---
layout: post
title: !binary |-
  T1NYIGFuZCBQYXJhbm9pYSA6OiBOb3QgRnJpZW5kcyE=
wordpress_id: 474
wordpress_url: !binary |-
  aHR0cDovL3d3dy5mcnltYW5ldC5jb20vP3A9NDc0
date: 2011-06-28 01:24:34.000000000 -05:00
---
Okay,  so I ran into yet another OSX bug that is both silly in nature and somewhat concerning. First, the error:
<blockquote>An error occurred. Unable to import "&lt;Certificate Name"&gt;.

Error: -2147415740</blockquote>
<a href="http://www.frymanet.com/wp-content/uploads/2011/06/RootCA-8192.png"><img class="size-medium wp-image-475 alignnone" title="RootCA-8192" src="http://www.frymanet.com/wp-content/uploads/2011/06/RootCA-8192-300x112.png" alt="" width="300" height="112" /></a>

ZOMG! What on earth did I do to cause OSX to spit out such a wonderful error to me? I must be doing something wrong, right?

Here is the scenario: I have a group of friends that have this self-signed certificate chain. I want to trust it, because I'm paranoid - they're paranoid, and I can relatively trust that they've done what they need to do in order to secure their website/blog/wiki/etc. More importantly, I'm tired of seeing a broken lock when I visit a webpage and getting nagged when I visit.

Easy solution - import some SSL certificates. However, when attempting to import, I received the above error. What is odd about this?

The certificate in question had some (what I would consider) benign options. Namely, SHA-1 Signature Algorithm.. but wait. 8192 bit keysize. Besides the neurons that had to die in order to compute these large primes, not that out of the ordinary. However...After some debugging and research, I found out that OSX Snow Leopard does not support certificates that are big! Well, how big? Let's do some experimentation.

Environment:
<ul>
	<li>OSX Snow Leopard 10.6.8</li>
	<li>OpenSSL 1.0.0d (Feb 2011) [MacPorts] and OpenSSL 0.9.8r (Feb 2011) [Native]</li>
</ul>
Did some random testing with the signature algorithm (MD5/SHA{1,256,384,[512}) only to find this had no bearing on the results. In my tests, any certificate that was greater than 4096 either produced this error or simply did not parse in the Apple Keychain Access. Not good!

The same issue occurs when I apply a key of 8192 or higher to a webserver. Safari does not like this, and decides to complain. This is not life altering, by any means, as NIST still only recommends keysizes of 2048 for CAs. (<a href="http://csrc.nist.gov/publications/nistpubs/800-57/sp800-57_PART3_key-management_Dec2009.pdf">REF</a>) However, Cryptanalytic attacks against a particular key size become more practical as computing power increases and new techniques are developed. Crypto only keeps us secure for a finite period of time. Eventually, we'll have some quantum computers that will be able to magically pull primes out of nothing, and then current cryptography will be all but useless.

Test along with me! My script to generate certificates quickly can be found <a href="http://dl.dropbox.com/u/443584/Scripts/snowleopard_ssl_testing">here</a>: Edit parameters and have fun!
