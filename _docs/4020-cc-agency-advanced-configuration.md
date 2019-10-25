---
title: "CC-Agency: Advanced Configuration"
permalink: /docs/cc-agency-advanced-configuration
---


# GPU Blacklist

Starting with version 8.1.0, CC-Agency will automatically detect available Nvidia GPUs.
You can exclude certain GPUs from being used by CC-Agency by defining a blacklist.

```yaml
controller:
  docker:
    nodes:
      gpu_node1:
        base_url: "tcp://192.168.0.101:2376"
        tls:
          verify: "/home/cc/.docker/machine/machines/gpu_node1/ca.pem"
          client_cert:
            - "/home/cc/.docker/machine/machines/gpu_node1/cert.pem"
            - "/home/cc/.docker/machine/machines/gpu_node1/key.pem"
          assert_hostname: False
        hardware:
          gpu_blacklist:
            - 0
            - 1
```

Here `0` and `1` are the GPU IDs as shown by the [/nodes](/docs/cc-agency-api#get-nodes) endpoint of the CC-Agency REST interface.


# Notification Hooks

To send HTTP notifications if a batch has entered a final state (`succeeded`, `failed` or `cancelled`), you can configure notification hooks in the agency configuration as follows.

```yaml
controller:
  notification_hooks:
    - url: "http://example.com/notify"
    - url: "http://example.com/auth-notify"
      auth:
        username: "username"
        password: "password"
```

This configuration will result in a HTTP Post to each url, as soon as a batch state changes. The `auth` field is optional.

The content of a notification is a list of batch information, specifying the `batchId` and the new `state` of the batch:


**Example:**

```json
{
  "batches": [
    {
      "batchId": "5c6sb537h4d0k4353ch27c",
      "state": "succeeded"
    }
  ]
}
```
