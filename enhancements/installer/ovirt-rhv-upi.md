---
title: ovirt-rhv-upi
authors:
  - "@gekorob"
reviewers:
  - "@sdodson"
  - "@abhinavdahiya"
approvers:
  - TBD
creation-date: 2020-05-06
last-updated: 2020-05-06
status: implementable
---

# oVirt-RHV UPI

## Release Signoff Checklist

- [x] Enhancement is `implementable`
- [x] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [x] Graduation criteria
- [ ] User-facing documentation is created in [openshift-docs](https://github.com/openshift/openshift-docs/)

## Summary

Ovirt support for OpenShift 4 was initially focused on the installer provisioned
(IPI) workflow. The IPI is itself a convenient and straightforward approach for the
oVirt/RHV end user that aims to create all the infrastructure elements needed to
install OpenShift in a highly opinionated way.
Right now, users can choose to adapt the [bare metal upi][baremetal-upi] instructions
to reuse their existing infrastructure elements.

With this enhancement we want to provide a reference and specific oVirt/RHV documentation
and tooling to allow users to manage OpenShift installation in their clusters in a more
flexible and familiar way.

## Motivation

oVirt/RHV infrastructures are generally highly customized by the owning organizations
and there's a high demand to have a UPI that can take advantage of the already
existing elements (e.g. DNS, DHCP, Load Balancer, ...) simplifying their configuration
to install OpenShift.

### Goals

The main goal of the UPI is to grant users a higher degree of freedom compared to the
more opinionated installer-provisioned installation.

The purpose of this enhancement is to obtain, as a fist step, the same result of a regular
IPI installation, granting the user the ability to reuse and customize pre-existing
resources and VMs, through a provisioning installer codebase and a set of documented
manual steps.
Following other UPI installer directives we are thinking to have

* oVirt-RHV UPI documentation available under:
https://github.com/openshift/installer/blob/master/docs/user/ovirt/install_upi.md
* Ansible playbooks for scripts for oVirt/RHV resource creation available under:
https://github.com/openshift/installer/tree/master/upi/ovirt
* CI job executing the provisioning scripts to test the UPI installer

### Non-Goals

It is outside the scope of this enhancement to provide explanations about the installation
of infrastructure elements that are considered as required and owned by the user (e.g. DNS,
DHCP, Load Balancer...)

## Proposal

This is where we get down to the nitty gritty of what the proposal actually is.

* Write Ansible playbooks that can automate as much as possible the creation of ovirt resources
like the VMs that will be used as masters and worker nodes of the cluster.
* Write Ansible scripts to configure pre-existing infrastructure elements like (DNS, DHCP, ...)
* Write the UPI documentation
* Setup the CI job to have running test suite


### Implementation Details/Notes/Constraints

The implementation of the UPI workflow aims to reproduce the same features of the IPI
allowing the user to deploy OpenShift on an existing infrastructure.

The user will have the possibility to specify custom configuration related to their pre-existing
infrastructure elements that will be useful for the installation: domain name, DNS, DHCP,
Load Balancer, Web server.

Stated the maturity level of the oVirt Ansible modules and the familiarity that administrators
tend to have with Ansible itself, a set of playbooks will be provided to automate as
much as possible the configuration and installation process. 

Depending on the user environment we will provide script that could help the user

* add mandatory records (A, PTR, SRV) to the user provided DNS
* upload the RHCOS image to use
* configure an httpd server for RHCOS customized parameters ignition
* configure a load balancer (HAProxy)
* configure a dhcp to assign IP addresses to bootstrapping machines
* bootstrap VMs

### Risks and Mitigations

This UPI will try to achieve the same results obtained by the automatic and opinionated
IPI installer but in a more customizable way using provisioning scripts like ansible
and the documentation provided.

One of the challenging problems will be represented by the CI.
In particular UPI jobs will add more load to the current CI and so they will
have to be carefully scheduled due to the currently limited capacity.

We cannot exclude that an additional quota must be required to fulfill all the UPI
CI needs.

## Design Details

### Test Plan

The testing strategy will be inspired by the other existing UPI platforms
(e.g. OpenStack, AWS, GCP):

- A new e2e job will be created to use the Ansible templates
- At the moment we think that unit tests will not probably be necessary, but in case
of necessary changes to the existing codebase, will cover them with appropriate
tests.

### Graduation Criteria

The proposal is to follow a graduation process based on the existence of a CI running
suite with end to end jobs and to evaluate it's feedback along with the ones provided by
the QE and testers.

We consider the following as part of the necessary steps

- UPI document published in the OpenShift repo
- Ansible playbooks exist
- CI jobs present and regularly scheduled
- End to end jobs are stable and passing and evaluated with the same criteria of the IPI
- Developers of the team have successfully deployed a UPI on RHV following the
documented procedure

## Implementation History

Major milestones in the life cycle of a proposal should be tracked in `Implementation
History`.

## Drawbacks

The UPI implementation is resource demanding not only from the development point of view
but also for CI, QE, Documentation, etc.
Resources could be spent to improve the IPI process but as already specified in the
`Motivation` section, the UPI was highly requested by the users.

## Alternatives

People not using the IPI workflow can follow the Bare Metal UPI document.
That implies more manual work and the necessary knowledge to identify oVirt/RHV specific
parts without any automation help.

Extending the IPI to cover several common use cases coming from UPI users is not
considered a valid option, because it's against the opinionated nature of the IPI itself.

## Infrastructure Needed

Developers have the infrastructure needed to approach the UPI basic feature development
Resources for the CI should be carefully evaluated in relation to the extra load due to
the UPI jobs and any future integration with storages etc.

[baremetal-upi]: https://github.com/openshift/installer/blob/master/docs/user/metal/install_upi.md
