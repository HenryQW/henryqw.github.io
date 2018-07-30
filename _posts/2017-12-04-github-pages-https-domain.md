---
key: 20171204
modify_date: 2018-05-14
tags: [GitHub, Nginx, Let's Encrypt, English]
title: Enable HTTPS for GitHub Pages with custom domain
---

~~[GitHub Pages](https://pages.github.com){:target="\_blank"} doesn't support HTTPS for sites with custom domain.~~ There is an easy workaround, which involves [Let's Encrypt](https://letsencrypt.org){:target="\_blank"} and [Nginx](https://nginx.org){:target="\_blank"}.

<!--more-->

### Update: GitHub is now working with Let's Encrypt to [support HTTPS for custom domain](https://blog.github.com/2018-05-01-github-pages-custom-domains-https/).

Simple adding at least one A record pointing the domain to any of the following:
- 185.199.108.153
- 185.199.109.153
- 185.199.110.153
- 185.199.111.153

## Nginx reverse proxy server
Instead of using CNAME as suggested by [GitHub Help](https://help.github.com/articles/using-a-custom-domain-with-github-pages/){:target="_blank"}, I setup an Nginx reverse proxy to serve all the contents from GitHub repository.


{% highlight Nginx %}

location / {
	proxy_pass https://HenryQW.github.io;
}
{% endhighlight %}

## Redirect all traffics to HTTPS

{% highlight Nginx %}

server {
	listen 80;
	server_name  henry.wang;
	return 301 https://$host$request_uri;
}
{% endhighlight %}


## Add SSL certificate

My 443 port is used by [sslh](https://github.com/yrutschle/sslh){:target="_blank"} for multiplexing, therefore 8443 is set as HTTPS port. Adding Let's Encrypt certificate to the configuration, and done!

{% highlight Nginx %}

server {
	listen 127.0.0.1:8443 ssl;
	server_name   henry.wang;

	ssl_certificate  /etc/letsencrypt/live/henry.wang/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/henry.wang/privkey.pem;

	location / {
        proxy_pass https://HenryQW.github.io;
	}
}
{% endhighlight %}

## Conclusion

The only down side is that your GitHub Pages doesn't redirect to your domain, but the rest will work just the same.

> But why all the hassle? Isn't there a server you already have? 

You get version control out of the box, updating the site is just commit and push. You don't need to run a database, which unleashes your server to do something else while GitHub is taking care of your sites.

With the great flexibility and availability offered by GitHub, it's unlikely your sites will produce error 4xx/5xx. Together with [CloudFlare's Always Online feature](https://www.cloudflare.com/always-online/){:target="_blank"}, your site will never go down, isn't that cool?