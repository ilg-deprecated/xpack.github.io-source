---
layout: page
lang: en
permalink: /xcdl/files/xcdl-json/
title: The XCDL format
author: Liviu Ionescu

date: 2017-10-10 19:59:00 +0300

---

## Overview

The **XCDL** format was originally inspired from eCos CDL (Configuration Definition Language), and later was extended to include definitions from CMSIS PDSC.

## Purpose

The **XCDL** files are used to

* define the device features
* define the board features
* define the software components and configurations (TBD)

## File conventions

### JSON Properties

As with any JSON file, the content is basically an object with several properties, represented as `"name": <value>`. The values may be strings, objects or arrays of other values.

Named children objects are stored in objects (not arrays), with the names as properties (implemented as maps, with names as keys). Compared to arrays, maps have the advantage of guaranteed unique names.

### Collections

Traditionally, collections are stored in JSON as arrays, which impose no restrictions on the content or uniqueness.

If object names must be unique, collections of named children can be stored as objects (implemented as maps) with names as properties and children objects as values.

To simplify usage, except named children, collections cannot have other properties.

## File names

Specific files have names with a prefix, in singular or plural:

* `devices-xcdl.json` or `device-xcdl.json`
* `boards-xcdl.json` or `board-xcdl.json`

## Objects

The XCDL file is a hierarchy of objects, with the JSON root on top.

### The root object

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `schemaVersion` | string | The version is used by the parser to identify different file formats. |
| `mcus` | collection | A map of mcu objects. MCU names are internal IDs used to refer to the MCU; externally use the device `displayName`; must be unique among all files. |

Example

```json
{
    "schemaVersion": "0.1.0",
    "mcus": {
        "families": {
            "fe": {
                "...": "..."
            }
        }
    },
}
```

### The _mcu_ object

The MCU is the top-most object, and usually contains one MCU family.

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `families` | collection | A map of family objects. The keys are internal IDs used to refer to the family. |

### The _family_ object

| Parent |
|:-------|
| The `mcus` property of the root object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `displayName` | string | A short string to externally identify the device family. Must be unique among all files. If missing, the internal name (the map key) is used. |
| `description` | string | A long string to describe the main features of the device family. |
| `supplier` | object | The device supplier. |
| `devices` | collection | A map of device objects. The keys are internal IDs used to refer to the devices. |

### The _supplier_ object

| Parent |
|:-------|
| A **family** object. |

The device supplier.

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `name` | string | The supplier short name; a single word that uses only legal variable characters. |
| `id` | string | The supplier numeric id; should not change over time. |
| `displayName` | string | A short string to externally identify the supplier. |
| `fullName` | string | A longer string to externally identify the supplier, like official company name. |
| `contact` | string | Contact information. |

### The _device_ object

The device is the basic object, and correspond to a single implementation of a MCU which has unique characteristics from the software point of view.

| Parent |
|:-------|
| The `devices` property of a **family** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `displayName` | string | A short string to externally identify the device. Must be unique among all files related to a supplier. If missing, the internal name (the map key) is used. |
| `description` | string | A long string to describe the main features of the device. |
| `url` | string | A full URL to a page describing the device. |
| `compile` | object | TBD |
| `features` | object | TBD |
| `memoryRegions` | collection | TBD |
| `debug` | object | TBD |

### The _debug_ object

An object with definitions useful for debug sessions.

| Parent |
|:-------|
| A **device** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `jtag` | object | TBD |
| `xsvd` | string | the relative path to the XSVD JSON file (POSIX paths). Used by the debugger to display the peripheral registers. |
| `svd` | string | the relative path to the CMSIS SVD XML file, (POSIX paths). Used by the debugger to display the peripheral registers. Ignored if the `xsvd` property is present. |


TBD

## Revision history

The format version is reflected in the `schemaVersion` property, present in the root object. The value of this property follows the semantic versioning requirements ([semver](http://semver.org)).

Versions are listed in reverse chronological order.

#### v0.1.0 (2017-12-08)

* preliminary version, only some device related fields.
