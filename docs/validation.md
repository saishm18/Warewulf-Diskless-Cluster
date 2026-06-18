# Validation

This page summarizes what I validated in the final cluster workflow and includes public-safe excerpts from real command output.

The point of validation was not just to show that services could start. The cluster had to work end to end: boot, enter runtime, expose shared storage, register with Slurm, accept user jobs, and execute real workloads.

## 1. Boot Validation

I treated boot as a layered process:

- UEFI PXE had to start correctly
- the bootloader had to be fetched correctly
- GRUB had to hand off to the right kernel and initramfs
- the node had to enter the intended diskless runtime

Public-safe boot excerpt:

```text
Start PXE over IPv4
NBP filename is warewulf/shim.efi
Fetching Netboot Image warewulf/grubx64.efi

Warewulf v4 (GRUB)
MARKER: using /usr/local/etc/warewulf/grub/grub.cfg.ww

This node:
  ImageName: rocky9-test
  KernelVersion: 5.14.0-611.27.1
```

That kind of evidence mattered because early boot failures were one of the hardest parts of the build.

## 2. Fleet Visibility and Provisioning State

![Warewulf node list proof](../assets/proofs/warewulf-node-fleet.png)

The cluster also had to prove that the provisioning layer really knew about the full worker fleet.

Public-safe excerpt:

```text
$ wwctl node list
NODE NAME  PROFILES  NETWORK
node01     node1     default, sfp
node02     node1     default, sfp
node03     node1     default, sfp
...
node38     node1     default, sfp
node39     node1     default, sfp
node40     node1     default, sfp
```

This shows that the project was not just a single-node proof of concept.

## 3. Worker Runtime Validation

![Rocky Linux worker OS proof](../assets/proofs/rocky-worker-os.png)

Once a node finished booting, I validated that it was actually running the intended worker environment.

Example operating system check:

```text
$ cat /etc/os-release
NAME="Rocky Linux"
VERSION="9.7 (Blue Onyx)"
ID="rocky"
VERSION_ID="9.7"
PRETTY_NAME="Rocky Linux 9.7 (Blue Onyx)"
```

That matters because a diskless cluster can look healthy at PXE or GRUB time and still fail later in runtime assembly.

## 4. Shared Storage Validation

The cluster relies on shared filesystems for the persistent parts of the user environment. I validated that those mounts were present from the running node environment.

Public-safe excerpt:

```text
$ mount | grep -E '/opt|/home'
<storage-host>:/opt  on /opt  type nfs4 (...)
<storage-host>:/home on /home type nfs4 (...)
```

That proved the worker nodes were not only booting, but also seeing the shared storage model needed for normal use.

## 5. Network and Multi-NIC Validation

The cluster uses separate roles for control-plane and workload-side networking, so I validated more than simple reachability. I also confirmed that the nodes came up with the intended dual-network identity and that the user and scheduler paths used the correct interfaces.

This mattered because wrong interface preference was one of the failure classes that could make the cluster look partially healthy while still breaking scheduler behavior.

## 6. Slurm Validation

Scheduler validation included:

- node visibility
- node registration
- interactive execution
- batch execution
- user-facing login-node workflow

Public-safe interactive example:

```text
$ srun -N4 -n4 hostname
node03
node01
node04
node02
```

That is one of the most convincing pieces of evidence in the project because it shows the scheduler, the login path, and the compute fleet all working together.

## 7. Non-Root User Validation

I also validated that the cluster worked through a normal user path rather than only through privileged administrative testing.

Public-safe excerpt:

```text
User info:
uid=10000(<user>) gid=10000(<user>)
Sudo check:
NO_SUDO_ACCESS
Hostnames:
node01
node03
node02
node04
```

This mattered because it proved that the cluster supported a real user workflow, not just admin-run test commands.

## 8. Login-Node Workflow Validation

Part of the final validation was confirming that the login node could act as the real entry point for cluster work:

- shared-home workflow available
- scheduler commands available
- job launch available
- user-facing access path usable without direct admin intervention

That was especially important because a major part of the project was moving from infrastructure bring-up into something users could actually use.

## 9. What Validation Proved

Taken together, the validation work proved that the project reached more than just boot success.

It demonstrated:

- repeatable diskless boot
- working runtime assembly
- shared storage availability
- login-node-based user access
- working Slurm scheduling
- non-root job execution
- readiness for real benchmarking and HPC workloads

That is the line between a partially assembled cluster and a working HPC environment.
