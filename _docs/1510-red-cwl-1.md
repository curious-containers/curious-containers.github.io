---
title: "RED-CWL 1 Specification"
permalink: /docs/red-cwl-1
---


**DRAFT: This spec is still subject to change.**

*Changes to the draft can viewed in the [git history of this page](https://github.com/curious-containers/curious-containers.github.io/blob/master/_docs/1510-red-cwl-1.md).*

RED-CWL 1 is a subset of the [CWL 1.1 CommandLineTool Description](https://www.commonwl.org/v1.1/CommandLineTool.html). RED 9 will be the first version that implements the RED-CWL 1 specification draft.

The CWL reference implementation `cwltool` requires a `*.cwl` file as generic description of CommandLineTool and a `*.yml` job file to define an input object, that contains actual parameters to run the experiment.
In contrast, a RED file embeds the contents of a `*.cwl` file under the `cli` keyword and embeds the input object under the `inputs` keyword.
Depending on the RED execution engine, defining an output object might be optional and can be embedded under the `outputs` keyword.

The RED-CWL description can be extracted from a RED file's `cli` section and stored in a `*.cwl` file.
Since RED-CWL is a subset of the CWL standard, this extracted description should always be compatible with CWL runtimes like `cwltool`.
On the other hand, not every CWL description can be embedded into a RED file without modification.

This spec refers to individial sections of the [CWL 1.1 CommandLineTool Description](https://www.commonwl.org/v1.1/CommandLineTool.html) to define wether a certain part of the standard is supported or not.


# Data model

## Identifiers

[CWL 1.1 - 2.3 Identifiers](https://www.commonwl.org/v1.1/CommandLineTool.html#Identifiers)

Arbitrary identifiers using Schema Salad definitions are not supported.

Only `id` fields that are explicitely described in the CWL 1.1 CommandLineTool Description are supported. This limitation is allowed by the CWL standard.


## Document Preprocessing

[CWL 1.1 - 2.3 Document preprocessing](https://www.commonwl.org/v1.1/CommandLineTool.html#Document_preprocessing)

`$import` and `$include` directives are not supported.

Data type simplifications are supported.


## Extensions and metadata

[CWL 1.1 - 2.4 Extensions and metadata](https://www.commonwl.org/v1.1/CommandLineTool.html#Extensions_and_metadata)

Extensions and metadata are not supported.


# Execution model

## Generic execution process

[CWL 1.1 - 3.2 Generic execution process](https://www.commonwl.org/v1.1/CommandLineTool.html#Generic_execution_process)

The generic execution process is not supported, because RED execution work inherently different than CWL runtimes.

RED execution engines execute the following steps using an "agent" program that supervises the processing in the experiment container:

1. Execute validate subcommands of input and output connectors.
2. Execute input connectors.
3. Execute CommandLineTool.
4. Execute output connectors.


## Requirements and hints

[CWL 1.1 - 3.3 Requirements and hints](https://www.commonwl.org/v1.1/CommandLineTool.html#Requirements_and_hints)

Requirements and hints are not supported.


## Parameter references

[CWL 1.1 - 3.4 Parameter references](https://www.commonwl.org/v1.1/CommandLineTool.html#Parameter_references)


## Expressions (Optional)

[CWL 1.1 - 3.5 Expressions (Optional)](https://www.commonwl.org/v1.1/CommandLineTool.html#Expressions_(Optional))


## Executing CWL documents as scripts

[CWL 1.1 - 3.6 Executing CWL documents as scripts](https://www.commonwl.org/v1.1/CommandLineTool.html#Executing_CWL_documents_as_scripts)


## Discovering CWL documents on a local filesystem

[CWL 1.1 - 3.7 Discovering CWL documents on a local filesystem](https://www.commonwl.org/v1.1/CommandLineTool.html#Discovering_CWL_documents_on_a_local_filesystem)


# Running a Command

## Input binding

[CWL 1.1 - 4.1 Input binding](https://www.commonwl.org/v1.1/CommandLineTool.html#Input_binding)


## Runtime environment

[CWL 1.1 - 4.2 Runtime environement](https://www.commonwl.org/v1.1/CommandLineTool.html#Runtime_environment)


## Execution

[CWL 1.1 - 4.3 Execution](https://www.commonwl.org/v1.1/CommandLineTool.html#Execution)


## Output binding

[CWL 1.1 - 4.4 Output binding](https://www.commonwl.org/v1.1/CommandLineTool.html#Output_binding)


# CommandLine Tool

[CWL 1.1 - 5 CommandLine Tool](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandLineTool)


## CommandInputParameter

[CWL 1.1 - 5.1 CommandInputParameter](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandInputParameter)


### CommandLineBinding

[CWL 1.1 - 5.1.1 CommandLineBinding](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandLineBinding)


### Expression

[CWL 1.1 - 5.1.2 Expression](https://www.commonwl.org/v1.1/CommandLineTool.html#Expression)


### SecondaryFileSchema

[CWL 1.1 - 5.1.3 SecondaryFileSchema](https://www.commonwl.org/v1.1/CommandLineTool.html#SecondaryFileSchema)


### LoadListingEnum

[CWL 1.1 - 5.1.4 LoadListingEnum](https://www.commonwl.org/v1.1/CommandLineTool.html#LoadListingEnum)


### Any

[CWL 1.1 - 5.1.5 Any](https://www.commonwl.org/v1.1/CommandLineTool.html#Any)


### CWLType

[CWL 1.1 - 5.1.6 CWLType](https://www.commonwl.org/v1.1/CommandLineTool.html#CWLType)


### File

[CWL 1.1 - 5.1.7 File](https://www.commonwl.org/v1.1/CommandLineTool.html#File)


#### Directory

[CWL 1.1 - 5.1.7.1 Directory](https://www.commonwl.org/v1.1/CommandLineTool.html#Directory)


### stdin

[CWL 1.1 - 5.1.8 stdin](https://www.commonwl.org/v1.1/CommandLineTool.html#stdin)


### CommandInputRecordSchema

[CWL 1.1 - 5.1.9 CommandInputRecordSchema](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandInputRecordSchema)


#### CommandInputRecordField

[CWL 1.1 - 5.1.9.1 CommandInputRecordField](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandInputRecordField)


##### CommandInputEnumSchema

[CWL 1.1 - 5.1.9.1.1 CommandInputEnumSchema](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandInputEnumSchema)


##### CommandInputArraySchema

[CWL 1.1 - 5.1.9.1.2 CommandInputArraySchema](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandInputArraySchema)


## CommandOutputParameter

[CWL 1.1 - 5.2 CommandOutputParameter](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandOutputParameter)


### stdout

[CWL 1.1 - 5.2.1 stdout](https://www.commonwl.org/v1.1/CommandLineTool.html#stdout)


### stderr

[CWL 1.1 - 5.2.2 stderr](https://www.commonwl.org/v1.1/CommandLineTool.html#stderr)


### CommandOutputRecordSchema

[CWL 1.1 - 5.2.3 CommandOutputRecordSchema](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandOutputRecordSchema)


### CommandOutputRecordField

[CWL 1.1 - 5.2.4 CommandOutputRecordField](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandOutputRecordField)


#### CommandOutputEnumSchema

[CWL 1.1 - 5.2.4.1 CommandOutputEnumSchema](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandOutputEnumSchema)


#### CommandOutputArraySchema

[CWL 1.1 - 5.2.4.2 CommandOutputArraySchema](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandOutputArraySchema)


#### CommandOutputBinding

[CWL 1.1 - 5.2.4.3 CommandOutputBinding](https://www.commonwl.org/v1.1/CommandLineTool.html#CommandOutputBinding)


## InlineJavascriptRequirement

[CWL 1.1 - 5.3 InlineJavascriptRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#InlineJavascriptRequirement)


## SchemaDefRequirement

[CWL 1.1 - 5.4 SchemaDefRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#SchemaDefRequirement)


## LoadListingRequirement

[CWL 1.1 - 5.5 LoadListingRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#LoadListingRequirement)


## DockerRequirement

[CWL 1.1 - 5.6 DockerRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#DockerRequirement)


## SoftwareRequirement

[CWL 1.1 - 5.7 SoftwareRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#SoftwareRequirement)


## SoftwarePackage

[CWL 1.1 - 5.8 SoftwarePackage](https://www.commonwl.org/v1.1/CommandLineTool.html#SoftwarePackage)


## InitialWorkDirRequirement

[CWL 1.1 - 5.9 InitialWorkDirRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#InitialWorkDirRequirement)


### Dirent

[CWL 1.1 - 5.9.1 Dirent](https://www.commonwl.org/v1.1/CommandLineTool.html#Dirent)


## EnvVarRequirement

[CWL 1.1 - 5.10 EnvVarRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#EnvVarRequirement)


## EnvironmentDef

[CWL 1.1 - 5.11 EnvironmentDef](https://www.commonwl.org/v1.1/CommandLineTool.html#EnvironmentDef)


## ShellCommandRequirement

[CWL 1.1 - 5.12 ShellCommandRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#ShellCommandRequirement)


## ResourceRequirement

[CWL 1.1 - 5.13 ResourceRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#ResourceRequirement)


## WorkReuse

[CWL 1.1 - 5.14 WorkReuse](https://www.commonwl.org/v1.1/CommandLineTool.html#WorkReuse)


## NetworkAccess

[CWL 1.1 - 5.15 NetworkAccess](https://www.commonwl.org/v1.1/CommandLineTool.html#NetworkAccess)


## InplaceUpdateRequirement

[CWL 1.1 - 5.16 InplaceUpdateRequirement](https://www.commonwl.org/v1.1/CommandLineTool.html#InplaceUpdateRequirement)


## ToolTimeLimit

[CWL 1.1 - 5.17 ToolTimeLimit](https://www.commonwl.org/v1.1/CommandLineTool.html#ToolTimeLimit)


## CWLVersion

[CWL 1.1 - 5.18 CWLVersion](https://www.commonwl.org/v1.1/CommandLineTool.html#CWLVersion)
