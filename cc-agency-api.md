# CC-Agency API

This is a documentation of the available REST API endpoints of CC-Agency. 

**Before using** the CC-Agency API, you should read the [user information](cc-agency.md#user-information) and [data security](cc-agency.md#data-security) sections in the CC-Agency documentation.

Usage examples are provided as Python code. Install and import the following Python modules to prepare for the examples.

```bash
pip3 install --user --upgrade requests ruamel.yaml
```

```python
import requests
from ruamel.yaml import YAML
yaml = YAML(typ='safe')
```

## Table of Contents

| Endpoint | Methods |
| --- | --- |
| / | [GET](#get-) |
| /token | [GET](#get-token) |
| /version | [GET](#get-version) |
| /red | [POST](#post-red) |
| /experiments/count | [GET](#get-experimentscount) |
| /experiments | [GET](#get-experiments) |
| /experiments/EXPERIMENT_ID | [GET](#get-experimentsexperiment_id) |
| /batches/count | [GET](#get-batchescount) |
| /batches | [GET](#get-batches) |
| /batches/BATCH_ID | [GET](#get-batchesbatch_id), [DELETE](#delete-batchesbatch_id) |
| /nodes | [GET](#get-nodes) |

## GET /

Hello World.

Request (Python):

```python
requests.get(
    'http://localhost/'
)
```

Response (JSON):

```json
{"Hello": "World"}
```

## GET /token

Request temporary password token, which replaces the actual user password for basic auth.

*Hint: This endpoints requires authorization with username and password credentials, not another token.*

Request (Python):

```python
requests.get(
    'http://localhost/token',
    auth=('guest', 'guest')
)
```

Response (JSON):

```json
{
    "token": "d474a3d73a57e5636b070b03eee8ac2a0d8c302798ffd38f",
    "valid_for_seconds": 86400
}
```

Token can be used for authorization with other endpoints as follows.

Request (Python):

```python
requests.get(
    'http://localhost/version',
    auth=('guest', 'd474a3d73a57e5636b070b03eee8ac2a0d8c302798ffd38f')
)
```


## GET /version

Installed CC-Core and CC-Agency versions.

Request (Python):

```python
requests.get(
    'http://localhost/version',
    auth=('guest', 'guest')
)
```

Response (JSON):

```json
{
    "agencyVersion": "5.3.0",
    "coreVersion": "5.3.0"
}
```


## POST /red

Send RED data to CC-Agency, which will result in one database entry in the `experiments` collection and one or more entries in the `batches` collection. The agency's scheduler will trigger the parallel processing of the batches in the distributed cluster of docker-engines.

Additionally **required** fields:

* `outputs`
* `container.settings.image.url`
* `container.settings.image.ram`

File red.yml:

```yaml
redVersion: "5"
cli:
  cwlVersion: "v1.0"
  class: "CommandLineTool"
  baseCommand: "grepwrap"
  doc: "Search for query terms in text files."

  inputs:
    query_term:
      type: "string"
      inputBinding:
        position: 0
      doc: "Search for QUERY_TERM in TEXT_FILE."
    text_file:
      type: "File"
      inputBinding:
        position: 1
      doc: "TEXT_FILE containing plain text."
    after_context:
      type: "int?"
      inputBinding:
        prefix: "-A"
      doc: "Print NUM lines of trailing context after matching lines."
    before_context:
      type: "int?"
      inputBinding:
        prefix: "-B"
      doc: "Print NUM lines of leading context before matching lines."

  outputs:
    out_file:
      type: "File"
      outputBinding:
        glob: "out.txt"
      doc: "Query results."

inputs:
  query_term: "QU"
  text_file:
    class: "File"
    connector:
      pyModule: "cc_core.commons.connectors.http"
      pyClass: "Http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/vagrant-quickstart/master/in.txt"
        method: "GET"
  before_context: 1

outputs:
  out_file:
    class: "File"
    connector:
      pyModule: "cc_core.commons.connectors.http"
      pyClass: "Http"
      access:
        url: "http://172.17.0.1:5000/server-out.txt"
        method: "POST"

container:
  engine: "docker"
  settings:
    image:
      url: "grepwrap-image"
    ram: 256

execution:
  engine: "ccagency"
  settings:
    disablePull: True
    protectedKeys: ['username', 'url']
```

Request (Python):

```python
with open('red.yml') as f:
    red = yaml.load(f)

requests.post(
    'http://localhost/red',
    auth=(guest, guest),
    json=red
)
```

Response (JSON):

```json
{"experimentId": "5b7b30f2aafce97767a50a95"}
```


## GET /experiments/count

Count experiments stored in database.

*Hint: Users without admin priviliges will only receive information about their own database entries.*

URL parameters:

| Parameter | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| username | string | yes | | Filter by username. |

Request (Python):

```python
requests.get(
    'http://localhost/experiments/count',
    auth=(guest, guest)
)
```

Response (JSON):

```json
{"count": 1}
```


## GET /experiments

List experiments stored in database sorted by registration time in descending order.

*Hint: Users without admin priviliges will only receive information about their own database entries.*

URL parameters:

| Parameter | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| skip | integer >= 0 | yes | 0 |Skip a certain number of experiments. |
| limit | integer >= 1 | yes | | Limit the amount of experiments returned. |
| ascending | boolean | yes | False | Sort results by registrationTime in ascending order. Order is descending (newest first) by default. |
| username | string | yes | | Filter by username. |

Request (Python):

```python
requests.get(
    'http://localhost/experiments?ascending=true&limit=10',
    auth=(guest, guest)
)
```

Response (JSON):

TODO


## GET /experiments/EXPERIMENT_ID

Get full experiment stored in database by its `experimentId`.

*Hint: Users without admin priviliges will only receive information about their own database entries.*

Request (Python):

```python
requests.get(
    'http://localhost/experiments/5b7b2c98aafce967d9e2709b',
    auth=(guest, guest)
)
```

Response (JSON):

TODO


## GET /batches/count

Count batches stored in database.

*Hint: Users without admin priviliges will only receive information about their own database entries.*

URL parameters:

| Parameter | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| username | string | yes | | Filter by username. |
| node | string | yes | | Filter by node. |
| state | enum: registered, processing, succeeded, failed, cancelled | yes | | Filter by state. |
| experimentId | string | yes | | Filter by experimentId. |

Request (Python):

```python
requests.get(
    'http://localhost/batches/count',
    auth=(guest, guest)
)
```

Response (JSON):

```json
{"count": 1}
```


## GET /batches

List batches stored in database sorted by registration time in descending order.

*Hint: Users without admin priviliges will only receive information about their own database entries.*

URL parameters:

| Parameter | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| skip | integer >= 0 | yes | 0 |Skip a certain number of experiments. |
| limit | integer >= 1 | yes | | Limit the amount of experiments returned. |
| ascending | boolean | yes | False | Sort results by registrationTime in ascending order. Order is descending (newest first) by default. |
| username | string | yes | | Filter by username. |
| node | string | yes | | Filter by node. |
| state | enum: registered, processing, succeeded, failed, cancelled | yes | | Filter by state. |
| experimentId | string | yes | | Filter by experimentId. |

Request (Python):

```python
requests.get(
    'http://localhost/batches?ascending=true&limit=10',
    auth=(guest, guest)
)
```

Response (JSON):

TODO


## GET /batches/BATCH_ID

Get full batch stored in database by its `batchId`.

*Hint: Users without admin priviliges will only receive information about their own database entries.*

Request (Python):

```python
requests.get(
    'http://localhost/batches/5b7b2c98aafce967d9e2709c',
    auth=(guest, guest)
)
```

Response (JSON):

TODO


## DELETE /batches/BATCH_ID

Cancel unfinished batch by its `batchId`. This will not delete their database entries.

*Hint: Users without admin priviliges can only cancel their own batches.*

Request (Python):

```python
requests.delete(
    'http://localhost/batches/5b7b2c98aafce967d9e2709c',
    auth=(guest, guest)
)
```

Response (JSON):

TODO


## GET /nodes

List docker-engine nodes available in the cluster.

Request (Python):

```python
requests.get(
    'http://localhost/nodes',
    auth=(guest, guest)
)
```
