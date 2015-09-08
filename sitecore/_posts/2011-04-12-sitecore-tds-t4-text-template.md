---
layout: post
title: Sitecore + TDS + T4 Text Template
redirect_from: /blog/2011/04/sitecore-tds-t4-text-template/
---

Inspired by the [Compiled Domain Model](http://trac.sitecore.net/CompiledDomainModel) and other code generation tools for Sitecore, I came up with a different approach to generating classes and eliminating magic strings.

In most cases the code generator uses the Sitecore API to traverse items in a Sitecore database to generate classes that represent templates and items. I cannot help feeling that basing the generated code on data from a database is a little detached from the codebase that lives in the versioning system.

For some time I have been using [TDS](http://hhogdev.com/products/team-development-for-sitecore.aspx "Team Development for Sitecore") to keep my items under version control (this could have been done using Sitecore&#39;s serialization feature - but TDS just makes it much more manageable). In a recent discussion with a client I had the idea of generating a model based on the serialized data from TDS.

This is a proof of concept using T4:

{% highlight csharp %}
<#@ template hostspecific="true" language="C#" #>
<#@ assembly name="C:\Program Files (x86)\Hedgehog Development\Team Development for Sitecore (VS2010)\HedgehogDevelopment.SitecoreCommon.Data.dll" #>
<#@ assembly name="C:\Program Files (x86)\Hedgehog Development\Team Development for Sitecore (VS2010)\HedgehogDevelopment.SitecoreCommon.Data.Parser.dll" #>
<#@ import namespace="HedgehogDevelopment.SitecoreCommon.Data" #>
using System;   

<# 
SerializedTreeDataSource sitecoreDataSource = new SerializedTreeDataSource(this.Host.ResolvePath("..\\Herskind.Tds"));

foreach (var template in sitecoreDataSource.Templates)
{    
#>
public class <#= template.Name.Replace(" ", "") #>
{
<#
    foreach(var section in template.Sections)
    {
#>
    // Section: <#= section.Name #>
<#
        foreach(var field in section.OwnFields)
        {
#>
    // Field: <#= field.Name #> - Data type: <#= field.Fields["type"] #>
<#
        }
    }

    foreach(var baseitemid in template.BaseTemplateIds)
    {
#>
    // Base: <#= baseitemid.ToString() #>
<#
    }
#>
}
<#
}
#>
{% endhighlight %}


I realise that there is some way from this to fully automated model generation but I thought it was a fun demo.