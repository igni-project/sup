# Scene Update Protocol

SUP/1.0, Draft 2
January 2025, Igni Project

This document is an informal draft. It may not be entirely unambiguous, grammatically correct or consistent. The document, as it stands, merely exists to assist development of a more refined document later on.

Despite the contents of this document being subject to change, the protocol itself shall remain unchanged.

## Table of Contents
1. Introduction
2. Definitions and Abbreviations
3. Table Formats
    1. Data Structures
    2. Enumerations
4. Data Formats
    1. String
    2. File Path
    3. URL
5. Error Response
6. Requests
    1. Configure Connection
    2. Load Asset
    3. Show Asset
    4. Hide Asset
    5. Transform Asset
    6. Destroy Asset
    7. Create Action Instance
    8. Set Action Frame
    9. Set Action Weight
    10. Destroy Action Instance
    11. Transform Viewpoint
    12. Set Viewpoint Field of View 

## 1. Introduction

The Scene Update Protocol (SUP) is an application layer communication protocol that is lightweight and extensible enough to transfer modern 3D graphics over a network. This protocol requires a bi-directional client-to-server network connection to operate.

Nearing the end of the development of the Visual Scene Update Protocol (VSUP), it quickly became evident that the protocol had a wider range of use than simply visual graphics. To better reflect its nature, VSUP is now known as SUP: the Scene Update Protocol. This rebrand was performed by first discontinuing VSUP. At the same time, the new SUP standard was created, forked from the first and only draft of VSUP. Because this is not VSUP but a newly created standard, an opportunity has opened to make changes to the protocol.

The new protocol features:

-   ability to load assets via URL
-   revised opcode values

## 2. Definitions and Abbreviations

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

**shader**
executable code which renders graphics

**uint**
unsigned integer

**opcode**
operation code – a unique number which identifies a command

## 3. Table Formats

To increase legibility and clarity, some data has been summarised as tables. The formats which these tables adhere to are detailed in the following subsections.

### 3.1. Data Structures

A data structure may be formatted as a table with 3 columns:

| **Size (bytes)**| **Value**           | **Description**                               |
|-----------------|---------------------|-----------------------------------------------|
| Size of a field | Contents of a field | A brief description on the purpose of a field |

Each row in the table shall outline the properties of a field in its data structure. Fields are ordered from top to bottom; The top row describing the first field, and the bottom row describing the last field.

### 3.2. Enumerations

An enumeration may be formatted as a 2 column table:

| Value           | Description                                         |
|-----------------|-----------------------------------------------------|
| Constant number | A brief description of the value and its properties |

Each row in the table shall list a constant value alongside its description. Values shall be listed lowest to highest, with the lowest value first at the top of the table.

## 4. Data Formats

### 4.1. String

The term ‘string’ in this document specifically refers to a null-terminated string in UTF8 (RFC [3629](https://datatracker.ietf.org/doc/html/rfc3629)) format.

### 4.2. File Path

The term ‘file path’ in this document specifically refers to a string of a UNIX-like absolute file path.

File paths shall obey the following rules:

1.  The directory separator is one character, a forward slash ( / ).
    
2.  A file path may be prefixed with forward slash to signify that it is an absolute path. This, however, shall not make a difference as all file paths are interpreted as absolute.

### 4.3. URL

The term 'URL' is an acronym for Uniform Resource Locator. URLs shall be interpreted in accordance with the URL Standard by WHATWG ([link](https://url.spec.whatwg.org/)).

## 5. Error Response

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 8-bit uint  | Request opcode  |
| 1               | 8-bit int   | Error code      |

The error response shall be comprised of 2 fields. The first field, **Request opcode**, shall hold the opcode of the command which caused the error. **Error code**, the second field, shall contain the correct identifier of the error which has occurred.

Each error has a unique identifier. Error responses may use the following error codes:

| Value | Description              |
|-------|--------------------------|
| 0     | Invalid protocol version |
| 1     | Asset not found          |
| 2     | Action not found         |
| 3     | File import failed       |
| 4     | ID already taken         |
| 5     | Invalid opcode           |

## 6. Requests

### 6.1. Configure Connection

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 0           | Opcode          |
| 4               | 0           | Version ID      |

The ‘Configure Connection’ request shall adjust the properties of a clients connection with the server.

**Version ID** shall determine the version of protocol to use during client-server communication. To inform server that the client is communicating through version 1.0 of the SUP protocol, **Version ID** shall be set to 0.

#### Errors

The ‘Configure Connection’ request shall fail if:

**Invalid protocol version**
-   **Version ID** does not identify an existing version of the SUP protocol.

### 6.2. Import Asset

| **Size (bytes)**| **Value**         | **Description** |
|-----------------|-------------------|-----------------|
| 1               | 1                 | Opcode          |
| 4               | 32-bit int        | Asset ID        |
| 1+              | File path or URL  | Asset path      |

The ‘Import Asset’ request shall instruct the server to import an asset from a file. If the file specified by **Asset path** is a shortcut, then the file loaded shall be that which the shortcut or chain of shortcuts point to. Files of the same base name as **Asset path** shall be imported and bound to the loaded asset automatically.

The asset created by the request must have the same ID number as **Asset ID**. Upon creation, the asset shall be hidden. Its initial location and rotation coordinates shall be 0 on all axes. Scale, however shall start out at 1 on asset load.

All coordinates of elements in the asset shall be interpreted as local to the transformation of the asset.

#### Errors

The ‘Import Asset’ request shall fail if:

**File import failed**
-   The server fails to import the asset file.

**ID already taken**
-   An asset of ID **Asset ID** already exists in the clients scene.

### 6.3. Show Asset

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 2           | Opcode          |
| 4               | 32-bit int  | Asset ID        |

This ‘Show Asset’ request shall make the asset specified by **Asset ID** visible to the viewer.

#### Errors
The ‘Show Asset’ request shall fail if:

**Asset not found**
-   **Asset ID** does not match the ID of any asset in the clients scene.

### 6.4. Hide Asset

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 3           | Opcode          |
| 4               | 32-bit int  | Asset ID        |

This request shall hide the asset identified by **Asset ID**. A asset which is hidden shall not be displayed onscreen.

#### Errors
The ‘Hide Asset’ request shall fail if:

**Asset not found**
-   **Asset ID** does not match the ID of any asset in the clients scene.

### 6.5. Transform Asset

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 4           | Opcode          |
| 4               | 32-bit int  | Asset ID        |
| 4               | float       | X location      |
| 4               | float       | Y location      |
| 4               | float       | Z location      |
| 4               | float       | X rotation      |
| 4               | float       | Y rotation      |
| 4               | float       | Z rotation      |
| 4               | float       | X scale         |
| 4               | float       | Y scale         |
| 4               | float       | Z scale         |

The ‘Transform Asset’ request shall set the location, rotation and scale of the asset referenced by **Asset ID**. The X, Y and Z location, rotation and scale coordinates of the selected asset are to be copied from the request.

#### Errors
The ‘Transform Asset’ request shall fail if:

**Asset not found**
-   **Asset ID** does not match the ID of any asset in the clients scene.

### 6.6. Destroy Asset

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 5           | Opcode          |
| 4               | 32-bit int  | Asset ID        |

This request shall remove the asset specified by **Asset ID** from its scene.

#### Errors
The ‘Destroy Asset’ request shall fail if:

**Asset not found**
-   **Asset ID** does not match the ID of any asset in the clients scene.

### 6.7. Create Action Instance

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 6           | Opcode          |
| 4               | 32-bit int  | Asset ID        |
| 4               | 32-bit int  | Action ID       |
| 4               | string      | Action name     |

Assets may contain a set of animation sequences called actions. These sequences may animate all properties of a asset, including those adjustable through requests.

This request shall create a reference to an action of the same name as **Action name** in the asset specified by **Asset ID**. Upon creation, the action instance shall have its scale set to 0. More information on action scaling and blending is available in section **6.9. Set Action Weight**.

#### Errors
The ‘Create Action Instance’ request shall fail if:

**Asset not found**
-   **Asset ID** does not match the ID of any asset in the clients scene.

**Action not found**
-   The asset specified by **Asset ID** has no action whose name matches the string provided by **Action name**.

**ID already taken**
-   An action of ID **Action ID** already exists in the clients scene.

### 6.8. Set Action Frame

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 7           | Opcode          |
| 4               | 32-bit int  | Action ID       |
| 4               | 32-bit int  | Frame index     |

The ‘Set Action Frame’ request shall set the current frame of an action. The action to have its frame number set must be identified by same ID as **Action ID**. The **Frame index** field shall specify the frame which the selected action shall skip to.

#### Errors
The ‘Set Action Frame’ request shall fail if:

**Action not found**
-   **Action ID** does not match the ID of any action in the clients scene.

### 6.9. Set Action Weight

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 8           | Opcode          |
| 4               | 32-bit int  | Action ID       |
| 4               | float       | Weight          |

The ‘Set Action Weight’ request shall adjust the blend weight of an action. The action identified by **Action ID** shall have its blend weight set to the value of the **Weight** field.

#### Errors
The ‘Set Action Weight’ request shall fail if:

**Action not found**
-   **Action ID** does not match the ID of any action in the clients scene.

### 6.10. Destroy Action Instance

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 9           | Opcode          |
| 4               | 32-bit int  | Action ID       |

The ‘Destroy Action Instance’ request shall remove the action instance identified by **Action ID** from its scene.

#### Errors
The ‘Destroy Action Instance’ request shall fail if:

**Action not found**
-   **Action ID** does not match the ID of any action in the clients scene.

### 6.11. Transform Viewpoint

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 10          | Opcode          |
| 4               | float       | X location      |
| 4               | float       | Y location      |
| 4               | float       | Z location      |
| 4               | float       | X rotation      |
| 4               | float       | Y rotation      |
| 4               | float       | Z rotation      |

The ‘Transform Viewpoint’ request shall set the location and rotation of the viewpoint. The command shall set the X, Y and Z location and rotation coordinates of the world viewpoint to those specified by the request.

#### Errors
The ‘Transform Viewpoint’ request has no errors.

### 6.12. Set Viewpoint Field of View

| **Size (bytes)**| **Value**   | **Description** |
|-----------------|-------------|-----------------|
| 1               | 11          | Opcode          |
| 4               | float       | Field of view   |

The ‘Set Viewpoint Field of View’ request shall set the angular extent of the viewpoints perspective. The field of view of the viewpoint shall be set to the value of **Field of view**.

#### Errors
The ‘Set Viewpoint Field of View’ request has no errors.

