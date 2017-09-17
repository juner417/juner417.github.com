---
layout: post
title: "mono in docker container"
date: 2016-03-31 01:00:00
categories: blog
tags: container
comments: true
---
[how to run mono in docker container(Ref link)](http://dotnetliberty.com/index.php/2015/10/04/mono-and-c-sharp-on-docker-hello-world-in-15-steps/)

{% highlight bash %}
# download docker ubuntu image
docker image
docker run -it ubuntu bash

{% endhighlight %}


{% highlight bash %}
# in ubuntu 
sudo apt-get update
sudo apt-get install -y mono-complete
ctrl p q

{% endhighlight %}

{% highlight bash %}
# after out of container
docker ps
docker commit containerID juner/monodev
docker images

{% endhighlight %}


run c# binary using mono 
{% highlight bash %}
# return contianer
sudo apt-get install -y vim
vim hellomono.cs

{% endhighlight %}


{% highlight cs %}
public class HelloMono {
        static public void Main() {
                System.Console.WriteLine("Hello Mono!");
        }
} 


{% endhighlight %}


{% highlight bash %}
mcs hellomono.cs
mono hellomono.exe
Hello Mono!
{% endhighlight %}
