# Viz: Chrome Graphics Futures

Title slide.


# About this talk

At the previous BlinkOn, 

## next 

I talked about Mus and Compositing. 

## next 

Some things have
changed since then, we've renamed some parts, we've refined our implementation
plan and (obviously) we've landed a lot of CLs.

## next

 So  I'm going to give you a progress update and
revised roadmap.

# Viz: Two contexualizations

There are two ways to put Viz in context. It's worth knowing both.

## next

First, we can consider Viz part of the Servification effort. 

## next

* Here, we
think of the Mojo UI service (or mus) as one core service in the
servificied Chrome 

## next

* And Viz is a sub-service responsible for producing
visual output (Viz is short for visuals.) (Yes, there are no Zs in
visuals. But Z makes it better.) And hence, Viz: a service responsible
for producing all visual output via use of the GPU to raster and
composite.

## next

Alternatively, we could think of Viz as an augmentation of the GPU
process.

## next

In existing Chrome, the GPU process is a virtualization of a hardware
GPU. 

## next 

But the process doess a lot more than that when it also is
responsible for rasterization and compositing.

## next

So we renamed it Viz. 


# Servicified Chrome Viz Context

I can show you a pretty picture to help make this clear..
You can see in the array of services that we have a UI 
service

We also have a closetly-coupled Viz service: a component responsible
for all of the browser's visual output.

# GPU process vs Viz

*em immediately next*

But Viz is more than just a tickbox in a list of servificiation 
components. 

## next 

It's about making Chrome graphics better
for the web platform.

## next

Viz does this by 

*	improving performance through direct compositing and centralized scheduling

## next

*	reducing rasterization work by centralizing raster decisions

## next

*	and supporting site-isolation without performance or memory penalities


# Current WebPlatform Graphics Pipeline

Oviously you're probably wondering how Viz will actually
improve performance.

It's worth explaining. Maybe I'll do a better job than I did in the last few talks.

Let's start with the current WebPlatform graphics compositing / raster pipeline.
It looks like this when using GPU-accelerated rasterization (the default as you
heard yesterday except on Linux)

Going from left to right, we have:

* blink
* and its paint subsystem. The result of layout and paint is a set of
paint items grouped into layers.
* this information is passed over to the CC thread via a *commit*
operation
* paint items go to the rasterization subsystem: Skia Ganesh emits
serialized GL corresponding to the various paint items. The serialized
GL goes via command buffer to the GPU process
* the layer compositor builds a CompositorFrame corresponding to the
layer structure of the page
* it ships the CF to the display compositor in the browser process.
* and the display compositor converts the union of CFs from all
clients into a serialized GL stream over a command buffer
* to the GPU process.
* where draw ops are sent to the graphics hardware


# Viz Tadpole

We are currently working on the first Viz milestone called tadpole
(yes, at least I have a penchant for silly names), 

From the viewpoint of the WebPlatform, Tadpole is a (in essence) a
refactoring to let us implement direct compositing.

We are merely relocating the display compositor from the browser
process to the Viz process. (You'll recall that Viz is the GPU process
augmented with more stuff -- like the DisplayCompositor.)

This box here... moved from the browser (which is no longer in the picture)
to here ... in Viz.

## next

We are actively working on Tadpole and expect it to come to Chrome
over the next 2-3 milestones. 

## next 

Tadpole is not an end in itself. It's a prepatory refactoring for direct compositing

And you might be wondering what direct compositing is: the intent is to
eliminate this bolded command buffer here...

## next

With the goal of getting a 10-15% performance improvement from reduced Command
Buffer overhead.


# Viz Tadpole with Direct Skia Compositing

We have several ways to implement our goal of removing the unnecessary
command buffer.

## next

The current Display Compositor has
a GL backend that  emits serialized GL over a command buffer (in red) to
Chrome's virtualized GPU

To eliminate the use of a command buffer in process...

We could modify (possibly extensively) the GLRenderer backend
to the display compositor to connect to GL directly.

## next

Instead, we plan to switch the compositor to a new Skia/Ganesh backend instead
for reasons that will become clear later in the talk.

# Aside: Why is Tadpole Hard?

You may have heard me talkign about this before. On several occasions
and might be wondering: why isn't Tadpole and whatnot done yet?

Well, Tadpole is hard. 

## next

It is simple for the web platform

## next 

But as you can see from the picture, the before and after renderer for tadpole
is effectively the same

## next

We (mostly) only have to alter the setup of CompositorFrame transport.

## next

But tadpole also requires many browser-side changes.

# UI Graphics Pipeline Changes

I know this is BlinkOn but bear with me while I talk about how the Chrome UI 
does graphics. Unlike the Web platform, the 
Chrome UI graphics pipe has to change a lot.

## next 

* Before Viz, the windowing and ui toolkits aura and views have a synchronous functional
call interface to the entire compositor.

## next

* But after relocating the display compositor, we have inserted the new highlighted
asynchronous IPC interface. 


# Tadpole Status [crbug/601863](http://crbug/601863)

Handling the consequences of Introducing this asynchrony is why Tadpole
is a large project.

You can see the bug for all the details. Or even better, find some some bugs
that you might like to help with.

A quick status update on where we are:

## next


# Parallel Effort: OOP Rasterization

Another parallel effort for Viz leading towards Salamander Viz is OOP
Rasterization.

## next

Centralizing the display compositor promises to reduce command buffer overhead. But
so does delegating all rasterization to the Viz process.

## next

This is the point of OOP rasterization: delegating raster to Viz via out of process
serialized paint operations.

## next

You can try on the work-in-progress implementation with the ... and  flags

# OOP  Rasterization

*always next*

* Currently, each web platform client generates paint ops and then
uses Skia/Ganesh to convert these into a serialized GL stream that it
sends these to the GPU process via a command buffer.

## next

With OOP rasterization, we instead serialize the paint ops in the
renderer and ship them (re-using the underlying CommandBuffer) to Viz
for rasterization.

This substitution will improve performance because this lower
highlighted bar of serialized paint ops will be smaller and cheaper to
serialize / deserialize than this upper highlighted bar of serialized
GL commands.

# Combined Tadpole and OOP Rasterization

We can combine Tadpole, OOP raster and the SkiaRenderer
compositor.

This is a significant milestone because it is the prerequisite for the
use of Vulkan. 

Going left to right...

* here we have the same CC paint / commit appratus as before
* paint ops are handled as I already discussed for OOP raster: serialized
in the renderer and shipped to Viz for rasterization
* the CC impl thread prepares CompositorFrame instances and sends them to Viz
* As with Tadpole, the display compositor is in Viz and handles the provided CompositorFrame
* The display compositor uses the direct compositing SkiaRenderer backend that I talked
about previously
* The Skia rasterizer handling the paintops has been modified. Here, Skia creates Chrome closures wrapping GL draw calls corresponding to the paint ops.
* These are PostTask-ed to the SkiaRenderer so that can choose how rasterization and compositing drawops are scheduled to the GPU

Note in particular: we have no GL CommandBuffer use.

# Enabling Vulkan

As has perhaps become obvious now, eliminating the GL command buffer from both webpage rasterization
and compositing has an ulterior motive beyond eliminating its serialization
overhead.

## next

It is not possible to implement a command buffer for the Vulkan API without
losing many of the benefits of the Vulkan API.

## 

There is however (conveniently) a Skia Ganesh Vulkan backend that 
we could use for both rasterization and the SkiaRenderer. I added it to the
diagram here. It should be obvious why I talked earlier about the 
SkiaRenderer.

## 

And: why do we care to use Vulkan? 

We expect an additional 10-15% performance improvement from the combination
of command buffer removal and the use of Vulkan.

# Aside: WebGL Requires CommandBuffer

If you care about WebGL, you're probably wondering what would happen
to it without a command buffer. Have no fear, it's still here because
WebGL requires it.

We have two ways to connect the GPU service 

## next

We could require the highlighted bar for texture interchange between native GL 
and Vulkan

## next

Or we could have a Vulkan backend for Angle that can produce the same
kind of Vulkan buffers that the Ganesh backend needs to produce. 
And it woudl submit these to the SkiaRenderer compositor in a similar way.

# Explaining the Site Isolation Graphics Toll

At the beginning of the talk, I said that Viz is also intended
to remove the  rasterization and compositing overhead 
needed to support for site-isolation.

## next

Site isolation is desirable because it improves 
security by placing each origin domain in a different
renderer process.

But currently, general use of site isolation imposes a graphics
performance toll.

## next

A single top-level page using site isolation might look like the
schematic. ...explain picture...

The embedded iframe in light red allocates GPU memory and rasterizes
the entire red region. (Though afaik, there is wip to clip it to the
bounds of the embedding page.)

## next

But the dark red region is unecessarily allocated and rasterized.


# Now: Aggregation *after* Rasterization

This problem exists because aggregation is out of order
with respect to rasterization.

* Blink paint prepares paint ops here  in the renderers for the visible page contents. 

* The lower renderer here has to indeed rasterize all of the blue region (though perhaps
it has chosen to not rasterized portions of the layer underneath the darker
fixed position element that I've shown.

## next

* But these raster decisions are made locally

## next

* And so the upper renderer has to rasterize all of the red page contents without
any knowledge of which portions of the page are occluded by the lower embedding
renderer

## next

* The display compositor then aggregates the contributed layers and draws
the minimal set needed.

## next

But by then, it's too late: we've already over-allocated GPU memory for all of the
red renderer's raster products and done unnecessary raster work.


# Salamander: Aggregation Before Raster

That brings us to Salamander. The cute amphibian name for what comes
after Tadpole and the last step of Viz development that has been reasonably
concretely defined.

* Salamander is about re-ordering aggregation with
rasterization.

*  I've adjusted the picture: the renderer no longer contains 
the layer compositor. 

* Instead Viz now contains a paint op aggregator and unified compositor.

## next

Blink keeps on submitting the same paint ops correpsonding to the red and blue
regions

## next

But adds additional layerization data by shipping property trees (or something them)

## next

And the paint op aggregator (here in the picture) uses this global layer knowledge to choose
the minimal set of paint ops.

## next

So that we can rasterize the smallest necessary set of paint ops corresponding to
these schematic regions here.


# Summary: Viz Status

Salmander development is probably still quite a ways out. Let me wrap
up by summarizing where we are now.

## next

Underway now we have

## next 

* The first part of Viz project is Tadpole were we relocate the DisplayCompositor
to Viz. On by 66 we hope.

## next

* OOP rasterization is happening in parallel but still requires a great deal of effort. Ideally not long after Tadpole. Say Q2 of 2018.

## next

* The SkiaRenderer compositor backend 

## next

With these three items complete, then we can start the next  additional in-parallel 
efforts. Ideally in Q3 of 2018:

## next

* Use of Vulkan graphics

## next

* Salamander's re-ordering of layerization and paint item selection


# Viz Take-aways

## next

first, remember that there are two ways to think about viz

## next

part of the Chrome servicification effort

## next

and a new and improved GPU process

## next

And second, remember that Viz improves graphics for the webplatform in three ways

## next

10 to 15% performance benefit from removing command buffers

## next

Vulkan for another...

## next

and centralized aggregation paint ops for toll-free site-isolation support


