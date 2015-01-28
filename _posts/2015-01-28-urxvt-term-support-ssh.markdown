---
layout: post
title:  "urxvt unknown terminal type with ssh"
date:   2015-01-28 13:55:51
categories: urxvt terminal ssh
---

If you use urxvt and ssh chances are you've ran into the same issue as I have:

{% highlight bash %}
$ clear
'rxvt-unicode-256color': unknown terminal type.
{% endhighlight %}

As it turns out, it's quite easy to fix. All you have to do is copy over your local
terminfo to the server which is giving you the problem:

create the terminfo folders on user@remotehost:

{% highlight bash %}
mkdir -p ~/.terminfo/r/
{% endhighlight %}

copy over the terminal profile ( urxvt in my case ):

{% highlight bash %}
scp /usr/share/terminfo/r/rxvt-unicode-256color user@remotehost:.terminfo/r/
{% endhighlight %}

All new ssh sessions will support the terminal profile you added :)
