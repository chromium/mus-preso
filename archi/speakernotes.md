# Title slide...

You may have heard of Mus, mustash, tadpoles, etc. I'm going to try
to give you a big-picture overview of all of the moving parts.

# What's Mus Anyway?

It's part of servifi..on effort.

The Mojo UI Service. It's a single centralized set of UI services for
a tree of windows. So, at it's simplest: it's like a window server:

Mus provides an aribtrary embedding tree of windows where each client
process can own an arbitrary subtree of windows

It de-multiplexes input across the tree

and multiplexes output onto the available displays

With the goal of making OOPIFs first class: to make input and graphics
efficient both for the Chrome UI embedding a web contents or the web
contents embedding an arbitrary nesting of OOPIFs.

So, before continuing, first an aside...

# Aside: What's Servicification?

# Servicified Chrome

Mus is the red bit. Click on it. It's called UI in the picture.

# Aside: Naming

Next aside: naming

Mus is divided into two processes. these are processes. But one of the
points of mojo is that services can be

# Centralized Event Dispatching

here... I can use pictures from the input world...

mutate the picture to deliver events when I click. To show how events
always go from UI to each of the clients.

and repeat the picture for graphics...


# Centralized Graphics

here I need a new pic that shows how everybody's contribution to the
scene flows down

mutate pictures to show how all of the CFs and raster data flow down

and edit the SVG... (how hard?) (do that last)

# First Class OOPIFs

I made a claim about first class OOPIF support. Mus has an
aspirational goal of equality: all clients whether ash, chrome, top
level renderers or oopif renderers are treated the same. In
particular:

The ui and viz service must meet (or exceed) the graphics perf bar
while embedding Chrome into Ash as is achieved by the current
directly-coupled in-browser implementation of Ash and Chrome.

This is hard. We're not going to get there all at once. Lots of
refactoring in our future. Which brings us to the Roadmap.

# Mus Roadmap

So how do we get there... Explain steps

# Chrome Now

# Mushrome

# Process Separation Forces Improvements

In this section.. drive the idea of changes make things better.

# Tadpole

Why take rasterization out?

To answer this, we have to look inside VIZ

# Inside VIZ

prep for direct compositing

# Direct Compositing and Vulkan!

Direct connection to the GPU will permit optimizations and performance
improvements up to perhaps 20% by improving our ability to overlap
the creation of (Vulkan) command buffers with the dispatching of them

# After Tadpole: Central Compositor

# Salamander

# Summary

