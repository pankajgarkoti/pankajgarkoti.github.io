---
layout: post
title: ChainFactory Specification Draft 001
---

# The Chain Factory Specification
**Draft 001**

## Structure
A .fctr file can be thought of as a YAML file with some syntax additions to make it easier to define chains.

- Specify the Prompt or list the Inputs (keyword: prompts & in)
- Define Models (keyword: def)
- Specify the Outputs (keyword: out)


## Typing
The typing system takes direct inspiration from Python's type annotations. The only difference is that the type system is stricter and more limited. The following atomic types are supported:

- str
- int
- float
- bool
- list
- dict

It is possible to define custom types in the `def` section of the .fctr file. The syntax for typing a field is as follows:

`[name]: [type][?]=[default_value]`

The `?` symbol indicates that the field is optional. If a field has a RHS value that is not a valid type, ChainFactory will assume that the field type is `str` and the RHS is a default value. 


### Definition
The def keyword is the part of the .fctr file that defines new types to be used in rest of the file.
To define a 


### Prompt
The prompt template related options can be set under this keyword. Keyword options are:

- type: template # can be template, auto. the template is generated automatically based on the purpose of the chain in the auto mode.
- purpose: null # a string that describes the purpose of the chain. this can be used for auto generating the prompt template.
- template: > # the template to use for the prompt. 


### In
The in keyword is the part of the .fctr file that defines the inputs of the chain. This is necessary if the prompt is set to auto mode. The in keyword is a list of objects that define the input properties of the chain. The following syntax is used to define the input properties:


### Out
The `out` keyword defines part of the .fctr file that defines the output structure of the chain. This can be skipped if the chain output is a single string. Otherwise the `out` section contains a list of attributes that define the output properties of the chain. The syntax is same as the `in` keyword.
