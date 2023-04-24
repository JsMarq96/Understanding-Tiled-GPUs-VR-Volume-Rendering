# Understanding Tiled Mobile GPUs through VR Volume Rendering

## NOTE: This is a WIP, the structure and data are due to change in the coming months; and corrections are always appreciated  :)

## Why

There is a lot of dogmatism and hearsay in the area of Computer Graphics, specially in the area of mobile GPUs. And it does not help that one of the biggest players in this hardware space (Qualcomm) is a *really* closed ecosystem, with not a lot of public documentation. This hardware is usually quite different from their desktop counterparts, with a distinct (tiled) infrastructure. Which brings a new set of constraints to the table: it is not just a less powerful version than its desktop counterparts.

My goal is to better understand the strengths & limitations of this hardware via empirical testing & debugging tools. I plan to do this through the lens of stereoscopic volume rendering.

## The platform

Since there is a lot of variability between manufacturers of mobile GPUs, I decided to go with the most ubiquitous one, Qualcomm and their Snapdragon XR2 SOC; in the Meta Quest 2 device.

My reasoning for this is straightforward: is the most ubiquitous and accessible XR device, and that allowed Qualcomm to stay as the de-facto chip maker for XR mobile-powered devices; and researching volume rendering in this application can bring benefits to more cost effective VR medical platforms.

This poses some problems, like the absolute absence of official documentation of the SOC, best practices, and this sort of techniques.

It also offers some interesting capabilities, in the form of Qualcomm's exclusive extensions, for Variable Rate Shading, Frame Reprojecting, Foveated Rendering, and so.

## The methodology

The main idea is to asses the cost of different rendering techniques, and try to remedy the issues of their implementation, and how they scale, the more percentage of the viewport is filled with the volume rendering in question.

The volume that is going to be presented is the ubiquitous bonsai 3D texture, and I am going to center in rendering an Isosurface, since when trying to achieve a volumetric or spectral representation there is not a lot of maneuverability with the cost.

The techniques present some old approaches, revisited in this paradigm; some usual suspects; and more modern ones:

* Raymarching
* MipMap Accelerated Raymarching
* Volumetric Billboards
* Surface nets
* Octree based rendering

If time allows, It would also be interesting how effective can be the different more traditional VR techniques on top of this list; like Foveated Rendering.

Using the RenderDoc Meta Fork, it is possible to obtain quite a lot of low-level information, in a per-drawcall matter, such as:

* Frame times
* Clocks
* Texture cache misses: L1, L2, total rates
* Total number of bytes read from memory
* ALU usage per vertex/fragment
* EFU usage per vertex/fragment
* Fragments shaded
* Usage per viewport tile

This can help paint the cost of each different technique; in different viewport configurations.

In order to reduce run-to-run variance, the perspective for each test will be fixed, and the results will be averaged between multiple samples.

The tests will be done in different fixed perspectives; and a final free-roam test, for an overall experience. The fixed perspectives differ on the scale of the rendered volume in the viewport. This helps stablish the overall the cost in a per-pixel basis.

(TODO add the viewport examples)

## The test & development environment

When working with a standalone headset like the Quest 2, remote development can be a challenge. The debugging utilities are limited, with a (comparatively) huge time to deploy, when compared to desktop development. 

Taking advantage of the cross-platform nature of OpenGL, the development was done on a desktop system with the following specs:


| Processor | i5-12600KF     |
| ----------- | ---------------- |
| GPU       | RTX 3080       |
| RAM       | 16 GB 4800 MHz |

This also helped to stablish a comparative baseline of performance.

The test environment is a Meta Quest 2, running the software version XXX. The performance indicator for both GPU & CPU are set to maximum.

The main environment is in OpenGL ES 3.3, due its comparable features to OpenGL 4.0. Since this tests are not in a complex scene, with different materials and meshes, I determined that the benefits of working in Vulkan in this set of test could be negligible (at least in most test). Most of the techniques require one draw-call per eye.

## Tiled GPUs

*on progress*

## Tecnhiques

* [Raymarching](https://github.com/JsMarq96/Understanding-Tileg-GPUs-VR-Volume-Rendering/blob/main/raymarching/raymarching.md)
* [Mipmap Accelerated Raymarching](https://github.com/JsMarq96/Understanding-Tileg-GPUs-VR-Volume-Rendering/blob/main/mipmap-accel-raymarching/mar.md)

## Repositories

* [Desktop, OpenGL 4.0 implementation](https://github.com/JsMarq96/Volume-Rendering-Desktop)
* [Quest, OpenGL ES 3.3 implementation](https://github.com/JsMarq96/Quest-Tiled-Volume-Rendering)
