---
layout: post
title:  "Handcrafting velocity fields"
date:   2022-09-30 14:26:32 +0200
author: Alex Majewski
categories: Houdini
keywords: houdini velocity field
thumbnail: /assets/posts/handcrafting_velocity_fields/velfields_thumbnail.jpg
---
Velocity fields are used to control simulations, especially things like smoke or fluids. They can be created from any set of points by giving them a vector attribute.

![Examples of different fields]({{ '/assets/posts/handcrafting_velocity_fields/velfields_examples.jpg' | relative_url }}){: .img-extra-post-width}

It's literally that simple. Or hard --- if you don't know a few easy techniques from this article which will teach you how to direct velocity vectors by hand.



## Creating a basic velocity field
We can start by creating points. For simplicity, we'll use a simple `Points From Volume`.<br>
(Worth noting operators like `Pyro Source` have point generation built in)

We then add velocity to the points with one of the methods listed below and then we run it through `Volume Rasterize Attributes` in which we specify which attribute we want to rasterize (velocity).

![An example of a simple vector field]({{ '/assets/posts/handcrafting_velocity_fields/velfields_simple_field.jpg' | relative_url }}){: .img-max-post-width}
This is the hardest part of creating a velocity field.
{: .img-caption}

For more awareness, we can preview our velocity field a bit more clearly by feeding it into `Volume Trail` along with our original points.

![Volume Trail SOP setup]({{ '/assets/posts/handcrafting_velocity_fields/velfields_volume_trail.jpg' | relative_url }}){:class="img-extra-post-width"}

Finally, we send the result into a solver. (Make sure to check out the hip file provided at the end of this post. It contains Pyro, Flip, Vellum and RBD both SOP and DOP level setups)

![Example of a velocity field plugged into Pyro solver]({{ '/assets/posts/handcrafting_velocity_fields/velfields_example_pyro.jpg' | relative_url }}){: .img-max-post-width}
A simplified example of a velocity field being used in a Pyro DOP network (it still needs @fuel, @temperature etc).
{: .img-caption}

![Example of a velocity field plugged into SOP-level Vellum Solver]({{ '/assets/posts/handcrafting_velocity_fields/velfields_example_vellum.jpg' | relative_url }}){: .img-max-post-width}
A possible way of applying a velocity field as a POP force inside a SOP-level Vellum Solver.
{: .img-caption}

{% capture video_url %}
  {{ "/assets/posts/handcrafting_velocity_fields/velfields_pyro.mp4" | relative_url }}
{% endcapture %}
{% include video.html url=video_url %}{: .img-extra-post-width}

## How to hand-craft velocity vectors

### 1. Plane deformation method
This method relies on adding velocity in uniform direction to a plane while it's still flat, and then transforming and warping it afterwards.

![Twisting a flat plane with vectors]({{ '/assets/posts/handcrafting_velocity_fields/velfields_twist.jpg' | relative_url }}){: .img-extra-post-width}
Most SOPs will be able to deform vector attributes as long as they are of the correct type. Default attributes such as @v usually work by default, but if you chose a different name, or they simply aren't being affected by transformations then you'll have to [set attribute type metadata](https://www.sidefx.com/docs/houdini/vex/attribtypeinfo_suite) to "vector" in a wrangle.
{: .img-caption}

From here, we can do pretty much anything --- duplicate the distorted plane, copy it to points in a circle, deform it along a curve using `Path Deform` --- the possibilities are endless.

![Example of copying the deformed plane to points]({{ '/assets/posts/handcrafting_velocity_fields/velfields_copytopoints.jpg' | relative_url }}){: .img-max-post-width}

### 2. Measure SOP method
If we set the `Measure` SOP to "Gradient" mode it will generate direction vectors on our geometry. We can rename the gradient attribute to "v".

![Example of Measure SOP in use]({{ '/assets/posts/handcrafting_velocity_fields/velfields_measure_example.jpg' | relative_url }}){: .img-max-post-width}

`Measure` generates these vectors in relation to the chosen axis, and they will flow around your geometry like smoke in a wind tunnel.


If we plan to deform the geometry after using Measure SOP, we will have to [change the velocity attribute's type metadata](https://www.sidefx.com/docs/houdini/vex/attribtypeinfo_suite) to "vector" in an `Attribute Wrangle`.

![Changing attribute type metadata]({{ '/assets/posts/handcrafting_velocity_fields/velfields_attribtypeinfo.jpg' | relative_url }}){: .img-max-post-width}

### 3. Curves method
We can use curve direction as velocity. I think that’s what `Curve Force` and `Gas Curve Force` SOPs already do, but if you need it as a velocity field, this could be one way to do it.

We pass the curve through a `Resample` SOP and export a Tangent attribute and then transfer it to our velocity field.

![Example of using the tangent as direction vector]({{ '/assets/posts/handcrafting_velocity_fields/velfields_curve.jpg' | relative_url }}){: .img-max-post-width}

Alternatively, we can use `Orient Along Curve` which lets us control the rotation of @N with pitch, yaw and roll, which we then can use as @v. (`@v = @N;`)

### Transferring to velocity field

After creating the general flow of vectors we need to transfer them to our field using the `Attribute Transfer` SOP.

![Example of transferring velocity attribute]({{ '/assets/posts/handcrafting_velocity_fields/velfields_attrib_transfer_nodes.jpg' | relative_url }}){: .img-max-post-width}

{% capture video_url %}
  {{ "/assets/posts/handcrafting_velocity_fields/velfields_attrib_transfer.mp4" | relative_url }}
{% endcapture %}
{% include video.html url=video_url %}{: .img-extra-post-width}

As always, that's just the tip of the iceberg of possibilities. There are many more ways to create and adjust velocity vectors.

{% capture hip_link %}
  {{ "/assets/posts/handcrafting_velocity_fields/velocity_fields_examples_v_1_0.hipnc" | relative_url }}
{% endcapture %}
{% include hipfile.html url=hip_link version="1.0" houversion="19.5.303" %}
