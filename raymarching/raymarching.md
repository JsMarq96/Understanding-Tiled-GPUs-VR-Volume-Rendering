# Raymarching

Raymarching is quite the ubiquious technique for representing volumes, mostly due to the straightworward & simple implementation, and the flexiblity that it presents, being able to render a isosurface or a spectral representation without changing nothing more than 2 lines.

But the cost of iteration is rather high, ending in a lot of texture sampling for each sample; and that being applied to mobile, I am expecting poor perfomance, due to the memory bandwith limitations.

In this implementation the step size is fixed to 0.005 and a maximun of 200 iterations, contained into the proxy geometry of a unit cube; and the algorithm was set to find de isosurface of at least a density of 0.15, and return its position as a fragment color. This was able to present the volume fully, without any percived loss in quality.

I will not be delving deeply in this technique, since its main interesintg point is to serve as a baseline & comparison point.

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

As expected the main issue with raymarching is its poor fragment scalability on the viewport, due its high cost per fragment, most probably due to a texture memory bandwidth issue, as we can see the texture vertex fetch stall rising to a 50% the more of the fragments that are on the viewport.
