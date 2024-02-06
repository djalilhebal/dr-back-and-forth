---
title: Using SVG <symbol> as a Camera (Danganronpa Back and Forth)
published: false
description: 
tags: svg
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-02-03 20:05 +0000
---

I like SVG. I like Danganronpa. I tried to recreate a Danganronpa-like camera panning effect in SVG.

**TLDR:**
We can use SVG `<symbol>` and `<use>` elements to create a camera, similar to the ones you may find in video editors, game engines, and the like (After Effects, Unity, and Blender, to name a few).
The goal was to recreate a panning effect as seen in Danganronpa games or fan projects.

**PS: This is not a step-by-step tutorial.**
- This is mostly a proof of concept.
- I will provide a high-level overview, mention specific implementation details, and link to further readings.
- I assume you are somewhat familiar with CSS animations and SVG.


## The idea

### Danganronpa panning effect

In official Danganronpa games and fan projects:
[Danganronpa F: Shattered Hope][danganronpa-f-prologue] (made in After Effects),
[Project: Eden's Garden][eg-prologue-walkthrough] (made in Unity),
and even [some memes/remixes][objection-fun-danganronpa].

[danganronpa-f-prologue]: https://youtu.be/EGU4w5C_WKI?list=PLw3Hoj70YKZBdEBcpDKu8GiXG_WBVysrp&t=1398 "Danganronpa F: Shattered Hope - Prologue (Full) | YouTube"

[eg-prologue-walkthrough]: https://youtu.be/-U6aCDrH3NA?t=8581 "[No Commentary] Official Prologue Walkthrough Part 1 | Project: Eden's Garden | YouTube"
[objection-fun-danganronpa]: https://www.youtube.com/watch?v=wEQNIgRscCY "objection funk but it's danganronpa v3 (remix) | YouTube"

If I recall correctly, in Danganronpa 2, this sequence happens:
- At the start of the animation, blur all characters except the active speaker.
- Move the camera to the active speaker.
- At the end of the animation, unblur all characters.

We could try to recreate that.

We also could make it more like Danganronpa F:
- make it 2.5D (3D transforms of 2D sprites),
- add depth of vision effect,
- add motion blur effect,
- etc.

But let's focus on camera panning for now.

### Danganronpa F tutorial

> ![drf-cancel]
> **Danganronpa F: Cancel.**
> Notice that there are two cameras: A complete view of the stage and a close-up view of the active speaker (again, same stage).

[drf-cancel]: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6zekoamw03v89t9zl2x9.jpg

One of Danganronpa F's creators made a video tutorial about it.
They use After Effects, but their techniques can be employed in other contexts, like Web technologies.

See [Panning and Camera - Fanganronpa Tutorial (AE) | YouTube](https://www.youtube.com/watch?v=iOlk6GDzS8M).

Essentially, they:
- Set up the composition (background and characters).
- Use a camera (Camera 1) to show the entire composition, covering the entire screen.
- Draw a semi-transparent overlay.
- Use another camera (Camera 2) to show a part of the composition.
- Update Camera 2's position (x, y, and z) to switch focus between characters... Panning.

### Main layers and the main idea

> ![drv3-letsgo-1]
> **Danganronpa V3: World.**

---

> ![drv3-letsgo-2]
> **Danganronpa V3: World + overlay.**

---

> ![drv3-letsgo-3]
> **Danganronpa V3: World + overlay + close-up camera + UI.**
> Notice that the girl wearing a hat (Himiko) is still visible behind the overlay.

<!--
[drv3-letsgo-1]: ./drv3-letsgo-1.jpg
[drv3-letsgo-2]: ./drv3-letsgo-2.jpg
[drv3-letsgo-3]: ./drv3-letsgo-3.jpg
-->
[drv3-letsgo-1]: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x0gbiwfvfmzdp41omeej.jpg
[drv3-letsgo-2]: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/agf9z42v0kn3khuf0pmy.jpg
[drv3-letsgo-3]: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t7xl71q4sh5wrrl4ztfn.jpg

### Translating concepts

Naming things is hard. So, let's reuse terms commonly found in game dev: scene, UI, world, and camera. (These will be our variables or element IDs.)

They are comparable to concepts found in game engines (e.g. Unity or Ren'Py), video editing software (e.g. After Effects), or 3D animation software (e.g. Blender).

Elements:
- The **scene** is the whole composition.
- The **world** contains "game" objects (the background, characters, items, etc.).
- The **camera** defines a viewport or perspective on the **world**.
- The **UI** acts as an overlay on top of the world and includes textboxes and whatnot.

### Pseudocode

```html
<svg id="scene">
    <g id="world">
        <image id="background" />
        <image id="nagito" />
        <image id="hajime" />
    </g>

    <symbol id="camera" viewBox="...">
        <use href="#world" />
    </symbol>
    <use href="#camera" />

    <rect id="overlay" />

    <g id="ui">
        <text id="speaker">???</text>
        <text id="dialog">No, that's wrong!</text>
    </g>
</svg>
```

## Impl details

Suppose this is our **world** and we want to change the camera focus from Nagito (blue rect) to Hajime (red rect):
![Inkscape](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/weti2e5r6dmlmhhy77oy.jpg)

To animate the `symbol`'s `viewBox`, we could use GSAP or a similar library, but since the animation is basic, I decided to use only the `<animate>` element.

**Alternating**: SVG does not have something similar to CSS `animation-direction: alternate`. To recreate it, we have to define two animations and make each one start at the end of the other. To start this loop, we also set one of them (`toHajime`) to start at 0s in the document's timeline (i.e. once the document loads).
```html
<!-- From Nagito to Hajime and then back to Nagito -->
<animate id="toHajime" begin="0s; toNagito.end" />
<animate id="toNagito" begin="toHajime.end" />
```

**Delaying**: We want the camera to pause for a brief moment (say, 1s) at each sprite before moving to the other.
Turns out, we can write something like `toNagito.end + 1s` to delay the execution of the animation, relative to another event.
We also use [`fill="freeze"`][mdn-svg-fill] to keep the animation's last frame's state, similar to CSS `animation-fill-mode: forwards`.


[mdn-svg-fill]: https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/fill#animate "fill - SVG: Scalable Vector Graphics | MDN"

**Custom easing**: Check _oak_'s post.
Easing function used: **easeInOutQuint** (`cubic-bezier(0.86,0,0.07,1)`). See https://easings.co.


## Conclusion

???

Takeaways:
- Transferable knowledge and whatnot.
- SVG is cool.

---

- Check it live: https://djalilhebal.github.io/dr-back-and-forth/svg-ver

- Source code: https://github.com/djalilhebal/dr-back-and-forth


## Further reading

- [Basic SVG viewBox animation | Motion Tricks](https://www.motiontricks.com/basic-svg-viewbox-animation/)
    * Visited and #archived/20240202092018

- [Animated SVGs: Custom easing and timing | oak](https://oak.is/thinking/animated-svgs/)
    * Visited and #archived/20240202113846

- [Animate SVG viewBox change | Stack Overflow](https://stackoverflow.com/a/33821641)
    * The question mentioned Velocity, but the answer suggests the `animate` SVG element.

- [SVG animation delay on each repetition | Stack Overflow](https://stackoverflow.com/a/31690969)
    * We can append `anim.end + 2s` (for example) in `begin`.
    * I don't believe [`begin`'s MDN page][mdn-svg-begin] mentions this.

[mdn-svg-begin]: https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/begin "begin - SVG: Scalable Vector Graphics | MDN" 
