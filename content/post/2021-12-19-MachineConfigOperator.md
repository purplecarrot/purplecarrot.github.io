---
title: "Getting Along with the OpenShift Machine Config Operator"
date: 2021-12-19T11:11:45Z'
description: "Understanding the OpenShift Machine Config Operator, and fixing it when things go wrong."
featured: true 
draft: true
toc: true
thumbnail: images/thumbnails/openshift.png 
categories:
  - Containers
tags:
  - Kubernetes
  - OpenShift
  - Linux
---

When I'm working with the Machine Config Operator (MCO) in OpenShift, and frequently typing `oc describe mcp` and `oc get mcp`, I'm often reminded of the MCP (Machine Control Program) in the film [Tron](https://en.wikipedia.org/wiki/Tron). I like the idea behind the MCO, and when it's working it's great, but perhaps this association in my mind is not only because of the shared acronym, but because you do sometimes feel you're fighting with it for control of your cluster!

## What is the OpenShift Machine Config Operator?

The [OpenShift Machine Config Operator](https://github.com/openshift/machine-config-operator) is a very powerful piece of software and an important part of the OpenShift Container Platform (OCP). It manages the Operating System versions, updates and configuration files of the master and worker nodes that make up an OpenShift cluster. Operating system configuration files such as NetworkManager config files, systemd units, sysctl files as well as kubelet config files and certificate bundles, are base64 encoded and held in `MachineConfig` resources inside Kubernetes itself. 

This might not sound that interesting in the context of today's public cloud service providers IaaS solutions, but when the nodes are running on HP and Dell bare metal physical servers in private disconnected data centres, having a software operator like this - which is run and managed using native Kubernetes primitives and commands - drastically reduces the SA workload required to manage and maintain multiple clusters made up of hundreds of bare metal physical servers


## Understanding the Machine Config Operator (MCO)
The MCO lives in the *openshift-machine-config-operator* project and is present in every OpenShift cluster. There are a number of different containers running in this namespace:

  * **machine-config-operator** - this is the main controller loop or ClusterOperator itself that deploys and manages everything else in this namespace.
  * **machine-config-server** - provides the endpoint that nodes connect to in order to get their configuration files.
  * **machine-config-controller** - co-ordinates upgrades and manages the lifecycle of nodes in the cluster by rendering (generating) `MachineConfigs (mc)`, managing `MachineConfigPools (mcp)` and by co-ordinating with the *machine-config-daemon* running on each node to keep all the nodes up to date with the correct configuration.
  * **machine-config-daemon** - responsible for updating nodes to a given `MachineConfig` when requested to by the machine-config-controller. This runs as a DaemonSet so there is one pod per node in the cluster.

The terminal output below illustrates these pods running in the *openshift-machine-config-operator* project in an OpenShift cluster:
```
$ oc get pods -n openshift-machine-config-operator -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP              NODE       NOMINATED NODE   READINESS GATES
machine-config-controller-7c4975f766-gbnh2   1/1     Running   1          20h   10.129.0.27     server01   <none>           <none>
machine-config-daemon-2kt5g                  2/2     Running   2          18h   192.162.1.203   server03   <none>           <none>
machine-config-daemon-jmhll                  2/2     Running   6          54d   192.168.1.206   server06   <none>           <none>
machine-config-daemon-rzwhv                  2/2     Running   6          54d   192.168.1.202   server02   <none>           <none>
machine-config-daemon-vqphs                  2/2     Running   24         54d   192.168.1.201   server01   <none>           <none>
machine-config-daemon-xqjgk                  2/2     Running   2          17h   192.168.1.205   server05   <none>           <none>
machine-config-operator-66f4cd998f-v8cfg     1/1     Running   1          20h   10.129.0.40     server01   <none>           <none>
machine-config-server-5hlsd                  1/1     Running   12         54d   192.168.1.101   server01   <none>           <none>
machine-config-server-dfvg8                  1/1     Running   3          54d   192.168.1.102   server02   <none>           <none>
machine-config-server-tr2fd                  1/1     Running   11         54d   192.168.1.103   server03   <none>           <none>
```

## MachineConfigs

So how does an update or change to a systemd unit file used to start the kubelet on a node make it to the OS disk of a node that requires it? 

One of the core utilities underpinning the MCO is [Ignition](https:/github.com/coreos/ignition). When RedHat acquired the CoreOS company in 2018, they merged the best parts of CoreOS, Red Hat Linux and Red Hat Atomic operating systems and the resulting product was named Red Hat CoreOS (RHCOS). This is the OS that runs on the master and worker nodes in an OpenShift cluster.

All the OS configuration files a node requires are encoded and encapsulated in ignition files. The *machine-config-controller* collates all these various snippets and OS configuration files each node requires, and then renders (generates) `MachineConfig` kubernetes resources with all the files.

The command below, shows the `MachineConfigs` available in an OpenShift cluster:

```
$ oc get mc --sort-by='{.metadata.creationTimestamp}'
NAME                                               GENERATEDBYCONTROLLER                      IGNITIONVERSION   AGE
99-worker-ssh                                                                                 3.2.0             216d
99-master-ssh                                                                                 3.2.0             216d
00-master                                          a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             216d
01-master-kubelet                                  a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             216d
01-worker-container-runtime                        a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             216d
01-master-container-runtime                        a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             216d
99-master-generated-registries                     a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             216d
00-worker                                          a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             216d
99-worker-generated-registries                     a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             216d
01-worker-kubelet                                  a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             216d
rendered-master-0a540fd2eb85335cab8066f0bb407d5b   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             216d
rendered-worker-9da78a560e7fa3c0cbd53d2bf0c17941   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             216d
rendered-master-248f9034db99de75517a88a2ae8d29e2   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             215d
rendered-worker-e323d4623ccb8cb71a4953f4ca81f8a4   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             215d
rendered-worker-5599b4631efd88b5834f0844bec5ce83   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             203d
rendered-master-8e5868011cc53d9697ac9dda69a64b39   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             203d
rendered-worker-77e79ef013b565633a4d04da28f7dda4   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             203d
rendered-master-c40185b01423956d1146b7dea0a3e265   116603ff3d7a39c0de52d7d16fe307c8471330a0   3.2.0             203d
rendered-worker-dee67e9dc2ba77876de8bc8d045740dd   c76785d62cc28ddf3390c865c9e999a02248cd84   3.2.0             57d
rendered-master-3fcbbc2dcbddc79435994c98b655b192   c76785d62cc28ddf3390c865c9e999a02248cd84   3.2.0             57d
rendered-worker-7373393797c4c7e5ae500ff51ba234a9   a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             55d
rendered-master-40ad8d11cc05c7079ee8f3bfa6df5540   a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             55d
rendered-worker-53656729c24a74935907c4fba2adc90a   a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             54d
rendered-master-1e2f18449f36cad8c9618c264d5cdc10   a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             54d
rendered-worker-82df13b281ba3ddb9ba2e8811ef7957c   a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             14d
rendered-master-7ed9a47c85ff478b341adaa20a55e942   a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             14d
rendered-master-98968a4a639e3efad0436d19d4a6c93f   a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             14d
rendered-worker-65ec18c5a3e4ebbf5f8ef459fb3c50b2   a537783ea4a0cd3b4fe2a02626ab27887307ea51   3.2.0             14d
```

In this command output, you can see that 216 days ago when the cluster was first installed, initial `MachineConfigs` were rendered - one for the master nodes (rendered-master-0a540f..) in the cluster, and one for the worker nodes (rendered-worker-9da78a...). Whilst these two `MachineConfigs` share common OS configuration files, there are also lots of differences and so they are managed separately for each `MachineConfigPool`.

Since the initial install, we can see at various intervals, new `MachineConfig` objects were rendered by the *machine-config-controller*. Each new rendered- `MachineConfig` is from when upgrades were made to the OpenShift cluster (because an OpenShift update may have updated an OS setting in say `/etc/crio/crio.conf`) 

As well as OCP version updates forcing updates, you can also manually edit `MachineConfigs` to modify or add you own OS config files. For what it's worth, I did have to do this with early OpenShift 4.1/4.2 versions, where I modified NTP and SSHD config files for custom settings and to modify the ca-bundle.crt as a workaround as early versions had issues with private CAs. But since those early versions, I have not had need to make any changes - the MCO just manages the complete OS lifecycle of the node.

When a new `MachineConfig` is generated, the *machine-config-controller* will work with the *machine-config-daemon* running on each node to reboot it and have it apply the latest ignition configuration held within the `MachineConfig`.

When a node boots Red Hat CoreOS, *ignition* connects to the *machine-config-server* endpoint `https://<cluster-api>:22623/config/master` (or `/config/worker` for worker nodes, or `/config/<machine-config-pool>`) and applies the *Ignition* config to the node.

## Quick Health Check of MCO

Most of the time, the MCO works very well. However, as with any highly automated process, on occasions it does go wrong. The first thing to be aware of with MCO is that you have to keep a close eye on it, particularly when it's rolling out updates. At a high level, the quickest ways to do this is by looking at status of the the ClusterOperator's status:

```
$ oc get co machine-config
NAME             VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
machine-config   4.8.10    True        False         False       14d
```

Here we can see that the MCO is **AVAILABLE=True**. This is good, and you nearly always want to see this. If it's **False**, something is very wrong with the cluster, or more likely perhaps, the cluster is in the middle of an OCP upgrade and the MCO pod is restarting on a different node.

If **PROGRESSING=False** is shown, that means that the MCO has no updates to rollout, and that the nodes in your cluster are all up to date and running on the latest OS version and using all the OS config files from the `MachineConfig`. If this is set to **False**, then that means the *machine-config-controller* is busy working through an update and asking the *machine-config-daemons* on each node to reboot the node and apply a `MachineConfig`.

For a healthy MCO, you also want to see **DEGRADED=False**. If you see this as **True**, then your MCO has a problem that usually requires manual intervention, and some of the key reasons that I've seen for this condition are documented below.

As well as looking at the overview status of the `ClusterOperator` resource, another way you can see the health of the MCO at a glance is to look at its `MachineConfigPools`. In a healthy cluster, these pools will show **UPDATED=True** and **DEGRADED=False**. If **UPDATING=True** is set, the *machine-config-controller* is in the middle of rolling out an update:

```
$ oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master   rendered-master-98968a4a639e3efad0436d19d4a6c93f   True      False      False      3              3                   3                     0                      216d
worker   rendered-worker-53656729c24a74935907c4fba2adc90a   True      False      False      3              3                   3                     0                      216d
```

## Monitoring an MCO Update

If your MCO is updating (**PROGRESSING=True**, **UPDATING=True**), then you can monitor its progress by watching the logs of the *machine-config-controller* pod. 

```
$ oc logs -l k8s-app=machine-config-controller -n openshift-machine-config-operator
```
In the logs from this pod, you'll see nodes in the cluster sequentially being cordoned (made `schedulable=False`) and then rebooted. Whilst rebooting and applying the new configuration, they will be seen as `NotReady` status in `oc get nodes`).

```
$ oc get nodes
NAME       STATUS                        ROLES           AGE    VERSION
server01   Ready                         master,worker   217d   v1.21.1+9807387
server02   NotReady,SchedulingDisabled   master,worker   217d   v1.21.1+9807387
server03   Ready                         master,worker   217d   v1.21.1+9807387
server04   NotReady,SchedulingDisabled   worker          203d   v1.21.1+9807387
server05   Ready                         worker          203d   v1.21.1+9807387
```

If something happens when the node reboots (eg it fails to DHCP, it can't boot because of a disk or grub error, fails to start the kubelet), the MCO rollout process will halt. You'll notice this because the node stays `NotReady` for longer than you'd expect and the status will become *degraded*. If this happens, you have to roll your sleeves up and work out why.

## Troubleshooting Degraded Nodes

The first place to look when troubleshooting a *degraded* node is its machine-config-daemon pod. In early OpenShift 4.x versions, there were also occasions when the reason for a degraded node was only apparent by looking in the Linux journald logs on the node itself. However in recent versions, I've not had to resort to journald logs because everything about an error has been available in the *machine-config-daemon* pod logs for the node. To find the correct *machine-config-daemon* pod for the node, you simply run `oc get pods -o wide`, or alternatively you could cut and paste the following:

```
$ MCDPOD=$(oc get pods -l k8s-app=machine-config-daemon -o jsonpath='{.items[?(@.spec.nodeName=="server01")].metadata.name')
$ oc logs $MCDPOD -c machine-config-daemon
```

Next, lets look at a few common reasons for a node being degraded.

### Unexpected on-disk state - content mismatch for file

If the files on a node's disk that are under control of the `MachineConfig` don't match the contents of those same files as specified in that `MachineConfig` (ie the `MachineConfig` specified in the node's `machineconfiguration.openshift.io/currentConfig` annotation), then the *machine-config-daemon* will refuse to apply the update and this will put the `MachineConfigPool` into a degraded state. You'll see a message event similar to this:

```
Type:                  RenderDegraded
Message:               Node server01 is reporting: "unexpected on-disk state validating against rendered-worker-82df13b281ba3ddb9ba2e8811ef7957c: content mismatch for file \"/etc/kubernetes/kubelet.conf\""
Reason:                1 nodes are reporting degraded status on sync
Status:                True
Type:                  NodeDegraded
```

This happens when somebody has manually edited a file on a node. Although remember, there should be no reason or need for a sysadmin (SA) to edit files on RHCOS, but sometimes junior SAs or SAs not familiar with OpenShift might be tempted to edit OS config files using ssh/vim.

You can get an indication of this by looking at the `machineconfiguration.openshift.io/ssh` annotation on the node. If this is set to accessed, then a SA has ssh'd onto the node and could have perhaps changed something manually. Ordinarily (except see troubleshooting below), SA shouldn't be ssh'ing onto RHCOS nodes.

The easiest way to quickly fix this is to restore the config file from the `MachineConfig`. To do this, output the correct `MachineConfig` in full YAML, and then search for the path entry for the file being reported as having a content mismatch. Alternatively, copy and run the following commands to get the file contents:

```
$ CURRENT_CONFIG=$(oc get node -o jsonpath="{.items[?(@.metadata.name=='server01')].metadata.annotations['machineconfiguration\.openshift\.io/currentConfig']}")
$ oc get mc $CURRENT_CONFIG -o jsonpath="{.spec.config.storage.files[?(@.path=='/etc/kubernetes/kubelet.conf)].contents.source}" > newkubelet.conf
```

Then you simply base64 decode (or urldecode) the file, and scp it back onto the degraded node in the correct location. This will overwrite the manual changes made to the file and restore it to what the MCO wants it to be. The MCO should then pick up that the `MachineConfig` now matches, and continue progressing with what it was doing.

### Error running rpm-ostree

Red Hat CoreOS uses something called [rpm-ostree]() which is a *"hybrid image/package system"*. It's basically an archive file that contain all the Linux software package (RPMS) that CoreOS requires and allows atomic upgrades (and downgrades) by being able to pivot between them. Sometimes this fails, and the node will become degraded. I've seen this fail for a couple of reasons - once with an issue where the node was unable to download an image from quay.io, and another time when a third party vendor (who wasn't familiar with how rpm-ostrees and Red Hat CoreOS worked) set the immutable ACL attribute on some of its files on the node, causing the node to fail drastically on each OS upgrade. The vendor has since removed these custom ACLs from their product after our experience.

The best way to fix this, if you can, is to correct the issue breaking the pivot. For example, by removing the immutable ACL on a system file. The Linux systemd journal on the node should help with identifying the error. If you still can't fix it, the last resort to fix this issue is to force a pivot.

### Forcing a Pivot

If you can't get the node to update, you'll have to force it. To do this, you connect to the node and run the `machine-config-daemon pivot` command with the correct OS image SHA (find this from the errors in the logs to be sure you get the correct one!)

```
$ oc debug node/server01
Starting pod/server01-debug ...
To use host binaries, run `chroot /host`
Pod IP: 192.168.1.201
If you don't see a command prompt, try pressing enter.
# sh-4.4
# chroot /host
# /run/bin/machine-config-daemon pivot "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:aaaabbbbccccdddd"
```


### Forcing a Node to Use a Specific MachineConfig

As a last resort - and experience has shown it really is better to address any MCO issues in other ways before doing this - you can force a node to reboot and use a different `MachineConfig`. This is done by manually editing the machineconfiguration annotations on the Node object itself.

```
oc patch node server01 --type merge --patch '{"metadata": {"annotations": {"machineconfiguration.openshift.io/currentConfig": "render-worker-52656729c24a7493"}}}'
oc patch node server01 --type merge --patch '{"metadata": {"annotations": {"machineconfiguration.openshift.io/desiredConfig": "render-worker-52656729c24a7493"}}}'
oc patch node server01 --type merge --patch '{"metadata": {"annotations": {"machineconfiguration.openshift.io/reason": ""}}}'
oc patch node server01 --type merge --patch '{"metadata": {"annotations": {"machineconfiguration.openshift.io/state": "Done"}}}'
```

Then you have to connect to the node and touch a machine-config-daemon-force file. This will trigger the *machine-config-daemon* to do a reboot and to force the update to the `MachineConfig`.

```
$ oc debug node/server01
[root@server01 core] chroot /host
[root@server01 core] touch /run/machine-config-daemon-force
```

On occasion, you might not be able to connect to the node using an oc debug pod. I've seen this happen when a systemd unit running podman early in the boot process has had an image pull failure, and the debug pod is also not able to run when kubelet is not running properly, mean SSH is the only option.

```
$ ssh core@server01
[root@server01 core] touch /run/machine-config-daemon-force
```

## Pausing MCO Updates

As described above, the MCO will automatically carry on the work of rebooting and managing your nodes if you let it. This is great, but does mean that nodes in the cluster can sometimes be rebooted without warning. Of course, true cloud-native Kubernetes ready applications should be running multiple replicas across nodes anyway, and this shouldn't normally be a problem. 

However, should you wish to pause the MCO so that it doesn't automatically reboot nodes when you don't want it to, you can "pause" the worker pools by setting the `.spec.paused` field in the `MachineConfigPool`. For example, by running the following patch commands:

```
$ oc patch mcp master --type merge --patch '{"spec":{"paused": true}}'
$ oc patch mcp worker --type merge --patch '{"spec":{"paused": true}}'
```

It should be noted that you shouldn't leave the `MachineConfigPool's` paused for long periods of times. This can cause issues for MCO if configuration updates combine with cluster cert rotation events. However, pausing them whilst the actual OCP control plane upgrade happens and then unpausing when you're ready for all the nodes to be rebooted can be very useful.

## Summary

The Machine Config Operator is a clever piece of software that significantly reduces workload of running and managing nodes in an OpenShift cluster. However, it's important for cluster administrators to understand how it works, and to be able to successfully recover it when any issues arise and it becomes degraded. Hopefully, this blog post will help you understand it a little better and to resolve any issues you encounter.
