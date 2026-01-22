# Scene Update Protocol

SUP/1.0, Draft 1 Revision 3
March 2025, Igni Project



## Table of Contents

1. Introduction
2. Definitions and Abbreviations
3. Data Formats

   1. String
   2. File Path
   3. URL

4. Requests

   1. Request Header
   2. Configure Connection
   3. Import Model
   4. Show Model
   5. Hide Model
   6. Transform Model
   7. Destroy Model
   8. Create Action Instance
   9. Set Action Frame
   10. Set Action Weight
   11. Destroy Action Instance
   12. Transform Viewpoint
   13. Set Viewpoint Field of View

5. Response

   1. Invalid Protocol Version
   2. Model Not Found
   3. Action Not Found
   4. Model Import Failed
   5. Identifier Already Taken
   6. Invalid Request Code

6. Authors Notes

## 1\. Introduction

The Scene Update Protocol (SUP) is an application layer communication protocol capable of transferring real-time 3D graphics over a network. Its use of actions and model file loading grants it great extendability and versatility while remaining lightweight and simple in design.

## 2\. Definitions and Abbreviations

**3D**
3-dimensional

**field**
variable in a data structure

**float**
IEEE 754 floating-point integer

**ID**
unique identifier

**int**
signed integer

**uint**
unsigned integer

## 3\. Data Formats

### 3.1. String

The term ‘string’ in this document specifically refers to a null-terminated string in UTF8 (RFC [3629](https://datatracker.ietf.org/doc/html/rfc3629)) format.

### 3.2. File Path

The term ‘file path’ in this document specifically refers to a string that points to a file. Format of file paths may vary implementation to implementation.

### 3.3. URL

The term 'URL' is an acronym for Uniform Resource Locator. URLs shall be interpreted in accordance with the URL Standard by WHATWG ([link](https://url.spec.whatwg.org/)).

## 4\. Requests

The term 'request' in this specification is defined as a message sent from a client to a server. Requests contain instructions, which when read by a server, are carried out by the server on the clients behalf.

### 4.1. Request Header

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|8-bit int|Request code|

Every request has its contents preceded by a header. The header has 1 field: an 8-bit code that identifies the request placed after the header.

### 4.2. Configure Connection

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|0|Opcode|
|4|0|Version ID|

The ‘Configure Connection’ request provides a server with details on the connection of the client that sent the message.

This request is comprised of 1 field: the version of protocol adhered to by the sender. To inform the server that version 1.0 of the SUP protocol is in use, this field shall be set to 0.

#### Errors

The ‘Configure Connection’ request shall fail if:

**Invalid protocol version**

* **Version ID** does not identify an existing version of the SUP protocol.

### 4.3. Import Model

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|1|Opcode|
|4|32-bit int|Model ID|
|1+|File path or URL|Model path|

The ‘Import Model’ request shall instruct the server to import a model from a file. If the file specified by **Model path** is a shortcut, then the file loaded shall be that which the shortcut or chain of shortcuts point to. Files of the same base name as **Model path** shall be imported and bound to the loaded model automatically.

The model created by the request must have the same ID number as **Model ID**. Upon creation, the model shall be hidden. Its initial location and rotation coordinates shall be 0 on all axes. Scale, however shall start out at 1 on model load.

This request consists of 2 fields. The first field is 32-bit identification code of the newly created model. The second field directly follows the first and is the model path as a null-terminated UTF-8 string.

#### Errors

The ‘Import Model’ request shall fail if:

**File import failed**

* The server fails to import the model file.

**ID already taken**

* A model of ID **Model ID** already exists in the clients scene.

### 4.4. Show Model

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|2|Opcode|
|4|32-bit int|Model ID|

The ‘Show Model’ request shall make the model specified by **Model ID** visible to the viewer.

#### Errors

The ‘Show Model’ request shall fail if:

**Model not found**

* **Model ID** does not match the ID of any model in the clients scene.

### 4.5. Hide Model

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|3|Opcode|
|4|32-bit int|Model ID|

This request shall hide the model identified by **Model ID**. A model which is hidden shall not be displayed onscreen.

#### Errors

The ‘Hide Model’ request shall fail if:

**Model not found**

* **Model ID** does not match the ID of any model in the clients scene.

### 4.6. Transform Model

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|4|Opcode|
|4|32-bit int|Model ID|
|4|float|X location|
|4|float|Y location|
|4|float|Z location|
|4|float|X rotation|
|4|float|Y rotation|
|4|float|Z rotation|
|4|float|X scale|
|4|float|Y scale|
|4|float|Z scale|

The ‘Transform Model’ request shall set the location, rotation and scale of the model referenced by **Model ID**. The X, Y and Z location, rotation and scale coordinates of the selected model are to be copied from the request.

#### Errors

The ‘Transform Model’ request shall fail if:

**Model not found**

* **Model ID** does not match the ID of any model in the clients scene.

### 4.7. Destroy Model

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|5|Opcode|
|4|32-bit int|Model ID|

This request shall remove the model specified by **Model ID** from its scene.

#### Errors

The ‘Destroy Model’ request shall fail if:

**Model not found**

* **Model ID** does not match the ID of any model in the clients scene.

### 4.8. Create Action Instance

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|6|Opcode|
|4|32-bit int|Model ID|
|4|32-bit int|Action ID|
|1+|string|Action name|

Models may contain a set of animation sequences called actions. These sequences may animate all properties of a model, including those adjustable through requests.

This request shall create a reference to an action of the same name as **Action name** in the model specified by **Model ID**. Upon creation, the action instance shall have its scale set to 0. More information on action scaling and blending is available in section **6.9. Set Action Weight**.

#### Errors

The ‘Create Action Instance’ request shall fail if:

**Model not found**

* **Model ID** does not match the ID of any model in the clients scene.

**Action not found**

* The model specified by **Model ID** has no action whose name matches the string provided by **Action name**.

**ID already taken**

* An action of ID **Action ID** already exists in the clients scene.

### 4.9. Set Action Frame

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|7|Opcode|
|4|32-bit int|Action ID|
|4|32-bit int|Frame index|

The ‘Set Action Frame’ request shall set the current frame of an action. The action to have its frame number set must be identified by same ID as **Action ID**. The **Frame index** field shall specify the frame which the selected action shall skip to.

#### Errors

The ‘Set Action Frame’ request shall fail if:

**Action not found**

* **Action ID** does not match the ID of any action in the clients scene.

### 4.10. Set Action Weight

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|8|Opcode|
|4|32-bit int|Action ID|
|4|float|Weight|

The ‘Set Action Weight’ request shall adjust the blend weight of an action. The action identified by **Action ID** shall have its blend weight set to the value of the **Weight** field.

#### Errors

The ‘Set Action Weight’ request shall fail if:

**Action not found**

* **Action ID** does not match the ID of any action in the clients scene.

### 4.11. Destroy Action Instance

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|9|Opcode|
|4|32-bit int|Action ID|

The ‘Destroy Action Instance’ request shall remove the action instance identified by **Action ID** from its scene.

#### Errors

The ‘Destroy Action Instance’ request shall fail if:

**Action not found**

* **Action ID** does not match the ID of any action in the clients scene.

### 4.12. Transform Viewpoint

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|10|Opcode|
|4|float|X location|
|4|float|Y location|
|4|float|Z location|
|4|float|X rotation|
|4|float|Y rotation|
|4|float|Z rotation|

The ‘Transform Viewpoint’ request shall set the location and rotation of the viewpoint. The command shall set the X, Y and Z location and rotation coordinates of the world viewpoint to those specified by the request.

#### Errors

The ‘Transform Viewpoint’ request has no errors.

### 4.13. Set Viewpoint Field of View

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|11|Opcode|
|4|float|Field of view|

The ‘Set Viewpoint Field of View’ request shall set the angular extent of the viewpoints perspective. The field of view of the viewpoint shall be set to the value of **Field of view**.

#### Errors

The ‘Set Viewpoint Field of View’ request has no errors.

## 5\. Response



Upon a failed request, an error response shall be sent to the sender of the unsuccessful request. Errors may be fatal or non-fatal.

|**Size (bytes)**|**Value**|**Description**|
|-|-|-|
|1|8-bit int|Error code|

Error responses shall be comprised of 2 fields. The first field, **Request opcode**, shall hold the opcode of the command which caused the error. **Error code**, the second field, shall contain the correct identifier of the error which has occurred.

Each error has a unique identifier. Error responses may use the following error codes:

|Value|Description|
|-|-|
|0|Invalid protocol version|
|1|Model not found|
|2|Action not found|
|3|Model import failed|
|4|ID already taken|
|5|Invalid opcode|

### 5.1. Invalid Protocol Version

If a client attempts to select an unsupported SUP version when configuring its connection, the client shall receive this error. The ‘invalid protocol version’ error is identified by an error code of 0 and is fatal.

### 5.2. File Not Found

This error signals to a client that a path does not point to a valid file. It has an error code of 1 and is not fatal.

### 5.3. Action Not Found

An 'action not found' error. This error has a code of 2.

### 5.4. Model Import Failed

This error has a code of 3.

### 5.5. Identifier Already Taken

This error has a code of 4.

### 5.6. Invalid Request Code

This error has a code of 5.

## 6\. Author's Notes

This document is an informal draft. It may not be entirely unambiguous, grammatically correct or consistent. The document, as it stands, merely exists to assist development of a more refined document later on.

