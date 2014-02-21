coreos-cluster
==============

A simple local experimental coreos cluster setup using Vagrant and Ansible

Getting started
---
Get into the working directory which contains the Vagrantfile and ./provision/
folder and:

$ vagrant up
$ ansible-playbook -i provision/hosts provision/playbook.yml
$ vagrant ssh core-01

[core-01]$ etcdctl ls
  should return quickly and return nothing, b/c there are no values stored on
  the cluster

[core-01]$ fleetctl list-machines
  should return quickly with the set of three machines with machine ID, IP
  address, and Metadata (which is from fleet.conf & contains hostname for now

Sky's the limit at this point to experiment around with fleet, etcd, and the cluster...

Notes of interest:
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


Vagrantfile
---
Creates 3 coreos VMs running at 192.168.65.2-4 named core-01, core-02, and core-03

Ansible playbook & configs in ./provision/
---
Bootstraps the coreos VMs after they're started with a (patched)
etcd-cluster.service and (simple, custom) fleet-cluster.service. The fleet
service reads /home/core/fleet.conf as it's config file which allows this setup
to include metadata and public ip in fleetctl results, and for other
experimentation with the feature set.

Would love your help to critique and improve this so it can be updated to
b
