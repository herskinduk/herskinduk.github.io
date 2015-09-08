---
layout: post
title: Sitecore 404 without 302
---

As I recently migrated the "Apropos Sitecore" blog from Typepad to Sitecore I put some effort in to setting up legacy URL rewriting. In case I missed anything I wanted to ensure that the 404 handling on the new blog was set up appropriately.

I have always felt that the default way Sitecore handles 404 request was a bit annoying. If a users misspells a URL or perhaps misses out a couple of characters when copying a URL they will be redirected to the /sitecore/service/notfound.aspx URL with a cryptic&nbsp;query string&nbsp;- not very&nbsp;helpful. I suspect that&nbsp;today's&nbsp;web savvy users find it pretty frustrating (I do!) having to retype the entire URL rather than just correcting what was wrong.

Luckily both IIS and Sitecore has support for rewriting error URLs rather than redirecting to a 404 page. Basically this means that the URL stays the same in the browser but on the server the request is&nbsp;transferred to another page - in this case a 404 error page.

There is [a document on SDN](http://sdn.sitecore.net/upload/sitecore6/handling_http_404_a4.pdf) with details of how to configure 404. There is a short chapter called "Consistent HTTP 404 Page Not Found Management" on how to configure a custom 404 page for Sitecore and IIS. Unfortunately it does not say anything about using rewrites rather than redirects.

After a bit of nosing around I found that Sitecore has a configuration option called RequestErrors.UseServerSideRedirect. Setting this to true will make Sitecore do URL rewrites rather than redirects.&nbsp;

## Configuration example for IIS7

First configure Sitecore to do rewrites by creating a file called CustomErrorRedirect.config in the /App_Config/Include folder with the following snippet:

{% highlight xml %}
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
    <sitecore>
        <settings>
            <setting name="ItemNotFoundUrl">
                <patch:attribute name="value">/path-to/404-file.aspx</patch:attribute>
            </setting>
            <setting name="LinkItemNotFoundUrl">
                <patch:attribute name="value">/path-to/404-file.aspx</patch:attribute>
            </setting>
            <setting name="RequestErrors.UseServerSideRedirect">
                <patch:attribute name="value">true</patch:attribute>
            </setting>
        </settings>
    </sitecore>
</configuration>
{% endhighlight %}

Then add this to <system.Webserver> section in the Web.config to have IIS follow the same behaviour (handles page not found for non-ASP.Net requests - eg. the .html extension):

	<httpErrors errorMode="Custom">
	    <remove statusCode="404" subStatusCode="-1" />
	    <error statusCode="404" path="/path-to/404-file.aspx" responseMode="ExecuteURL" />
	</httpErrors>

Also remember to have your "404-file.aspx" page set the Response.StatusCode = 404; and make it look pretty!