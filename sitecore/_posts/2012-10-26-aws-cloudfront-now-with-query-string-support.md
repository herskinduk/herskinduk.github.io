---
layout: post
title: AWS CloudFront - now with query string support
---

Great news - or should I say old news!?

Apparently Amazon CloudFront have added support for query string forwarding for custom origin distributions. This feature was announced approximately two weeks after I had written this [lengthy blog post](/blog/2012/04/using-cloudfront-for-sitecore-media-content) on how to mitigate this deficiency&nbsp;when using CloudFront and Sitecore with dynamic media URLs.

With the introduction of query string forwarding you can you can drop all the rewriting and just set the Media.MediaLinkPrefix (or do something [slightly more elegant](http://www.cognifide.com/blogs/sitecore/the-ultimate-approach-to-storing-sitecore-media-items-in-cdn/)). Remember to configure the cache HTTP headers for Sitecore media:

{% highlight xml %}
<!-- allow proxies to cache -->
<setting name="MediaResponse.Cacheability" value="public" />
<!-- lifetime 12 hours -->
<setting name="MediaResponse.MaxAge" value="0.12:00:00:00" />
{% endhighlight %}

The last thing to do is to go to the AWS management console and edit the&nbsp;behaviour&nbsp;setting on your custom origin CloudFront distribution to ensure query string forwarding is enabled.

![AWS CloudFront distrubution configured to respect query string parameters](http://static1.herskind.co.uk/%5Epub=NjM0NzAyMTEzNjYwMDAwMDAw%5Eh=546%5Ew=725/~/media/Images/Projects/aws-edit-behaviour.ashx)

This&nbsp;set-up&nbsp;is a lot lower in complexity than my initial post and it is much more in line with what I initially had hoped for when I started researching using CloudFront with Sitecore. This approach does not have any considerations around edge cache invalidation (cache beyond&nbsp;time-out) - this would probably need to be addressed production use.