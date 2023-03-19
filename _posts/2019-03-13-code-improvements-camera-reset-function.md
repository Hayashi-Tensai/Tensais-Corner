---
layout: post
title:  "Code Improvements - Camera Reset Function"
author: Hayashi Tensai
excerpt: Hi there! Today I will be featuring part of the script that I will be using for an ongoing project (still in early stages). It is an improved version of a function that I used back in one of my previous projects, Mirage Forest.
permalink: blog/2019/03/13/code-improvements-camera-reset-function
date:   2019-03-13 13:51:00 +0800
updated: 2020-09-03 14:15:00 +0800
categories: game-development
---

Hi there!

Today I will be featuring part of the script that I will be using for an ongoing project (still in early stages). It is an improved version of a function that I used back in one of my previous projects, Mirage Forest. Here I will be sharing the difficulties and problems I faced in the older version and what I have changed.

Firstly let's look into the hierarchy of the player, the script is located on the **Player GameObject** (I'll call this as **"parent"**) of the highest hierarchy and has the **camera** and the **character model** as the children.

<img style="max-width:80%;" src="/assets/blog-images/game-dev/camera-reset-1.png" />

Here are two different clips featuring the effects of the old scripts and the new scripts.

**Scenario 1: Both models are set to face left and then reset camera is pressed.**
![Camera Reset Left Old](/assets/blog-images/game-dev/camera-reset-left-old.gif)  
Old : camera turns counter-clockwise, taking the long way around.

![Camera Reset Left New](/assets/blog-images/game-dev/camera-reset-left-new.gif)  
New : camera turns clockwise, taking the shortest path

**Scenario 2: Both models are set to face right and then reset camera is pressed.**
![Camera Reset Right Old](/assets/blog-images/game-dev/camera-reset-right-old.gif)

![Camera Reset Right New](/assets/blog-images/game-dev/camera-reset-right-new.gif)  
Both turns clockwise in this scenario

Following on, the scripts. Let's look at the old script first.

**Old Script**
```csharp
void LerpAngle ()
{
    transform.eulerAngles = new Vector3(transform.eulerAngles.x, transform.eulerAngles.y + (lerpDamping * Time.deltaTime), transform.eulerAngles.z);
    model.transform.localEulerAngles = new Vector3(model.transform.localEulerAngles.x, model.transform.localEulerAngles.y - (lerpDamping * Time.deltaTime), model.transform.localEulerAngles.z);

    if(model.transform.localRotation.y <= 0.0f)
        isLerping = false;
}
```
The old script is relatively simple, **the parent** will turn clockwise on an a certain interval wihile, the **character model** will turn counter-clockwise countering the **parent's** movement, and it will continue until the **character model's** local rotation goes less than or equal to zero.

Problems with this code:  
- It only turns clockwise, potentially taking a longer path to rotate as seen the left clip above.  
- The angle after it stops is not at zero (It will overshoot by around 8 - 11 degrees).

**New Script**
```csharp
void LerpAngle()
{
    Vector3 modelAngle = model.transform.localEulerAngles; //! Euler angles of the character model
    Vector3 parentAngle = transform.eulerAngles; //! Euler angles of the parent object
    float lerpRate = lerpDamping * Time.deltaTime;


    if (modelAngle.y <= 12.0f)
    {
        transform.eulerAngles = new Vector3(parentAngle.x, parentAngle.y + modelAngle.y, parentAngle.z);
        model.transform.localEulerAngles = new Vector3(modelAngle.x, 0.0f, modelAngle.z);
        isLerping = false;
    }
    else if(modelAngle.y >= 348.0f)
    {
        transform.eulerAngles = new Vector3(parentAngle.x, parentAngle.y - (360.0f - modelAngle.y), parentAngle.z);
        model.transform.localEulerAngles = new Vector3(modelAngle.x, 0.0f, modelAngle.z);
        isLerping = false;
    }
    else if(modelAngle.y > 12.0f && modelAngle.y <= 180.0f)
    {
        transform.eulerAngles = new Vector3(parentAngle.x, parentAngle.y + lerpRate, parentAngle.z);
        model.transform.localEulerAngles = new Vector3(modelAngle.x, modelAngle.y - lerpRate, modelAngle.z);
    }
    else if(modelAngle.y > 180.0f && modelAngle.y < 348.0f)
    {
        transform.eulerAngles = new Vector3(parentAngle.x, parentAngle.y - lerpRate, parentAngle.z);
        model.transform.localEulerAngles = new Vector3(modelAngle.x, modelAngle.y + lerpRate, modelAngle.z);
    }

}
```
As for the new script, I made the variables modelAngle, parentAngle and lerpRate, to reduce the amount of long repetitions of long lines of codes. I also made a change to the if conditions to check for localEularAnlgles rather than the local rotation, as it makes more sense that way since the euler angles is the one being modified. It also makes it a lot easier to use because euler angles only stays within the range of 0 - 360 degrees.

Then, as for the issues above:  
- The code has now several conditons to check for the current euler angles of the **character model** to consider whether to turn clockwise or anticlockwise to take the shortest path.  
- The top two conditions on the other hand handles the problem where the angle does not end up at zero, which when the lerping processing reaches less than a 12 degree difference from zero from either direction, it clips right away to zero.

That's for for now, hope to write more about the project soon!