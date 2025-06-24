---
layout: default
title: Computational Robotics Design Library
---

# Framework for Automated Design of Tubular Kinematic Robots

## Description 

I am working on a python library that helps automate the design process for tubular robots, with an algorithm that generates collision-free tubular kinematic trees given Denavit-Hartenberg parameters to specify the joint axes. However, ensuring collision-free configurations necessitates a greedy approach, meaning the joint placement needs to be further optimized (as seen in the before and after images) in order to meet workspace constraints. To enhance space efficiency and ensure the reachability of specific end-effector positions, I use a combination of downhill simplex and particle swarm optimization techniques with modified constraints and considerations for particular geometry. This allows for a discrete set of reachable joint states/end effector poses to be computed while avoiding self-collisions, making it easier for designers to visualize the resulting behavior and make any input modifications to the algorithm as they deem necessary. I also fully automate the generation and compilation of CAD files using a custom joint module library, streamlining the fabrication process for these designs. The library is also integrated into an intuitive GUI, which will make it accessible to users without coding experience.

[Back](../index.html)