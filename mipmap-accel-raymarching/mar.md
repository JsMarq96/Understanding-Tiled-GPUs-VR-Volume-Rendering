# Mipmap Accelerated Raymarching

An expansion to clasical raymarching, that aims to minimize the number of steps taken. Described originaly on the paper **[Indirect Illumination with Efficient Monte Carlo Denoising](https://link.springer.com/article/10.1007/s11042-020-09884-5)** the MAR technique applies a dynamic step size depending on the current mipmap level.

The main idea is to harness the use of mipmap, for skipping empty regions. This presents a good alternative to clasic raymarching, bringing some advantages:

* Dynamic step length with skipping of empty spaces
* Fewer texture fetches
* Not needed any aditional infrastructure (with the exception of the mipmap of the 3D texture)

But it also brings its own set of drawbacks:

* More complex algorithm
* More parameters to tune
* A lot more of the nasty kind of branching
* Might not be coherent to the texture cache system

## Pseudocode

```
current_mip_level = lowest one
current_sample_pos = starting position

while not found a surface:
    density = sample volume at current_sample_pos
    if density > 0.0:
        if current_mip_level == 0:
            found surface
        else:
            current_mip_level--
            current_sample_pos = go back 1 step in the current level
    else:
        if current_sample_pos.outside(get_current_voxel(current_sample_pos, current_mip_level)):
            current_mip_level++
            current_sample_pos = go back 1 step in the current level
        else:
            current_sample_pos = ray_direction * texel_size_in_mip_level(current_mip_level) * delta
```

## Comparison with traditional raymarching

Here we can see the number of steps taken for each algorithm (the whiter the pixel is, the closer it is to 150 steps, the maximun of the test).


| ![Clasical Raymarching](assets\20230414_112642_Raymarching_iterations_150_steps.PNG) | ![MipMap Accelerated raymarching](assets\20230414_112642_MAR_iterations_150_steps.PNG) |
| -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Raymarching                                                                          | Mipmap Accelerated raymarching                                                         |

## First implementation: Issues & a closer look

When working of this algorithms, I deploy them first on a desktop computer, for a more confortable workflow. After finishing the V0.1 of this techique I found great results on the desktop computer, but promplty found this results did not translate to the Quest 2.


|             | Frametimes (Desktop RTX 3080) | Frametimes (Quest 2) |
| ------------- | ------------------------------- | ---------------------- |
| Raymarching | 2.39 ms                       | 15 ms                |
| MAR         | 1.2 ms                        | 15.5 ms              |
| Speedup     | 1.991x                        | 0.98x                |

A almost 2x performance increase for just a shader change is a huge speed up. But on Quest, I found that at best, it performed slightly better. Usually, it resulted on a slightly worse result.

### Closer look

When using RenderDoc, I saw the following


|             | Texture Fetch Stall | Texture L1 miss | Textrue L2 miss | ALU instructions per Fragment | EFU instructions per Fragment | Fragment instructions | Texture Pipes busy |
| ------------- | --------------------- | ----------------- | ----------------- | ------------------------------- | ------------------------------- | ----------------------- | -------------------- |
| Raymarching | 10.835%             | 23.14%          | 19.84%          | 2206.24633789                 | 5.19638395                    | 1329271680            | 89.00164032        |
| MAR         | 46.35%              | 23.27%          | 11.09%          | 2255.65307617                 | 0.99751121                    | 1356445568            | 94.78199768        |

* The texture stall, when compared to raymarching, has gone down from and average of 46.34% to a 10.83%. This proves that there are fewer texture lookups being done.
* There are fewer texture cache misses.
* There is a increment to the EFU load from 0.99 calls per fragment, to 5.19
* The ALU load stays mostly the same, at 2206.24 vs 2256.059 per fragment.
* The fragment instructions are very high on both options.

This confirms some gains from the fewer texture lookups, but it results in a net loss from the complexity of the algorithm. But thanks to that complexity, we can try to optimize it, using this information.

### Optimizations

My first instinct is to reduce the use of ALU & EFU instructions.

According to the [Qualcomm's Official Snapdragon Mobile Platform OpenCL Optimization Guide](https://developer.qualcomm.com/download/adrenosdk/adreno-opencl-programming-guide.pdf?referrer=node/6114https:/), on the section 8.3 *Conformant vs. fast vs. native math functions* they clasify math functions based on their performance going from A (highest performance and good practice) to D (lowest performance).

The current implementation used functions of grade A (pure ALU calls), some grade B (EFU only, or EFU plus simple ALU calls) & grade C (combination of ALU and EFU with some bit manouvers in the middle).

For fixing this, the result must try to only use grade A ALU functions. Since this functions where only used on the grid size & intersections code, I wrapped it on simple array based LUTs.

This gave back some performance, experimenting frame times of arround 12.0 ms. But I felt that I could go a bit better.

#### The bigger cost

Since the bigger cost (in branching and instruction count) came from the mechanism to go up and down on the mip level. This system is fundamental since the mip level determines the size of the step size. But I tried to remove the mechanism to go up a level.

This system is in place for augmenting the step size when there has not been a ray intersection, but have passed near a surface. This makes the mip level go down, and reduce the step size. But this added a ray intersection test for each interation, with branching operators.

This test was meant to reflect what would be preffered on this hardware: less texture lookups with a higher instruction cost; more texture lookups, but with cheaper iterations.

On a worst case, the it would perform with slightly worse performance to raymarching.

One thing that I noted what some banding artifacts, but they where easily solved by adding blue noise jittering.

### The result


|               | Texture Fetch Stall | Texture L1 miss | Textrue L2 miss | ALU instructions per Fragment | EFU instructions per Fragment | Fragment instructions | Texture Pipes busy | Frametime |
| --------------- | --------------------- | ----------------- | ----------------- | ------------------------------- | ------------------------------- | ----------------------- | -------------------- | ----------- |
| Raymarching   | 10.835%             | 23.14%          | 19.84%          | 2206.24633789                 | 5.19638395                    | 1329271680            | 89.00164032        | 14.5 ms   |
| MAR naive     | 46.35%              | 23.27%          | 11.09%          | 2255.65307617                 | 0.99751121                    | 1356445568            | 94.78199768        | 15 ms     |
| MAR optimized | 22.557%             | 23.21%          | 18.4278%        | 1258.46496582                 | 0.99751121                    | 757047808             | 93.34055328        | 9.3 ms    |

As we can see, in this case the gamble paid off. With frametimes of 9.3 ms it is a very viable alternative to raymarching.

* The texture stall indicates that we are not saturating as much the texture pipelines.
* But we are not under-using then, as indicated on the busyness of the texture pipelines.
* There has been a notable reduction in the fragment instructions and the ALU use.

## The Benchmark results

Using the optimized version, I ran the benchmaks.


| Perspective  | Frametimes              | Total FPS         | Fragments Shaded   | Texture L1 misses   | Texture L2 misses   | % Time ALUs working | % Time EFUs working | ALU / Vertex        | ALU / Fragment          | EFU / Vertex | EFU / Fragment    | % Texture pipes busy |
| -------------- | ------------------------- | ------------------- | -------------------- | --------------------- | --------------------- | --------------------- | --------------------- | --------------------- | ------------------------- | -------------- | ------------------- | ---------------------- |
| Not looking  | 0.01913ms / 0.01837ms   | 120hz (0.06261ms) | 0 / 0              | 0 / 0               | 0 / 0               | 9.09091% / 9.09091% | 0% / 0%             | 28 / 28             | 0 / 0                   | 0 / 0        | 0 / 0             |                      |
| Looking far  | 1.4657ms / 1.45377ms    | 120hz (2.947ms)   | 33453 / 33455      | 60.51189 /60.63144  | 100.66218 / 100     | 12.5865% / 12.4932% | 0.05193% / 0.05153% | 28 / 28             | 1320.08215 / 1320.31787 | 0 / 0        | 0.99488 / 0.99523 |                      |
| Near         | 2.48241ms / 2.62131ms   | 120hz (5.14249ms) | 394785 / 394785    | 27.02943 / 27.60523 | 25.13162 / 27.03773 | 50.25291 / 48.46212 | 0.34695 / 0.33271   | 30 / 30             | 906.05292 / 905.28394   | 0 / 0        | 0.99849 / 0.99852 |                      |
| Close        | 5.42298ms / 5.26525ms   | 93hz (10.64436ms) | 1080614.2 / 927972 | 19.14908 / 17.30471 | 13.1689 / 11.89132  | 52.38562 / 47.14788 | 0.44555 / 0.3912    | 30.98863 / 28.224   | 791.54568 / 726.92281   | 0 / 0        | 0.99907 / 0.89918 |                      |
| FullViewport | 10.89429ms / 10.73793ms | 43hz (21.70803ms) | 2073600 / 2073600  | 10.67951 / 10.70527 | 8.05422 / 8.02575   | 54.87321 / 54.95217 | 0.41815 / 0.4231    | 32.66667 / 33.12676 | 924.71814 / 912.84009   | 0 / 0        | 0.99985 / 0.99986 |                      |

As we can see, this technique scales a bit better than raymarching, but presents another sweet spot.

We can see that the further distance presents a unexpected cost. Even thought the overall performance does not suffer due to the reduce amount of viewport fragments, I think is interesting to dig a bit depper. As we can see, both the cache misses and the ALU calls per fragment are unexpectedly high. My idea is that due to the local position disparity between pixels, is the responsible between cache misses. If the branching in the same workgroup is registered in the ALU calls per fragment, it would also explain the increase.

When looking at this data, we can see that the sweetspot for this technique is in the Close to Near range, as we can see a significant drop on perfomance when it fills the viewport.
