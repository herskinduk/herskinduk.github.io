---
layout: post
title: Sitecore item paths with IntelliSense (auto-generated)
---

A while ago I presented my [Sitecore model auto-generation concept](/blog/2012/03/automatic-sitecore-model-generation) to [Sitecore’s UK User Group](http://www.sitecore.net/unitedkingdom/landing/unitedkingdom/usergroups). One of the perspectives I mentioned in [my presentation](/blog/2012/03/sug-notes-automatic-sitecore-model-generation) was using the technique to generate item paths. The idea was that this would eliminate a lot of the magic strings representing paths and IDs.

My initial thought was to generate a list of constants that could be used in conjunction with the item factory. Before I ever got started on this [Jason](http://twitter.com/#!/jason_bert)&nbsp;from [Lightmaker](http://www.lightmaker.com/#/uk/) showed me a much more elegant way of representing the item tree by auto-generating nested static classes. This way you get very nice IntelliSence in Visual Studio when typing what is effectively the path to the desired item.

Example:

![](http://static1.herskind.co.uk/%5Epub=NjM0NzAyMTEzNjYwMDAwMDAw%5Ew=720%5Eh=364/~/media/Images/Projects/ItemTree.ashx)

Here is my code for the ItemTree.tt (can be dropped into the auto-generation [project available on Sitecore's GitHub](https://github.com/Sitecore/TDS-T4-Model-Generation)):

{% highlight csharp %}
<#@ template hostspecific="true" debug="false" language="C#" #>
<#@ assembly name="%ProgramFiles%\Hedgehog Development\Team Development for Sitecore (VS2010)\HedgehogDevelopment.SitecoreCommon.Data.dll" #>
<#@ assembly name="%ProgramFiles%\Hedgehog Development\Team Development for Sitecore (VS2010)\HedgehogDevelopment.SitecoreCommon.Data.Parser.dll" #>
<#@ assembly name="System.Core" #>
<#@ import namespace="HedgehogDevelopment.SitecoreCommon.Data" #>
<#@ import namespace="HedgehogDevelopment.SitecoreCommon.Data.Items" #>
<#@ import namespace="HedgehogDevelopment.SitecoreCommon.Data.Fields" #>
<#@ import namespace="System.Collections" #>
<#@ import namespace="System.Collections.Generic" #>
<#@ import namespace="System.Globalization" #>
<#@ import namespace="System.Linq" #>
<#@ import namespace="System.Text" #>
<#@ import namespace="System.Text.RegularExpressions" #>
//
// This file is auto-generated - do not edit
//
using System;
using Sitecore.Data.Items;
using Herskind.Model.Helper;
using Herskind.Model.Helper.FieldTypes;
  
namespace Herskind
{
    public static class ItemTree
    {
        private T InstantiateItemWrapper<T>(string id)
        {
            return ItemFactory.Instance.SelectSinglePath<T>(id);
        }
<# 
    // CONFIGURE: Change this to the the path of your TDS project (relative to current project)
    string tdsPath = "..\\Herskind.Tds";
    SitecoreDataSource = new SerializedTreeDataSource(this.Host.ResolvePath(tdsPath));
 
    PushIndent("\t");
    GenerateClasses(SitecoreDataSource.Items, "");
    PopIndent();
#>
    }
}
<#+
public void GenerateClasses(IEnumerable<IItem> items, string parentName)
{
    PushIndent("\t");
    foreach (IItem item in items)
    {
        var template = SitecoreDataSource.Templates.FirstOrDefault(t => t.Properties["id"].ToLower() == item.Properties["template"].ToLower());
        var templateInterfaceName = 
            template == null ? "global::" + BaseNameSpace + ".Helper.IItemWrapper" : "global::" + FullNamespace(template) + "." + InterfaceName(template.Name);
        var className = ParentSafeClassName(parentName, item.Name);
             
#>
/// <summary>
/// Item path:
/// <#= item.Path #>
/// </summary>
public static class <#= className #>
{
    public static <#= templateInterfaceName #> <#= InstancePropertyName(parentName, item, "Instance") #>
    {
        get { return InstantiateItemWrapper<<#= templateInterfaceName #>>("<#= item.ID.ToString() #>"); }
    }
    public static string <#= InstancePropertyName(parentName, item, "ItemID") #>
    {
        get { return "<#= item.ID.ToString() #>"; }
    }
<#+
        GenerateClasses(item.Children, className);
#>
}
<#+
    }
    PopIndent();
}
#>
<#+ 
public string BaseNameSpace = "Herskind.Model";
public SerializedTreeDataSource SitecoreDataSource = null;
 
public string InstancePropertyName(string parentName, IItem item, string name)
{
    if (ParentSafeClassName(parentName, item.Name) != name)
    {
        return name;
    }
    return InstancePropertyName(parentName, item, name+"_");
}
 
 
public string ParentSafeClassName(string parentName, string name)
{   
    if (parentName!=ClassName(name))
    {
        return ClassName(name);
    }
    return ClassName(name)+"_";
}
 
public string ClassName(string name)
{
    return TitleCase(name);
}
 
 
public string InterfaceName(string name)
{
    return "I" + TitleCase(name);
}
 
public string TitleCase(string name)
{
    name = Regex.Replace(name, "([a-z](?=[A-Z])|[A-Z](?=[A-Z][a-z]))", "$1 ");
    name = CultureInfo.InvariantCulture.TextInfo.ToTitleCase(name);
    name = Regex.Replace(name, @"[^a-zA-Z0-9]", String.Empty);
    name = Regex.Replace(name, @"(^[0-9])", "_$1");
     
    return name;
}
 
public string RelativeNamespace(TemplateItem template)
{
    var sb = new StringBuilder();
    var pathList = new List<string>(template.Path.Split('/'));
     
    try
    {
        return string.Join(".", pathList.Take(pathList.Count - 1).Skip(3).Select(p => TitleCase(p)));
    }
    catch
    {
    }
     
    return "";
}
 
public string FullNamespace(TemplateItem template)
{
    return BaseNameSpace + "." + RelativeNamespace(template);
}
#>
{% endhighlight %}
