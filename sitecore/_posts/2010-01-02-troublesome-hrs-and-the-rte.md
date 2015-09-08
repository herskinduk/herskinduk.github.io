---
title: Troublesome HR's and the RTE
layout: post
redirect_from: /blog/2010/01/troublesome-hrs-and-the-rte/
---

I recently came across a (yet another) IE7 annoyance. If you try and style a HR tag with a background image it is seemingly impossible to avoid the border around the image in IE7. After scouring the web I came across [a solution that solves the problem](http://www.sovavsiti.cz/css/hr.html) and retains the HR tag (I reckon that is the best solution from a semantic point of view).

So how do I go about having Sitecore's Rich Text Editor insert HR's like this:

<pre class="brush: html">&lt;div class="hr"&gt;&lt;hr /&gt;&lt;/div&gt;
</pre>

There are two obvious solutions:

1.  Add the above code as a HTML snippet in the RTE or
2.  Rely on the RTE to insert plain HR tags and then replace HR tags in the fieldRenderer pipeline.

I chose to explore the 2nd option as I'm skeptical about editors adding HTML snippets.

First I did the CSS to handle div.hr&gt;hr as suggested in previously mentioned article. The I created a class to handle the string replacement:

<pre class="brush: c#">namespace Herskind.Processors
{
    public class RichTextFixes
    {
        public void Process(RenderFieldArgs args)
        {
            switch (args.FieldTypeKey)
            {
                case "rich text":
                    {
                        args.Result.FirstPart = ReplaceHRs(args.Result.FirstPart);
                        args.Result.LastPart = ReplaceHRs(args.Result.LastPart);
                        break;
                    }
            }
        }

        private string ReplaceHRs(string text)
        {
            text = text.Replace("&lt;hr&gt;", "&lt;hr/&gt;");
            text = text.Replace("&lt;hr/&gt;", "&lt;hr /&gt;");
            return text.Replace("&lt;hr /&gt;", "&lt;div class=\"hr\"&gt;&lt;hr/&gt;&lt;/div&gt;");
        }
    }
}
</pre>

I know that there might be a more elegant way of doing the replacement, but for illustration purposes this will do.&nbsp;

And added it as the last step of the _renderField _pipeline by creating a configuration include file (ie. website/App_Config/Include/RteFix.config):

<pre class="brush: xml">&lt;configuration xmlns:patch="http://www.sitecore.net/xmlconfig/"&gt;
  &lt;sitecore&gt;
    &lt;pipelines&gt;
      &lt;renderField&gt;
        &lt;processor type="Herskind.Processor.RichTextFixes, Herskind" /&gt;
      &lt;/renderField&gt;
    &lt;/pipelines&gt;
  &lt;/sitecore&gt;
&lt;/configuration&gt;
</pre>

Note: The processor type consists of to pieces of information. The fully expended class name (basically the namespace + the class name) followed by the name of the assembly.

Now the editors can insert plain HR's in the RTE and when the field is rendered using &lt;sc:text /&gt; all hr tags are enclosed in the &lt;div class"hr"/&gt;.
