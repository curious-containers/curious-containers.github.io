---
title: "CC-Agency: Advanced Configuration"
permalink: /docs/cc-agency-advanced-configuration
---


## GPU Nodes

If Nvidia GPUs are available on a node, they can be configured as follows.

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
          gpus:
            - id: 0
              vram: 1024
            - id: 1
              vram: 1024
```

This configuration means that two GPUs are present on a node "gpu\_node1". Each GPU has 1024 MB VRAM.
Currently only Nvidia-GPUs are supported. To make the GPUs accessible for docker, you have to install the proprietary Nvidia GPU driver and the [Nvidia Container Toolkit](https://github.com/NVIDIA/nvidia-docker) or its predecessor [Nvidia-Docker 2](https://github.com/NVIDIA/nvidia-docker) on each GPU node.

The IDs shown in the configuration, are the nvidia-device-IDs, which can be identified with `nvidia-smi` (see [nvidia-smi](https://developer.nvidia.com/nvidia-system-management-interface)).


## Notification Hooks

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
