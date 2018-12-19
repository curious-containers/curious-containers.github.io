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

This configuration means that two GPUs are present on a node "gpu\_node1".
Each GPU has 1024 MB VRAM.
Currently only Nvidia-GPUs are supported. To make the GPUs accessible for docker, [Nvidia-Docker](https://github.com/NVIDIA/nvidia-docker) has to be installed on each GPU node.

The IDs shown in the configuration, are the nvidia-device-IDs, which can be identified with `nvidia-smi` for example
(see [nvidia-smi](https://developer.nvidia.com/nvidia-system-management-interface)).
