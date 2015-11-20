---
layout: post
category : pages
tags : [NGinx, Load Balancing]
---

In order to get [NGINX](http://nginx.org) working as load balancer the following steps are needed on a Mac:

#### 1. Install [NGINX](http://nginx.org) via brew

{% highlight bash%}
brew install nginx
{% endhighlight %}

#### 2. Create/Edit nginx.conf *(default: /usr/local/etc/nginx/)*

{% highlight bash%}
events {
    worker_connections  1024;
}


http {
  upstream customers {
    server localhost:8080 weight=3;
    server localhost:8081;
  }

  server {
    listen 80;
    location / {
      proxy_pass http://customers;
    }
  }
}
{% endhighlight %}

#### 3. Update /etc/hosts (optional)

{% highlight bash%}
127.0.0.1       localhost,customers
{% endhighlight %}

#### Some Notes

{% highlight bash%}
nginx
{% endhighlight %}

{% highlight bash%}
nginx -s stop
{% endhighlight %}

A very interesting read: http://tenzer.dk/nginx-with-dynamic-upstreams/
