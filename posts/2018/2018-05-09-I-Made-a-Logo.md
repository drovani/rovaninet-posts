---
title: I Made a Logo
category: Rovani in C♯
tags:
- svg
- experiments
date: 2018-05-09
---

Scalar Vector Graphics (SVG) has been one of those many mysteries that I've only vaguely been familiar with. Picking up bits of knowledge here and there, I surmised that an SVG image was a set of instructions for how to draw the picture, as opposed to being a mapping of the color of specific pixels. SVG images, thus, could scale to any level and still be the same image. What I had no clue about, though, was what the XML inside of the SVG element was doing. As with all things, I started with a simple Google search that led me to the [SVG Tutorial](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial) on MDN Web Docs. It is a well written breakdown of how to draw your first SVG image. I highly recommend this, even for those that intend to just use a painting program to spit out the raw SVG. It was a highly illuminating journey.

<figure class="blockcenter">
    <svg version="1.1" xmlns="http://www.w3c.org/2000/svg" width="270" height="270" class="blockcenter">
        <defs>
            <g style="fill: #F00" transform="rotate(30) scale(.33)" id="6-point-star">
                <polygon points="15 0 4.33 2.5 7.5 13 0 5 -7.5 13 -4.33 2.5 -15 0 -4.33 -2.5 -7.5 -13 0 -5 7.5 -13 4.33 -2.5" />
            </g>
            <g id="logo">
                <g style="stroke:#B3DDF2; stroke-width:10; fill:none">
                    <path d="M35 5 v80 M35 45 h-30 M35 50 l-20 20" id="letter-r" />
                    <path d="M55 5 v80 M55 45 h30" id="letter-p" />
                    <circle cx="45" cy="45" r="40" />
                </g>
                <use href="#6-point-star" x="45" y="20" />
                <use href="#6-point-star" x="45" y="36.67" />
                <use href="#6-point-star" x="45" y="53.33" />
                <use href="#6-point-star" x="45" y="70" />
            </g>
        </defs>
        <use href="#logo" x="0" y="0" transform="scale(3)" />
    </svg>
    <figcaption class="blockcenter" style="width: 50%">A backwards, capital r; a forwards, capital p; contained and encircled; all in a light shade of blue. Vertically aligned are four six-pointed stars, all of which reflects the Chicago origins of Rovani Projects.</figcaption>
</figure>

The first realization I had is that I retained almost nothing from the trigonometry courses I took in High School and College. I remembered the general concepts, so searching online for what I needed wasn't too painful. However, remembering the difference between _sin_ and _cos_ and when to use each? That knowledge is completely gone. For example - how do I figure out the ratio between the inner radius and the outer radius of the star in order to obtain a 30&deg; angle? No freakin' clue! I guessed, and hoped that it looked good enough. If I were to put together a formal description of the logo, then I would figure out that ratio.

How could I make it better? If you deconstruct the SVG, the legs of the R and P are blunt and just happen to be covered by the circle. It is terribly ugly if you strip the circle away. As such, I feel a better way to do it would be to draw the edges of the polygon and fill it all in from there. I think a better version of the logo wouldn't have the circle going all the way around - but instead just being the curve of the R and the P. That would make those letters more distinguishable.

It was an entertaining exercise. I will probably use this logo as a temporary graphics that is better than nothing (or maybe nothing is better - I think I can still claim that I'm new to this). At some point, I think I will engage the services of someone better suited to doing this sort of work. But in the meantime, at least I got to learn something.