---
layout: post
title: Automatic Sitecore Model Generation
redirect_from: /blog/2012/03/automatic-sitecore-model-generation/
---

There are several model frameworks available for Sitecore. They all have their strengths and weaknesses – but common for all of them are that they represent Sitecore items as strongly typed objects in code.

My motivation for creating what one guy at Sitecore User Group London described as “yet another model generation framework” was not to invent a new way of composing strongly typed objects. It was an attempt to introduce an alternative technique for auto-generating the objects with focus on the following goals:&nbsp;

*   It should respect the principle of single source of truth (solution files under version control in this case).
*   It should validate of field/property name and type mapping at compile-time.
*   It should work in a continuous integration environment.
*   It should be flexible and utilise standard tools.

To achieve the first of these goals it is important to think about where the auto-generation gets information about the item data structures. A popular approach is to look in the master database using the Sitecore API however this has some serious drawbacks:

*   Master databases across environments and individual developer machines may vary. This may result in disparity between data structures in Sitecore and in code.*   You need an HttpContext to for easy access to the Sitecore API – this could make it tricky to integrate in a continuous integration process.

As an alternative I think serialised data structures (residing with your solution under version control) would be a more reliable source of information and as I use [Team Development for Sitecore](http://www.hhogdev.com/products/team-development-for-sitecore.aspx "Team Development for Sitecore") on nearly all of my projects this information is already available to me. To generate the model based on the templates in my TDS project I used a T4 template. I elaborated on the technique I described in a [previous blog post](http://sitecore.herskind.co.uk/2011/04/sitecore-tds-t4-text-template.html) to generate interfaces/classes matching the Sitecore data structures. It is possible to have Visual Studio [run the T4 transformation on pre-build](http://stackoverflow.com/questions/1646580/get-visual-studio-to-run-a-t4-template-on-every-build/3041089#3041089) so that your model always reflects a fresh view of the data structure.

This illustration shows the basic elements of the process. You start out by adding your templates to your TDS project. Next time you build the Model.tt is transformed to Model.cs before the build runs.&nbsp;

![](http://static1.herskind.co.uk/%5Epub=NjM0NzAyMTEzNjYwMDAwMDAw/~/media/Images/Projects/ScreenShot20120301at213542.ashx)

Here is the Model.tt code used to generate the model in the illustration above:

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
	using System;
	using Sitecore.Data.Items;
	  
	<# 
	SerializedTreeDataSource sitecoreDataSource = new SerializedTreeDataSource(this.Host.ResolvePath("..\\Herskind.Tds"));
	 
	foreach (var template in sitecoreDataSource.Templates)
	{    
	    var baseTemplates = BaseTemplateList(template, sitecoreDataSource, false);
	    var baseTemplatesRecursive = BaseTemplateList(template, sitecoreDataSource, true);
	    var combinedTemplateList = new List(baseTemplatesRecursive);
	    combinedTemplateList.Add(template);
	     
	#>
	 
	#region <#= template.Name #> (<#= RelativeNamespace(template) #>)
	namespace <#= FullNamespace(template) #>
	{
	    // Template interface
	    public partial interface <#= InterfaceName(template.Name) #> : IItemWrapper <# 
	    foreach (var baseTemplate in baseTemplates)
	    {   
	        #>, <#= RelativeNamespace(baseTemplate) + "." + InterfaceName(baseTemplate.Name)  #><#
	    }
	#>
	 
	    {       
	<#
	    foreach(var fieldTemplate in combinedTemplateList)
	    {       
	        foreach(var section in fieldTemplate.Sections)
	        {
	            foreach(var field in section.OwnFields)
	            {
	#>
	        <#= GetFieldWrapperType(field.Fields["type"]) #> <#= TitleCase(field.Name) #> { get; }      
	<#
	            }
	        }
	    }
	#>           
	    }
	 
	    // Template class
	    [TemplateMapping("<#= template.ID.ToString("b").ToUpper() #>")]
	    public class <#= ClassName(template.Name) #> : BaseItemWrapper, <#= InterfaceName(template.Name) #>
	    {
	        private Item _innerItem = null;
	        public <#= ClassName(template.Name) #>(Item item) : base(item)
	        {
	            _innerItem = item;
	        }
	<#
	    foreach(var fieldTemplate in combinedTemplateList)
	    {       
	        foreach(var section in fieldTemplate.Sections)
	        {
	            foreach(var field in section.OwnFields)
	            {
	#>
	 
	        public <#= GetFieldWrapperType(field.Fields["type"]) #> <#= TitleCase(field.Name) #>
	        {
	            get
	            {
	                return (<#= GetFieldWrapperType(field.Fields["type"]) #>)GetField("<#= field.Name.ToLower() #>"); 
	            } 
	        }   
	<#
	            }
	        }
	    }
	     
	#>
	    }
	}
	#endregion
	 
	<#
	}
	#>
	<#+ 
	private const string BaseNameSpace = "Herskind.Model";
	 
	public string GetFieldWrapperType(string typeName)
	{
	    var wrapperType = "IFieldWrapper";
	     
	    switch (typeName.ToLower())
	    {
	        case "checkbox":
	            wrapperType = "IBooleanFieldWrapper";
	            break;
	        case "image":
	            wrapperType = "IImageFieldWrapper";
	            break;
	        case "date":
	        case "datetime":
	            wrapperType = "IDateFieldWrapper";
	            break;
	        case "checklist":
	        case "treelist":
	        case "treelistex":
	        case "multilist":
	            wrapperType = "IListFieldWrapper";
	            break;
	        case "droplink":
	        case "droptree":
	        case "general link":
	            wrapperType = "ILinkFieldWrapper";
	            break;
	        case "single-line text":
	        case "multi-line text":
	        case "rich text":
	        wrapperType = "ITextFieldWrapper";
	            break;
	    default:
	            wrapperType = "ITextFieldWrapper";
	            break;
	    }
	 
	    return wrapperType;
	}
	 
	 
	public string ClassName(string name)
	{
	    return TitleCase(name);
	}
	 
	public string DataClassName(string name)
	{
	    return TitleCase(name) + "FieldData";
	}
	 
	public string DataInterfaceName(string name)
	{
	    return "I" + TitleCase(name) + "FieldData";
	}
	 
	public string DataPropertyName(string name)
	{
	    return TitleCase(name) + "Fields";
	}
	 
	public string DataPrivateMemberName(string name)
	{
	    return "_m" + TitleCase(name) + "Fields";
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
	    name = Regex.Replace(name, @"(^[0-9])", "Z$1");
	     
	    return name;
	}
	 
	public string RelativeNamespace(TemplateItem template)
	{
	    var sb = new StringBuilder();
	    var pathList = new List(template.Path.Split('/'));
	     
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
	 
	public IEnumerable BaseTemplateList(TemplateItem template, SerializedTreeDataSource sitecoreDataSource, bool recursive)
	{
	    var list = new List();
	     
	    foreach (var templateId in template.BaseTemplateIds)
	    {
	        var baseTemplates = sitecoreDataSource.Templates.Where(t => t.ID.Equals(templateId));
	        if (baseTemplates.Count() == 1)
	        {
	            if (!list.Where(t => t.ID == baseTemplates.First().ID).Any())
	            {
	                list.Add(baseTemplates.First());
	                if (recursive)
	                {
	                    foreach (var baseTemplate in BaseTemplateList(baseTemplates.First(), sitecoreDataSource, true))
	                    {
	                        if (!list.Where(t => t.ID == baseTemplate.ID).Any())
	                        {
	                            //System.Diagnostics.Debugger.Launch();
	                            //Console.WriteLine(baseTemplate.ID.ToString());
	                            list.Add(baseTemplate);
	                        }
	                    }
	                }
	            }
	        }
	    }
	     
	    return list;
	}
	#>

Note how the above T4 script utilises the TDS assemblies to parse the serialized data. It also contains a number of useful methods for generating namespace-, property-, class- and interface names that will compile and (sort of) follow C# best practices.

In theory this technique could generate whatever model you want - it doesn't even have to be a model, another example could be generating constants based on a list of dictionary items, or something completely different.

In this example I generated a model to experiment with some of the challenges of auto-generated models:

## Sitecore's templates supports multiple inheritance - .Net classes do not

I have been experimenting with different ways of representing the multiple inheritances feature of Sitecore templates in C#. The most obvious solution is to make good use of interfaces as a class may implement as many interfaces as needed.

I experimented with two approaches for adding fields and inherited fields as properties on my objects; 1) by association and 2) by direct implementation. The two approaches can happily co-exist – but I found the latter to be easier to live with and opted for that in this release.&nbsp;

## How to extend/customise the auto-generated model

I found two ways of customising my model; 1) using partial classes and partial interfaces and 2) using extension methods. Using partial classes and interfaces gives access to private members of the class but must exist in the same assembly as the model. Using extension methods has some limitations but is still my favorite out of the two.

## Mapping Sitecore items to model objects

Like most other Sitecore model frameworks I have added an item factory that can spawn model objects. My item factory is very basic and works a somewhat like a facade for the Sitecore Database.

Example of the item factory:

	var itemFactory = new ItemFactory() as IItemFactory;
	var wrappedItem = itemFactory.GetSiteHome<Model.Herskind.Pages.IHomePage>();

## Maintaining Sitecore functionality (ex. RenderField functionality) whilst encapsulating dependence on the Sitecore API

I also created a number of field wrappers that gives access to the raw field value as well as a rendered value and extended functionality for reference types, list types etc.

Example of using a wrapped field:

	// Continued from example above
	if (wrappedItem != null)
	{
	  // Outputting fields raw or rendered value
	  Page.Title = wrappedItem.Title.RawValue;
	  Response.Write(wrappedItem.Title.Render());
	}

## Summary

This post describes the process of generating a model using T4 and TDS. I have not included all the code that is required for this model to work as my main objective was to describe the mechanics of auto generation rather than providing a model. The model shown here is just a testbed for some of the issues described above. If you would like a copy of the source code to try it out [drop me an email](mailto:kern@herskind.co.uk).

_**Update: The code is now available on [Sitecore's Github](https://github.com/Sitecore/TDS-T4-Model-Generation).**_
