---
title: "RED-CWL 0 Specification"
permalink: /docs/red-cwl-0
---


RED-CWL 0 is a subset of the [CWL 1.0 CommandLineTool Description](https://www.commonwl.org/v1.0/CommandLineTool.html).
RED-CWL 0 is a pre-release version of [RED-CWL 1](/docs/red-cwl-1) and reflects the current implementation status in RED 8.

The CWL reference implementation `cwltool` requires a `*.cwl` file as generic description of CommandLineTool and a `*.yml` job file to define an input object, that contains actual parameters to run the experiment.
In contrast, a RED file embeds the contents of a `*.cwl` file under the `cli` keyword and embeds the input object under the `inputs` keyword.
Depending on the RED execution engine, defining an output object might be optional and can be embedded under the `outputs` keyword.

The RED-CWL description can be extracted from a RED file's `cli` section and stored in a `*.cwl` file.
Since RED-CWL is a subset of the CWL standard, this extracted description should always be compatible with CWL runtimes like `cwltool`.
On the other hand, not every CWL description can be embedded into a RED file without modification.

This spec refers to individial sections of the [CWL 1.0 CommandLineTool Description](https://www.commonwl.org/v1.0/CommandLineTool.html) to define wether a certain part of the standard is supported or not.


# Data model

## Document preprocessing

[CWL 1.0 - 2.4 Document preprocessing](https://www.commonwl.org/v1.0/CommandLineTool.html#Document_preprocessing)

Optional inputs and outputs can be defined as `<T>?`. Input arrays can be defined as `<T>[]`. Optional input arrays can be defined as `<T>[]?`.
`<T>` refers to the input type. Output arrays are not allowed.


# Running a Command

## Runtime Environment

[CWL 1.0 - 4.2 Document preprocessing](https://www.commonwl.org/v1.0/CommandLineTool.html#Runtime_environment)

Setting `runtime` parameters like `runtime.ram` is not supported.
In the context of RED, this information is part of the `container` section, because the container engine enforces these settings.

In addtion, the resource requirements of a CLI tool are in many cases influenced by the input data size.
This makes resource requirements more of an experiment setting than a CLI tool setting.


# CommandLineTool

[CWL 1.0 - 5 CommandLineTool](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandLineTool)

| Field | Supported | Description |
| --- | --- | --- |
| inputs | partly | only <code>map<id, type &#124; CommandInputParameter></code> |
| outputs | partly | only <code>map<id, type &#124; CommandInputParameter></code> |
| class | yes |  |
| id | no |  |
| requirements | no | |
| hints | no | |
| label | no | |
| doc | yes | |
| cwlVersion | yes | |
| baseCommand | yes | |
| arguments | no | |
| stdin | no | |
| stdout | partly | only `string` |
| stderr | partly | only `string` |
| successCodes | no | |
| temporaryFailCodes | no | |
| permanentFailCodes | no | |


## CommandInputParameter

[CWL 1.0 - 5.1 CommandInputParameter](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandInputParameter)

| Field | Supported | Description |
| --- | --- | --- |
| id | no | because `inputs` only supports <code>map<id, type &#124; CommandInputParameter></code> |
| label | no | |
| secondaryFiles | no | |
| streamable | no | |
| doc | yes | |
| format | no | |
| inputBinding | yes | |
| default | no | |
| type | partly | only `string`, `CWLType` |


### Expression

[CWL 1.0 - 5.1.1 Expression](https://www.commonwl.org/v1.0/CommandLineTool.html#Expression)

| Symbol | Supported | Description |
| --- | --- | --- |
| ExpressionPlaceholder | partly | only in RED `outputs` to reference `inputs` |


### CommandLineBinding

[CWL 1.0 - 5.1.2 CommandLineBinding](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandLineBinding)


| Field | Supported | Description |
| --- | --- | --- |
| loadContents | no | |
| position | yes | |
| prefix | yes | |
| separate | yes | |
| itemSeparator | yes | |
| valueFrom | no | |
| shellQuote | no | |


### Any

[CWL 1.0 - 5.1.3 Any](https://www.commonwl.org/v1.0/CommandLineTool.html#Any)

Not supported.


### CWLType

[CWL 1.0 - 5.1.4 CWLType](https://www.commonwl.org/v1.0/CommandLineTool.html#CWLType)

| Symbol | Supported | Description |
| --- | --- | --- |
| null | no | |
| boolean | yes | |
| int | yes | |
| long | yes | |
| float | yes | |
| double | yes | |
| string | yes | |
| File | yes | |
| Directory | yes | |


### File

[CWL 1.0 - 5.1.5 File](https://www.commonwl.org/v1.0/CommandLineTool.html#File)

Supported in RED `inputs` section.

| Field | Supported | Description |
| --- | --- | --- |
| class | yes | |
| location | no | because RED introduces the `connector` field, that replaces `location` |
| path | auto | automatically set by execution engine |
| basename | yes | |
| dirname | yes | |
| nameroot | auto | automatically set by execution engine |
| nameext | auto | automatically set by execution engine |
| checksum | yes | |
| size | yes | |
| secondaryFiles | no | |
| format | no | |
| contents | no | |


#### Directory

[CWL 1.0 - 5.1.5.1 Directory](https://www.commonwl.org/v1.0/CommandLineTool.html#Directory)

Supported in RED `inputs` section.

| Field | Supported | Description |
| --- | --- | --- |
| class | yes | |
| location | no | because RED introduces the `connector` field, that replaces `location` |
| path | auto | automatically set by execution engine |
| basename | yes | |
| listing | yes | |


### CommandInputRecordSchema

[CWL 1.0 - 5.1.6 CommandInputRecordSchema](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandInputRecordSchema)

Not supported.


### CommandInputRecordField

[CWL 1.0 - 5.1.7 CommandInputRecordField](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandInputRecordField)

Not supported.


### CommandInputRecordSchema

[CWL 1.0 - 5.1.6 CommandInputRecordSchema](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandInputRecordSchema)

Not supported.


### CommandInputRecordField

[CWL 1.0 - 5.1.7 CommandInputRecordField](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandInputRecordField)

Not supported.


#### CommandInputEnumSchema

[CWL 1.0 - 5.1.7.1 CommandInputEnumSchema](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandInputEnumSchema)

Not supported.


#### CommandInputArraySchema

[CWL 1.0 - 5.1.7.2 CommandInputArraySchema](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandInputArraySchema)

https://www.commonwl.org/v1.0/CommandLineTool.html#Document_preprocessing

Not supported, but arrays can be specified using `type: <T>[]` as described in [CWL 1.0 - 2.4 Document preprocessing](https://www.commonwl.org/v1.0/CommandLineTool.html#Document_preprocessing).


## CommandOutputParameter

[CWL 1.0 - 5.2 CommandOutputParameter](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandOutputParameter)


| Field | Supported | Description |
| --- | --- | --- |
| id | no | because `inputs` only supports <code>map<id, type &#124; CommandInputParameter></code> |
| label | no | |
| secondaryFiles | no | |
| streamable | no | |
| doc | yes | |
| outputBinding | yes | |
| format | no | |
| type | partly | only `string`, `CWLType`, where `CWLType` is one of `File`, `Directory` |


### stdout

[CWL 1.0 - 5.2.1 stdout](https://www.commonwl.org/v1.0/CommandLineTool.html#stdout)

| Symbol | Supported | Description |
| --- | --- | --- |
| stdout | yes | |


### stderr

[CWL 1.0 - 5.2.2 stderr](https://www.commonwl.org/v1.0/CommandLineTool.html#stderr)

| Symbol | Supported | Description |
| --- | --- | --- |
| stderr | yes | |


### CommandOutputBinding

[CWL 1.0 - 5.2.3 CommandOutputBinding](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandOutputBinding)

| Field | Supported | Description |
| --- | --- | --- |
| glob | yes | |
| loadContents | no | |
| outputEval | no | |


### CommandOutputRecordSchema

[CWL 1.0 - 5.2.4 CommandOutputRecordSchema](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandOutputRecordSchema)

Not supported.


### CommandOutputRecordField

[CWL 1.0 - 5.2.5 CommandOutputRecordField](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandOutputRecordField)

Not supported.


#### CommandOutputEnumSchema

[CWL 1.0 - 5.2.5.1 CommandOutputEnumSchema](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandOutputEnumSchema)

Not supported.


#### CommandOutputArraySchema

[CWL 1.0 - 5.2.5.2 CommandOutputArraySchema](https://www.commonwl.org/v1.0/CommandLineTool.html#CommandOutputArraySchema)

Not supported.


## InlineJavascriptRequirement

[CWL 1.0 - 5.3 InlineJavascriptRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#InlineJavascriptRequirement)

Not supported.


## SchemaDefRequirement

[CWL 1.0 - 5.4 SchemaDefRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#SchemaDefRequirement)

Not supported.


## DockerRequirement

[CWL 1.0 - 5.5 DockerRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#DockerRequirement)

Not supported, because container engine settings must be defined in the `container` section of a RED file.


## SoftwareRequirement

[CWL 1.0 - 5.6 SoftwareRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#SoftwareRequirement)

Not supported.


## SoftwarePackage

[CWL 1.0 - 5.7 SoftwarePackage](https://www.commonwl.org/v1.0/CommandLineTool.html#SoftwarePackage)

Not supported.


## InitialWorkDirRequirement

[CWL 1.0 - 5.8 InitialWorkDirRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#InitialWorkDirRequirement)

Not supported.


## EnvVarRequirement

[CWL 1.0 - 5.9 EnvVarRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#EnvVarRequirement)

Not supported.


## EnvironmentDef

[CWL 1.0 - 5.10 EnvironmentDef](https://www.commonwl.org/v1.0/CommandLineTool.html#EnvironmentDef)

Not supported.


## ShellCommandRequirement

[CWL 1.0 - 5.11 ShellCommandRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#ShellCommandRequirement)

Not supported.


## ResourceRequirement

[CWL 1.0 - 5.12 ResourceRequirement](https://www.commonwl.org/v1.0/CommandLineTool.html#ResourceRequirement)

Not supported.


## CWLVersion

[CWL 1.0 - 5.13 CWLVersion](https://www.commonwl.org/v1.0/CommandLineTool.html#CWLVersion)


| Field | Supported | Description |
| --- | --- | --- |
| draft-2 | no | |
| draft-3.dev1 | no | |
| draft-3.dev2 | no | |
| draft-3.dev3 | no | |
| draft-3.dev4 | no | |
| draft-3.dev5 | no | |
| draft-3 | no | |
| draft-4.dev1 | no | |
| draft-4.dev2 | no | |
| draft-4.dev3 | no | |
| v1.0.dev4 | no | |
| v1.0 | yes | |
