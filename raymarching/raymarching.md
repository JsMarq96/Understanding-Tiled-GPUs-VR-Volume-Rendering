# Raymarching

Raymarching is quite the ubiquitous technique for representing volumes. Thanks to the straightforward & simple implementation, and the flexibility that it presents. My implementation is based on the article [GPU Gems 2: Per pixel displacement mapping with Distance functions](https://developer.nvidia.com/gpugems/gpugems2/part-i-geometric-complexity/chapter-8-pixel-displacement-mapping-distance-functions). This was adapted to work without distance fields, by setting a constant step.

But the cost of iteration is rather high, ending in a lot of texture sampling for each sample; and that being applied to mobile, I am expecting poor performance, due to the memory bandwidth limitations.

## Implementation

The fundamental idea of raymarching lies in intersecting a ray with a fixed step. The ray's origin lies in the center of the camera, with the direction of a fragment in the projection plane. When one of the rays intersect with the volume, the raymarching process begins.

![Raymarching diagram, from Transmittance function mapping (DOI:10.1145/1944745.1944751)](https://github.com/JsMarq96/Understanding-Tiled-GPUs-VR-Volume-Rendering/blob/main/raymarching/assets/20230425_152811_Notations-and-principle-of-a-classical-ray-marching-algorithm-to-compute-single.png?raw=true)

In this current implementation we are interested to find the isosurface. In order to achieve that, we stop the iteration the moment that we detect the sampled density goes up a certain threshold.

```
while point p is inside volume or has not exceeded the max number of iterations:
     p = p + ray_direction * step_size
     density = sample volume at p
     if p > density threshold:
          return p as isosurface point
```

This algorithm presents 3 parameters:

* Step size: distance between sampling points along the ray. A huge step could lead to a loss in detail and inconsistency between viewpoints. But a small step could lead to poor performance since there are unnecessary samples taken along the way.
* Maximum number of iterations: in order to restrict the number of samples, for a guaranteed minimum in performance. Few steps can lead to only a partial render; and too much can lead to performance drops in tricky perspectives.
* Density threshold: since the interest of this test is to find a isosurface, we use the density threshold as a way to determine when the surface starts. A high number can include noise present in the original image, or unwanted data; and setting it to low, can lead to skipping some

For this test the step size is fixed to 0.005 and a maximum of 200 iterations, with a density threshold of 0.15. This was able to present the volume fully, without any perceived loss in quality.

## Results

###### TODO: Add shader ALU & EFU total use per frame & % Of Busy Texture pipelines; and reference viewport pictures


| Perspective  | Fragments shaded  | Frametime (ms)                          | Texture L1 Miss       | Texture L2 Miss       | % Time ALUs Working   | % Time EFUs Working | ALU/ Vertex         | ALU/ Fragment           | EFU/ Vertex       | EFU/ Fragment     | % Texture Pipes Busy |
| -------------- | ------------------- | ----------------------------------------- | ----------------------- | ----------------------- | ----------------------- | --------------------- | --------------------- | ------------------------- | ------------------- | ------------------- | ---------------------- |
| Not looking  | 0 / 0             | 0.01875 /0.01837 (0.0632 -> 120 fps)    | 0% / 0%               | 0% / 0%               | 8.71659% / 9.0909%    | 0% / 0%             | 28 / 28             | 0 / 0                   | 0 / 0             | 0 / 0             | TODO                 |
| Looking Far  | 33453 / 33455     | 1.72726 / 1.7800 (3.5039 -> 120fps)     | 53.84705% / 54.14334% | 71.53591% / 72.11881% | 18.73432% / 18.27434% | 0.04497% / 0.04368  | 28 / 28             | 2303.60571 / 2303.37256 | 0 / 0             | 0.99488 / 0.99523 |                      |
| Near         | 394785 / 394785   | 3.95576 / 4.13165 (8.12841 -> 120fps)   | 23.40274% / 23.98612% | 18.48331% / 18.57747% | 52.61275% / 50.77347% | 0.21843% / 0.20895% | 30 / 30             | 1611.70117 / 1611.56494 | 0.99849 / 0.99852 | 0 / 0             |                      |
| Close        | 1086118 / 1031080 | 8.82709 / 8.46807 (17.23671 -> 58fps)   | 16.01881% / 16.23618% | 9.13429% / 8.90372%   | 53.85233% / 53.84847% | 0.27456% / 0.26864% | 30.94737 /31.36     | 1421.67712 / 1450.03772 | 0.99906 / 0.99909 | 0 / 0             |                      |
| FullViewport | 2073600 / 1866240 | 17.42682 / 17.33124 (34.80482 -> 28fps) | 7.37855% / 6.59615%   | 6.18813% / 5.73779%   | 55.20058% / 49.56052% | 0.25943% / 0.23534% | 32.71268 / 30.07588 | 1612.01041 / 1430.41827 |                   |                   |                      |

## Conclusions

As expected the main issue with raymarching is its poor fragment scalability on the viewport, due its high cost on texture reads per fragment.

One interesting behaviour observed is that the highest texture stall (77%) and the highest total of readded bytes from a texture (52538700) came from the far rendering perspective. This can be explained when looking at the cache misses on texture memory: since we are looking at fragments close to each other in screen-space, but are further on texture space. This is not presented as an issue in the overall frametime, due to the small number of fragments shaded in those stages.

But the closer we are to the model, we can observe a relieve on the the texture pipeline, and minimization on the cache misses. (NOTE & TODO: Contrast this with the Textures/Fragment Metric! Is the number of calls higher near vs far??)

This indicated that the texture cache is extremely important mechanism in this architecture; and that in order to achieve good performance there need to be a balance between locality & number of fragments. The following methods will try to remedy this problems.
