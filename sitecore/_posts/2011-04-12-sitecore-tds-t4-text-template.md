---
layout: post
title: Sitecore + TDS + T4 Text Template
---

Inspired by the [Compiled Domain Model](http://trac.sitecore.net/CompiledDomainModel) and other code generation tools for Sitecore, I came up with a different approach to generating classes and eliminating magic strings.

In most cases the code generator uses the Sitecore API to traverse items in a Sitecore database to generate classes that represent templates and items. I cannot help feeling that basing the generated code on data from a database is a little detached from the codebase that lives in the versioning system.

For some time I have been using [TDS](http://hhogdev.com/products/team-development-for-sitecore.aspx "Team Development for Sitecore") to keep my items under version control (this could have been done using Sitecore&#39;s serialization feature - but TDS just makes it much more manageable). In a recent discussion with a client I had the idea of generating a model based on the serialized data from TDS.

This is a proof of concept using T4:

<pre class="brush: csharp">&lt;#@ template hostspecific=&quot;true&quot; language=&quot;C#&quot; #&gt;
&lt;#@ assembly name=&quot;C:\Program Files (x86)\Hedgehog Development\Team Development for Sitecore (VS2010)\HedgehogDevelopment.SitecoreCommon.Data.dll&quot; #&gt;
&lt;#@ assembly name=&quot;C:\Program Files (x86)\Hedgehog Development\Team Development for Sitecore (VS2010)\HedgehogDevelopment.SitecoreCommon.Data.Parser.dll&quot; #&gt;
&lt;#@ import namespace=&quot;HedgehogDevelopment.SitecoreCommon.Data&quot; #&gt;
using System;   

&lt;# 
SerializedTreeDataSource sitecoreDataSource = new SerializedTreeDataSource(this.Host.ResolvePath(&quot;..\\Herskind.Tds&quot;));

foreach (var template in sitecoreDataSource.Templates)
{    
#&gt;
public class &lt;#= template.Name.Replace(&quot; &quot;, &quot;&quot;) #&gt;
{
&lt;#
    foreach(var section in template.Sections)
    {
#&gt;
    // Section: &lt;#= section.Name #&gt;
&lt;#
        foreach(var field in section.OwnFields)
        {
#&gt;
    // Field: &lt;#= field.Name #&gt; - Data type: &lt;#= field.Fields[&quot;type&quot;] #&gt;
&lt;#
        }
    }

    foreach(var baseitemid in template.BaseTemplateIds)
    {
#&gt;
    // Base: &lt;#= baseitemid.ToString() #&gt;
&lt;#
    }
#&gt;
}
&lt;#
}
#&gt;
</pre>

I realise that there is some way from this to fully automated model generation but I thought it was a fun demo.