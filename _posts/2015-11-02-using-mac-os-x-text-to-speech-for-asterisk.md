---
layout:     post
title:      Using Mac OS X Text-to-speech for Asterisk
date:       2015-11-02 11:21:29
summary:    text-to-speech via the Terminal.
categories: aster
---

Recently I had been working on a project with a customer who has a old AT&T PagePac to for paging. You can read more about how I got the PagePac working with SIP here. The customer has a requirement to send notifications via the paging system to alert for breaks and lunch's. Since I use a Mac (and I hate recording my own voice) I thought about using the built-in text-to-speech.

Lets open Terminal

{% highlight bash %}
say -v Vicki --file-format=WAVE --data-format=LEI16@8000
-o "sample.wav" ",,,,,,,,, The time is now 8am"
{% endhighlight %}

Lets break this down:

-v lets you choose the voice you want the defaults installed are Kathy, Vicki, Victoria, Alex, Bruce and Fred. If you open your System Preferences you can click on the drop down for the "System Voice:" I think that Ava and Tom are the two best sounding.

--file-format=WAVE --data-format=LEI16@8000 Asterisk wants the format in WAVE, specifically a 16bit 8000hz.

-o outputs the recorded audio to a filename of your choosing.

Now between the quotes you can enter the text you want recorded. If you are going to possibly use this for IVR you may want to use multiple semi-colons between your
sentences to add a pause.
