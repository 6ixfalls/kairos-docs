+++
title = "Full P2P mesh support"
date = 2022-02-09T17:56:26+01:00
weight = 4
chapter = false
pre = "<b>- </b>"
+++

{{% notice note %}}
This feature is crazy and experimental!
{{% /notice %}}

This section will guide on how to leverage the peer-to-peer (P2P) full-mesh capabilities of Kairos.

Kairos supports P2P full-mesh out of the box. That allows to seamlessly interconnect clusters and nodes from different regions into a unified overlay network, additionally, the same network is used for coordinating nodes automatically allowing self-automated, node bootstrap.

A hybrid network is automatically set up between all the nodes, as such there is no need to expose them over the Internet or expose the Kubernetes management API outside, reducing attacker's exploiting surface.

Kairos can be configured to automatically bootstrap a Kubernetes cluster with the full-mesh functionalities or include an additional interface to the machines to let them communicate within a new network segment.

If you are not familiar with the process, it is suggested to follow the [quickstart](/quickstart/installation) first and the steps below in sequence.
The section below explains the difference in the configuration options to enable P2P full-mesh during the installation phase.

## Prerequisites

- Kairos CLI

## Configuration

To configure a node to join over the same P2P network during installation, add a `kairos` block in the configuration, like the following:

```yaml
kairos:
  network_token: "...."
  # Optionally, set a network id (for multiple clusters in the same network)
  # network_id: "dev"
  # Optionally set a role
  # role: "master"

```

The `kairos` block is used to configure settings to the mesh functionalities. The minimum required argument is the `network_token`.

### `network_token`

The `network_token` is a unique, shared secret which is spread over the nodes and can be generated with the Kairos CLI.
It will make all the node connect automatically to the same network. Every node will generate a set of private/public key pair automatically on boot that are used to communicate securely within an end-to-end encryption (E2EE) channel.

To generate a new network token, you can use the Kairos CLI:

```bash
kairos generate-token
```

### `network_id`

This is an optional, unique identifier for the cluster. It allows bootstrapping of multiple cluster over the same underlying network.

### `role`

Force a role for the node. Available: `worker`, `master`.

For a full reference of all the supported use cases, see [cloud-init](https://rancher.github.io/elemental-toolkit/docs/reference/cloud_init/).

## Join new nodes

To join new nodes, reapply the process to new nodes by specifying the same `config.yaml` for all the machines. Unless you have specified a role for each of the nodes, the configuration doesn't need any further change. The machines will connect automatically between themselves, either remotely on local network.

## Connect to the nodes

The Kairos CLI can be used to establish a tunnel with the nodes network given a `network_token`.


```bash
sudo kairos bridge --network-token <TOKEN>
```

This command will create a TUN device in your machine and will make possible to contact each node in the cluster.


{{% notice note %}}
The command requires root permissions in order to create a tun/tap device on the host
{{% /notice %}}

An API will be also available at [localhost:8080](http://localhost:8080) for inspecting the network status.


## Get kubeconfig

To retrieve `kubeconfig`, it is sufficient to either log in to the master node and get it from the engine (e.g., K3s places it `/etc/rancher/k3s/k3s.yaml`) or use the Kairos CLI.

By using the CLI, you need to be connected to the bridge or be logged in from one of the nodes and perform the commands in the console.

If you are using the CLI, you need to run the bridge in a separate window.

To retrieve `kubeconfig`, run the following:

```bash
kairos get-kubeconfig > kubeconfig
```

{{% notice note %}}
`kairos bridge` acts like `kubectl proxy`. You need to keep it open to operate the Kubernetes cluster and access the API.
{{% /notice %}}