# Volumetric Billboards

The next technique presents a departure from the other methods. Published on 2009, [Volumetric Billboards]([https://doi.org/10.1111/j.1467-8659.2009.01354.x](https://doi.org/10.1111/j.1467-8659.2009.01354.x)) aims to achieve volumetric rendering on super imposed proxy geometry, that samples the volume on slices.

The main idea is to take advangate of the structure and normal behaviour of a GPU; but achiving a look and feel of a volumetric render with a more "unatural" technique, like raymarching.

## The techique

The paper proposes the use of a slized bounding box, with the slizes aligned to the viewport. This slices are computed using mesh shaders. When rendering this slices, they sample the volume based on the fragment position, inside the cube.

![Fig taken from the volumetric billboards paper](assets\20230426_153603_bill.PNG)

The distance between the slices and the number of slices can vary, depending on the distance to the camera.

By superimposing all the slices (in a back-to-front manner, with alpha blending), you get not only a volume render, but also a volumetric render, for the same cost. This, convined to the straightfoward use of the GPU, made me thik that could be extreamly usefull for mobile render devices.

But one limitation of this techique is, apart from the number of drawcalls, the memory usage & the overdraw. 

## Implementation

My implementation slightly differs from the proposed on the paper. Following the advice of MediaMolecule's Alex Evans on the [SIGGRAPH presentation about the Dreams render](https://advances.realtimerendering.com/s2015/AlexEvans_SIGGRAPH-2015-sml.pdf), there is no need for expensive mesh shader or on-the-fly mesh generation.

The proposed solution is to use a succession of quads, aligned with the most facing axis to the viewport. This worked excelently, and when adding some margin to the changes on aligment, the change on aligment is quite seamless.

One other change that I decided is to order the render of the quads front-to-back. This is oposed to the recomendation of the paper,  since the goal there is to accumulate the samples for rendering volumetrics.

But here, my goal is to render a volume, the isosurface. In order to achieve this, I am only rendering the fragments that cross the depth threshold, and making it opaque. This mixed with the front-to-back ordering, allows to prevent overdraw, enhancing performance.

One pitfall easy to have in this technique is to mess the rendering order. Since we are super-impossing quad after quad, it can lead to a ton of overdraw. This can bring a worse case (full viewport) from a acceptable 16ms to a stuttery 29ms. In order to avoid this, the render order must only start drawing from the closest of the camera.

### Implementation: Drawcalls and OpenGL

Previusly, I talked about how for this project, it does not make a lot of sense to work with Vulkan due to the nature of the algorithms on this project. But this is the only exception: here we are rendering 200 to 125 drawcalls per eye, per frame. On OpenGL, the cost of the drawcall can start to make a difference; end that can be mitigated with the use of instancing.

But here, I decided not to implement this (yet). Since the cost of the amount of drawcalls can be intuited on the "Not looking" perspective. If I have time, I will come back to this with this changes.