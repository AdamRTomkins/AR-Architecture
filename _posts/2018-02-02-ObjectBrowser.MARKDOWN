---
layout: post
title: "Project: Sheffield AR Building Browser"
img: ObjectBrowser/WestStreet.jpg
date: 2018-01-29 15:20:00
description: An example project using the tracked markers for a Sheffield building browser.
tag: [Augmented Reality, Architecture, Project, Shedfield, SketchUp, C#]
---

In this example, we are going to build a simple object browser, building off the Vuforia Turorial and the SketchUp tutorial, and introduce two new concepts:

1. Scripting in C#
2. The UI System.

The end goal of this tutorial is an Android App which:
1. Recognises a Map-Based Marker
2. Displays Sheffield Building Models
3. User interaction can cycle through buildings

This should look something along the lines of:

<div class="img_row">
	<img style="max-height: 100%;max-width: 100%"  src="{{ site.baseurl }}/img/Blogs/ObjectBrowser/WestStreet.jpg" alt="West Street in Sheffield ExplorAR" title="West Street in Sheffield ExplorAR"/>
</div>

#### 1. Prepare your models.

Using the steps from the [SketchUp tutorial](https://adamrtomkins.github.io/AR-Architecture/SketchUp/), prepare a selection of fbx models and import them as assets into unity. In our case, we're using:

1. [St Pauls Tower](https://3dwarehouse.sketchup.com/model/4bfd1f05c40c64246657bcb628d5eb5/St-Pauls-Tower-Sheffield-UK)
2. [The Cathedral](https://3dwarehouse.sketchup.com/model/834d263c39313e4d2cd20d41da6192f3/Cathedral-Sheffield-UK)
3. [The Cavendish](https://3dwarehouse.sketchup.com/model/6a8d33f318df7d744522e2cc67bda51/The-Cavendish-Building-West-Street)
4. [The Arts Tower](https://3dwarehouse.sketchup.com/model/5bb83ee87189242bf462e854274aefd8/Arts-Tower-Sheffield-UK)

#### 2. Create your scene

Using the [Vuforia](https://adamrtomkins.github.io/AR-Architecture/Unity_Vuforia/) Tutorial, and the [Custom Marker](https://adamrtomkins.github.io/AR-Architecture/Custom_Markers/) Tutorial. 

Here I am going to go with my favourite Sheffield Map tracker from [Stamen Maps](http://maps.stamen.com/).

<div class="img_row">
	<img style="max-height: 100%;max-width: 100%"  src="{{ site.baseurl }}/img/Blogs/ObjectBrowser/Sheffield_Map.png" alt="Sheffield Marker" title="Sheffield Marker"/>
</div>

From here, you will need to import each of the FBX models you have prepared, and attach them as children of your Target Image. This will look quite hectic, and you can disable them in the object inspector for clarity.

We are attaching them as children, so that they appear and disapear along with the trarget image detection, but we will need to develop some extra logic in order to only display a single one at a time.

#### 3. The Custom Tracker Script

We are going to create our own custom tracker called BrowserTracker, and replace the Default Trackable Event Handler. First, start creating a new C# Script and copying across the contents of the vuforia DefaultTrackableEventHandler.

Then we have to create a 'Selection' logic to ensure that only one object is displayed at a time.

First we will need to set up the variables required for this logic:
1. tracked (store whether objects are currently displayed)
2. c (a counter variable)
3. Selection (the index of the currently selected object)

~~~~
using UnityEngine;
using UnityEngine.UI;
using Vuforia;
using System.Collections;

/// <summary>
///     A custom handler that implements the ITrackableEventHandler interface.
/// </summary>
public class BrowserTracker : MonoBehaviour, ITrackableEventHandler
{

    #region PRIVATE_MEMBER_VARIABLES

    protected TrackableBehaviour mTrackableBehaviour;
  
    #endregion // PRIVATE_MEMBER_VARIABLES

    #region UNTIY_MONOBEHAVIOUR_METHODS

    protected virtual void Start()
    {
        mTrackableBehaviour = GetComponent<TrackableBehaviour>();
        if (mTrackableBehaviour)
            mTrackableBehaviour.RegisterTrackableEventHandler(this);
        tracked = false;
    }

    #endregion// UNTIY_MONOBEHAVIOUR_METHODS

    public int Selection;


    private int c;
    private bool tracked;

    

    #region PUBLIC_METHODS

    /// <summary>
    ///     Implementation of the ITrackableEventHandler function called when the
    ///     tracking state changes.
    /// </summary>
    public void OnTrackableStateChanged(
        TrackableBehaviour.Status previousStatus,
        TrackableBehaviour.Status newStatus)
    {
        if (newStatus == TrackableBehaviour.Status.DETECTED ||
            newStatus == TrackableBehaviour.Status.TRACKED ||
            newStatus == TrackableBehaviour.Status.EXTENDED_TRACKED)
        {
            Debug.Log("Trackable " + mTrackableBehaviour.TrackableName + " found");
            tracked = true;
            OnTrackingFound();
        }
        else if (previousStatus == TrackableBehaviour.Status.TRACKED &&
        newStatus == TrackableBehaviour.Status.NOT_FOUND)
        {
            Debug.Log("Trackable " + mTrackableBehaviour.TrackableName + " lost");
            tracked = false;
            OnTrackingLost();
        }
        else
        {
            // For combo of previousStatus=UNKNOWN + newStatus=UNKNOWN|NOT_FOUND
            // Vuforia is starting, but tracking has not been lost or found yet
            // Call OnTrackingLost() to hide the augmentations
            tracked = false;
            OnTrackingLost();
        }
    }


    #endregion // PUBLIC_METHODS

    #region PRIVATE_METHODS



    protected virtual void OnTrackingLost()
    {
        var rendererComponents = GetComponentsInChildren<Renderer>(true);
        var colliderComponents = GetComponentsInChildren<Collider>(true);
        var canvasComponents = GetComponentsInChildren<Canvas>(true);

        // Disable rendering:
        foreach (var component in rendererComponents)
            component.enabled = false;

        // Disable colliders:
        foreach (var component in colliderComponents)
            component.enabled = false;

        // Disable canvas':
        foreach (var component in canvasComponents)
            component.enabled = false;
    }

    #endregion // PRIVATE_METHODS
}

~~~~

Next we need to create the logic for when a tracker has been tracker:

~~~~
    protected virtual void OnTrackingFound()
    {
        var rendererComponents = GetComponentsInChildren<MeshRenderer>(true);
        var colliderComponents = GetComponentsInChildren<Collider>(true);
        var canvasComponents = GetComponentsInChildren<Canvas>(true);

        // Enable rendering:

        c = 0;
        foreach (var component in rendererComponents)

        {
            Debug.Log(component.name);
            if (c != Selection)
            {
                component.enabled = false;
            }
            else
            {
                component.enabled = true;
            }
            c++;
        }
        c = 0;



        // Enable colliders:
        foreach (var component in colliderComponents)
            component.enabled = true;

        // Enable canvas':
        foreach (var component in canvasComponents)
            component.enabled = true;
    }

~~~~

Finally we need to create the incrementation logic, that we can hook into with the UI

~~~~

    public void TestIndex()
    {
        if (Selection >= GetComponentsInChildren<MeshRenderer>(true).GetLength(0))
        {
            Selection = 0;
        }

        if (Selection < 0)
        {
            Selection = GetComponentsInChildren<MeshRenderer>(true).GetLength(0) - 1;
        }
    }

    public void Decrement()
    {
        // First, hide every object
        Selection--;
        TestIndex();
        if (tracked)
        {
            OnTrackingFound();
        }
    }

    public void Increment()
    {
        // First, hide every object
        Selection++;
        TestIndex();
        if (tracked)
        {
            OnTrackingFound();
        }
    }
~~~~

#### 4. Create the User Interface

For this we can create a simple button in a canvas object, and see the on-click event to trigger the 'Increment' function above.



<div class="img_row">
	<img style="max-height: 100%;max-width: 100%"  src="{{ site.baseurl }}/img/Blogs/ObjectBrowser/UI.PNG" alt="User Interface" title="User Interface"/>
</div>


And that should be all that it takes, to get this simple browser up and running.

<div class="img_row">
	<img style="max-height: 100%;max-width: 100%"  src="{{ site.baseurl }}/img/Blogs/ObjectBrowser/Button.PNG" alt="Button Trigger" title="Button Trigger"/>
</div>

Now, you should see something like this, when you target your Tracker

<div class="img_row">
	<img style="max-height: 100%;max-width: 100%"  src="{{ site.baseurl }}/img/Blogs/ObjectBrowser/ArtsTower.jpg" alt="Arts Tower in Sheffield ExplorAR" title="Arts Tower in Sheffield ExplorAR"/>
</div>