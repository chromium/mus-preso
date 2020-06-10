# mus-preso
A directory of public presentations for mus. Mus is the mojo UI service -- a window
server extracted from the existing graphics, compositing and input management
code in Chrome. 

Slide decks here:

*  [Blink-on 7](https://chromium.github.io/mus-preso/blinkon/index.html)
*  [BlinkOn 8](https://chromium.github.io/mus-preso/blinkon8/index.html) Viz: Chrome Graphics Futures
*  [BlinkOn 9](https://chromium.github.io/mus-preso/blinkon9/index.html) Chrome Graphics: Viz Update 
*  [Event Targeting](https://chromium.github.io/mus-preso/events/index.html)
*  [Architecture Update](https://chromium.github.io/mus-preso/archi/index.html)
*  [Event Targeting Update](https://chromium.github.io/mus-preso/eventupdate/index.html) All the latest (August 8, 2017) on the status of the event targeting rework for viz.
*  [Seprable Objectives](https://chromium.github.io/mus-preso/twogoals/index.html) Overview of decoupling viz and mus launch schedules.
*  [Viz: Roadmap 2018](https://chromium.github.io/mus-preso/roadmap18/index.html) Roadmap for Viz team work in 2018.
*  [GPU Team Meetup 2019](https://chromium.github.io/mus-preso/gpumeetup19/index.html) Vulkan/Chrome Overview slides from GPU Team meetup.

# Production Notes
Some people asked me how I made these slides. I did this:

* I wrote `html` content using the [Google HTML5 slides template](https://code.google.com/archive/p/html5slides/). It makes very pretty slides. Good use of the web platform seems appropriate for slides about Chrome.
* Or I used [pandoc](http://pandoc.org) to process Markdown into HTML5 slides.
* I drew diagrams in OmniGraffle.
* I exported SVG per diagram and removed the layout-affecting spaces between `<tspan>` elements with the following [sam](http://doc.cat-v.org/bell_labs/sam_lang_tutorial/) script: `X/\.svg$/  ,x:</tspan>[\n ]*<tspan: x:>[\n ]*<: c:><:`.
* The slides are served via GitHub pages.
