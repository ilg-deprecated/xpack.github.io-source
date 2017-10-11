---
layout: page
lang: en
permalink: /xsvd/files/xsvd-json/
title: The xsvd format 
author: Liviu Ionescu

date: 2017-10-10 20:00:00 +0300

---

## Overview

The **xsvd** format is inspired from ARM [CMSIS SVD](http://www.keil.com/cmsis/svd), which is based on XML and was influenced by [IP-XACT](https://en.wikipedia.org/wiki/IP-XACT).

The xsvd content is generally similar to the SVD content, but it has a better structure, using a hierarchy of objects with properties. As such, the natural format to represent it is JSON, which is simpler and easier to parse than XML.

To help the migration from CMSIS Packs to xPacks, an automated tool was written to convert ARM SVD to xsvd.

## Purpose

Contrary to the wider scope of IP-XACT, the xsvd format was intentionally kept as simple as possible, since it is currently intended only for: 

* allowing debuggers to display memory mapped peripherals, including separate register bit-fields
* supporting the automated generation of device header files
* supporting the automated generation of source files implementing the peripherals in emulators (like QEMU).

## File conventions

### JSON Properties

As with any JSON file, the content is basically an object with several properties, represented as `"name": <value>`. The values may be strings, objects or arrays of other values.

Named children objects are stored in objects (not arrays), with the names as properties (implemented as maps, with names as keys).

### Names

Since some xsvd definitions may be used to support the generation of C/C++ code files (object names may be used to compose type and macro definitions), the names must comply with ANSI C naming restrictions. In particular, they must not contain any spaces or special characters.

### Values

All scalar values, even those representing numeric values, should be encoded as strings.

Number constants shall be entered as hex or non-negative decimal integers:

* hexadecimal (`"0x..."`)
* decimal (`"123"`)

### Measuring units

Numeric values that may represent values with different units may have the unit explicitly added to the value, after a space separator:

* `"width": "32 bits"`
* `"hfosc": "13800 kHz"`
* `"lfosc": "32768 Hz"`

Note: currently not used.

### Repeat generators

The syntax used in the `repeatGenerator` property is:

* a sequence of numbers, like `0,1,2` 
* a range of numbers, like `0-31`
* a sequence of letters, like `A,B,C`
* a range of letters, like `A-G`.

## Revision history

The format version is reflected in the `schemaVersion` property, present in the root object.

Versions are listed in reverse chronological order.

#### v0.2.0 (2017-10-11)

First formal specifications defined. Used to define the SiFive devices (like the FE310-G000, the first RISC-V physical device).

Differences from SVD:

- explicit separate definitions for repetitions and arrays
- accept repetitions for fields
- accept `resetValue` and `resetMask` for fields
- the `access` property was simplified to `[“ro”,”rw”]`
- use `busWidth` (instead of `<width>`)
- use `regWidth` (instead of `<size>`)
- use `repeatIncrement` (instead of `<dimIncrement>`)
- use `repeatGenerator` (instead of `<dimIndex>`)
- use `arraySize` (instead of `<dim>`)

#### v0.1.0 (2016-12-14)

Initial version, a direct correspondence to the ARM SVD format. Firstly implemented by the xcdl program used by GNU ARM Eclipse QEMU.

## Objects

The entire xsvd file is basically a hierarchy of objects, with the JSON root on top.

### The root object

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `schemaVersion` | string | The version is used by the parser to identify different file formats. |
| `devices` | object | A map of device objects. Device names are internal IDs used to refer to the device; externally use the device `displayName`; must be unique among all files. |
| `generators` | array | Array of generator objects describing the tools that generated/modified the current file. |

Example

```json
{
    "schemaVersion": "0.2.0",
    "devices": {
        "fe310-g000": {
            "...": "..."
        }
    },
    "...": "..."
}
```

### The _device_ object

The device is the top-most object, and contains one or more peripherals.

| Parent |
|:-------|
| The `devices` property of the root object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `version` | string | The version of the device. |
| `displayName` | string | A short string to externally identify the device. Must be unique among all files. If missing, the internal name (the map key) is used. |
| `description` | string | A long string to describe the main features of the device. |
| `busWidth` | string | The width of the maximum single data transfer supported by the bus infrastructure, in bits. This information is relevant for debuggers when accessing registers, because it might be required to issue multiple accesses for resources of a bigger size. |
| `regWidth` | string | Default width of any register contained in the device, in bits. |
| `access` | string | Default access rights for all registers. Values: ["ro","rw"]. |
| `resetValue` | string | Default value for all registers at RESET. |
| `resetMask` | string | The register bits that have a defined reset value. |
| `cpu` | object | An object defining the CPU characteristics. |
| `peripherals` | object | A map of peripheral objects. The keys are internal IDs used to refer to the peripherals; externally use the `displayName`; must be unique among devices. |

Example

```json
{
    "...": "...",
    "devices": {
        "fe310-g000": {
            "version": "1.0.0",
            "displayName": "Freedom E310-G000",
            "description": "The FE310-G000 is the first Freedom E300 SoC, ...",
            "busWidth": "32",
            "regWidth": "32",
            "resetValue": "0x00000000",
            "resetMask": "0xFFFFFFFF",
            "access": "rw",
            "peripherals": {
                "...": "..."
            }
        }
    },
    "...": "..."
}
```

### The _cpu_ object

| Parent |
|:-------|
| A **device** object. |

TBD


### The _peripheral_ object

At least one peripheral has to be defined.

Each peripheral describes all registers belonging to that peripheral.
Each peripheral has a memory block where the registers are allocated, defined as a base address and a size.
Register addresses are specified relative to the base address of a peripheral.

Multiple similar peripherals sharing a common name can be defined as repetitions using the `repeatGenerator` property; for example instead of defining `GPIOA`, `GPIOB`, `GPIOC`, use `GPIO%s` (or simply `GPIO`, since `%s` at the end is default) and define separate generators like `A,B,C` or `A-C`.

The `repeatIncrement` property specifies the address offset between two peripherals. 

To create copies of a peripheral using different names, use the `derivedFrom` property.

| Parent |
|:-------|
| The `peripherals` property of a **device** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `derivedFrom` | string | The peripheral name from which to inherit properties. Subsequent properties override inherited values. |
| `displayName` | string | A short string to externally identify the peripheral. Must be unique among the device. If missing, the internal name (the map key) is used. |
| `description` | string | A long string to describe the peripheral. |
| `baseAddress` | string | The lowest address of the memory block used by the peripheral. |
| `size` | string | The size in bytes of the memory block used by the peripheral. |
| `regWidth` | string | Default width of any register contained in the peripheral, in bits. |
| `access` | string | Default access rights for all peripheral registers. Values: ["ro","rw"]. |
| `resetValue` | string | Default value for all peripheral registers at RESET. |
| `resetMask` | string | The register bits that have a defined reset value. |
| `repeatIncrement` | string | The address increment, in bytes, between two neighbouring peripherals in a repetition or an array. |
| `repeatGenerator` | string | A generator of strings that substitute the placeholder `%s` within the `displayName`. 
| `groupName` | string | Define a name under which the Peripheral Registers Viewer is showing this peripheral. |
| `headerStructName` | string | Specify the base name of C structures. The header file generator uses the peripheral `displayName` as the base name for the C structure type. If `headerStructName` is specified, then this string is used instead of the peripheral name; useful when multiple peripherals get derived and a generic type name should be used. |
| `clusters` | object | A map of cluster objects. The keys are internal IDs used to refer to the clusters; externally use the cluster name. |
| `registers` | object | A map of register objects. The keys are internal IDs used to refer to the registers; externally use the `displayName`. |
| `interrupts` | object | A peripheral can have multiple associated interrupts. This entry allows the debugger to show interrupt names instead of interrupt numbers. |

```json
{
    "...": "...",
    "peripherals": {
        "clint": {
            "description": "Coreplex-Local Interruptor (CLINT)",
            "baseAddress": "0x02000000",
            "size": "0x10000",
            "registers": {
                "...": "..."
            }
        }
    },
    "...": "..."
}
```

### The _cluster_ object

A cluster describes a sequence of neighbouring registers within a peripheral. A cluster specifies the `addressOffset` relative to the `baseAddress` of the grouping element. All register objects within a cluster specify their `addressOffset` relative to the cluster base address (peripheral.baseAddress + cluster.addressOffset).

Nested clusters express hierarchical structures of registers. It is targeted at the generation of device header files to create a C-data structure within the peripheral structure instead of a flat list of registers. It is also possible to specify an array of a clusters using the `arraySize` property.

Multiple similar clusters sharing a common name can be defined as repetitions using the `repeatGenerator` property. 

Multiple similar clusters can also be grouped in arrays. The size of the array is specified by the `arraySize` property. 

The `repeatIncrement` property specifies the offset in bytes between two clusters. If not specified, the cluster size is used.

| Parent |
|:-------|
| The `clusters` property of a **cluster** or **peripheral** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `derivedFrom` | string | The cluster name from which to inherit properties. Subsequent properties override inherited values. |
| `displayName` | string | A short string to externally identify the cluster. Must be unique among the parent object. If missing, the internal name (the map key) is used. For repetitions, use the placeholder `%s`, which is replaced by strings from `repeatGenerator`; if missing, `%s` will be added at the end. For arrays, the placeholder `[%s]` will be automatically added at the end. |
| `description` | string | A long string to describe the register. |
| `addressOffset` | string | The address offset relative to the enclosing element (peripheral or cluster). |
| `regWidth` | string | The width of the register, in bits. |
| `access` | string | Default access rights for the register. Values: ["ro","rw"]. |
| `resetValue` | string | Default value for the register at RESET. |
| `resetMask` | string | The register bits that have a defined reset value. |
| `repeatIncrement` | string | The address increment, in bytes, between two neighbouring clusters in a repetition or an array. |
| `repeatGenerator` | string | A generator of strings that substitute the placeholder `%s` within the `displayName`. 
| `arraySize` | string | The size of the array of clusters. |
| `clusters` | object | A map of cluster objects. The keys are internal IDs used to refer to the clusters; externally use the cluster name. |
| `registers` | object | A map of register objects. The keys are internal IDs used to refer to the registers; externally use the `displayName`. |

```json
{
    "...": "...",
    "clusters": {
        "target-enables": {
            "displayName": "target-enables[%s]",
            "arraySize": "1",
            "description": "Hart M-Mode interrupt enable bits",
            "addressOffset": "0x0C002000",
            "repeatIncrement": "0x20",
            "registers": {
                "...": "..."
            }
        },
        "targets": {
            "displayName": "targets[%s]",
            "arraySize": "1",
            "description": "Hart M-Mode interrupt thresholds",
            "addressOffset": "0x0C200000",
            "repeatIncrement": "0x8",
            "registers": {
                "...": "..."
            }
        }
    },
   "...": "..."
}
```

### The _register_ object

The description of registers is the most important part of this file format. If the properties `regWidth`, `access`, `resetValue`, and `resetMask` have not been specified on a higher level, then these properties are mandatory on register level.

A register can represent a single value or can be subdivided into individual bit-fields of specific functionality and semantics. From a schema perspective, the property `fields` is optional, however, from a specification perspective, `fields` are mandatory when they are described in the device documentation.

Multiple similar registers sharing a common name can be defined as repetitions using the `repeatGenerator` property. 

Multiple similar registers can also be grouped in arrays. The size of the array is specified by the `arraySize` property. 

The `repeatIncrement` property specifies the offset in bytes between two registers. If not specified, the register width is used.

| Parent |
|:-------|
| The `registers` property of a **cluster** or **peripheral** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `derivedFrom` | string | The register name from which to inherit properties. Subsequent properties override inherited values. |
| `displayName` | string | A short string to externally identify the register. Must be unique among the parent. If missing, the internal name (the map key) is used. For repetitions, use the placeholder `%s`, which is replaced by strings from `repeatGenerator`; if missing, `%s` will be added at the end. For arrays, the placeholder `[%s]` will be automatically added at the end. |
| `description` | string | A long string to describe the register. |
| `addressOffset` | string | The address offset relative to the enclosing element (peripheral or cluster). |
| `regWidth` | string | The width of the register, in bits. |
| `access` | string | Default access rights for the register. Values: ["ro","rw"]. |
| `resetValue` | string | Default value for the register at RESET. |
| `resetMask` | string | The register bits that have a defined reset value. |
| `repeatIncrement` | string | The address increment, in bytes, between two neighbouring registers in a repetition or an array. |
| `repeatGenerator` | string | A generator of strings that substitute the placeholder `%s` within the `displayName`. 
| `arraySize` | string | The size of the array of registers. |
| `fields` | object | A map of field objects. The keys are internal IDs used to refer to the fields; externally use the `displayName`. |

```json
{
   "...": "...",
    "registers": {
        "msip0": {
            "description": "MSIP for hart 0",
            "addressOffset": "0x0000"
        },
        "mtimecmp0": {
            "description": "Timer compare register for hart 0",
            "addressOffset": "0x4000",
            "regWidth": "64"
        },
        "mtime": {
            "description": "Timer register",
            "addressOffset": "0xBFF8",
            "access": "ro",
            "regWidth": "64"
        }
    },
   "...": "..."
}
```

### The _field_ object

A bit-field has a name that is unique within the register. The position and size within the register can be described as
the lsb and the bit-width of the field.

Multiple similar fields sharing a common name can be defined as repetitions using the `repeatGenerator` property. 

The `repeatIncrement` property specifies the offset in bits between two bit-fields. If not specified, the bit-field width is used.

A field may define an enumeration in order to make the display more intuitive to read.

| Parent |
|:-------|
| The `fields` property of a  **cluster** or **peripheral** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| parent | | A cluster object or a peripheral object |
| `derivedFrom` | string | The field name from which to inherit properties. Subsequent properties override inherited values. |
| `displayName` | string | A short string to externally identify the field. Must be unique among the register. If missing, the internal name (the map key) is used. For repetitions, use the placeholder `%s`, which is replaced by strings from `repeatGenerator`; if missing, `%s` will be added at the end. |
| `description` | string | A long string to describe the field. |
| `bitOffset` | string | The position of the least significant bit of the field within the register. |
| `bitWidth` | string | The bit-width of the field within the register. |
| `access` | string | Default access rights for the register. Values: ["ro","rw"]. |
| `resetValue` | string | Default value for the field at RESET. |
| `resetMask` | string | The field bits that have a defined reset value. |
| `repeatIncrement` | string | The offset increment, in bits, between two neighbouring fields in a repetition. |
| `repeatGenerator` | string | A generator of strings that substitute the placeholder `%s` within the `displayName`. 
| `enumeration` | object | A map with a single property. The key is the enumeration name. |

Example

```json
{
    "...": "...",
    "fields": {
        "data": {
            "description": "Transmit data",
            "bitOffset": "0",
            "bitWidth": "8"
        },
        "full": {
            "description": "Transmit FIFO full",
            "bitOffset": "31",
            "bitWidth": "1"
        }
    },
    "...": "..."
}
```

### The _enumeration_ object

The concept of enumerated values creates a map between unsigned integers and an identifier string. In addition, a description string can be associated with each entry in the map.

* `0` ↔︎ disabled → "The clock source clk0 is turned off."
* `1` ↔︎ enabled  → "The clock source clk1 is running."
* `*` ↔︎ reserved → "Reserved values. Do not use."

This information generates an enum in the device header file. The debugger may use this information to display the identifier string as well as the description. Just like symbolic constants making source code more readable, the system view in the debugger becomes more instructive. The detailed description can provide reference manual level details within the debugger.

| Parent |
|:-------|
| The `enumeration` property of a **field** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `derivedFrom` | string | The enumeration name from which to inherit properties. Subsequent properties override inherited values. |
| `displayName` | string | A short string to externally identify the enumeration. Must be unique among the enumerations. If missing, the internal name (the map key) is used. |
| `description` | string | A long string to describe the enumeration. |
| `headerEnumName` | string | An alternate identifier for the enumeration section when used to generate the device header. The default is the hierarchical enumeration type in the device header file. The user is responsible for the uniqueness across the device. |
| `values` | object | A map of enumerated values objects. The map keys are the enumeration numeric values (unsigned integers). |

Example

```json
{
    "...": "...",
    "enumeration": {
        "clk-src-enum": {
            "displayName": "Clock source",
            "description": "Values to select the clock source.",
            "values": {
                "0": {
                    "displayName": "disabled",
                    "description": "The clock source clk0 is turned off."
                },
                "1": {
                    "displayName": "enabled",
                    "description": "The clock source clk1 is running."
                },
                "*": {
                    "displayName": "reserved",
                    "description": "Reserved values. Do not use."
                }
            }
        }
    },
    "...": "..."
}
```

### The _enumerated value_ object

An enumerated value defines a map between an unsigned integer (the map key) and a string.

| Parent |
|:-------|
| The `values` property of an **enumeration** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `displayName` | string | A short string to externally describe the semantics of the value. Usually displayed together with the value. |
| `description` | string | A long string to describe the value. |

### The _interrupt_ object

A peripheral can have multiple interrupts. This object allows the debugger to show interrupt names instead of interrupt numbers.

| Parent |
|:-------|
| The `interrupts` property of a **peripheral** object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `displayName` | string | A short string to externally identify the interrupt. Must be unique among all interrupts. If missing, the internal name (the map key) is used. |
| `description` | string | A long string to describe the interrupt. |
| `value` | string | The enumeration index value associated to the interrupt. |
| `type` | string | The interrupt type, if the architecture defines multiple interrupt spaces. Architecture specific. For RISC-V use `["local","global"].` |
| `headerIrqName` | string | The name used for the interrupt handler. |

Example

```json
{
    "...": "...",
    "interrupts": {
        "uart0": {
            "description": "UART0 global interrupt",
            "value": "3",
            "type": "global",
            "headerIrqName": "riscv_interrupt_global_handle_uart0"
        }
    }
    "...": "..."
}
```

### The _generator_ object

This is an informative object, used to trace which tools generated or modified the xsvd file.

| Parent |
|:-------|
| The root object. |

| Properties | Type | Description |
|:-----------|:-----|:------------|
| `tool` | string | A string describing the tool. |
| `version` | string | The tool version, as returned by `--version`. |
| `command` | string[] | Array of strings, the full command with all parameters. |
| `date` | string | A string with the date, in ISO format. |

Example

```json
{
    "...": "...",
    "generators": [
        {
            "tool": "xcdl",
            "version": "1.6.2",
            "command": [
                "svd-convert",
                "--file",
                "/Users/ilg/Library/xPacks/Keil/STM32F0xx_DFP/1.5.0/SVD/STM32F0x1.svd",
                "--output",
                "STM32F0x1-xsvd.json"
            ],
            "date": "2016-12-14T22:50:48.956Z"
        }
    ],
    "...": "..."
}
```


## TODO

- clarify how multi-core devices can be described
- clarify if the `cpu` object is needed, since most of its definitions are already in the `device-xcdl.json` file
- define where the interrupt names are added
- define the QEMU mode bits, that define if unaligned or smaller size accesses are allowed.
