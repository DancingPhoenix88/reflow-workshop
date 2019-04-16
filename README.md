# reflow-workshop
Coding materials for my workshop about *"Reflow & Repaint events"*.

___
## 1 - Rendering pipeline
![Javascript > Style > Layout > Paint > Composite](https://developers.google.com/web/fundamentals/performance/rendering/images/intro/frame-full.jpg)

    Javascript:     Execute JS code to interact with DOM tree
    Style:          Calculate style in CSSOM tree to apply to DOM tree
    Layout:         Calculate position, size to arrange elements in the web page
    Paint:          Draw elements into layers (separated by 3D attributes or manually)
    Composite:      Apply transform to layers (without re-painting !!!)

___
## 2 - What are restyle, reflow, repaint events
    Recomposite:    Changing opacity & transform requests GPU to apply effects to rendered textures
    Repaint:        Changing color, border radius, shadow ... requests browser to re-paint element (then re-composite)
    Reflow:         Changing position, size ... requests browser to re-arrange / re-layout ALL RELATED elements, THEN re-paint & re-composite them
    Restyle:        Changing CSS atributes in general, leadaing to ONE of following events: reflow, repaint, recomposite
   
___
## 3 - Why are they slow
    One element changes in layout forces browser to reflow before processing next requests
    Many element changes ? -> Repeat forcing reflow (-> repaint & recomposite)
    .
    Add / remove a DOM node ? Reflow + Repaint
    Hide element by 'display:none' ? Reflow + Repaint
    Hide element by 'visibility:hidden' ? Repaint
    Hide element by 'transform:scale(0)' ? None

___
## 4 - Workshop - part 1
**Target**: Draw balls moving horizontally, smoothly with 60FPS. More balls are better.

    File: `index.html`
    Screenshots: `screenshots/*.png`
    Check commit messages for more details.
    
- [v0](https://github.com/DancingPhoenix88/reflow-workshop/commit/d1a4ac64be51f4fa7dcccd9ccc7fd2d95b4ad2ba):     setInterval -> Unstable frame rate with 100 balls (Introduce Chrome Devtools, FPS meter, tab Performance)
- [v1](https://github.com/DancingPhoenix88/reflow-workshop/commit/ebeb8910027446fe42a83d176b723ef0b64d8b40):     requestAnimationFrame -> 100 balls but still force reflow
- [v2](https://github.com/DancingPhoenix88/reflow-workshop/commit/a7cfbf92281db5f4caf45f1242401c2f4804c241):     separate get & set style -> 600 balls
- [v3](https://github.com/DancingPhoenix88/reflow-workshop/commit/f08f1afb9ce0282d536bc3b25ac7d03b145652df):     cache offsetTop -> 800 balls
- [v4](https://github.com/DancingPhoenix88/reflow-workshop/commit/b3ab67707fbb012b68a89d1c0e0c821f5c3d760f):     use `transform` instead of `position` -> 1K balls (replace Repaint by Restyle, Introduce `sessions` in tab Performance)
- [v5](https://github.com/DancingPhoenix88/reflow-workshop/commit/59d5bc86281fe9fe6d4ba9f76fa01efc2950f1a2):     apply culling -> 12K balls !!! (Tab `Layer`, use "Camera" for visual debugging)
- [v6](https://github.com/DancingPhoenix88/reflow-workshop/commit/e2284fd0b369de3317a89264d76d22e10fd15dd8):     hide elements by moving to a hidden node -> 80K with hiccup (DOM GC)
- [v6b](https://github.com/DancingPhoenix88/reflow-workshop/commit/3bb90fd056f06e1a7e40de17a9c3262bb053b64f):    batch showing elements by DocumentFragment -> better but it seems GC appears more often
- [v7](https://github.com/DancingPhoenix88/reflow-workshop/commit/e8aca601180e1597e42fd2e1aa5a4bbed2cdc268):     pre-calculate visible index range -> 140K balls, and split functions in profiler
- [v7b](https://github.com/DancingPhoenix88/reflow-workshop/commit/faa669e159bda36467c7141e7a39eea122e90a1f):    separate updating position and display ? nothing improved -> ignore
- [v8](https://github.com/DancingPhoenix88/reflow-workshop/commit/2b5f0c1eab7105bc9546e2a7e9c0a8be2d8ae866):     only update dirty elements by comparing index range -> 1M balls !!!


**Lessons**:
- `top/left` with `position:absolute` will not trigger many reflow events
- Put restyle, reflow, repaint commands in `requestAnimationFrame` instead of `setInterval/setTimeout` for a stable frame rate
- Group restyle, reflow repaint commands to make them trigger all at once. Check FastDOM for explaination.
- Optimize more and more if you can (culling, dirty techniques). 
- Profiler is your friend. Trust your tools, not someone's rule, because browser is changing quickly.
- Batch `appendChild` to DOM using `DocumentFragment`
- Generate many elements by `innerHTML` & `cloneNode`

**Some more improvements / experiments**:
- Find & fix memory leaks
- Re-use `tops[m]` to reduce `tops.length`
- Create default `movers` as hidden ones. Check branch `feature/animate-position/v8-advanced` for more details.
- Use `ArrayBuffer` to store movers
- Promote `#inputContainer` into a separate layer using `transform`
- Disable each CSS in `.move`r to measure which style is heavy (try `border-radius`, `radial-gradient`)


Since v8 will not be affected by number of balls anymore:
- Script:   Number of balls change each frame <= 2 * max-visible-balls
- Restyle:  Number of balls move = max-visible-balls
- Reflow:   Number of new visible/hidden balls <= 2 * max-visible-balls
- Repaint:  Number of visible balls is almost fixed

=> No need to improve more

___
## 5 - Workshop - part 2
**Target**: Improve balls drawing with opacity and transform to mimic 3D space

    File: `index_3d.html`
    Screenshots: `screenshots/animate-3d/*.png`
    Check commit messages for more details.

- [v8.1](https://github.com/DancingPhoenix88/reflow-workshop/commit/1c3e1b250edd3d685d3f648e203ff84c85dee450):   Animate 'width' & 'height' -> 200 balls / 15 FPS (unstable)
- [v8.1b](https://github.com/DancingPhoenix88/reflow-workshop/commit/8d0a6aa1c2f4f94b4fbc288cc73410c67cd53dbb):  Use 'cssText' for batching changes -> not improve (+ Performance Monitor)
- [v8.2](https://github.com/DancingPhoenix88/reflow-workshop/commit/c82907d4adf4f7e582a8190d5aa51a4563dd0dcf):   Use 'transform' ('translate' & 'scale') to animate -> 100K balls / 60FPS
- [v8.2b](https://github.com/DancingPhoenix88/reflow-workshop/commit/1db447982db1dc24733cbf1c36a8db9b4a366d1d):  Add 'opacity: 0.8' -> 200 balls / 30FPS (unstable)
- [v8.3](https://github.com/DancingPhoenix88/reflow-workshop/commit/6fa164236fd3bed61fda8460ceb45d12315ed630):   Add 'will-change' -> 1K balls / 60FPS
- [v8.4](https://github.com/DancingPhoenix88/reflow-workshop/commit/800804f6cdf7110ee4e58da203f891678ffa3868):   Animate 'opacity' -> 1K balls / 60FPS

```
Check 'layer borders' option and you will see that each ball is in a separate layer (orange border), this is why FPS is low.
    More specific, open `Layers` tab and each .mover is in a layer with `Compositing reasons` = `willChange`.
    Go back to v8 and check layer borders, they are cyan -> many elements are combined into one single layer
```

**Lessons**:
- Best CSS attributes for animation: opacity, transform
- `will-change` separates element in its own layer (best for opacity, transform animation)
- Some more improvements / experiments:
- CSS custom properties (force separate paint each ball !!!)

___
## 6 - Workshop - part 3
**Target**: Replace JS animation by CSS animation

    File: `index_3d_css.html`
    Screenshots: `screenshots/animate-3d-css/*.png`
    Check commit messages for more details.

- [v9.0](https://github.com/DancingPhoenix88/reflow-workshop/commit/9df3e3a77ba7f9e1ffdcb13e6280d5c7c9e1e031):   Move 1 ball by CSS
- [v9.1](https://github.com/DancingPhoenix88/reflow-workshop/commit/da6cd37450538fe3438d0f440fe85bd6a79d3a2a):   Generate many balls
- [v9.2](https://github.com/DancingPhoenix88/reflow-workshop/commit/ed08725cd346d61cc373ce9084ab3337c17036ea):   Add delay to each ball
- [v9.3](https://github.com/DancingPhoenix88/reflow-workshop/commit/3531076d5e26ac2ab3063840d3b6b9cb1bcf9de3):   Pause
- [v9.4](https://github.com/DancingPhoenix88/reflow-workshop/commit/6e4ef53114d89b7fcf57b17a501b00b5aef71133):   Apply culling
- [v9.5](https://github.com/DancingPhoenix88/reflow-workshop/commit/193bf8d251f08482f029e4ee6aa685f124eb760c):   Dynamic CSS animation

**Lessons**:
- CSS animations are useful and handy for simple animations
- CSS animations start immediately when assigned class, or becoming visible
- It is hard to control and optimize CSS animations (time, delay, dynamic parameter like position ... ), but we can try CSS custom var

___
## 7 - List of properties and functions forcing reflow
JS:     https://gist.github.com/paulirish/5d52fb081b3570c81e3a

CSS:    https://csstriggers.com/
   
___
## 8 - More references
Presentations:
- https://speakerdeck.com/addyosmani/velocityconf-rendering-performance-case-studies?slide=1

Documents hub:
- http://jankfree.org/
- https://developers.google.com/web/
- https://developers.google.com/web/fundamentals/codelabs/web-perf/
- https://developers.google.com/web/fundamentals/performance/rendering/
- https://developers.google.com/web/tools/chrome-devtools/rendering-tools/
- http://wilsonpage.co.uk/preventing-layout-thrashing/
- https://www.igvita.com/slides/2012/web-performance-for-the-curious/#29
- https://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome

Advanced profiling tools:
- http://dev.chromium.org/developers/how-tos/trace-event-profiling-tool
- https://www.html5rocks.com/en/tutorials/games/abouttracing/
- https://www.gamasutra.com/view/news/176420/Indepth_Using_Chrometracing_to_view_your_inline_profiling_data.php
