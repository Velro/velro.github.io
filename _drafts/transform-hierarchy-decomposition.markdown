---
title: Transform Hierarchy Decomposition
layout: post
---

Deleting a large number of objects at once can cause a Unity game to skip a couple of frames. In our game, Shadows of Isolation, there are a number of times that we have to delete several hundred objects while maintaining a seamless experience as the games load in and out different sub scenes. For my example I’ll be using an asteroid field with a few thousand rigidbodies.

![My helpful screenshot](/assets/asteroid-field.png)


Lets start by just calling `Delete(gameObject)` on the parent object. Here’s our latency on an i7 processor on a single frame.

profiler image

My strategy to get around this spike was to first disable all of the objects. This way are functionally out of the scene, then to start deleting one object per frame. I could have just left the objects disabled in my scene, but those objects were still taking up space in memory. For this particular example the memory footprint was low, as there were only a few meshes and textures, but when destroying more varied scenes it had a large impact on reducing my memory foot print.

In my case, I tend to keep my sub scenes as different groups of objects parented to one thing. That sub scene game object then consists of a deep hierarchy. We are going to need a shallow list of objects starting with the deepest and ending with the shallowest. Otherwise we will have spikes in performance as objects with deep transform hierarchies are deleted before being decomposed.

4 3
2 1

I decided the best way to do this was to create separate lists for each depth level.

One of the sometimes fun, sometimes hard parts of programming is naming your file. I decided to go with the macabre name `DecomposeChildren`, to each his own.

To start, here’s how we declare a generic list of type `GameObject`. This list will eventually contain all of our objects in the order we want.

{% highlight csharp linenos %}

using UnityEngine;
using System.Collections;
using System.Collections.Generic;
 
public class DecomposeChildren : MonoBehaviour
{
    private List<GameObject> shallowList;
 
    void Start()
    {
        shallowList = new List<GameObject>();
    }
}
{% endhighlight %}


more text