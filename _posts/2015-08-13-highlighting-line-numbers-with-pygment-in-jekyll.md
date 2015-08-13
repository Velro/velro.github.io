---
title: Highlighting line numbers in code snippets with Jekyll/Pygments
layout: post
category: Jekyll
author: James Fulop

excerpt: This info was not easily found online and I think other tech blog people using this website stack will want to be able to do this.
---
{% raw  %}
//TLDR: `{% highlight csharp linenos=table hl_lines="8 13" %}`
{% endraw %}

That's it really. To be more general:
{% raw  %}
`{% highlight languagehere hl_lines="8 13"%}`
{% endraw %}

Here's an example. 

{% highlight csharp linenos=table hl_lines="6 10" %}
using UnityEngine;
using System.Collections;
 
public class TestCode : MonoBehaviour
{
    private GameObject someObject;
 
    void Start()
    {
        someObject.SetActive(false);
    }
}
{% endhighlight %}

Isn't that nice!


Note: The `linesnos=table` separates the line numbers into a separate element. This way the code can be selected separately from the line numbers.

This was so hard to find because, as I understand, Jekyll has wrapped Pygments in the Liquid template system, so the syntax is different from what is described in the [Pygments documentation](http://pygments.org/docs/formatters/). I finally got on the right track by looking at the [source code for the Jekyll highlighter](https://github.com/jekyll/jekyll/blob/b4ac044c2979f4c9b028df7aa724065b0ed6e80d/lib/jekyll/tags/highlight.rb). 

It'd be nice to be able to put in range type parameter, like "1 3 5-10". Maybe I could do that with some quick JS/HTML string variable replacement. I'll have to try that sometime.