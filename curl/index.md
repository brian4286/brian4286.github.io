---
layout:     post
title:      cURL Cheat Sheet
---

Curl is an awesome tool. I keep forgetting the switches though, so here's a cheatsheet.

**Simple GET request**

{% highlight text %}
curl -k "https://localhost/foo?bar=baz&amp;abc=def"
{% endhighlight %}

**JSON POST or PUT request**

{% highlight text %}
curl -k -H "Content-Type: application/json" -X POST -d '{"accountName":"test","value":"hello"}' https://localhost/foo
curl -X "PUT"
... for a PUT
{% endhighlight %}

**POST a file**

{% highlight text %}
curl ... --data-binary @filename
{% endhighlight %}

**Fake a /etc/hosts entry and a Host: header with curl**

{% highlight text %}
curl -vvv --resolve 'book.mixu.net:80:123.145.167.189' http://book.mixu.net/
{% endhighlight %}

Make a request with basic auth enabled

{% highlight text %}
curl -vvv -u name@foo.com:password http://www.example.com

or:

curl --user name:password http://www.example.com
{% endhighlight %}

**Set the Referer header**

{% highlight text %}
curl -e http://curl.haxx.se daniel.haxx.se
{% endhighlight %}

**Set the User Agent header**

{% highlight text %}
curl -A "Mozilla/4.73"

or

curl --user-agent "Mozilla".
{% endhighlight %}

**Set Cookies**

{% highlight text %}
curl -b "name=Daniel"

or

curl --cookie "name=Daniel"
{% endhighlight %}

**Time a request (connect time + time to first byte + total tile)**

{% highlight text %}
curl -o /dev/null -w "Connect: %{time_connect} TTFB: %{time_starttransfer} Total time: %{time_total} \n" http://google.com
{% endhighlight %}

**Downloading files from Github**

{% highlight text %}
curl -O  https://raw.github.com/username/reponame/master/filename
{% endhighlight %}
