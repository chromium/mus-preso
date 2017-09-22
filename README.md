# mus-preso
A directory of public presentations for mus. Mus is the mojo UI service -- a window
server extracted from the existing graphics, compositing and input management
code in Chrome. 

Slide decks here:
*  [Blink-on 7](https://cdn.rawgit.com/chromium/mus-preso/a5701889/blinkon/index.html)
*  [BlinkOn 8](https://goo.gl/3RLBav) Viz: Chrome Graphics Futures
*  [Event Targeting](https://cdn.rawgit.com/chromium/mus-preso/706199ba/events/index.html)
*  [Architecture Update](https://goo.gl/Kd0jy8)
*  [Event Targeting Update](https://goo.gl/YKM4tF) All the latest (August 8, 2017) on the status of the event targeting rework for viz.

# Production Notes
Some people asked me how I made these slides. I did this:

* I wrote `html` content using the [Google HTML5 slides template](https://code.google.com/archive/p/html5slides/). It makes very pretty slides. Good use of the web platform seems appropriate for slides about Chrome.
* I drew diagrams in OmniGraffle.
* I exported SVG per diagram and removed the layout-affecting spaces between `<tspan>` elements with the following [sam](http://doc.cat-v.org/bell_labs/sam_lang_tutorial/) script: `X/\.svg$/  ,x:</tspan>[\n ]*<tspan: x:>[\n ]*<: c:><:`.
