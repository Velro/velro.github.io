---
title: Transform Hierarchy Decomposition
layout: post
category: Unity
author: James Fulop

excerpt: Learn how to reduce overhead from destroying complex objects by spreading out the destroy call.
---

**TLDR:** Is your game lagging from destroying really big, complex objects? [Use this script to spread out the Destroy calls.](https://gist.github.com/Velro/3b10f4de8ba7188602a9)


Deleting a large number of objects at once can cause a Unity game to skip a couple of frames. In our game, Shadows of Isolation, there are a number of times that we have to delete several hundred objects while maintaining a seamless experience as the games load in and out different sub scenes. For my example I’ll be using an asteroid field with a few thousand rigidbodies.

![My helpful screenshot]({{! site.url }}/assets/transform-hierarchy-decomposition/asteroid-field.png)


Lets start by just calling `Delete(gameObject)` on the parent object. Here’s our latency on an i7 processor on a single frame.

![My helpful screenshot]({{! site.url }}/assets/transform-hierarchy-decomposition/Destroy1500AtOnce.png)

My strategy to get around this spike was to first disable all of the objects. This way are functionally out of the scene, then to start deleting one object per frame. I could have just left the objects disabled in my scene, but those objects were still taking up space in memory. For this particular example the memory footprint was low, as there were only a few meshes and textures, but when destroying more varied scenes it had a large impact on reducing my memory foot print.

In my case, I tend to keep my sub scenes as different groups of objects parented to one thing. That sub scene game object then consists of a deep hierarchy. We are going to need a shallow list of objects starting with the deepest and ending with the shallowest. Otherwise we will have spikes in performance as objects with deep transform hierarchies are deleted before being decomposed.

<head>
  <meta charset=utf-8 />
  <title></title>
  <style>
    div.container {
      display:inline-block;
    }

    p {
      text-align:center;
    }
  </style>
</head>
<body>
  <div class="container">
    <p>1st Tier</p>
    <img src="{{! site.url }}/assets/transform-hierarchy-decomposition/Heirarchy1Deep_2.png" title="heirarchy 1." alt="heirarchy 1." style="width:224px;height:324px;"/>
  </div>
  <div class="container">
  	<p>2nd Tier</p>
    <img src="{{! site.url }}/assets/transform-hierarchy-decomposition/Heirarchy2Deep_2.png" title="heirarchy 2." alt="heirarchy 2." style="width:224px;height:324px;"/>
  </div>
</body>
<body>
  <div class="container">
  	<p>3rd Tier</p>
    <img src="{{! site.url }}/assets/transform-hierarchy-decomposition/Heirarchy3Deep_2.png" title="heirarchy 3." alt="heirarchy 3." style="width:224px;height:324px;"/>
  </div>
  <div class="container">
  	<p>4th Tier</p>
    <img src="{{! site.url }}/assets/transform-hierarchy-decomposition/Heirarchy4Deep_2.png" title="heirarchy 4." alt="heirarchy 4." style="width:224px;height:324px;"/>
  </div>
</body>

I decided the best way to do this was to create separate lists for each depth level.

We'll name our class 'EnumeratedDestroy'.

To start, here’s how we declare a generic list of type `GameObject`. This list will eventually contain all of our objects in the order we want.

{% highlight csharp linenos=table hl_lines="3 7 11" %}
using UnityEngine;
using System.Collections;
using System.Collections.Generic;
 
public class EnumeratedDestroy : MonoBehaviour
{
    private List<GameObject> shallowList;
 
    void Start()
    {
        shallowList = new List<GameObject>();
    }
}
{% endhighlight %}


We can then make a list of lists so the script is flexible and will work with any sort of transform hierarchy.

{% highlight csharp linenos=table hl_lines="8 13"%}
using UnityEngine;
using System.Collections;
using System.Collections.Generic;
 
public class EnumeratedDestroy : MonoBehaviour
{
    private List<GameObject> shallowList;
    private List<List<GameObject>> masterList;
 
    void Start()
    {
        shallowList = new List<GameObject>();
        masterList = new List<List<GameObject>>();
    }
}

{% endhighlight %}

Now to fill these lists with objects. The best way seems to be with recursion for its flexibility. While iterating the children of a transform, if that child has its own children (read:grandchild), then recursively call at that depth. If while iterating over the hierarchy we end up deeper than we have before, add a list to our masterList for that depth. After generating these lists we can just iterate through the list of lists from last to first, adding their values to shallowList.

{% highlight csharp linenos=table hl_lines="15 16 17 18 19" %}
using UnityEngine;
using System.Collections;
using System.Collections.Generic;
 
public class EnumeratedDestroy : MonoBehaviour
{
    private List<GameObject> shallowList;
    private List<List<GameObject>> masterList;
 
    void Start ()
    {
        shallowList = new List<GameObject>();
        masterList = new List<List<GameObject>>();
 
        PopulateLists(transform, 0);
        for (int i = masterList.Count - 1; i >= 0; i--)// fill from deepest to shallowest
        {
            shallowList.AddRange(masterList[i]);
        }
    }
 
    private void PopulateLists(Transform parent, int depth) 
    {
        for (int i = 0; i < masterList.Count - 1; i++)
        {
            if (depth > masterList.Count-1)//initialize new lists for each depth
            {
                List<GameObject> list = new List<GameObject>();
                masterList.Add(list);
            }
            masterList[depth].Add(parent.GetChild(i).gameObject);
 
            //recurse into next layer
            if (parent.GetChild(i).childCount != 0)
            {
                PopulateLists(parent.GetChild(i), depth + 1);
            }
        }
    }
}
{% endhighlight %}

Now we just need to actually destroy objects, which we put into a very simple coroutine. Here’s the full script!

{% highlight csharp linenos=table hl_lines="20"%}
using UnityEngine;
using System.Collections;
using System.Collections.Generic;
 
public class EnumeratedDestroy : MonoBehaviour
{
    private List<GameObject> shallowList;
    private List<List<GameObject>> masterList;
 
    void Start ()
    {
        shallowList = new List<GameObject>();
        masterList = new List<List<GameObject>>();
 
        PopulateLists(transform, 0);
        for (int i = masterList.Count - 1; i >= 0; i--)// fill from deepest to shallowest
        {
            shallowList.AddRange(masterList[i]);
        }
        StartCoroutine(Delete(shallowList));
    }
 
    private void PopulateLists(Transform parent, int depth) 
    {
        for (int i = 0; i < masterList.Count - 1; i++)
        {
            if (depth > masterList.Count-1)//initialize new lists for each depth        
            {
                List<GameObject> list = new List<GameObject>();
                masterList.Add(list);
            }
            masterList[depth].Add(parent.GetChild(i).gameObject);
 
            //recurse into next layer
            if (parent.GetChild(i).childCount != 0)
            {
                PopulateLists(parent.GetChild(i), depth + 1);
            }
        }
    }
 
    //call once
    private IEnumerator Delete(List<GameObject> shallowList)
    {
        for (int i = 0; i < shallowList.Count; i++ )
        {
            Destroy(shallowList[i]);
            yield return new WaitForSeconds(0);
        } 
        Destroy(this.gameObject);
    }
}
{% endhighlight %}

Lets see how this performs.

<img src="{{! site.url }}/assets/transform-hierarchy-decomposition/decomposechildren_performance_highlighted1.png" title="Performance recap." alt="Performance recap." style="width:692px;height:166px;"/>


So our performance is about one fourth of the 'Destroy(gameObject)'' call, with miniscule 'Destroy' calls on subsequent frames. As long as the hierarchy doesn't change during gameplay, since the Start() function, where we determine the nesting, can be pretty expensive on a large enough object.

I hope this helps someone out there! Thanks for reading!

[Download the class here](https://gist.github.com/Velro/3b10f4de8ba7188602a9)