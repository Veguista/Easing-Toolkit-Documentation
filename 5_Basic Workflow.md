# Easing Toolkit - Basic Workflow

This document contains information conveying the basic workflow of the systems inside the Easing Toolkit package. For a more in-depth exploration, visit [Advanced Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/6_Advanced%20Workflow.md).

**Easing Toolkit** is comprised of 2 complimentary but separate parts:

##

## Index

1. [**SecondOrderDynamics**](#secondOrderDynamics)
 
    **Debug Options**
	1.1  [Running Dynamics In Editor](#runDynamicsInEditor) (_Debug Only_) 

    **Configuring the Dynamics**
    1.2  [Dynamic Type](#dynamicType)
    1.3  [Axis Configuration](#axisConfiguration)
    1.4  [Input Method](#inputMethod)
    1.5  [Data Origin](#dataOrigin)
    1.6  [Output Space](#outputSpace)
    1.6  [Refresh Mode](#refreshMode)

    **Control Parameters**
    1.7  [Second Order Parameters](#secondOrderParameters)

###

2. [**EasingTools.ApplyEase**](#applyEase)

##

-------------------------------------

## SecondOrderDynamics: {#secondOrderDynamics}

A collection of tools to ease movement, rotation, scaling, and other numerical variables procedurally.

Most users will mainly use the **SecondOrderTransform** component. When active, **SecondOrderTransform **components take control of one Transform element (position, rotation, or scale) of the GameObject they are attached to. Generally, that element's information is updated by following another Transform's position, rotation, or scale and applying an easing process. As a result, **SecondOrderTransform** allows users to create easing effects similar to those made by **EasingTools.ApplyEase()**, without the limitation of needing an endpoint to the interpolation.

To start using the **SecondOrderTransform** component, drag it into a GameObject's inspector from the Project tab or add it using the "Add Component" button. The following are the options that the component offers to users:


### Run Dynamics in Editor {#runDynamicsInEditor}
When enabled, the SecondOrderTransform script will work without entering Play Mode.

**_Access through code:_ **
This element should only be accessed through the inspector, as it is only meant for debug purposes.

**_WARNINGS:_ **
Use only as a Debug tool. SecondOrderTransform might display unintended behavior during Editor Mode:
-  Due to Unity's restrictions on script execution, the "_FixedUpdate_" refresh mode is incompatible with Editor Mode. Instead, SecondOrderTransform will default to "_Update_" refresh mode during Editor Mode.
-  Execution during Editor Mode is incompatible with the "Stored _Transform Data_" input method.

##

### Dynamics Configuration - Dynamic Type {#dynamicType}
Determines whether this component should apply to the Position, Rotation, or Scale of the attached GameObject.

**_Access through code:_ **
The Dynamic Type can be accessed and changed through the public "_WhichDynamicType_" field.

> SecondOrderTransform soTransform = GetComponent<SecondOrderTransform>();
> soTransform.WhichDynamicType = SecondOrderTransform.DynamicsType.position;

**_WARNINGS:_ **
- Changing the dynamic type forces a reset of the dynamics. The SecondOrderTransform component only tracks one dynamic at a time, and as such transitions between dynamics will not be smooth. Instead, to create such effects, it is recommended to use multiple SecondOrderTransform components simultaneously while altering their parameters.

##

### Dynamics Configuration - Axes configuration (Apply X / Apply Y / Apply Z) {#axisConfiguration}
Determines whether the SecondOrderTransform component applies its output to certain axes. It works only while using the "_Position_" and "_Scale_" dynamic types.

**_Access through code:_ **
The axes configuration can be accessed and changed through the public "axisToFollow" bool array.
- axisToFollow[0] => Apply to X Axis.
- axisToFollow[1] => Apply to Y Axis.
- axisToFollow[2] => Apply to Z Axis.


> SecondOrderTransform soTransform = GetComponent<SecondOrderTransform>();
> soTransform.axisToFollow[1] = false;	// Disabling the application of the SecondOrderTransform component to the Y Axis. 

**_WARNINGS:_ **
- The "_Rotation_" dynamic type ignores this configuration.
- There is a known bug, where altering the Axes configuration through code will not be visually reflected in the Inspector. However, the changes are still taking effect.
- Turning on the configuration of an Axis during runtime will result in a sudden change, as that is not the axes' intended use case. Instead, it is recommended to use multiple SecondOrderTransform components, each with different Axes Configurations, and manipulate their parameters.

##

### Dynamics Configuration - Input Method {#inputMethod}
Determines which method the SecondOrderTransform uses to obtain the input Transform information. There are possible 2 options:
- _Follow Transform_: The Transform information is obtained from another GameObject. The inspector shows the field "Follow Transform" that allows users to select which Transform to obtain the information from.
- _Stored Transform Data_: The Transform data is inputted by the user through code. The user needs to initialize and update the selected Dynamic manually.


**_Access through code:_ **
To change the Input method, users can access and alter the public "inputMode" field.

> SecondOrderTransform soTransform = GetComponent<SecondOrderTransform>();
> soTransform.inputMode = SecondOrderTransform.TypeOfDataInput.storedTransformData;

While using the "_Follow Transform_" input mode, users can access and alter the Transform that is inputting information through the public "_followTransform_" field.

> soTransform._followTransform _= < otherTransform >;

Further information on how to use the _Stored Transform Data_ input method can be found in the [Advanced Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/6_Advanced%20Workflow.md) document.

**_WARNINGS:_ **
- When using the "_Stored Transform Data_" input method, updating the component's dynamics before initializing them will result in an error. The error will indicate that the component is trying to set certain Transform values to NaN.
- The "_Stored Transform Data_" input method requires users to initialize the Dynamics of the component every time the Dynamic type changes.

##

### Dynamics Configuration - Data Origin {#dataOrigin}
Determines whether the Transform information is fetched locally or globally while using the "_Follow Transform_" input mode (Ex. Whether we are getting a Transform's "localPosition" or "worldPosition" to feed into the SecondOrderTransform component).

There are 2 possible options:
- _Local Space._
- _World Space._


_**Access through code:**_
The data origin can be accessed and changed through the public "obtainTransformDataFromLocalOrWorld" field.

> SecondOrderTransform soTransform = GetComponent<SecondOrderTransform>();
> soTransform.obtainTransformDataFromLocalOrWorld = TypeOfSpace.worldSpace;	// The TypeOfSpace enum is used here and at the Output Location section.

**_WARNINGS:_ **
- This configuration is ignored while using the "_Stored Transform Data_" input mode (because the system does not need to fetch its Transform information).

##

### Dynamics Configuration - Output Space {#outputSpace}
Determines whether the results from the Second Order Dynamics are applied to the Transform locally or globally (Ex. We obtain an eased Quaternion as a result of updating our Second Order Dynamics. Do we set the Transform's "localRotation" or its "worldRotation"?).

There are 2 possible options:
- _Local Space._
- _World Space._


**_Access through code:_ **
The output space can be accessed and changed through the public "applyDynamicsToLocalOrWorld" field.

> SecondOrderTransform soTransform = GetComponent<SecondOrderTransform>();
> soTransform.applyDynamicsToLocalOrWorld= TypeOfSpace.localSpace;	// The TypeOfSpace enum is used here and in the Data Origin section.

##

### Dynamics Configuration - Refresh Mode {#refreshMode}
Determines when the dynamics of the Second Order Transform component update while using the "_Follow Transform_" input mode. There are 3 possible options:
- _Update_: Every frame before all physics calculations have been performed.
- _Fixed Update_: Every 0.02 secs.
- _Late Update_: Every frame after all physics calculations have been performed.


_**Access through code:**_
The refresh mode can be accessed and changed through the public "refreshMode" field.

> SecondOrderTransform soTransform = GetComponent<SecondOrderTransform>();
> soTransform.refreshMode = TypeOfDynamicsRefresh.fixedUpdate;

**_WARNINGS:_ **
- This configuration is ignored while using the "_Stored Transform Data_" input mode (because the system updates at the user's command).
- Be careful with updating physics-sensitive Transforms using the "Update" mode, as it will update its dynamics before any internal Unity-physics calculations.

##

### Second Order Parameters {#secondOrderParameters}
Three main parameters control a Second Order Transform component:
- Frequency: It controls the speed at which the system develops. Higher values will result in a shorter response time. [Warning => Must be bigger than 0]
- Dampening: It controls the amount of dampening applied to the system. Values of 1 or higher produce steady transitions to the target value. Values between 0 and 1 will result in a vibrating system, where the target value will be overshot and the system will vibrate around it. A value of 0 will result in no dampening at all, thus creating a system that will vibrate around a value forever (not recommended).  [Warning => Must be equal to or bigger than 0]
- Initial Response: The initial velocity of the system whenever a new input is placed. When positive and under 1, the system will respond faster to changes. When positive and over 1, the system will have so much initial velocity that it will overshoot its target. When negative, the system will start with an opposite velocity to the target value.


**_Access through code:_ **
The three parameters can be accessed and changed through the following public pointers:
- Frequency => "_Frequency_" public float.
- Dampening => "_Dampening_" public float.
- Initial Response => "InitialResponse" public float.


> SecondOrderTransform soTransform = GetComponent<SecondOrderTransform>();
> soTransform.InitialResponse = 0;

**_WARNINGS:_ **
- Trying to set a parameter's value to one outside its range will not be allowed by the system. Instead, a Warning message will be produced.

##

## EasingTools.ApplyEase(): {#applyEase}

A function that applies an easing style from a broad collection to a value from 0 to 1.

It is found inside the **EasingTools** class. To gain access to the **EasingTools **class, users can either:
1. (Recommended) Specify that they are using the **EasingToolkit namespace**. To do so, add the following line of code at the beginning of your script "**using EasingToolkit;**".
2. Write the namespace before every instance of the **EasingTools **class (Ex. "**EasingToolkit.EasingTools.ApplyEase(inputFloat, EasingToolkit.EaseType.typeOfEase)**").


The **ApplyEase()** method takes 2 arguments:
1. A float (**inputFloat**), in the [0,1] Range (inclusive). This is the value that will be eased.
2. An EaseType enum (**typeOfEase**), indicating which easing process to be performed.


All possible Easing Functions and their visualizations can be found at [easings.net](https://easings.net/).

A usual use case for the ApplyEase() method is to process an interpolation value before it is fed into a Lerp().
For example, the following code would calculate an eased translation of [type EaseInOutQuint](https://easings.net/#easeInOutQuint) between positions A and B.

> Vector3 position_a = new Vector3(0, 0, 0), position_b = new Vector3(1, 0, 3);
> float interpolationValue = < timePastInMovement / totalMovementDuration >;
> float eased_interpolationValue = EasingTools.ApplyEase(interpolationValue, EasingTools.EaseType.EaseInOutQuint);
> Vector3 easedMovement = Vector3.Lerp(position_a, position_b, eased_interpolationValue);

##

-------------------------------------

**Easing Toolkit** is a Unity Package developed by Pablo Granado Valdivielso ("Veguista").
