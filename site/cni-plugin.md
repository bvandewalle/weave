---
title: Integrating Kubernetes via the CNI Plugin
menu_order: 65
---

CNI, the [_Container Network Interface_](https://github.com/appc/cni#cni---the-container-network-interface),
is a proposed standard for configuring network interfaces for Linux
application containers.  CNI is supported by
[Kubernetes](http://kubernetes.io/), an open source container cluster
manager built by Google.

###Installing the Weave Net CNI plugin

The Weave Net CNI plugin is installed when you run `weave setup`, if
your machine has the directories normally used to host CNI plugins.

To create those directories, run (as root):

    mkdir -p /opt/cni/bin
    mkdir -p /etc/cni/net.d

Then run:

    weave setup

###Launching Weave Net

To create a network which can span multiple hosts, the Weave peers must be connected to each other, by specifying the other hosts during `weave launch` or via
[`weave connect`](/site/using-weave/finding-adding-hosts-dynamically.md).

See [Using Weave Net](/site/using-weave.md#peer-connections) for a discussion on peer connections. 

    weave launch <peer hosts>

Then you have to run an extra command to perform some additional
configuration required by Kubernetes:

    weave expose

This assigns an IP address to the Weave bridge and
[enables access to containers from the host](/site/using-weave/host-network-integration.md).

###Configuring Kubernetes to use the CNI Plugin

After you've launched Weave and peered your hosts, you can configure
Kubernetes to use Weave, by adding the following options to the
`kubelet` command:

    --network-plugin=cni --network-plugin-dir=/etc/cni/net.d

See the [`kubelet` documentation](http://kubernetes.io/v1.1/docs/admin/kubelet.html)
for more details.

Now, whenever Kubernetes starts a pod, it will be attached to the Weave network.

###Using the CNI network configuration file

All CNI plugins are configured by a JSON file in the directory
`/etc/cni/net.d/`.  `weave setup` installs a minimal configuration
file named `10-weave.conf`, which you can alter to suit your needs.

See the [CNI Spec](https://github.com/appc/cni/blob/master/SPEC.md#network-configuration)
for details on the format and contents of this file.

By default, the Weave CNI plugin adds a default route out via the Weave bridge, so your containers can access resources on the internet.  If you do not want this, add a section to the config file that specifies no routes:

    "ipam": {
        "routes": [ ]
    }

The following other fields in the spec are supported:

- `ipam / type` - default is to use Weave's own IPAM
- `ipam / subnet` - default is to use Weave's IPAM default subnet
- `ipam / gateway` - default is to use the Weave bridge IP address (allocated by `weave expose`)

###Caveats

- The Weave Net router container must be running for CNI to allocate addresses
- The CNI plugin does not add entries to weaveDNS.
