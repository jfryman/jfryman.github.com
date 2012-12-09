---
layout: post
title: !binary |-
  U3RvcmluZyBQdXBwZXQgZmFjdHMgaW4gTERBUCB3aXRoIE94Zm9yZA==
wordpress_id: 509
wordpress_url: !binary |-
  aHR0cDovL3d3dy5mcnltYW5ldC5jb20vP3A9NTA5
date: 2011-11-30 23:17:22.000000000 -06:00
---
Let me preface this post... I like LDAP. A lot. I think it gets a bad rap because of its complexity. However, this post is not about the positive merits of LDAP and using it for more than address book storage or AAA within Linux. I'll leave this <a href="http://sysadvent.blogspot.com/2010/12/day-11-journey-to-nosql.html">fine post here</a> if you'd like to learn more. I highly encourage it, as many of my future posts will discuss how to start using LDAP for a lot of things within a Puppet Deployment. Now, let's begin with this topic...

<!--more-->
<h2>Getting Started</h2>
Here was my challenge: I am in the beginning stages of designing a full-fledged Puppet environment for a client that I am working with. They have a very minimal Linux infrastructure, and so I have the opportunity to do things the way that I'd like to ideally see from a management/tool/etc perspective. In designing the overall architecture for this deployment, I chose LDAP as my storage backend for data. Everything from AAA (Authentication, Authorization, Audit), the ENC (External Node Collector) for Puppet configuration, to Facts about a system were to be published in LDAP, and more.

Let's get started with using Oxford. Once you've installed Puppet and Facter, simply install oxford using rubygems.
<blockquote>
<pre>gem install oxford</pre>
</blockquote>
Once this is installed, simply edit the /etc/oxford.conf to match your environment.
<blockquote>
<pre>ldap:
  host: ldap.frymanet.com
  port: 389
  method: plain
  base: dc=frymanet,dc=com
  user: cn=oxford,ou=Special,dc=frymanet,dc=com
  pass: &lt;hidden&gt;</pre>
</blockquote>
The final step to this puzzle is to extend the LDAP schema to allow it to understand the new schema. Grab the schema from <a href="https://github.com/jfryman/oxford/blob/master/inc/websages.schema">GitHub</a>, and add it to your LDAP server. OpenLDAP has two ways to import this schema depending on the version. Let's assume you're using a non cn=config setup, and download the schema to your LDAP server(s) and import. Make sure to restart your OpenLDAP server if you use this method.
<blockquote>
<pre>include /etc/openldap/schema/websages.schema</pre>
</blockquote>
Make sure that in your LDAP tree you create the following Organizational Unit (OU) at the top-level to contain host-data. Your tree should include an entry like this.
<blockquote>
<pre>ou=Hosts,dc=frymanet,dc=com</pre>
</blockquote>
Optionally, create an LDAP user to write data to this OU. The OpenLDAP ACL looks like this:
<blockquote>
<pre>access to dn.subtree="ou=Hosts,dc=frymanet,dc=com"
    by dn.base="cn=oxford,ou=Special,dc=frymanet,dc=com" write</pre>
</blockquote>
Finally, kick off oxford and watch the magic happen!
<blockquote>
<pre>oxford -c /etc/oxford.conf</pre>
</blockquote>
If everything is setup properly, you should see some host entries appear in the Hosts OU. If you want this to happen automagically, consider adding a cron-entry to update facts on a periodic basis.
<h2>Too Much Work!</h2>
If that seems like too much work (it is quite a bit, and LDAP already has a bit of a learning curve with it), consider using some pre-built modules that I've also built.
<ul>
	<li><a href="http://github.com/jfryman/puppet-openldap">Puppet Module for LDAP</a></li>
	<li><a href="http://www.github.com/jfryman/puppet-oxford">Puppet Module for Oxford</a></li>
</ul>
<div>Now, your definitions to make all of this happen simply look like this. Use your existing puppet infrastructure to make your life easier!</div>
<blockquote>
<pre> class { 'oxford':
    conn_type   =&gt; 'ldap',
    ldap_host   =&gt; 'ldap.frymanet.com',
    ldap_port   =&gt; '389',
    ldap_method =&gt; 'plain',
    ldap_user   =&gt; 'cn=oxford,ou=Special,dc=frymanet,dc=com',
    ldap_pass   =&gt; 'I shouldn never give out passwords!',
   }</pre>
<pre>  class { 'ldap':
    server      =&gt; 'true',
    server_type =&gt; 'openldap',
  }
  ldap::define::domain { 'frymanet.com':
    basedn =&gt; 'dc=frymanet,dc=com',
    rootdn =&gt; 'cn=admin',
    rootpw =&gt; &lt;hidden&gt;,
  }
  ldap::define::schema { 'websages':
    ensure =&gt; 'present',
    source =&gt; 'puppet:///modules/ldap/schema/websages.schema',
  }
  ldap::define::acl { 'dn.subtree="ou=Hosts,dc=frymanet,dc=com"':
    ensure =&gt; 'present',
    domain =&gt; 'frymanet.com',
    access =&gt; {
      'dn.base="cn=oxford,ou=Special,dc=frymanet,dc=com"' =&gt; 'write'
    },
  }</pre>
</blockquote>
<h2>Commentary:</h2>
The key drivers for using LDAP as a data backend included:
<ul>
	<li>Ability to rapidly scale data across the organization (Multi-Master replication).</li>
	<li>Libraries to interface with LDAP already exist across the enterprise, as systems are using LDAP for AAA.</li>
	<li>Transforming the fact data from Facter allows me to do some creative Meta-Programming</li>
	<li>I don't have to administer yet another MySQL database to store data.</li>
</ul>
The end-state overall high-level design looks like the graphic below.

<img class="size-full wp-image-526 alignnone" title="PuppetandLDAPIntegration" src="http://www.frymanet.com/wp-content/uploads/2011/11/PuppetandLDAPIntegration.png" alt="" width="400" height="242" />

In order to accomplish this task, I had to write some new code. Facter does not currently provide a facility to push facts to LDAP, so I wrote a wrapper around Facter to collect facts and push them into LDAP called Oxford. Oxford's job is to grab the key-value pairs of facts from Facter, transform them to adhere to an LDAP compliant schema, and then push new facts to LDAP.

The other challenge in this puzzle was figuring out how to model dynamic data in LDAP (things like Network Adapters or Processor Information). Other non-dynamic facts are stored at the host level. In order to work around this, I'm leveraging the creation of additional objects underneath the host entry. In this sense, network adapters would be accessed in LDAP with this CN:
<blockquote>
<pre>cn=eth0,cn=galactica,ou=Hosts,dc=frymanet,dc=com</pre>
</blockquote>
The next addition we gain here is the ability to do some fun meta-programming. I made a conscious decision to prefix all of the fact names with 'fact' in LDAP, and so queries to all facts in a system become ridiculously easy. This is important to me, as I am really only leveraging LDAP as a data store.

In order to be successful here, there has to be a front-end that is easily digestible by humans. The reputation that LDAP has in being difficult to interface with is not without merit. A nice front-end accompanies this setup that allows pretty much *anyone* to browse facts about systems. This is excellent as it provides a degree of transparency to management types about what is happening on a system without being destructive to the production environment.

The downside to this approach with any LDAP is having to continually extend schema to accommodate new facts. I'm still looking for a way to dynamically extend LDAP schema as custom facts are generated. However, in all reality - given that the entire infrastructure is programmatically managed with Puppet already, any additions are trivial to manage. Schema updates are no longer painful with the help of an <a href="https://github.com/jfryman/puppet-openldap">LDAP Puppet Module. </a>

The great thing about Oxford is that it was architected in such a way that we can continue to expand the fact sources - custom facts, different fact generation tools (like Ohai, pfacter, etc) and continue to aggregate into my central store. As the various tools in the OSS space continue to evolve and grow, I've provided a mechanism where I am no longer locked into a specific technology. If a better/faster/stronger fact generation capability arises - I can change on a dime.

This is all still a work in progress. Right now, we're only grabbing Linux specific facts, but this will grow to include AIX. However, this works remarkably well for me today.

Likewise, if you find this useful - please drop me a note or contribute to this project.

Many thanks to <a href="http://about.me/azizshamim">Aziz Shamim</a> for his assistance in refactoring the code for Oxford. Additional thanks to the folks at <a href="http://blog.websages.com">Websages</a> for allowing me to hijack some of their OID space for this work.

<strong>TL;DR - <em>gem install oxford, customize /etc/oxford.conf, profit. </em></strong>
