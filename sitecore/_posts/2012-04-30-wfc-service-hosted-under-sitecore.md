---
layout: post
title: WCF Service hosted under Sitecore
redirect_from: /blog/2012/04/wfc-service-hosted-under-sitecore/
---

I just spend a little to long getting a [Windows Communication Foundation](http://msdn.microsoft.com/en-us/netframework/aa663324) service working with Sitecore. It should be straight forward (and I guess it used to be!?). I followed the instructions from this blog post on [configuring Sitecore to host WCF services](http://sitecoreblog.alexshyba.com/2009/03/attach-wcf-services-to-sitecore-context.html).

This would suffice on Sitecore installations up to version 6.4.1 Update-4. From that version onwards Sitecore introduced a RewriteModule that does not like the URL format of WCF service methods (i.e. ServiceName.svc/MethodName). Sitecore released a fix for this a couple of months ago. The [issue notes](http://sdn.sitecore.net/Products/Sitecore%20V5/Sitecore%20CMS%206/ReleaseNotes/KnownIssues%20Recommended/Sitecore%20Rewrite%20Module%20prevents%20executing%20ASP,-d-,NET%20Web%20Service%20calls.aspx) only mentions ASP.Net web services, but is essentially the same problem experienced with WCF.

Hope this will save you some time and head scratching. I suggested that Sitecore support puts a mention of WCF on the issue notes.