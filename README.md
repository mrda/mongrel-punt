# mongrel-punt
[![License: GPL v3+](https://img.shields.io/badge/license-GPL%20v3%2B-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

It's an OpenStack Devstack builder!

More specifically, it's a set of Ansible plays to build VMs, run devstack on them, and perform various openstacky activities such as baremetal introspection.

## Quickstart

You can use mongrel-punt on the following operating systems:
* Centos 7
* Fedora 29 (f29)
* Ubuntu 18.04 (u1804)

The problem is that both f29 and u1804 don't fully work for different reasons.  Please see [Limitations](#limitations) for the reasons why, and how you can work around these issues.  But once you address these, you can make use of the following plays.

### Inventory

Each supported operating system has a separate inventory file `inventory-*.yml` in the top-level directory.  This is where you can customise what mongrel-punt will do.

The very first thing you want to do is to update the `name:` definition.  It is suggested that you use `OS-date-rev`, where `OS` is one of `[centos7, f29, u1804]`; `date` is today's date in YYYYMMDD format i.e. 20190812; and `rev` is an integer indicating a revision number to distinguish multiple builds on the same day.

### Plays

* `build-vm.yml` - builds a virtual machine
* `start-devstack.yml` - configures your VM ready to run devstack.  Note that currently it does not automatically start devstack.
* `introspect-nodes.yml` - performs introspection on the (nested) virtual baremetal nodes, adding traits to them

Or, if you're feeling brave, you can run this one single play which invokes the above playbooks in sequence:
```sh
ansible-playbook  -K -i inventory-centos7.yml playbooks/main.yml
```
(As a guide, running `playbooks/main.yml` takes around 52 minutes of wall clock time on a Lenovo X1 Carbon Gen 6 laptop)

### Utility Plays
* `get-ip-address.yml` - returns you the routable IP address of the VM

### Running plays

This is how a typical build will go.

1. Choose your operating system
1. Customise your inventory file, one of `[inventory-centos7.yml, inventory-f29.yml, inventory-u1804.yml]`
1. Build your VM, either using a golden image or the build-vm.yml playbook.  i.e. `ansible-playbook  -K -i inventory-centos7.yml playbooks/build-vm.yml`
1. Prepare your VM for devstack and start it going, noiting that it will take around 30 minutes before it completes. i.e. `ansible-playbook  -K -i inventory-centos7.yml playbooks/start-devstack.yml`
1. Introspect your nodes.  i.e. `ansible-playbook  -K -i inventory-centos7.yml playbooks/introspect-nodes.yml`

## Limitations

Right now, there are some challenges with two of the target operating systems:

### Fedora 29

You can build a Fedora 29 VM by doing this:

`ansible-playbook  -K -i inventory-f29.yml playbooks/build-vm.yml`

But currently Fedora can't run an Ironic-ised devstack, due to virtual baremetal nodes entering "clean failed" state.  Once that is fixed upstream, you can follow [Running Plays](#Running-plays).

### Ubuntu 18.04

U1804 is not supported for automated kickstart based builds (or is hard to work out without up to date documentation), so the play to build a VM is not complete.  You'll have to build that by hand first - see [Building an U18.04 VM](doc/Building-U1804-VM.md) for ideas.  After that's done, you can follow [Running Plays](#Running-plays).

## Colophon

What is the meaning of the term, "mongrel punt"?

Australia's national sport is Australian Rules Football (AFL), which is different from rugby, gridiron and soccer.  It's a game where people kick the football, pass the football via punching ("handball"), ferociously tackle each other, and score by kicking the ball between the goal posts.

A poorly executed kick is referred to as a "mongrel punt".  It gets the job done, but is not considered orthodox, reliable, or stylish.  According to wikipedia, it is "a tumbling, awkward looking kick"[1].  It describes what a set of Ansible plays to standup an OpenStack devstack is.

Here's a video of what a mongrel punt is: https://www.youtube.com/watch?v=vNlm5Xwf51c

[1] https://en.wikipedia.org/wiki/Glossary_of_Australian_rules_football
