---
layout: post
title: !binary |-
  Rml4aW5nIENoZWYgRXJyb3I6IFlvdSBtdXN0IGhhdmUgYSBzdHJpbmcgbGlr
  ZSByZXNvdXJjZV90eXBlW25hbWVd
wordpress_id: 557
wordpress_url: !binary |-
  aHR0cDovL3d3dy5mcnltYW5ldC5jb20vP3A9NTU3
date: 2012-05-23 00:06:31.000000000 -05:00
---
Every once in a while, I run into a weird error with the various configuration management tools I use for our clients. In this case, we ran into one today that the stack trace didn't tell me anything, and so it became difficult to debug. In giving back to the Internet that has given so much to me, here is the error and solution.
<pre>ArgumentError: You must have a string like resource_type[name]</pre>
Here is the dump I received:
<blockquote>
<pre>ArgumentError: You must have a string like resource_type[name]!
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource_collection.rb:205:in `find_resource_by_string'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource_collection.rb:139:in `find'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource_collection.rb:134:in `each'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource_collection.rb:134:in `find'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource.rb:46:in `resolve_resource_reference'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource.rb:294:in `resolve_notification_references'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource.rb:294:in `each'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource.rb:294:in `resolve_notification_references'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/runner.rb:72:in `converge'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource_collection.rb:86:in `each'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/resource_collection.rb:85:in `each'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/runner.rb:71:in `converge'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/client.rb:312:in `converge'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/client.rb:160:in `run'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/application/solo.rb:192:in `run_application'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/application/solo.rb:183:in `loop'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/application/solo.rb:183:in `run_application'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/../lib/chef/application.rb:67:in `run'
/var/lib/gems/1.8/gems/chef-0.10.8/bin/chef-solo:25
/var/lib/gems/1.8/bin/chef-solo:19:in `load'
/var/lib/gems/1.8/bin/chef-solo:19</pre>
</blockquote>
No clue as to which cookbook this might belong to. The hint here is 'string like resource_type[name]', meaning some string somewhere in code doesn't follow the chef convention named above. It turns out, this relates to a malformed :notifies block somewhere. Sure enough, I mistyped. So, here's a diff:
<pre>- notifies :reload, "nginx-#{@params[:name]}"
+ notifies :reload, "service[nginx-#{@params[:name]}]"</pre>
Notice that I didn't include the resource_type? Yup, my bad. However, the debugging here was a PITA... so here's hoping you don't have to do the same hard digging.
<blockquote>
<pre></pre>
</blockquote>
