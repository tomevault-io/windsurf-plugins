---
trigger: always_on
description: Made up of two types of classes:
---


# PsyNeuLink Organization, Coding, and Documentation Conventions

## REPOSITORY ORGANIZATION:

### Core:
Made up of two types of classes:
- *abstract base classes* (italicized) - cannot be instantiated.
- **core classes** (bold) - most basic (abstract) level of objects that can be instantiated. 

#### Components
"Building blocks"
- *Mechanism*
    - *ProcessingMechanism*
        - **TransferMechanism**
        - **IntegratorMechanism**
        - **ObjectiveMechanism**
    - *AdaptiveMechanism*
        - **LearningMechanism**
        - **ControlMechanism**
        - **GatingMechanism**
- *Projection*
    - *PathwayProjection*
        - **MappingProjection**
    - *ModulatoryProjection*
        - **LearningProjection**
        - **ControlProjection**
        - **GatingProjection**
- *Port*
    - **InputPort**
    - **ParameterPort**
    - **OutputPort**
    - *ModulatorySignal*
        - **LearningSignal**
        - **ControlSignal**
        - **GatingSignal**
- *Function*
    - *TransferFunction*
    - *CombinationFunction*
    - *IntegratorFunction*
    - *DistributionFunction*
    - *LearningFunction*
        
#### Composisitons
Objects that compose building blocks and control their execution.
- *Composition*
    - **System**
    - **Process**

#### Scheduler
Objects used by Compositions to control the execution of Components and Compositions.
- **Scheduler**
- **Condition** 

### Library
Extensions of Core objects
- *Components:* classes derived from Core objects
- *Compositions:*  models
- *Models:*  published, implemented models

### NAMING:

#### Content:
- All class (and corresponding module) names should be singular  
- Component names always end in their type (e.g., TransferMechanism, LearningProjection)
  (the only exception is the DDM)
- Components and Compositions should *always* be referred to in caps
  (e.g., All Mechanisms have Projections; the receiver for a Projection is an InputPort; etc.).

#### Format:
- class names:
    fully capitalized camelCase [ClassName]
- classAttributes: camelCase without initial capitalization
- arguments_of_constructors, instance_attributes and instance_methods:
      lowercase and underscore separator(s) [constructor_arg, method_arg, object_attribute]
- keywords:
  - all capitals and underscore separator(s) [KEY_WORD]
  - assigned values:
     - argument of a method or function: lower case [KEY_WORD = 'argument_value']
     - names of a Components: upper case [KEY_WORD = 'NAME'] 

DEPRECATED:
    - internal keywords:
        prepend kw followed by camelCase [kwKeyword]

####Errors and Warnings:

warnings.warn("WARNING:..."):
  - user's input/action may produce an unexpected outcome/behavior
  
raise \<Class\>Error("PROGRAM ERROR:..."):  
  - disallowed coding practice or PsyNeuLink problem
  
Assertion:  
  - as yet unresolved/unhandled condition;  can be in devel but NOT in master (except in test scripts)


### GRAMMATICAL:

#### Elements and items of lists and arrays:
- "value": any specified token (numeric or string);
    generally references to the entity received, represented or output by a state or projection,
    but can also refer to the specification of an attribute
- "element": refers to the finest grade constituent (highest dimension / axis)
- "item" refers to any constituent at any level higher than the highest dimension / axis
- Example:  [[a, b, c] [d, e, f]]
            a, b, and c are elements of the first item
            d, e, and f are elements of the second item

#### Parameters, arguments and attributes:
- "parameter" refers to any specifiable attribute of a PsyNeuLink component
- "argument" refers to a specifiable value in a method or function call
- "attribute" is the generic Python term for an object member
- arguments "specify" a value or an assignment;  attributes "determine" a value or some outcome

#### Referencing:
- <definite article> `item`;  <indefinite article> item; e.g.: the `errorSource`;  an errorSource
- a value is "assigned" to an attribute; the value of an attribute is specified...
- a run is multiple executions;  accordingly, plural for "input" and "target" refers to multiple executions,
    not number of items in the array  (e.g., the input for an execution, the inputs for a run)

### DOCSTRING ORGANIZATION:

#### Module docstring sections:

  .. _<X>_Overview:

  Overview 
  --------
  High level description of object and its relationship to others (including its super).
  
  .. _<X>_Creation:
  
  Creating a(n) <X>
  -----------------
  Description of use of constructor, context of creation, and/or conditions of automatic creation by other objects
  
  .. _<X>_Structure:

  Structure
  ---------
  Explanation of all class-specific attributes; this should be the most elaborate explanation of each attribute, 
  that is referenced by briefer descriptions of each in the Attributes section.
       
  .. _<X>_Execution:
  
  Execution
  ---------
  Details of how the object executes, including what its `function` does
  
  .. _<X>_Class_Reference:
  
  Class Reference
  ---------------

  
#### Module / Class docstrings:
  [COMMENTED OUT:
      Technical information, with headings:
        - Description: technical description (including categor/type)
        - Class Attributes: full list of any class-specific attributes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PrincetonUniversity/PsyNeuLink](https://github.com/PrincetonUniversity/PsyNeuLink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
