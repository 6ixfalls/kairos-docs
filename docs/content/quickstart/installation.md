+++
title = "Installation"
date = 2022-02-09T17:56:26+01:00
weight = 1
chapter = false
+++

## Introduction

For user convenience, Kairos releases ship a set of artifacts that can be used to install Kairos on a node. 
However, Kairos Kubernetes Native components allow the creation of these artifacts inside Kubernetes from a set of input images. 

In the quickstart below, we will use the artifacts generated by the Kairos releases, as we assume we don't have any prior Kubernetes cluster(s) available.

Most of the operations are completely automated, so there is little interaction needed with the Kubernetes components.

Installation can be also completely driven by the Kubernetes Native components if you already have a Kubernetes cluster to manage installations, with zero touch configuration.

The goal of this quickstart is to install Kairos on a node and create a single-node Kubernetes cluster with [k3s](https://k3s.io).

To simplify, we will install Kairos inside a virtual machine (VM); however, your mileage and configuration may vary based on your setup. See the documentation for a more exhaustive list of examples.


### Prerequisites

- A VM hypervisor that boots ISOs
- A Linux or a Windows machine where to run the `kairos` CLI (optional, we will see)
- A configuration file `cloud-init`

### Download

Kairos can be used to turn any distro in an immutable system; however, for user convenience, there are several artifacts published as part of the releases to get started.

You can find the latest releases in the [release page on GitHub](https://github.com/kairos-io/provider-kairos/releases). For instance, download the [kairos-opensuse-v1.0.0-k3sv1.24.3+k3s1.iso](https://github.com/kairos-io/provider-kairos/releases/download/v1.0.0/kairos-opensuse-v1.0.0-k3sv1.24.3+k3s1.iso)  ISO file to pick the openSUSE based version, where `v1.24.3+k3s1` in the name is the `k3s` version and `v1.0.0` is the Kairos one.

{{% notice note %}}

The releases in [kairos-io/provider-kairos](https://github.com/kairos-io/provider-kairos/releases) ships `k3s` and `p2p` full-mesh. These options need to be explicitly enabled. In follow-up releases, _k3s-only_ artifacts will also be available.

The releases in the [kairos-io/kairos](https://github.com/kairos-io/kairos/releases) repository are the Kairos core images that ship without K3s and P2P full-mesh functionalities; however further extensions can be installed dynamically in runtime by using the Kairos bundles mechanism.

{{% /notice %}}

### Booting up

Download the ISO, and boot it up on a VM running on the hypervisor of your choice. When deploying on a bare metal server, directly flash the image into a USB stick with DD:

```bash
dd if=/path/to/iso of=/path/to/dev bs=4MB
```

Another alternative is to use [Etcher](https://www.balena.io/etcher/).

You should be greeted with a GRUB boot menu:
Multiple entries are available. Choosing the appropriate entry depends on how you plan to install Kairos. For a standard installation, pick the first entry, or wait until the default timeout expires.
The machine will boot, and eventually a QR code will be printed out of the screen:

![livecd](https://user-images.githubusercontent.com/2420543/189219806-29b4deed-b4a1-4704-b558-7a60ae31caf2.gif)

### Configuration

At this stage the machine is waiting for the configuration to continue further with the installation process. Configuration can be either served via QR code, or manually via logging into the box and starting the installation process with a config file. The config file is a YAML file mixed with `cloud-init` syntax and the config of Kairos itself.

In this example, we will configure the node as a single-node Kubernetes cluster. So, we enable K3s, and we set a default password for the Kairos user to later access the box. We also need to define SSH keys:

```yaml
#node-config

stages:
   initramfs:
     - name: "User settings"
       hostname: "hostname.domain.tld"
       authorized_keys:
         kairos:
         - "ssh-rsa AAA..."
         - "github:mudler"
       users:
        kairos:
          passwd: "kairos"
       dns:
        path: /etc/resolv.conf
        nameservers:
        - 8.8.8.8

k3s:
  enabled: true
```

Save the config file as `config.yaml` as we will use it later in the process.

**Note**:
- The `stages.initramfs` block will configure the Kairos user (default) with the Kairos password. Note, the Kairos user is already configured with `sudo` permissions.
- `authorized_keys` can be used to add additional keys to the user in order to SSH into
- `hostname` sets the machine hostname
- `dns` sets the DNS for the machine
- `k3s.enabled=true` enables `k3s`. 

{{% notice note %}}

Several configurations can be added at this stage. [See the configuration reference](/reference/configuration) for further reading.

{{% /notice %}}

### Deployment

{{% notice note %}}

You can find instructions showing how to use the Kairos CLI below. In case you prefer to install via SSH and login into the box, see the [Manual installation](/installation/manual) section or the [Interactive installation](/installation/interactive) section to perform the installation manually from the console.

{{% /notice %}}

To trigger the installation process via QR code, you need to use the Kairos CLI. The CLI is currently available only for Linux and Windows. It can be downloaded from the release artifact:

```bash
curl -L https://github.com/kairos-io/provider-kairos/releases/download/v1.0.0/kairos-cli-v1.0.0-Linux-x86_64.tar.gz -o - | tar -xvzf - -C .
```

The CLI allows to register a node with a screenshot, an image, or a token. During pairing, the configuration is sent over, and the node will continue the installation process.

In a terminal window from your desktop/workstation, run:

```
kairos register --reboot --device /dev/sda --config config.yaml
```

**Note**:
- By default, the CLI will automatically take a screenshot to get the QR code. Make sure it fits into the screen. Alternatively, an image path or a token can be supplied via arguments (e.g. `kairos register /img/path` or `kairos register <token>`).
- The `--reboot` flag will make the node reboot automatically after the installation is completed.
- The `--device` flag determines the specific drive where Kairos will be installed. Replace `/dev/sda` with your drive. Any existing data will be overwritten, so please be cautious.
- The `--config` flag is used to specify the config file used by the installation process.

After a few minutes, the configuration is distributed to the node and the installation starts. At the end of the installation, the system is automatically rebooted.

### Accessing the Node

After the boot process, the node starts and is loaded into the system. You should already have SSH connectivity when the console is available.

To access to the host, log in as `kairos`:

```bash
ssh kairos@IP
```

**Note**:
- `sudo` permissions are configured for the Kairos user.

You will be greeted with a welcome message:

```
Welcome to Kairos!

Refer to https://kairos.io for documentation.
kairos@kairos:~>
```

At this point, it can take a few moments to get the K3s server running. However, you should be able to inspect the service and see K3s running. For example, with systemd-based flavors:

```
$ sudo systemctl status k3s
● k3s.service - Lightweight Kubernetes
     Loaded: loaded (/etc/systemd/system/k3s.service; enabled; vendor preset: disabled)
    Drop-In: /etc/systemd/system/k3s.service.d
             └─override.conf
     Active: active (running) since Thu 2022-09-01 12:02:39 CEST; 4 days ago
       Docs: https://k3s.io
   Main PID: 1834 (k3s-server)
      Tasks: 220
```

The K3s `kubeconfig` file is available at `/etc/rancher/k3s/k3s.yaml`. Please refer to the [K3s](https://rancher.com/docs/k3s/latest/en/) documentation.

## See Also

There are other ways to install Kairos:

- [Automated installation](/installation/automated)
- [Manual login and installation](/installation/manual)
- [Create decentralized clusters](/installation/p2p)
- [Take over installation](/installation/takeover)
- [Raspberry Pi](/installation/raspberry)
- [Netboot (TODO)]()
- [CAPI Lifecycle Management (TODO)]()

## What's Next?

- [Upgrade nodes with Kubernets](/upgrade/kubernetes)
- [Upgrade nodes manually](/upgrade/manual)
- [Immutable architecture](/architecture/immutable)
