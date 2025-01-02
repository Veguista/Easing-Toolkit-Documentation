# Easing Toolkit - Advanced Workflow

This document is an exploration of more advance workflows inside the Easing Toolkit package. It assumes that the user has already read the relevant sections in the previous document [Basic Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/5_Basic%20Workflow.md).

All Advanced Workflows revolve around using **SecondOrderDynamics**:

<br>

## Index

1. [The *Stored Transform Data* Input Method](#stored-transform-data) <br> <br>
2. [Custom SecondOrder_Scripts](#custom-second-order-script) <br>
	2.1 [Namespace](#custom-SO-namespace) <br>
    	2.2 [Constructor](#custom-SO-constructor) <br>
	2.3 [Functions - Update](#custom-SO-update) <br>
    	2.4 [Functions - Change Constants](#custom-SO-constants) <br>
    	2.5 [Functions - Reset](#custom-SO-reset)

<br>

-------------------------------------

### Dynamics Configuration - The _Stored Transform Data_ Input Method <a name="stored-transform-data"></a>
The differences between Input Methods, as well as how to select one, are explained in the [Basic Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/5_Basic%20Workflow.md) document.

<br>

While using the "_Follow Transform_" input mode, users can access and alter the Transform that is inputting information through the public "_followTransform_" field.

> soTransform._followTransform _= < otherTransform >;

<br>

While using the "_Stored Transform Data_" input mode, users need to make use of the "_TransformData_" struct.
The "_TransformData_" struct contains 3 data fields (Vector3 "_position_", Quaternion "_rotation_", and Vector3 "_scale_"), replicating the structure of a Transform class.

<br>

To store information in a "_TransformData_" struct, create a new struct and alter a field (only those fields that will be used need to be altered):

> TransformData myTransformData = new TransformData(); <br>
> myTransformData.position = new Vector3(1, 0, -1); <br>
> myTransformData.rotation = Quaternion.identity;

<br>

To initialize a Dynamic in "_Stored Transform Data_" input mode, call the InitializeDynamicsThroughTransformData() method and pass the TransformData struct with your desired starting Transform information.

> soTransform.InitializeDynamicsThroughTransformData(myTransformData);

<br>

Updates to the Transform the SecondOrderTransform component is attached to do not occur autonomously while using the "_Stored Transform Data_" input mode. Instead, they occur whenever the system receives a new input of TransformData. To input TransformData to the component and refresh the dynamics, use the InputTransformDataAndUpdateDynamics() method:

> soTransform.InputTransformDataAndUpdateDynamics(myTransformData);

<br>

#### _WARNINGS:_ <br>
- When using the "_Stored Transform Data_" input method, updating the component's dynamics before initializing them will result in an error. The error will indicate that the component is trying to set certain Transform values to NaN.
- The "_Stored Transform Data_" input method requires users to initialize the Dynamics of the component every time the Dynamic type changes.

<br>

### Custom Second Order Scripts <a name="custom-second-order-script"></a> <br>
Experienced users might realize the potential of incorporating Second Order Dynamics into non Transform-related workflows. However, they will also realize the limitations of using the **SecondOrderTransform** script for such purposes:

<br>

*Ex. A user wants to apply Second Order Dynamics to a float value and obtain its eased value. If they were following the [Basic Workflow](https://github.com/Veguista/Easing-Toolkit-Documentation/blob/main/5_Basic%20Workflow.md), they would need to apply the value to an empty GameObject's Transform to then extract it and store it.*

<br>

A much better alternative exists, which is to use the core Second Order structs:

- SecondOrder_1D => Applies Second Order Dynamics to **floats**.
- SecondOrder_2D => Applies Second Order Dynamics to **Vector2**s.
- SecondOrder_3D => Applies Second Order Dynamics to **Vector3**s.
- SecondOrder_Rotation => Applies Second Order Dynamics to **Quaternions**. <br>

<a name="custom-SO-namespace"></a> <br>
### Namespace 

Second Order structs are only accessible through code when using the **EasingToolkit.SecondOrderDynamics** namespace. To access it, users can add the following line of code at the beginning of their script:
> **using EasingToolkit.SecondOrderDynamics;**
 <br>

<a name="custom-SO-constructor"></a>  <br>
### Contructor 

To apply a Second Order Dynamic to a supported value, first create a Second Order Struct using the following constructor, replacing the Struct Type with one of the supported types (*float, Vector2, Vector3, Quaternion*):

> SecondOrder_**Type** (float _frequency, float _dampening, float _initialResponse, **Type** _initialValue);

<br>

For example, this is how we could declare a Vector3 struct:
> // The Vector3 that the Second Order Dynamic will use as its starting value. <br>
> Vector3 startingPosition = Vector3.zero;

> float frequency = 1f; <br>
> float dampening = 1f; <br>
> float intialResponse = 1f; <br>

> SecondOrder_3D mySecondOrder3D = SecondOrder_3D(frequency, dampening, intialResponse, startingPosition); <br>


<a name="custom-SO-update"></a> <br>
### Functions - Update <br>

Second order scripts act as post-processors to data. Users update their value by calling the Update() function, and it returns the eased value at that point in time.


#### DECLARATION <br>
public _Type_ Update(float deltaTime, _Type_ targetValue); <br>

#### DECLARATION *(Not avalible for SecondOrder_Rotation)* <br>
public _Type_ Update(float deltaTime, _Type_ targetValue, _Type_ inputVelocity);

<br>

| Parameter | Description |
| --- | ----------- |
| deltaTime | The ammount of elapsed time since the last Update() call.<br> Cannot be negative or 0. |
| targetValue | The value of the uneased data the SecondOrder struct is tracking at the time of calling. |
| inputVelocity | (Optional) The derivate of the value with respect to time. Providing it can improve the accuracy of the SecondOrder system.<br>In those cases where it is not provided, the second order system will approximate it with the data it already has.|

<br>

#### RETURNS <br>
The resulting eased value of the Type of the Second Order system. 
> Ex. SecondOrder_3D.Update() will return an eased Vector3.

<br>

#### EXAMPLE <br>

> using UnityEngine; <br>
> <br>
> public class SecondOrder_Vector3Easing : MonoBehaviour <br>
> { <br>
> &nbsp;&nbsp;[SerializedField] float frequency = 1.0f; <br>
> &nbsp;&nbsp;[SerializedField] float dampening = 1.0f; <br>
> &nbsp;&nbsp;[SerializedField] float initialResponse = 0.f; <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;SecondOrder3D mySecondOrder3D; <br>
> &nbsp;&nbsp;Vector3 uneasedVector3; <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;private void Start() <br>
> &nbsp;&nbsp;{ <br>
> &nbsp;&nbsp;&nbsp;&nbsp;//Initializing our Vector3. <br>
> &nbsp;&nbsp;&nbsp;&nbsp;uneasedVector3 = Vector3.zero; <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;&nbsp;&nbsp;// Initializing our Second Order Script with the starting value of uneasedVector3. <br>
> &nbsp;&nbsp;&nbsp;&nbsp;mySecondOrder3D = new SecondOrder_3D(frequency, dampening, initialResponse, uneasedVector3); <br>
> &nbsp;&nbsp;} <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;private void Update() <br>
> &nbsp;&nbsp;{ <br>
> &nbsp;&nbsp;&nbsp;&nbsp;// Updating the uneased Vector3 in uneasedVector3. <br>
> &nbsp;&nbsp;&nbsp;&nbsp;uneasedVector3 += Time.deltaTime * new Vector3(1, 0, 0); <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;&nbsp;&nbsp;// (Optional) Making sure that time scale hasn't been used to pause the game. <br>   
> &nbsp;&nbsp;&nbsp;&nbsp;if(Time.deltaTime == 0) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return; <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;&nbsp;&nbsp; Vector3 easedValue = mySecondOrder3D.Update(Time.deltaTime, uneasedVector3); <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;&nbsp;&nbsp;// Do smth with the eased value. <br>
> &nbsp;&nbsp;} <br>
> } <br>


<a name="custom-SO-constants"></a> <br>
### Functions - Change Constants

Allows users to change the parameters (frequency, dampening, and initial response) of a Second Order system after it has been initialized. <br>

#### DECLARATION <br>
public void ChangeConstants(float frequency, float dampening, float initialResponse);

#### DECLARATION *(Intended for internal use only)* <br>
public void ChangeConstants(SO_Constants constants);

<br>

| Parameter | Description |
| --- | ----------- |
| frequency | The new value for the system's **frequency parameter**.<br>It controls the speed at which the system develops.<br>Higher values will result in a shorter response time. <br>**Warning** => Must be bigger than 0. |
| dampening | The new value for the system's **dampening parameter**.<br>It controls the amount of dampening applied to the system.<br>Values of 1 or higher produce steady transitions to the target value. Values between 0 and 1 will result in a vibrating system, where the target value will be overshot and the system will vibrate around it. A value of 0 will result in no dampening at all, thus creating a system that will vibrate around a value forever (not recommended).<br>**Warning** => Must be equal to or bigger than 0. |
| initialResponse | The new value for the system's **initial response parameter**.<br>It controls the initial velocity of the system whenever a new input is placed.<br>When positive and under 1, the system will respond faster to changes. When positive and over 1, the system will have so much initial velocity that it will overshoot its target. When negative, the system will start with an opposite velocity to the target value.

<br>

#### EXAMPLE <br>

> using UnityEngine; <br>
> <br>
> public class SecondOrder_Vector3Easing : MonoBehaviour <br>
> { <br>
> &nbsp;&nbsp;SecondOrder3D mySecondOrder3D; <br>
> &nbsp;&nbsp;Vector3 uneasedVector3 = Vector3.zero; <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;float frequency = 3.0f; <br>
> &nbsp;&nbsp;float dampening = 1.0f; <br>
> &nbsp;&nbsp;float initialResponse = 0.f; <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;private void Start() <br>
> &nbsp;&nbsp;{ <br>
> &nbsp;&nbsp;&nbsp;&nbsp;// Initializing our Second Order Script with the starting value of uneasedVector3. <br>
> &nbsp;&nbsp;&nbsp;&nbsp;mySecondOrder3D = new SecondOrder_3D(frequency, dampening, initialResponse, uneasedVector3); <br>
> &nbsp;&nbsp;} <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;private void Update() <br>
> &nbsp;&nbsp;{ <br>
> &nbsp;&nbsp;&nbsp;&nbsp;// Updating the uneased Vector3 in uneasedVector3. <br>
> &nbsp;&nbsp;&nbsp;&nbsp;uneasedVector3 += Time.deltaTime * new Vector3(1, 0, 0); <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;// Making our frequency fluctuate with time. (Do not use this line in a real implementation, as Mathf.Sin has a valid range for its parameters.) <br>
> &nbsp;&nbsp;frequency = Mathf.Sin(Time.time) + 2; <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;// Updating the parameters in our Second Order Script. <br>
> &nbsp;&nbsp;mySecondOrder3D.ChangeConstants(frequency, dampening, initialResponse); <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;&nbsp;&nbsp; Vector3 easedValue = mySecondOrder3D.Update(Time.deltaTime, uneasedVector3); <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;&nbsp;&nbsp;// Do smth with the eased value. <br>
> &nbsp;&nbsp;} <br>
> } <br>


<a name="custom-SO-reset"></a> <br>
### Functions - Reset <br>

Resets the internal speed of the second order system to 0 and sets its value to the last target value passed through the Update() function.
Effectively, it leaves the Second Order script in the same state as if it had just been created with the last target value passed through the Update() function.
 <br>

#### DECLARATION <br>
public void Reset();

<br>

| Parameter | Description |
| --- | ----------- |
| None | - |

#### EXAMPLE <br>

> using UnityEngine; <br>
> <br>
> public class SecondOrder_Vector3Easing : MonoBehaviour <br>
> { <br>
> &nbsp;&nbsp;SecondOrder3D mySecondOrder3D; <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;// Code initializing and using mySecondOrder3D. <br>
> &nbsp;&nbsp; <br>
> &nbsp;&nbsp;OnDisable() <br>
> &nbsp;&nbsp;{ <br>
> &nbsp;&nbsp;&nbsp;&nbsp;mySecondOrder3D.Reset(); <br>
> &nbsp;&nbsp;} <br>
> }

<br>

-------------------------------------

**Easing Toolkit** is a Unity Package developed by Pablo Granado Valdivielso ("Veguista").
