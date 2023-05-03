# State of the Art & Vocabulary: Mobile VR rendering

Before starting with the techniques and results, I will be looking closely at the State of the Art in VR rendering & the rendering of mobile VR specially.

## Stereoscopic rendering

## VR Runtime: OpenXR & similars

## Foveated rendering

Based around the biology of the eye, is the most common optimization in VR. Getting its name from the Fovea Centralis, a part of the eye that is responsible the points of "high resolution" in the human eye. This technique mirrors this, reducing the number of pixels rendered the more distant they are from the designed fovea center.

There are two types, Fixed Foveated Rendering or Eye tracked (or Dynamic) Foveated Rendering. The main difference is that on fixed the fovea center is fixed on the center of the screen. But on the Eye tracked version the fovea's center follows the user's vision, via an eye tracking system in the headset. This can bring an important advantage in performance, since the area of the foviation can be very aggressive, lowering the pixel count for the render by a huge margin.

But the Eye tracked version has its own set of problems.

The first one is the lack of accessible hardware with eye tracking, being the only hardware incorporating starting at 700-800 euros (TODO: reference Vive pro 2 & Meta Quest pro).

The other problem is the speed of capture. Being camera dependant, with a pipeline of complex image processing techniques; some implementation on limited hardware might not present as huge of a performance boost. Being the main issue the speed and precision needed for tracking the eye movement at a sub 8.0 ms range; while rendering an interactive experience on top of that. (TODO: Link to John Carmack talk).

On the Meta Quest 2 & Snapdragon hardware, the Foveated rendering has native support, and is enabled by the extension QCOM_texture_foveated. This extension is always active on the hardware in order to reduce the pixel density of the areas occluded by the lenses. But the range of the foveation can be incremented, with minimal losses in quality, and a huge performance increase.

I will be leaving this configuration as is for the benchmarks.

## Frame Reproyection & Frame Generation

One crucial step of VR rendering is achieving a smooth as possible visual experience, with high-framerate & low frametimes. But as we are going to see, that is not always possible, but is fundamental for the user experience, so for that frame reprojection was used.

Incorporated on the VR application Runtime
