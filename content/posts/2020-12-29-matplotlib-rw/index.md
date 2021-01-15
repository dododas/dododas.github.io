---
title: "Animations with matplotlib"
date: 2020-12-29T22:36:55-05:00
katex: True
tags: ["python", "matplotlib"]
---

This post explains how to use the `matplotlib.animation` module to create animated plots.
<!--more-->

## Introduction

`matplotlib`[^mpl] is a python library for creating high quality scientific plots. It contains a module `matplotlib.animation` to create animations. This post shows examples of using this module to create some interesting visualizations of 2D random walks. 
[^mpl]:  https://matplotlib.org/

### Random walks

Random walks[^rw] are models for many physical phenomena, such as the movement of molecules in a liquid or gas, and the diffusion of proteins on the cell membrane. To simulate a random walk, a particle is placed at some initial position $\bold{r}_0 = (x_0, y_0)$ and advanced using the recursion relation:

$$ \bold{r}_{j+1} = (x_j + \delta x, y_j + \delta y) $$

[^rw]: https://en.wikipedia.org/wiki/Random_walk

The displacement along each axis is a normally distributed random variable: 
$$ \delta x, \delta y \sim \cal{N}(0, \sigma) $$

This algorithm produces a Gaussian random walk. The standard deviation $\sigma$ of the normal distribution is related to the diffusion coefficient of the particle[^d]. 

[^d]: $\sigma = \sqrt{2Dt}$ where $D$ is the diffusion coefficient, and $t$ is the time interval between observation
[^dif]: https://en.wikipedia.org/wiki/Diffusion

In the examples below, particles are initially placed within a small central region of the simulation domain. Each particle moves independently, following the random walk algorithm above. Over time, the aggregated random motion of the particles disperses them like a drop of ink mixing in water.

## Examples

These examples follow a common template:

1. Set up an empty plot
2. Initialize with plot elements, eg: points in their initial location
3. Update plot element attributes, eg: change point coordinates 

Step 3 is repeated at each frame to advance the animation. The function `FuncAnimation()` implements these steps. 


First, Load the needed libraries
```python 
import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl
import matplotlib.animation as anim
from IPython.display import HTML
from pathlib import Path
```

### Example 1: Diffusing particles

The first simulation shows the position of particles over time. In the next code block, `init()` plots the particles in their initial position, `move()` updates particle positions using the random walk algorithm above, and `FuncAnimation()` uses these functions as inputs to generate the animation:

{{< highlight python "linenos = table" >}}
# Initialize particle positions within a central square 
nparticles = 50
r0 = 0.45 + 0.1 * np.random.rand(nparticles, 2)
    
# Create figure
fig, ax = plt.subplots(figsize = (5, 5))
pos = [ax.plot([], [], "o", color = "C3", ms = 6, alpha = 0.8)[0] 
       for j in range(nparticles)]

# Plot initial particle positions
def init():
    for jj, p in enumerate(pos):
        p.set_data(r0[jj, 0], r0[jj, 1])
    # Fix plot appearance
    ax.set_xlim([0, 1])
    ax.set_xticks([])
    ax.set_ylim([0, 1])
    ax.set_yticks([])
    ax.set_title("Example 1: Brownian motion")
    return(pos)

# Update particle positions
def move(k, sigma = 0.01):
    # Apply random normal displacement along each axis
    for p in pos:
        x, y = p.get_data()
        dx, dy = tuple(sigma * np.random.randn(2))
        p.set_data(x + dx, y + dy)
    return(pos)

ani = anim.FuncAnimation(fig, move, init_func = init,
                         frames = 300, interval = 50, blit = True)
{{< / highlight >}}

The resulting output shows how particles diffuse and disperse away from their initial locations.

{{< iframe src="rw-particles.html" >}}

### Example 2: Particle tracks

The next example plots the trajectory of each particle. To do this efficiently, first simulate the entire trajectory for each particle, and store all trajectories in a `numpy` array:

```python {linenos=table}
def simulate_rw(nparticles = 15, nframes = 200, sigma = 0.01):
    """Simulate trajectories of diffusing particles"""
    # Initialize particle positions
    displacements = np.zeros((nframes, 2, nparticles))
    displacements[0, :, :] = 0.45 + 0.1 * np.random.rand(2, nparticles)
    # Simulate Brownian random walk
    displacements[1:, :, :] = np.random.normal(scale = sigma, 
                                               size = (nframes-1, 2, nparticles))
    # Compute positions
    r = np.cumsum(displacements, axis = 0)
    return(r)
```

Then plot the trajectories by showing the progress up to each frame. Note how the function `set_markevery()` is used to display both the marker and the line for each track. 

```python {linenos=table,linenostart=12}
# Generate tracks
positions = simulate_rw()
nframes, _, nparticles = positions.shape

# Create figure
fig, ax = plt.subplots(figsize = (5, 5))
pts = [ax.plot([], [], "-o", alpha = 0.8)[0] 
       for j in range(nparticles)]

# Plot initial particle positions
def init():
    r0 = positions[0,:,:]
    for jj, p in enumerate(pts):
        p.set_data(r0[0, jj], r0[1, jj])
    # Set plot appearance
    ax.set_xlim([0, 1])
    ax.set_xticks([])
    ax.set_ylim([0, 1])
    ax.set_yticks([])
    return(pts)

# Update particle positions
def move(k):
    for jj, p in enumerate(pts):
        r = positions[:k+1, :, jj]
        p.set_data(r[:, 0], r[:, 1])
        p.set_markevery((k, k+1))
    return(pts)

ani = anim.FuncAnimation(fig, move, init_func = init,
                         frames = nframes, interval = 50, blit = True)
```

The output shows how diffusing particles explore space:

{{< iframe src="rw-tracks.html" >}}

### Example 3: Particle trails

In this example, the animaiton shows a "trail" behind each particle rather than the complete trajectory. The maximum trail length can be changed in the `move()` function below.

{{< highlight python "linenos=table" >}}
# Create figure
fig, ax = plt.subplots(figsize = (5, 5))
pts = [ax.plot([], [], "-o", alpha = 0.85)[0] 
       for j in range(nparticles)]

# Plot initial particle positions
def init():
    r0 = positions[0,:,:]
    for jj, p in enumerate(pts):
        p.set_data(r0[0, jj], r0[1, jj])
    # Set plot appearance
    ax.set_xlim([0, 1])
    ax.set_xticks([])
    ax.set_ylim([0, 1])
    ax.set_yticks([])
    return(pts)

# Update particle positions
def move(k, trail = 30):
    for jj, p in enumerate(pts):
        if (k <= trail):
            r = positions[:k+1, :, jj]
            n = k
        else:
            r = positions[(k+1 - trail):k+1, :, jj]
            n = trail-1
        p.set_data(r[:, 0], r[:, 1])
        p.set_markevery((n, n+1))
    return(pts)

ani = anim.FuncAnimation(fig, move, init_func = init,
                         frames = nframes, interval = 50, blit = True)
{{< / highlight >}}

which produces the following output

{{< iframe src="rw-trails.html" >}}

## Summary and references

These examples demonstrate how the `matplotlib.animation` module can be used to create dynamic visualizations. 

**References**:
- https://matplotlib.org/3.3.3/api/animation_api.html
- https://towardsdatascience.com/animations-with-matplotlib-d96375c5442c
- https://brushingupscience.com/2016/06/21/matplotlib-animations-the-easy-way/
- https://brushingupscience.com/2019/08/01/elaborate-matplotlib-animations/
