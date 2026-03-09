---
date: 2026-02-23
draft: true
---

# ROS 2 and IPC for Robotics Applications

A deep dive into the internal of ROS 2 and other alternative Inter-Process 
Communication (IPC) options.

<!-- more -->

During the work for my latest robotics project for autunomous navigation, I encoutered 
some challenges with the default ROS 2 DDS middleware that bottlenecked the 
transmission frequency of LiDAR stream as well as image stream, which both have pretty 
high throughput. This has led to the discovery and investigation of the options of 
ROS 2 middlewares, different Inter-Process Communication (IPC) protocols and how ROS 
works internally which isn't being well documented 
