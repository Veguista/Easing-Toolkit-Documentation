# Easing Toolkit - Advanced Workflow

This document is an exploration of more advance workflows inside the Easing Toolkit package. It assumes that the user has already read the relevant sections in the previous document [Basic Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/5_Basic%20Workflow.md).

All Advanced Workflows revolve around using **SecondOrderDynamics**:

##

## Index

1. [The *Stored Transform Data* Input Method](#storedTransformData)
2. [Custom SecondOrder_Scripts](#customSecondOrderScripts)

##

-------------------------------------

### Dynamics Configuration - The _Stored Transform Data_ Input Method {#storedTransformData}
The differences between Input Methods, as well as how to select one, are explained in the [Basic Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/5_Basic%20Workflow.md) document.

While using the "_Follow Transform_" input mode, users can access and alter the Transform that is inputting information through the public "_followTransform_" field.

> soTransform._followTransform _= < otherTransform >;

While using the "_Stored Transform Data_" input mode, users need to make use of the "_TransformData_" struct.
The "_TransformData_" struct contains 3 data fields (Vector3 "_position_", Quaternion "_rotation_", and Vector3 "_scale_"), replicating the structure of a Transform class.

To store information in a "_TransformData_" struct, create a new struct and alter a field (only those fields that will be used need to be altered):

> TransformData myTransformData = new TransformData();
> myTransformData.position = new Vector3(1, 0, -1);
> myTransformData.rotation = Quaternion.identity;

To initialize a Dynamic in "_Stored Transform Data_" input mode, call the InitializeDynamicsThroughTransformData() method and pass the TransformData struct with your desired starting Transform information.

> soTransform.InitializeDynamicsThroughTransformData(myTransformData);

Updates to the Transform the SecondOrderTransform component is attached to do not occur autonomously while using the "_Stored Transform Data_" input mode. Instead, they occur whenever the system receives a new input of TransformData. To input TransformData to the component and refresh the dynamics, use the InputTransformDataAndUpdateDynamics() method:

> soTransform.InputTransformDataAndUpdateDynamics(myTransformData);

**_WARNINGS:_ **
- When using the "_Stored Transform Data_" input method, updating the component's dynamics before initializing them will result in an error. The error will indicate that the component is trying to set certain Transform values to NaN.
- The "_Stored Transform Data_" input method requires users to initialize the Dynamics of the component every time the Dynamic type changes.

##

### Custom Second Order Scripts {#customSecondOrderScripts}
Experienced users might realize the potential of incorporating Second Order Dynamics into non Transform-related workflows. However, they will also realize the limitations of using the **SecondOrderTransform** script for such purposes:

*Ex. A user wants to apply Second Order Dynamics to a float value and obtain its eased value. If they were following the [Basic Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/5_Basic%20Workflow.md), they would need to apply the value to an empty GameObject to then extract it and store it.*

A much better alternative exists, which is to use the core Second Order structs:

- SecondOrder_1D => Applies Second Order Dynamics to **float** values.
- SecondOrder_2D => Applies Second Order Dynamics to **Vector2**.
- SecondOrder_3D => Applies Second Order Dynamics to **Vector3**.
- SecondOrder_Rotation => Applies Second Order Dynamics to **Quaternions**.

##

Second Order structs are only accessible through code when using the **EasingToolkit.SecondOrderDynamics** namespace. To access it, users can add the following line of code at the beginning of their script:
> **using EasingToolkit.SecondOrderDynamics;**

##

To apply a Second Order Dynamic to a supported value, first create a Second Order Struct using the following constructor, replacing only the struct name:

SecondOrder_Type(float _frequency, float _dampening, float _initialResponse, Type _initialValue);

For example, this is how we could declare a SecondOrder_3D struct:
> // The Vector3 that the Second Order Dynamic will use as its starting value.
> Vector3 startingPosition = Vector3.zero;

> float frequency = 1f;
> float dampening = 1f;
> float intialResponse = 1f;

> SecondOrder_3D mySecondOrder3D = SecondOrder_3D(frequency, dampening, intialResponse, startingPosition);

##

Every time that we wish to update the 

##

-------------------------------------

**Easing Toolkit** is a Unity Package developed by Pablo Granado Valdivielso ("Veguista").
