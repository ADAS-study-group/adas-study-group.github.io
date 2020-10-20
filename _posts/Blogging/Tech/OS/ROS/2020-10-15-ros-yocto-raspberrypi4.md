---
layout: post
title: ROS with Yocto for Raspberry Pi 4
categories: [Blogging, Tech, OS, ROS]
tags: [ros, yocto, raspberrypi]
---
## Reference Links

Links | Description
----- | -------------
[ROS2 yocto meta layer](<https://discourse.ros.org/t/ros2-yocto-meta-layer/9643>) Discourse | Discussion about how to install ROS2 into a yocto based system
[Best ARM board for ROS](<https://discourse.ros.org/t/best-arm-board-for-ros/152>) Discourse | Discussion about Best ARM board for ROS
[Creating_Recipes_for_ROS_modules](<https://wiki.yoctoproject.org/wiki/TipsAndTricks/Creating_Recipes_for_ROS_modules>) Yocto Wiki | This article covers creating a recipe for a ROS module. For this article we'll use the vectornav module as an example. We'll also assume you already have meta-ros in your bblayers.conf.
[meta-ros](<https://layers.openembedded.org/layerindex/branch/master/layer/meta-ros/>) Yocto Wiki | ROS (Robot Operating System) support layer for Yocto Recipe
[bmwcarit/meta-ros](<https://github.com/ros/meta-ros>) Github | building ROS 2 dashing, eloquent, foxy, and rolling and ROS 1 melodic using OpenEmbedded release series thud (Yocto 2.6), warrior (Yocto 2.7), zeus (Yocto 3.0), dunfell (Yocto 3.1), or the yet to be released gatesgarth (Yocto 3.2) on a 64-bit build machine running Ubuntu bionic (18.04).

