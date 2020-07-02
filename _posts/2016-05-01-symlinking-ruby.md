---
layout: post
title: "A Quick Fix For An Issue With Symlinking Ruby"
categories: ruby
---

Ran into this problem the other day.  Thought I’d blog about it before it got too stale in my mind.
In most typical entry points to an application you’ll see something like this:

{% highlight ruby %}
$LOAD_PATH << File.join( File.dirname(__FILE__), '..', 'lib' )
require 'your-lib-name'
{% endhighlight %}

p. Here it is adding “../lib” to the load path and then loading your-lib-name.rb from that directory.  However, when this script is being run via a symlink, __FILE__ will be interpretted as the location of the symlink.  Obviously, because “../lib” is a relative location this can cause problems. This will give an error that I am sure most developers will be familiar with:

{% highlight ruby %}
:in `require': no such file to load -- yourlib (LoadError)
{% endhighlight %}

And absolute paths really aren’t a very good option either.  So what to do?  Well, luckily, you can do this:

{% highlight ruby %}
require 'pathname'
$LOAD_PATH << File.join( File.dirname(Pathname.new(__FILE__).realpath), '..', 'lib' )
require 'your-lib-name'
{% endhighlight %}

As you can see, Pathname.realpath resolves the symlink to the real location.  I thought it was an interesting problem and am surprised I haven’t seen this sooner.
