---
layout: post
title: !binary |-
  QnJpZGdpbmcgdGhlIFNlY3VyaXR5IEdhcCB3aXRoIFB1cHBldA==
wordpress_id: 457
wordpress_url: !binary |-
  aHR0cDovL3d3dy5mcnltYW5ldC5jb20vP3A9NDU3
date: 2011-08-16 22:39:23.000000000 -05:00
---
One of the things that I hear as I interface with technology professionals and management alike are the complex challenges that they face every day with developing and maintaining appropriate security within their organization. Security practitioners within organizations are often battling an immense amount of complexity within their organization. These complexities are only becoming more tangled as companies venture into new competitive markets, and as a result fall within the oversight of regulatory bodies and regulations (examples include HIPAA and PCI).

<!--more-->

The development of a well-rounded security program includes elements that appropriately weigh out three dimensions of risk within an organization - Confidentiality, Integrity, and Availability. Security practitioners must look at the domains of people, process, and toolsets in order to accurately assess how to design and implement appropriate security controls for an organization. Puppet as a tool provides both direct and indirect mechanisms to assist in the strengthening of internal security policies and external regulatory requirements.
<h3>Direct Impact to Security Compliance:</h3>
Puppet is built around the general management and automation of system configuration and administration. The Puppet Domain Specific Language (DSL) is built to allow quick and easy modeling of security policies in a manner that can be quickly learned by administrators and security analysts alike. In turn, Puppet code can be provided to auditors who can quickly and easily understand the underlying language to see how policies are being implemented and enforced.

Puppet provides the framework to build compliance solutions for your enterprise. Several guidelines have been published on the web today that help outline a good baseline security setting for any organization and can be tweaked and tuned as appropriate. Examples include Center for Internet Security [CIS] or National Institute of Standards and Technology [NIST]). Using the Puppet DSL, complex requirements can be quickly and easily modeled within an organization.

<em>Example Requirement:</em>
<pre>for DIR in `awk -F: '( $3 &gt;= 500 ) { print $6 }' /etc/passwd`; do
    if [ $DIR != /var/lib/nfs ]; then
      chmod -R g-w   $DIR

      chmod -R o-rwx $DIR
    fi
done</pre>
<em>Puppet Code Conversion:</em>
<pre>$homedir  = '/home'        # Sets a variable $homedir that can be referenced later in code
$filemode = $homedir ? {   # Selector conditional - making sure that $homedir still has read rights.
  $homedir =&gt; '0755',
  default  =&gt; '0750', 
}

file { $homedir:            # File Reference in Puppet - allows management of files/directories/symlinks
  recurse      =&gt; true,     # Ensures a recursive check of the specified directory
  recurselimit =&gt; '1',      # Limits a recursive check to only one level deep
  mode         =&gt; $filemode
}</pre>
Example Requirement retrieved from: CIS RHEL 5.0 Benchmark: <a href="https://benchmarks.cisecurity.org/tools2/linux/CIS_RHEL_5.0-5.1_Benchmark_v1.1.2.pdf">Source</a>

The added benefit with Puppet is the ability to enforce state of an Operating System. Once security settings are defined within Puppet, and deviation to these settings are automatically corrected and reported. Audits can be reduced from multiple day sessions discussing technology implementations to a short conversation demonstrating the state and how it is enforced. As long as your machines are up and running, Puppet is enforcing the known state. Take it a step further and integrate your Puppet implementation with a Source Code Management system (like Perforce, Subversion, or GIT), and you can also demonstrate an audit trail of changes made to your systems

For example: Within the Payment Card Industry - Data Security Standard (PCI-DSS), Puppet can assist in the management of over 100 controls proposed in Version 2.0 of the DSS. A complete list of how Puppet can assist with PCI can be found <a href="http://db.tt/C4vJ7hM">here</a>
<h3>Indirect Impact to Security Compliance:</h3>
A major problem within the development and implementation of a security program includes battling perceptions of additional and unnecessary complexity, or being perceived as overhead that only inhibits or slows down business development. Puppet has the ability to bridge the gaps between Security/Operations/Development by introducing a common language (the Puppet DSL) between all three groups in how machines are managed. This common language can help development and operations teams. This allows Security teams to come to the Operations/Development tables and introduce transparency around the how and why of security, enforcing existing process and easily tying into existing processes across an organization.
<h3>Conclusion</h3>
Puppet as a tool solves several direct problems of security management, providing a mechanism for teams to quickly and easily deploy security policies across a any number of machines. Likewise, Puppet effectively enforces a known state across these machines, reducing any configuration drift while also ensuring policies are consistently enforced across an organization. Finally, Puppet can help influence behavior in people and processes across an organization, with the ultimate goal of reducing risk by allowing security to become a part of everyone's core job.
