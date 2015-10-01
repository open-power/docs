OPAL Documentation
==================

There are several parts to the OpenPower Abstraction Layer (OPAL) documentation
and specification. This README is designed to orient you around where
documentation exists.

Background
----------
OPAL firmware first shipped as part of the first IBM POWER8 8xx-L servers with
PowerKVM and FW810 in June 2014.

Development to that first release and afterwards has basically followed an
Agile manifesto for software development:
- Individuals and interactions over processes and tools
- Working software over comprehensive documentation
- Customer collaboration over contract negotiation
- Responding to change over following a plan
(which is straight from http://agilemanifesto.org/ - It's recommended to also
read http://agilemanifesto.org/principles.html )

Thus there was not a 2,000+ page specification documented written years before
the first release. Where there are gaps in documentation, we refer to the
existing working implementations.

Our documentation is designed to be in small, manageable and easily updatable
chunks.

Patches to documentation is STRONGLY ENCOURAGED.

What is OPAL?
-------------
OPAL is boot and runtime firmware for POWER.

It consists of the following components:
- boot & runtime firmware (skiboot being the reference implementation)
  - this provides OPAL calls, including the PRD facility.
- An optional bootloader environment (petitboot running in a linux
  environment being the reference implementation)

The bootloader environment is optional as embedded systems may want to
not have the option to boot anything but their own payload and it leaves
options open for other boot components.

The boot environment coming from skiboot into the bootloader environment
is essentially identical to the environment coming from the bootloader.
Thus any Operating System SHOULD support booting from the bootloader OR
as the payload itself.

It is INCREDIBLY STRONGLY RECOMMENDED that systems ship the reference
implementation of the bootloader environment.

TODO: Come up with wording that says that an exception for certification
is needed to ship anything but the petitboot bootloader environment?

OPAL API
--------
The boot interface and runtime ABI/API are documented as part of the
reference implementation, skiboot.

Currently, this is all housed in the doc/ directory of the skiboot
git repository.
This can be found here: https://github.com/open-power/skiboot/tree/master/doc

Where there are gaps in documentation, we refer to the implementation and
encourage patches

PAPR
----
See: https://members.openpowerfoundation.org/document/dl/469
The PAPR specification is the Linux on Power Architecture Platform Reference
(sometimes referred to as LoPAPR).

This is the specification for the virtualized environment available under
KVM on POWER (e.g. guests in PowerKVM) and under the PowerVM hypervisor.

The PAPR interfaces are *NOT* something that OPAL provides.

We do share some data structures though, and there is ongoing work to split
out common documentation into their own specs rather than one giant spec.

Boot loader
-----------
The linux kernel + petitboot userspace is the OPAL bootloader environment.