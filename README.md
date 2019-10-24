# mongrel-punt
[![License: GPL v3+](https://img.shields.io/badge/license-GPL%20v3%2B-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

It's an OpenStack Devstack builder!

More specifically, it's a set of Ansible plays to build VMs, run devstack on them, and perform various openstacky activities such as baremetal introspection.

## Quickstart

mogrel-punt has been tested on Fedora 30.  You can use mongrel-punt to build VMs for the following operating systems:
* Centos 7 (centos7)
* Centos 8 (centos8)
* Fedora 29 (f29)
* Ubuntu 18.04 (u1804)

The problem is that several operating systems don't fully work for various reasons.  Please see [Limitations](#limitations) for the reasons why, and how you can work around these issues.  But once you address these, you can make use of the following plays.

### Inventory

Each supported operating system has a separate inventory file `inventory-*.yml` in the top-level directory.  This is where you can customise what mongrel-punt will do.

The very first thing you want to do is to update the `name:` definition.  It is suggested that you use `OS-date-rev`, where `OS` is one of `[centos7, f29, u1804]`; `date` is today's date in YYYYMMDD format i.e. 20190812; and `rev` is an integer indicating a revision number to distinguish multiple builds on the same day.

### Plays

In mongrel-punt, we have some higher-level playbooks that run individual plays.  These are:
* `00-ipa.yml` - build an ironic-python-agent disk image
* `00-start-devstack.yml` - builds and starts a devstack

You can run these on the supported operating systems as follows:

|              | 00-ipa | 00-start-devstack |
|:------------:|:------:|:-----------------:|
| Fedora 29    |:heavy_check_mark:|:x:|
| Ubuntu 18.04 |:heavy_check_mark:|:heavy_check_mark:|
| Centos 7     |:heavy_check_mark:|:heavy_check_mark:|
| Centos 8     |:heavy_check_mark:|:x:|

Alternatively, you can run individuals plays.  See [Limitations](#limitations) to determine which plays are supported where.

* `build-ipa-image.yml` - build an ironic-python-agent image and copies the resultant images back to the invoking host
* `build-ironic-node.yml` - builds virtual baremetal nodes alongside the devstack vm
* `build-vm.yml` - builds a virtual machine for the devstack node
* `copy-ipa-image.yml` - copies ip images from localhost into the devstack vm
* `enroll-nodes.yml` - enrolls virtual baremetal ironic nodes into devstack
* `get-vm-ip-address.yml` - utility play to obtain the devstack IP address
* `introspect-nodes.yml` - performs introspection on the (nested) virtual baremetal nodes, adding traits to them based upon instrospection rules configured in `/opt/stack/introspection/rules.json`
* `metalsmith-provision-traits.yml` - provision virtual baremetal ironic nodes based upon traits added via introspection
* `prepare-devstack.yml` - configures your VM ready to run devstack
* `provide-tinyipa.yml` - copy ip disk images from localhost onto the devstack node
* `build-ipa-image.yml` - builds a customised tinyipa image for introspection
* `start-devstack.yml` - runs devstack.

### Running plays

This is how a typical build will go.

1. Choose your operating system
1. Customise your inventory file, one of `[inventory-centos7.yml, inventory-f29.yml, inventory-u1804.yml]`
1. Build your VM, either using a golden image or the build-vm.yml playbook.  i.e. `ansible-playbook  -K -i inventory-centos7.yml playbooks/build-vm.yml`
1. Prepare your VM for devstack. i.e. `ansible-playbook  -K -i inventory-centos7.yml playbooks/prepare-devstack.yml`
1. Build a customised tinyipa image.  i.e. `ansible-playbook  -K -i inventory-f29.yml playbooks/build-ipa-image.yml`
1. Start devstack, noting that it will take around 30 minutes before it completes. i.e. `ansible-playbook  -K -i inventory-centos7.yml playbooks/start-devstack.yml`
1. Introspect your nodes.  i.e. `ansible-playbook  -K -i inventory-centos7.yml playbooks/introspect-nodes.yml`

## Limitations

Right now, there are some challenges with various target operating systems.

|              | build-vm | prepare-devstack | build-ipa-image | provide-tinyipa | start-devstack |
|:------------:|:--------:|:----------------:|:---------------:|:---------------:|:--------------:|
| Fedora 29    |:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:x:               |
| Ubuntu 18.04 |:x:               |:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|
| Centos 7     |:heavy_check_mark:|:heavy_check_mark:|:x:               |:heavy_check_mark:|:heavy_check_mark:|
| Centos 8     |:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|:x:|

### Fedora 29

You can build a Fedora 29 VM by doing this:

`ansible-playbook  -K -i inventory-f29.yml playbooks/build-vm.yml`

But currently Fedora can't run an Ironic-ised devstack, due to virtual baremetal nodes entering "clean failed" state.  Once that is fixed upstream, you can follow [Running Plays](#Running-plays).

### Ubuntu 18.04

U1804 is not supported for automated kickstart based builds (or is hard to work out without up to date documentation), so the play to build a VM is not complete.  You'll have to build that by hand first - see [Building an U18.04 VM](doc/Building-U1804-VM.md) for ideas.  After that's done, you can follow [Running Plays](#Running-plays).

### Centos 7

Centos 7 uses an old kernel version, which is too old for "ironic-python-agent-builder" to use.  Instead, use either Ubuntu 18.04 or Fedora 29 to build your customised tinyipa images for baremetal node introspection.

### Centos 8

Centos 8 is supported for all operations, except for actually running devstack.  It is expected that support for Centos 8 / RHEL-8 will be added to devstack, but it is not yet available.

## Colophon

What is the meaning of the term, "mongrel punt"?

Australia's national sport is Australian Rules Football (AFL)[1], which is different from rugby, gridiron and soccer.  It's a game where people kick the football, pass the football via punching ("handball"), ferociously tackle each other, and score by kicking the ball between the goal posts.

A poorly executed kick is referred to as a "mongrel punt".  It gets the job done, but is not considered orthodox, reliable, or stylish.  According to wikipedia, it is "a tumbling, awkward looking kick"[2].  It describes what a set of Ansible plays to standup an OpenStack devstack is.

Here's a video of what a mongrel punt is: https://www.youtube.com/watch?v=vNlm5Xwf51c

[1] https://www.youtube.com/watch?v=XMZYZcoAcU0
[2] https://en.wikipedia.org/wiki/Glossary_of_Australian_rules_football
