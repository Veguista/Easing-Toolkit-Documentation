# Easing Toolkit - Advanced Workflow

This document is an exploration of more advance workflows inside the Easing Toolkit package. It assumes that the user has already read the relevant sections in the previous document [Basic Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/5_Basic%20Workflow.md).

All Advanced Workflows revolve around using **SecondOrderDynamics**:

##

## Index

**Configuring the Dynamics**
1. [The _Stored Transform Data_ Input Method](#storedTransformData)


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

-------------------------------------

**Easing Toolkit** is a Unity Package developed by Pablo Granado Valdivielso ("Veguista").
