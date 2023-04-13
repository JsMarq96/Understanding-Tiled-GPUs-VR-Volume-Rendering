# Understanding Tiled Mobile GPUs throught VR Volume Rendering

## NOTE: This is a WIP, the structure and data are due to change in the coming months; and corrections are always appreciated  :)

## Why

There is a lot of dogmatism and hearsay in the area of Computer Graphics, specially on regard of the mobile GPUs area; and it doesnt help that one of the biggest players on this hardware space (Qualcomm) is a *really* closed ecosystem, without a lot of documentation.

My goal is to understand better the strenghts & limitations of this hardware via empirical testing & debuging tools; on the usecase of rendering techniques that are not as straightforward as the explicit geometry rendering, and that being Volume rendering, on a VR platform.

## The platform

Since there is a lot of variablity between manufacturers of mobile GPUs, I decided to go with the most ubiquous one, Qualcomm and their Snapgradragon XR2 SOC; in the Meta Quest 2 device.

My reasoning for this is straightworward: is the most ubiquouts and accessible XR device, and that allowed Qualcomm to stay as the de-facto chip maker for XR mobile-powered devices; and researching volume rendering in this aplication can bring benefits to more cost efective VR medical platforms.

This poses some problems, like the absolute absense of official documentation of the SOC, best practices, and this sort of techniques.

It also offers some interesting capabilities, in the form of Qualcomm's exclusive extensions, for Variable Rate Shadding, Frame Reprojection, Foveated Rendering, and so.

## The metodology

The main idea is to asses the cost of different rendering techiques, and try to remedy the issues of their implementation, and how they scale, the more percentage of the viewport is filled with the volume rendering in question.

The volume that is going to be presented is the ubiquous bonsai 3D texture, and I am going to center in rendering an Isosurface, since when trying to achieve a volumetric or spectral representation there is not a lot of manouverability with the cost.

The tecnhiques present some old aproaches, revisited in this paradigm; some usual suspects; and more mother aproaches:

* Raymarching
* MipMap Accelerated Raymarching
* Volumetric Billboards
* Surface nets
* Octree based rendering

If time allows, It would also be interesting how effective can be the different more traditional VR techniques on top of this list; like Foveated Rendering.

Using the RenderDoc Meta Fork, it is posible to obtain quite a lot of low-level information, in a per-drawcall matter, such as:

* Frame times
* Clocks
* Texture cache misses: L1, L2, total rates
* Total number of bytes read from memmory
* ALU usage per vertex/fragment
* EFU usage per vertex/fragment
* Fragments shaded
* Usage per viewport tile

This can help paint the cost of each different technique; in different viewport configurations

## Tiled GPUs

*on progress*

## Tecnhiques

* [Raymarching](https://github.com/JsMarq96/Understanding-Tileg-GPUs-VR-Volume-Rendering/blob/main/raymarching/raymarching.md)
