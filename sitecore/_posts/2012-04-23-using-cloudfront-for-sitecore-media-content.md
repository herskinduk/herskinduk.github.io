---
layout: post
title: Using CloudFront CDN for Sitecore media content
---
Ever since I discovered that Amazon CloudFront supported custom origins I have been thinking about how it could be used in a Sitecore solution. If you [set up a custom origin](http://docs.amazonwebservices.com/AmazonCloudFront/latest/DeveloperGuide/CreatingDistributions.html) pointing to your website it will effectively work as a caching proxy (edge cache). This can take some of the load off your Sitecore server and give a general boost to page performance* which again could have a positive affect on your SEO ranking.

## URL rewriting

In this post I want show how CloudFront can be leveraged as an edge cache for Sitecore media library content. One problem that&nbsp;immediately&nbsp;arises&nbsp;is that CloudFront does not forward query string parameters to the origin server. This is a problem if you use query string parameters to control your media requests. The&nbsp;obvious&nbsp;example is requesting images form the media library with different parameters.&nbsp;

![](http://static1.herskind.co.uk/%5Epub=NjM0NzAyMTEzNjYwMDAwMDAw%5Ew=720%5Eh=196/~/media/Images/Projects/cloudfront-missing-qs.ashx)

Note how the query string disappears as CloudFront forwards the requests to the origin.

Hopefully one day AWS will support query string parameters - in the meantime we will have to do a little bit of URL rewriting. If we were to move the query string to the beginning of the part and substitute &amp; and ? with ^ (%5E when URL encoded) - like this:

http://xxxxxx.cloudfront.net/%5Ew=100%5Eh=100/~/media/images/photo.ashx

CloudFront will forward the path in full to the origin server. The URL can easily be re-written using the [URL Rewrite module for IIS](http://http://www.iis.net/download/urlrewrite) to restore the query string.

![](http://static1.herskind.co.uk/%5Epub=NjM0NzAyMTEzNjYwMDAwMDAw%5Ew=720%5Eh=287/~/media/Images/Projects/cloudfront-qs-rewrite.ashx)

The rewrite rule could look like this:

{% highlight xml %}
<system.webServer>
    <rewrite>
        <rules>
            <clear />
            <rule name="Un-jumble Media Parameters" stopProcessing="true">
            <match url="(\^([^\^]+))?(\^([^\^]+))?(\^([^\^]+))?(\^([^\^]+))?(\^([^/]+))?/~/media/(.*)" />
            <conditions logicalGrouping="MatchAll" trackAllCaptures="false" />
            <action type="Rewrite" url="/~/media/{R:11}?{R:2}&{R:4}&{R:6}&{R:8}&{R:10}" appendQueryString="false" logRewrittenUrl="false" />
            </rule>
        </rules>
    </rewrite>
    ...
<system.webServer>
{% endhighlight %}

## Outputting CDN image paths

A while ago I blogged about [customising the Image field renderer](/blog/2011/07/imagecropper-and-scimage-the-missing-piece) to enable the crop parameter. It's possible to override the Sitecore.Pipelines.RenderField.GetImageFieldValue class to change the generated image source URL. The same technique can be used to substitute local media library URLs with CDN URLs.

{% highlight csharp %}
public class GetImageFieldValue : Sitecore.Pipelines.RenderField.GetImageFieldValue
{
    public class CdnEnabledImageRenderer : Sitecore.Xml.Xsl.ImageRenderer
    {
        protected override string GetSource()
        {
            var cdnService = new AwsCloudFrontCdnService();
 
            if (Sitecore.Context.PageMode.IsNormal && cdnService.CdnActive)
            {
                var baseUrl = new UrlString(base.GetSource().Replace("&", "&"));
                return cdnService.RewriteUrl(baseUrl.GetUrl(false));
            }
            else
            {
                return base.GetSource();
            }
        }
    }
 
    protected override Sitecore.Xml.Xsl.ImageRenderer CreateRenderer()
    {
        return new CdnEnabledImageRenderer();
    }
}
{% endhighlight %}

You will also need to handle media library URLs that are being output in-line in Rich Text fields. That can be done by elaborating on the functionality of the Sitecore.Pipelines.RenderField.ExpandLinks in the RenderField pipeline.

{% highlight csharp %}
public class ExpandLinks : Sitecore.Pipelines.RenderField.ExpandLinks
{
    private static Regex regex = new Regex("([^\"]*\\~/media/[^\"]*\\.ashx\\??[^\"]*)", RegexOptions.Compiled);
    private static MatchEvaluator evaluator = new MatchEvaluator(MakeCdnUrl);
    private static ICdnService cdnService = new AwsCloudFrontCdnService();
 
    public override void Process(RenderFieldArgs args)
    {
        Assert.ArgumentNotNull((object)args, "args");
        if (Context.PageMode.IsPageEditorEditing)
            return;
        args.Result.FirstPart = ReplaceCdnPaths(DynamicLink.ExpandLinks(args.Result.FirstPart, Settings.Rendering.SiteResolving));
        args.Result.LastPart = ReplaceCdnPaths(DynamicLink.ExpandLinks(args.Result.LastPart, Settings.Rendering.SiteResolving));
    }
 
    private string ReplaceCdnPaths(string html)
    {
        if (cdnService.CdnActive)
        {
            return regex.Replace(html, evaluator);;
        }
        else
        {
            return html;
        }
    }
 
    public static string MakeCdnUrl(Match match)
    {
        return cdnService.RewriteUrl(match.Value);
    }
}
{% endhighlight %}

Both of these uses this code to generate CDN URLs:

{% highlight csharp %}
public class AwsCloudFrontCdnService : ICdnService
{
    protected string CdnHostName
    {
        get { return Settings.GetSetting("Herskind.CdnHostName"); }
    }
 
    public string RewriteUrl(string originalUrl)
    {
        if (CdnActive && Sitecore.Context.PageMode.IsNormal)
        {
            if (!originalUrl.StartsWith("http://"))
            {
                if (!originalUrl.StartsWith("/"))
                {
                    originalUrl = "/" + originalUrl;
                }
                originalUrl = "http://x" + originalUrl;
            }
            var uri = new Uri(originalUrl);
            var masterdb = Sitecore.Configuration.Factory.GetDatabase("master");
            var pubHash = Encode(masterdb.Properties.GetLastPublishDate(Sitecore.Context.Database, Sitecore.Context.Language).Ticks.ToString());
            return CdnHostName + "/%5Epub=" + pubHash + uri.Query.Replace("?", "%5E").Replace("&", "&").Replace("&", "%5E") + uri.AbsolutePath;
        }
        return originalUrl;
    }
 
    protected string Encode(string toEncode)
    {
        byte[] toEncodeAsBytes
              = System.Text.ASCIIEncoding.ASCII.GetBytes(toEncode);
        string returnValue
              = System.Convert.ToBase64String(toEncodeAsBytes);
        return returnValue;
    }
 
    public bool  CdnActive
    {
        get { return !CdnHostName.IsNullOrEmpty(); }
    }
}
{% endhighlight %}

You may at this point think that it would be easier to just have a outbound URL rewrite rule (I did) but you would loose some control over when/where URLs are being rewritten. There could be cases (e.g. Preview and Page Editor mode) where you would want to disable the CDN URL rewriting.&nbsp;

## Content expiry

One big problem faced when using edge caching is invalidating content when it is updated at the origin. Most CDN's - CloudFront included - supports some sort of content invalidation. Ideally you would want to expire content on the edge as soon as a new version has been published by Sitecore. Due to the fact that we include dynamic parameters in the URLs and that CloudFront does not support a full purge this is difficult. To circumvent this issue I choose to embed a token in the CDN urls. There are two tokens that could be used; item revision or last published timestamp. The latter will basically generate new URLs for all content after a publish and forces CloudFront to visit origin for all requests. Note that the old media content may linger on the edge for however long the expires HTTP header dictates.You would be able to invalidate content on the edge using the AWS API if you know the exact URLs.&nbsp;If you are governed by legislation that requires the ability to invalidate the entire edge cache instantly, the approach described above is not appropriate.

Another consideration is how long content should live on the edge. By default Sitecore will not allow media content to be cached on the edge but this can be adjusted by changing the following configuration options:

{% highlight xml %}
<!-- allow proxies to cache -->
<setting name="MediaResponse.Cacheability" value="public" />
<!-- lifetime 12 hours -->
<setting name="MediaResponse.MaxAge" value="0.12:00:00:00" />
{% endhighlight %}

## Final notes

Sitecore already has a powerful caching layer for media library content. The performance gain from using a CDN may be limited depending on your set-up and the nature of your website.

I would not suggest using a CDN for caching pages as you would loose the ability to do personalisation. You will also loose the ability to record DMS goals on media library downloads.

When creating a custom origin CloudFront distribution it will mirror your entire site (pages included). You may want your origin server to filter out non-media requests coming form CloudFront. You can use a Request Blocking rule in IIS Rewrite for requests where the user agent HTTP header matching&nbsp;"Amazon CloudFront".

When trying to establish if content is being cached you can look for these HTTP headers coming from CloadFront: "X-Cache: Miss from cloudfront" or&nbsp;"X-Cache: Hit from cloudfront".

There are many other CDN products available besides Amazon CloudFront (Akamai is popular) but none other with such a transparent price policy as far as I'm aware. CloudFront does not require a heavy financial commitment to start using edge caching and should be affordable by most.

You can read more about Sitecore performance optimisation in this [blog post by Alex Shyba](http://sitecoreblog.alexshyba.com/2011/10/sitecore-media-library-performance.html), this blog&nbsp;[post by John West](http://www.sitecore.net/Community/Technical-Blogs/John-West-Sitecore-Blog/Posts/2011/05/All-About-Performance-Optimization-and-Scalability-with-the-Sitecore-ASPNET-CMS.aspx) or in the [Sitecore Performance Tuning Guide on SDN](http://sdn.sitecore.net/Reference/Sitecore%206/CMS%20Performance%20Tuning%20Guide.aspx).

*) Be warned: It's important to realise that rewriting the URLs in the FieldRenderer will degrade field rendering performance slightly and that the impact of any modifications to the RenderField pipeline will be amplified by the number of fields output via the pipeline. You would need to evaluate whether the performance hit to the FieldRenderer will be outweighed by the potential gains of using a CDN. The code provided in this post has not been performance tested or tuned - it's purely written to provide examples to help understanding the processes involved.
