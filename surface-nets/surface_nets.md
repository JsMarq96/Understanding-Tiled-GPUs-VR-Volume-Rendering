# Surface nets

An algorithm that generates a triangle mesh, from an implicit surface representation.
Originally presented on the paper [Constrained Elastic Surface Nets: generating smooth surfaces from binary segmented data (1999)](https://www.merl.com/publications/docs/TR99-24.pdf), the Surface Nets method present an alternative to the more traditional [Marching cubes (1987)](https://doi.org/10.1145/37402.37422) meshing algorithm.

## Meshing algorithms: Marching cubes

The main goal of this algorithms is to generate a (most times) triangular mesh; from volume data or function (like a distance field).

One of the more common techniques is the Marching cubes algorithm. The overall idea is to iterate thought the volume, taking 8 samples around each point, in a disposition of a square. We use the detected density in this points as an index for a look-up-table that contains 256 different triangles configurations.

![Some of the marching cubes special cases.  (c) Wikipedia, created by Jean-Marie Favreau](https://github.com/JsMarq96/Understanding-Tiled-GPUs-VR-Volume-Rendering/blob/main/imgs/marchingcubes.webp?raw=true)

This makes building the mesh a very cheap process; but it brings a few problems:

* Not all cases are defined: there is a need to check for abhorrent cases (?) that are not contemplated on the base cases.
* High vertex & face count: the resulting meshes have a lot of extra vertices, and optimizing the mesh adds an extra step before rendering.
* Shape irregularities: since we are sampling using a cube formation, and the cases are build around that, its unavoidable that some areas appear "voxelized", specially when you are using a smaller sample size.

After looking at the pros and cons of this technique, I decided to go with Surface Nets.

## Meshing algorithms: Surface nets

Proposed in 1999 by Sarah F. F. Gibson, this algorithm aims to mitigate the issues present in marching cubes.

We can define this algorithm in 3 different stages:

1. Surface detection: similar to the Marching cubes, the volume is iterated and sampling around the point, 8 times, in a square formation. The main difference here is the goal here is to generate the point (if there is) of the isosurface at that position. In order to achieve this, we can either interpolate or average between the samples with density detected. On this step we only need to check for two edge cases: all the samples present density (we are inside the volume) or no sample present density (we are outside the volume), which means that we do not create a point for that position.
2. Surface triangularization: looking the presence of the neighboring points, and generate the triangle list according to that. The original paper proposes a look up of five neighbours per axis; only adding triangles where the vertices point backwards from each sampling cube.

   ![Arm Mali GPU developer guide: surface net triangularization step](https://github.com/JsMarq96/Understanding-Tiled-GPUs-VR-Volume-Rendering/blob/main/imgs/arm_surface_nets.png?raw=true)

   If we look at this image, we would only add a triangle on the vertices that make the green triangles.
3. (Optional) Smoothing step: usually, look at the connected vertices and interpolate between then. This step can lead to smoother vertices; but at the risk of "shrinking" the resulting mesh. This is because the average position of all vertices converges to the center of the mesh. In order to fix this, is possible to use a constraint in the smoothing step, but can add to the performance.

This results in vertices with a more natural shape with less vertices, when compared to Marching Cubes; with a straightforward implementation.

The following table presents the difference between volumes build from Marching cubes and Surface nets; a comparison done by [Mikola Lysenko](https://0fps.net/2012/07/12/smooth-voxel-terrain-part-2/).


| ![Sphere with Marching Cubes](https://github.com/JsMarq96/Understanding-Tiled-GPUs-VR-Volume-Rendering/blob/main/imgs/spheremc.webp?raw=true) | ![Sphere with Surface Nets](https://github.com/JsMarq96/Understanding-Tiled-GPUs-VR-Volume-Rendering/blob/main/imgs/spheredc.webp?raw=true) | ![Sphere with Marching Cubes](https://github.com/JsMarq96/Understanding-Tiled-GPUs-VR-Volume-Rendering/blob/main/imgs/goursatmc.webp?raw=true) | ![Sphere with Marching Cubes](https://github.com/JsMarq96/Understanding-Tiled-GPUs-VR-Volume-Rendering/blob/main/imgs/goursatdc.webp?raw=true) |
| :---------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Sphere with Marching cubes                                                                                                                    | Sphere with Surface nets                                                                                                                    | Goursat Surface with Marching cubes                                                                                                            | Goursat Surface with Surface net                                                                                                               |
| 1140 verts, 572 faces                                                                                                                         | 272 verts, 270 faces                                                                                                                        | 80520 verts, 40276 faces                                                                                                                       | 20122 verts, 20130 faces                                                                                                                       |

## Implementation of Surface nets

In my implementation I omitted the third step, since the result was visually close enough to the result of the other methods; and I can increase the number of sampling points in order to increase the resolution.

One other aspect is that via the use of Shader Storage Buffer Objects (SSBOs) and compute shaders, the whole process can be done in the GPU; freeing up completely the CPU:

1. Generate the three needed SSBOs: a counter storage buffer (single uint); a surface point storage buffer (a 3D vector with the space position and a integer that indicates the state of the current point); and a finalized vertices storage buffer (a series of 3D vectors).
2. Upload the volume texture to the GPU memory.
3. Dispatch the Surface detection shader with the number of samples that we want to take. This also serves as a resolution indicator for the resulting mesh.
4. Dispatch the Surface triangularization shader, with the same number of parameters. This fills the finalized vertices storage buffer.
5. Delete the surface point storage buffer.
6. Configure the VAO using the finalised vertices buffer; in order to render directly from the generated buffers.

The rendering now is of a normal mesh with Opengl: bind the VAO and render with glDrawArrays.

## Performance

### The Mesh generation step

Since there is only the need to generate the mesh once; I do it just once per run while initializing.
