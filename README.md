# mongrel-punt
[![License: GPL v3+](https://img.shields.io/badge/license-GPL%20v3%2B-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

It's a devstack builder!

More specifically, it's a set of Ansible plays to build VMs, run devstack on them, and perform various openstacky activities such as baremetal introspection.

## Limitations

Right now, things are in a little bit of a mess.

### Fedora 29

You can build a Fedora 29 VM by doing this:

`ansible-playbook  -K -i inventory-f29.yaml playbooks/build-vm.yml`

But currently Fedora can't run an Ironicised devstack, due to virtual baremetal nodes entering "clean failed" state.  Once that is fixed upstream, the same plays identified below for U1804 can be used.

## Ubuntu 18.04

U1804 is not supported for automated kickstart based builds (or is hard to work out without up to date documentation), so the play to build a VM is not complete.  You'll have to build that by hand first - see `doc/Building-U1804-VM.md` for ideas.  But once you've done that, you can use the included ansible plays to do intersting things, such as:

* `ansible-playbook -K -i inventory-u1804.yaml playbooks/run-devstack.yml` which will set up your VM for running devstack
  * You'll need to edit inventory-u1804.yaml first to change the `name` of your VM
  * You can find out the IP address of your VM by running `ansible-playbook -K -i inventory-u1804.yaml playbooks/get-ip-address.yml`
  * Once this play has completed, you can `ssh root@<ipaddr>` and then run `cd /opt/stack/devstack; ./stack.sh` to start your devstack.  Note that it takes about 30 minutes or so to complete.
* `ansible-playbook -K -i inventory-u1804.yaml playbooks/inspect-nodes.yml` which places notes into maintenance mode, introspects them, and adds traits to them according to the introspection rules found in `/opt/stack/introspection`
