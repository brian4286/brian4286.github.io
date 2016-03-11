---
layout:     post
title:      Tuning php-fpm
date:       2015-11-12 20:29:15
summary:    Observations while tuning php-fpm for a PHP heavy website.
categories: aster
---

I had a inquiry from a customer about a web servers that was producing random 502 errors. This is the type of error you typically see from php-fpm when something upstream has not worked properly (which most of us think of downstream but that's a different story). Thankfully php-fpm was logging so that helped me see that the server was hitting the MaxClient frequently.

Show performance metrics

{% highlight bash %}
php-fpm config
{% endhighlight %}
