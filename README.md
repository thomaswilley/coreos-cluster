coreos-cluster
==============

A simple local experimental coreos cluster setup using Vagrant and Ansible

### Getting started
---
Get into the working directory which contains the Vagrantfile and ./provision/
folder and:

```sh
$ vagrant up --no-provision
$ vagrant provision
$ vagrant ssh core-01
```

yes, it's a little wierd to need to run with --no-provision, then later
with provision only after all the VMs are setup. but this way does work
just fine.
 
```sh
[core-01]$ etcdctl ls
```
should return quickly and return nothing, b/c there are no values stored on
the cluster

```sh
[core-01]$ fleetctl list-machines
```
should return quickly with the set of three machines with machine ID, IP
address, and Metadata (which is from fleet.conf & contains hostname for now

```sh
[core-01]$ fleetctl submit registry-cluster.service
[core-01]$ fleetctl start registry-cluster.service
[core-01]$ fleetctl status registry-cluster.service
```

note: you'll probably get some panic messages from go when you ask for
status or try anything ssh related. the provisioner took the liberty of
pushing your ~/.vagrant.d/insecure_private_key into the /home/core/.ssh/
folder of each node. it's a little tedious but would suggest you simply
go into the node you want to mostly use fleetctl from and acquaint the
image with that key so the coreos, fleet, etc ssh commands resume
functioning

```bash
core@core-02 ~ $ eval `ssh-agent -s`
Agent pid 964
core@core-02 ~ $ ssh-add ~/.ssh/insecure_private_key 
Identity added: /home/core/.ssh/insecure_private_key
(/home/core/.ssh/insecure_private_key)
core@core-02 ~ $ fleetctl list-machines -l
MACHINE                 IP      METADATA
a8505f9e-e163-45f9-9ca1-bac65dd983bc    192.168.65.2    name=core-01
f1912f0d-4fbb-4645-b101-c0ac0d66dff8    192.168.65.4    name=core-03
49ecaa92-3f25-456c-9f72-b55751fb19d9    192.168.65.3    name=core-02
core@core-02 ~ $ fleetctl ssh a8505f9e-e163-45f9-9ca1-bac65dd983bc
Last login: Mon Feb 24 03:55:23 UTC 2014 from 192.168.65.1 on pts/0
   ______                ____  _____
  / ____/___  ________  / __ \/ ___/
 / /   / __ \/ ___/ _ \/ / / /\__ \
/ /___/ /_/ / /  /  __/ /_/ /___/ /
\____/\____/_/   \___/\____//____/
core@core-01 ~ $ exit
logout
```

more explanation and fix options are here:
https://github.com/coreos/fleet/issues/143

...Sky's the limit at this point to experiment around with fleet, etcd, and the cluster...

### Notes of interest
---
Ansible works because it's either doing local_actions (like template
rendering for configs) or 'raw' actions which are raw commands issued over ssh
by ansible. This is because ansible itself requeires python (I believe 2.4+) on
all nodes and coreos doesn't (and doesn't plan to AFAIK) include python in the
image.

Because of this, and because I'm pretty new to ansible and probably made some
mistakes in the playbook, it does sometimes happen that if you run the same
ansible-playbook command twice, you actually have better results with the
various services functioning correctly. Also, keep your ~/.ssh/known_hosts in
mind a bit since it will get updated with the hashes of the VMs as you use them
so if you spin up clusters then [$ vagrant destroy && vagrant up] them, you can
get errors about ssh which are typically because the hash in known_hosts simply
changes when you re-create VMs [troubleshoot with -vvvv anyway to be sure].

### File function summary
* Vagrantfile: Creates 3 coreos VMs running at 192.168.65.2-4 named core-01, core-02, and core-03
* Ansible playbook & configs in ./provision/: Bootstraps the coreos VMs
  after they're started with a (patched) etcd-cluster.service and
  (simple, custom) fleet-cluster.service. The fleet service reads
  /home/core/fleet.conf as it's config file which allows this setup to
  include metadata and public ip in fleetctl results, and for other
  experimentation with the feature set.

Would love your help to critique and improve this so it can be updated to
better reflect best practices.
