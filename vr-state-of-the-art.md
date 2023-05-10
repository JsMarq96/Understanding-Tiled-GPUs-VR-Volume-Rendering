# State of the Art & Vocabulary: Mobile VR rendering

Before starting with the techniques and results, I will be looking closely at the State of the Art in VR rendering & the rendering of mobile VR specially.

## Stereoscopic rendering

## VR Runtime: OpenXR & similar APIs

With the current rise of XR manufacturers, and a lot of different XR enabled devices, there is a need for a OpenGL like API, to abstract manufacturer explicit code form the applications. Khronos Group's response is OpenXR, a cross platforms that generalises all XR devices. From a flat-screen AR application to a virtual production set, all these cases are covered on the API.
This system provides a runtime, that interfaces with the XR devices and its capabilities, and connect them to the user application.

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

Incorporated on the VR application Runtime, a is system that morphs a old frame in order to match the current head movement. This can alleviate the perceived smoothens, for parts that the application cannot serve frames as the specified rate.
They can range from a basic projection step; to a depth corrected re-projection. But due to the stopgap nature of this technique, usually its a more simple and less expensive approach. The main issue with this techniques si that, even thought they work fine for momentary frametime spikes, the artifacts can be noticeable, and might contribute to nausea during a continuous exposure.

This techniques have multiple names, depending on the manufacturer: Asynchronous Spacewarp from Meta or Asynchronous reprojection from Valve. But last year, Meta released a new API fro their mobile SDK, the Application Spacewarp system. When enabled, the XR runtime limits the framerate of the main application to half of its target; and generate a new frame, based on the data from the previous frame.

The main idea is to increase the frametimes for more complex rendering; while not having a huge cost in perceived smoothness.

One main advantage of this technique is that is included in the application, via the SDK, and it can use the pipeline state. The other systems only have the final frame, in order to make occlusion & motion estimates. In order for this to work, you need to provide the color result of the last frame, the depth buffer and a velocity buffer.

The results, for most users, are slightly imperceptible from the natural render, with a visual gain in fluidity. This system can really make a difference, specially when targeting lower framerates, in some cases gaining an up to 70% gain in frametimes.

However, this functionality is not going to be enabled in my test, since it muddies the direct capabilities of the hardware. And also, when deploying in the real world, you can always enable this features for lighter frametimes.
