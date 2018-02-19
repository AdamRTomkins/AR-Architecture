---
layout: post
title: "Ground Plane Detection"
img: ObjectBrowser/WestStreet.jpeg
date: 2018-02-19 15:20:00
description: Release your AR projects from trackers using this Tutorial on Ground Plane detection.
tag: [Augmented Reality, Architecture, Unity, Ground Plane, Plane Detection]
---

In this tutorial we are going to unleash our AR apps, using the Vuforia ground place detection API. First, this is not supported by the same range of devices that the tracked AR experiences are, and you should make sure your device is on the [List of supported devices](https://library.vuforia.com/articles/Solution/ground-plane-supported-devices.html).

The Vuforia API makes this an easy enough process, which starts out identical to our previous tracked examples.

These steps will guide you through the process of creating your first Ground Plane experience in Unity, quickly passing through the first few identical steps.

1. Open Unity 2017.3 or newer
2. Create a New Project
3. In the Edit Menu select: Project Settings> Player. In the “XR Settings” section of the PlayerSettings, make sure that “Vuforia Augmented Reality Supported” is checked.

4. Replace the default “Main Camera” with an ARCamera. In the Game Object Menu select: Vuforia> AR Camera to create a new AR Camera object in the scene. Remove the default “Main Camera” from the scene.
5. In the Game Object Menu select: Vuforia> Ground Plane> Ground Plane Stage.

Now this is where the process diverts from the usual set up, by adding a ground plane object.

#### 6. Add a Ground Plane Stage

On the Game object menu, add a Ground Stage Plane by going to Vuforia > Ground Plane > Ground Plane Stage.

The Ground Plane Stage GameObject serves as a parent GameObject like the ImageTarget from other tutorials. And so any AR content can be added as a child of the Ground Plane Stage GameObject.

<div class="img_row">
	<img style="max-height: 100%;max-width: 100%"  src="{{ site.baseurl }}/img/Blogs/GroundPlane/Plane.PNG" alt="Ground Plane Stage" title="Ground Plane Stage"/>
</div>

You can see that the Ground Plane stage has a 100cm grid, these are only rendered in the Editor.

#### 7. Add your AR content

Akin to the Target image, you can now add your AR models as children of your Ground Plane Scene.

#### 8. Add a Ground Plane Finder

In the Game Object Menu and select: Vuforia> Ground Plane> Plane Finder. The Plane Finder does the following:

    a. Listens to input from the user (such as a tap on the device screen) with the Anchor Input Listener Behaviour
    b. Attempts to find an appropriate plane to place content in the real world with the Plane Finder Behaviour
    c. Places the content into the real world with the Content Positioning Behaviour

Drag your “Ground Plane Stage” object into the “Anchor Stage” parameter of the “Content Positioning Behaviour” component

<div class="img_row">
	<img style="max-height: 100%;max-width: 100%"  src="{{ site.baseurl }}/img/Blogs/GroundPlane/Finder.PNG" alt="Ground Plane Finder" title="Ground Plane Finder"/>
</div>

Now we can deploy our App and test the functionality. when you looka round the environment, there is a finder that is projected onto planes. Once you find a surface you like, tap on the screen. You should notice that the AR Content is placed on the plane where the reticle is pointing to.

</div>