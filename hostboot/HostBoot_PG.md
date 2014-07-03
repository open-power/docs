# Hostboot POWER Systems Initialization Firmware #

The hostboot firmware performs all processor, bus, and memory initialization within IBM&reg; POWER&reg; based systems. It initializes the central electronics complex (CEC) of the IBM POWER8&trade; processor and the memory subsystem. It prepares the hardware to load and run an appropriate payload and provides runtime services to the payload. The hostboot firmware is easily configurable for a variety of POWER systems.



# Overview #
The hostboot firmware runs on the POWER8 processor and subsequent platforms. The following figure shows the major components involved in the initialization process and how they interact with the hostboot firmware.
![](http://i.imgur.com/kKcWzOd.png)

## Functional Overview ##
The hostboot firmware provides the following initial program load (IPL) capabilities:


- Initializes and runs diagnostics on the central electronics complex (CEC) processor and memory subsystems for various types of IPLs with controllable diagnostics levels and controllable sequencing modes.

- Configures, controls, packages, debugs, and tests the supplied firmware function as well as software and hardware procedures.

- Configures or overrides policies and loads and runs architected verification programs (AVPs) to support the manufacturing line.

- Adapts to arbitrary, but compatible, system topologies, defined system policies, hardware procedure attribute sets, and other conditions with no code impact.

- 	Detects, reports, and handles errors in hardware and software during IPL activities and reports IPL status.

The following simplified scenario describes what happens when the user pushes the button to power on the system, when the system is already in the standby state.

1.	The baseboard management controller (BMC) powers the system on.
2.	The BMC selects the master chip and releases the self-boot engines (SBEs) on the POWER8 chips, master last.
3.	The BMC relinquishes control of the flexible service interface (FSI) SCAN/SCOM engines.
4.	The hostboot firmware IPLs the system. It initiates a secondary power-on sequence through a digital power systems sweep (DPSS).
5.	The hostboot firmware loads the OPAL image and moves all processors to their execution starting points.

The following figure shows the hostboot flow.
![](http://i.imgur.com/tk1H8ls.png)

## Software Architecture ##
The following figure and the accompanying text describe the hostboot IPL firmware. The “Surrounding Ecosystem” is not part of the hostboot firmware; it is a set of external support deliverables. Any component above the kernel resides in user space.

 ![](http://i.imgur.com/aBV3AJF.png)

**Hostboot kernel**: The operating system that runs on the host processors and supports and otherwise enables execution of user-space hostboot IPL firmware and services. The kernel provides the general execution environment, message passing, task control, memory management, interrupt support, and any other functions to support the initialization activities of the user space. It is a micro-kernel that pushes as much function as possible to the user space, which has more safety protections. For more information, see <a href="#Kernel">Kernel</a>.

**System call wrappers**: A set of light-weight wrappers that bridge hostboot IPL user-space calls into the kernel space, which runs in the hypervisor state.

**Device drivers**: A set of device drivers to read and write hardware devices and perform presence detection. This area includes support for device and hardware access. For more information, see <a href="#devdrive">Device Drivers</a>.

**Interrupt service**: A facility that allows a hostboot device driver or service to register a callback for a particular type of interrupt. When the kernel notifies the service of an interrupt, it invokes the appropriate registered callback.

**Device routing/translation**: A firmware layer that provides device routing and translation services. Device drivers can register with this layer to provide device services to other device drivers, when needed.

**Device driver user-space API**: A firmware layer designed to expose a common device-driver API to non-device-driver user-space services. It abstracts hostboot IPL and runtime firmware from the underlying device drivers, which can have different implementations between these environments

**Base enablement libraries**: A set of libraries designed to mimic parts of libc/libc++/STL to support other hostboot firmware.  These libraries also include support for a virtual file system.

**Base enablement services**: A collection of basic services, mostly RAS related, that are almost universally required by other user-space services and devices. Includes support for logging errors, component trace, and progress codes and bridging API requests to and from the BMC.

**Hardware initialization services**: A collection of services that collaborate to initialize the CEC hardware. These services include major and minor IPL step (Istep) logic, hardware procedures and the hardware procedure framework, services to support winkle, initialize the payload, configure memory settings, and ramp memory voltage.

**System information abstraction services**: A collection of services that abstract access to and manipulation of system topology, hardware data, software policies, configuration data, vital product data (VPD), and so on from the underlying hardware devices or software layers that provide backing store for the information.

**Initialization service**: A service that manages the IPL flow sequencing from the moment the kernel gives it initial control until it boots the payload, determining which major and minor Isteps to run and when. It also acts as the final authority on handling errors in the IPL and performs processes such as hardware reconfigure loop processing and resource recovery as necessary.

**Surrounding ecosystem**: A suite of command-line utilities, compilers, simulator macros, build scripts, packaging tools, automated unit tests, debug tools, processes, and so on. Collectively, the utilities allow stakeholders to build, configure, debug, and test hostboot firmware within various environments and with high levels of quality.
## <a id="Kernel">Kernel</a> ##
The kernel consists of the following kernel sub areas:

**Execution environment**. Kernel support establishes the initial hostboot operating environment after the self-boot engine (SBE) hands off to it. It provides support for floating-point operations, and hands off execution to a hostboot user-space initialization service to drive the remainder of the IPL flow. It also provides facilities for critical first failure data collection (FFDC) and to support interrupt handling.

**Message passing support**. Kernel facilities provide support for interprocess communication (IPC) through message queues so that tasks and the kernel can communicate with command and control messages

**Task management**. Kernel facilities provide support for management and synchronization of user space tasks to enable parallel or concurrent processing, background services, device handling, and so on.

**Memory management**. Kernel facilities provide support for physical and virtual memory management, including dynamic memory allocation, memory-mapped I/O (MMIO), dynamic stack growth, and special register access, on behalf of the user space.

**Time management**. Kernel facilities support time management features such as allowing a task to delay for specified periods of time

**Interrupt handling.** Kernel facilities handle interrupts and exceptions.

## <a id="devdrive"> Device Drivers </a> ##
The following device drivers are provided. All device drivers register their services with the device routing layer and ask the routing layer to perform any operation that they are incapable of handling directly.

**SCAN device driver**. This device driver reads and writes SCAN rings through SCOM operations.

**SCOM device driver**. This “abstract” device driver provides a scan communication (SCOM) interface for the processor and memory buffer chips. It must determine which concrete device driver to use based on the supplied target.

**FSI-SCOM device driver**. This device driver provides an SCOM interface for the slave processor and memory buffer chips when the processor buses are not configured. It is slower than other SCOM types due to the overhead of using FSI.

**In-band SCOM device driver**. This device driver provides an SCOM interface for the memory buffer chips when the DMI buses are configured.

**XSCOM device driver.** This device driver provides an SCOM interface for the master processor chip always, or to the slave processor chips only when the processor buses are configured.

**I2C device driver**. This device driver is used to read and write inter-integrate cicuit (I2C) devices. It must also provide presence-detect capabilities to determine the presence of dual inline memory modules (DIMMs).

**VPD device driver**. This device driver reads or writes the field replaceable unit (FRU) or module vital product data (VPD) and abstracts the caller from knowing the details of how to access a VPD device.

**EEPROM device driver**. This device driver reads or writes EEPROM devices (such as the SEEPROM) and abstracts the caller from knowing the details of EEPROM access

**PNOR device driver**. This device driver reads or writes either side of the processor NOR (PNOR) flash device attached to the master or alternative master processor chip to provide access to code and data images and configuration settings.

**FSI master device driver**. This device driver performs a flexible service interface (FSI) bus walk to configure and detect FSI-based devices/engines. It provides support for FSI-based device drivers and supports presence detect. It does not support read/write access from outside the device-driver framework.

## Base Enablement Libraries and Services ##
The following base enablement libraries and services are required by the hostboot.

**Error service**. This service and client interface creates and logs errors with the FFDC, persists them for offload, and propagates them to the BMC, if present, to facilitate problem isolation.

**Trace service**. This service logs component traces, persists them for offload, and propagates them to the BMC, if present, to facilitate problem isolation

**Progress code service**. This service logs IPL progress codes, persists them for offload, and propagates them to the BMC, if present. Ultimately, the progress codes are displayed on a BMC console, management software, progress code buffer, or other aggregation point, as necessary, to inform the customer of IPL progress and potential hang conditions.

**Host/BMC communication service**. This service allows the hostboot to perform asynchronous, bidirectional communication with a BMC, if present, and vice versa.

**Virtual file system (VFS)**. This service loads and executes code images or modules and provides a file abstraction for various data sources.

**Minimal Libc**. This library reproduces an optimized set of typical libc APIs to support basic string and memory manipulation.

**Minimal LibC++**. This library reproduces an optimized set of typical libc++ APIs to support C++-style memory management and provides symbols required to compile hostboot with C++ support.

**Minimal STL**. This template library minimally reproduces the standard template library (STL) container and iterator support for aggregations of arbitrary types.

**PNOR resource provider**. This resource provider abstracts access to sections of PNOR flash of interest to the hostboot from the actual hardware layout, by mapping the data to virtual memory.

# Hostboot Software #

## Software Layout ##

The hostboot source tree layout follows the standard Linux kernel layout.  All source code resides under the src/ tree.  The hostboot kernel source (.C files) are under src/kernel.  The include files for the kernel can be found under src/include/kernel/.  Similarly, the user space source can be found under src/usr/ and src/include/usr.

There are a variety of other directories under usr/src.  The naming convention of these directories and the organization described previously will help you navigate through these directories.

Hostboot firmware makes extensive use of [Doxygen](http://www.doxygen.org/) for its documentation.  Type  `make docs` to generate the HTML documentation based on the doxygen in the code.  The documentation will then be in obj/doxygen/.  Load obj/doxygen/html/index.html in a web browser to view. Our team usually views the doxygen when looking at the source code directly, but the html output can be useful at times.

## Compiling ##
Hostboot firmware uses [buildroot](http://buildroot.uclibc.org/about.html) as its build infrastructure to pull in a variety of required packages such as cross compilers, flash image generators, and source code.To generate the flash image that you put on your system, type:

    source op-build-env
    op-build palmetto_defconfig && op-build

Initial support for compiling hostboot is only provided for recent distributions of x86 Linux&reg; systems. The required cross compilers and all other dependencies are a part of the buildroot package that you  downloaded.  See the included README for more information about generating the required flash image and getting it loaded on your system.

## Targeting Model ##

The targeting model is a software construct within hostboot that provide an association between "targets" within the system. It also provides a way to associate data with those targets.  For example, there is a processor chip target that contains up to 12 EX chiplet targets (core+l2+l3). That processor chip also contains up to eight MCS targets (memory controller unit).

Those MCS units have an association with one or more memory DIMM targets, with which they communicate over a bus.  All of these relationships are present within the targeting model and can be used in software.  One of the first things that the hostboot firmware does when it starts is to inventory the hardware in the system and set the appropriate present and functional states of all targets.

In regards to data, each of the targets has a set of attributes associated with it. These attributes can be used for a variety of purposes. For example, an attribute can be used to track the state of the target or an attribute can be a read-only item that describes characteristics of the hardware such as the target type or the hardware address.

The majority of the targeting model is defined within the target_types.xml file, which is described later.

## Data Driven Design ##

The hostboot software was deliberately designed to have a separation of data and software. The data drives the software.  This makes it easy to put new data (such as new systems with new CPU, memory, I/O, and layouts) in place and have the majority of the hostboot software work as-is. Although there are exceptions, this is a key point to remember as you start contributing to hostboot and building your own systems.

The layout of what images go where within PNOR and the tools to build the PNOR image are defined within the src/build/buildpnor/ directory.

The primary script that is run to generate the data for hostboot is in src/usr/targeting/common/genHwsvMrwXml.pl. The data that genHwsvMrwXml.pl looks at primarily comes from the MRW files.

To modify or create a new attribute, use src/usr/targeting/common/xmltohb/attribute_types.xml.

To create a new target or to associate a new attribute with a target, use src/usr/targeting/common/xmltohb/target_types.xml.

# Hardware Procedure Framework #

The chip initialization is driven by the hardware procedures. Hardware procedures are C++ code that does all of the hardware accesses required to test and initialize the hardware. The order and name of the hardware procedures is defined within the IPL flow document found within this repo. These procedures run within the hardware procedure framework. This framework enables the hardware procedures to be run in a variety of environments and  allows a clear distinction between them and the software around them.

The source code for the hardware procedures and the framework are all under src/usr/hwfp/. The fapi/ directory is the framework that all environments must support. The plat/ directory is the hostboot-specific implementation of required fapi interfaces. The hwp/ directory is where all of the actual hardware procedures reside.

**Note:** Changes to the hardware procedures are strongly discouraged. Most necessary changes in this area can be made by changing the attribute data used by the procedures.

# Hostboot Startup and Boot #

Hostboot firmware is loaded into the L3 cache of the first functioning core.  As hostboot starts to run, it executes the following high-level steps:

- Initialize the kernel and start all threads within the core.
- Load all base libraries (PNOR, VFS).
- Load the extended image and its base libraries (utilities, targeting, mailbox, trace, and so on).
- Run the IPL flow, calling the hardware procedures and initializing the hardware.
- Load OPAL, the kernel-based virtual machine (KVM), into host memory and start executing it.

See the following files:

- src/kernel/kernel.C
- src/usr/initservice/baseinitsvc/initsvctasks.H
- src/usr/initservice/extinitsvc/extinitsvctasks.H
- src/include/usr/isteps/istepmasterlist.H
- src/usr/initservice/istepdispatcher/istepdispatcher.C


# Use Cases and Examples #

## <a id="attrusecase">Add a New Attribute to an Existing Target</a> ##

The attributes and targets within your system are defined within two xml files.  There are a variety of configuration options.

    vi src/usr/targeting/common/xmltohb/attribute_types.xml

Attributes are pieces of data that you  associate with a target type.  Consider the following key points when creating an attribute:

- Name. Conform to the naming conventions of  other attributes and be sure that its name is relevant to its function.
- Data type. Always use the smallest and simplest type for the attribute.
- Persistence. Always use the least amount of persistence required for the attribute.
- Read/write characteristics. Always make it read-only if possible.

Add the new attribute to the target type that you want to associate it with.

	vi src/usr/targeting/common/xmltohb/target_types

Now open up your <SYSTEM>.system.xml and add the new attribute and its default value with each of the required targets.

## Support a new System ##

Start with the palmetto xml file and make a new copy for your new system. Modify it as needed. Update the makefile to generate your new system.

## Support a new PNOR Chip ##

The source code for the PNOR device driver can be found in src/usr/pnor/pnordd.C.

## Support a new LPC Device ##

TBD

## Support a new I2C Device ##

The source code for the I2C devices can be found in src/usr/i2c/.
This directory includes the base I2C DD support (i2c.C) as well as support for an EEPROM device (eepromdd.C), which sits on top of the I2C DD.

Add the appropriate attributes (see <a href="#attrusecase">Add a New Attribute to an Existing Target</a>) as needed to support the new I2C device.

## Support a new BMC ##

TBD



# Glossary #

The following terms are used in this document.

**API** 		Application programming interface. An interface that allows an application program that is written in a high-level language to use specific data or functions of the operating system or another program.

**AVP**		 Architected verification program. A custom payload used to test a processor or other host hardware or software function.

**BMC**		 Baseboard management controller. An industry-standard service processor.

**CE**	 Correctable error. A hardware error that the firmware detects and corrects without impacting the state of the system.

**CEC**	    Central electronics complex.

**DCM**	    Dual-chip module.

**DIMM**	Dual inline memory module. A small circuit board with memory-integrated circuits containing signal and power pins on both sides of the board.

**DMI**	Differential memory interface. The memory bus instance of the EDI, which connects a memory buffer chip to the processor.

**DPSS**	Digital power systems sweep. A field-programmable gate array (FPGA) that controls power to various hardware components and performs such functions as controlling memory DIMM voltages, triggering platform power state changes, and managing a timer.

**DRAM**	Dynamic random access memory. Storage in which the cells require repetitive application of control signals to retain stored data.

**EDI**	Elastic differential interface. A bus that consists of high-speed differential I/O links. The memory bus instance of an EDI bus is a “DMI bus”, and the off-module, fabric bus (between processors) instance of an EDI bus is an “A bus”.

**FFDC**	First failure data capture.

**FRU**	Field-replaceable unit. An assembly that is replaced in its entirety when any one of its components fails.

**FSI** Flexible service interface. The POWER8 diagnostic and control interface used by firmware to initialize and service the CEC hardware. It runs between all processor chips in the system.

**Hardware reconfigure loop**	Refers to the process of automatically rerunning select parts of the IPL sequence when DIMMs, processor sockets, or other qualifying pieces of hardware are found to be faulty and  are subsequently deconfigured. The need for such processing arises because, for operational simplicity, the hostboot performs initializations based on the assumption that those parts remain functional throughout the IPL. When the assumed configuration changes due to hardware faults, the initialization sequence must be run with a new set of assumptions.

**HDAT**	Host data area. Collection of attributes, device information, and other derived information used by the payload to manage the computer system.

**Hostboot firmware**	Refers to firmware that runs on the host processors to initialize the memory and processor bus during IPL and at runtime, as well as a supporting an ecosystem of firmware such as command-line utilities, attribute compilers, and simulator macros.

**Hostboot IPL firmware**	A subset of hostboot firmware only active during the IPL.

**HW**	Hardware. With respect to the hostboot, any hardware that it interacts with.

**HWP**	Hardware procedure. A “black box” code module supplied by the hardware team that initializes CEC hardware in a platform-independent fashion.

**HWPF**	Hardware procedure framework. A code layer that executes black box hardware procedures for different platforms and environments. Also refers to a hostboot component that integrates the HWPF along with additional platform-unique implementations.

**I2C** Inter-integrated circuit.

**IPC**	Interprocess communication. Refers to communication between two or more processes or tasks.

**IPL**	Initial program load. The time between when power is applied to the platform hardware and when the payload is fully functional.

**Istep**	IPL step (major or minor), which encompasses the logic and procedures defined for it by the hostboot IPL Flow document. A minor Istep is the smallest quantum of IPL execution.

**KVM**	Kernel-based virtual machine. A virtual machine implementation that uses the operating system kernel (typically Linux).

**LPC**	Low pin count. Refers to a bus used to connect low bandwidth devices to a processor

**MCS**	Memory controller synchronous. A processor unit that provides the memory controller interface to the fabric at processor bus speeds and supports selective memory mirroring.

**MMIO**	Memory-mapped I/O. Refers to using the same address bus to access both memory and I/O devices (that is, the CPU instructions used to access the memory are also used to access devices).

**MRW**	Machine readable workbook. An XML description of a machine as specified by the system owner.

**NOR**	In Boolean logic, the negation of a logical OR.

**OPAL**    OpenPOWER abstraction layer. The software that runs underneath the operating system and contains kernel-based virtual machine (KVM) and hostboot runtime services.

**Payload**	Any software entity to which the hostboot hands off execution and that acts as the primary server operating environment.

**PCB**	Pervasive control bus. Processor logic that provides a generic, modular structure for communication between pervasive elements (the glue logic between chiplets).

**PIB**	Pervasive interconnect bus. A bus that provides access from masters through external interfaces and internal masters to common PIB attached slaves.

**PNOR**	Processor NOR. NOR memory device where all firmware, including the hostboot firmware, is stored and from which it is loaded. It is attached to the master or alternate master processor through an SPI bus.

**PORE**	Power-on reset engine. A processor engine that initializes various other hardware entities using a simple instruction image.

**RAS**	Reliability, availability, and serviceability. A combination of design methodologies, system policies, and intrinsic capabilities that, taken together, balance improved hardware availability with the costs required to achieve it. Reliability is the degree to which the hardware remains free of faults. Availability is the ability of the system to continue operating despite predicted or experienced faults. Serviceability is how efficiently and nondisruptively broken hardware can be fixed.

**SBE**	Self-boot engine. Specialized PORE that initializes the processor chip, and then loads or invokes the hostboot IPL firmware base image.

**SCAN**	Refers to shifting groups of latch states, internal to a chip,  used to read or write it when functionally not in use.

**SCOM**	Scan communications interface. A specialized interface into the chip pervasive logic that allows specific latches to be updated while functionally in use.

**SDRAM**	Synchronous dynamic random access memory. A type of dynamic random access memory (DRAM) with features that make it faster than standard DRAM.

**SEEPROM**	Serial electrically erasable programmable read-only memory. An EEPROM memory device that can only be read, but which can be reprogrammed by an external programmer. Note that the hostboot is able to initiate writes to SEEPROMS connected to the circuit board through a serial bus.

**Service** A set of public APIs, tasks, and internal functions that logically perform a major function within the hostboot environment. In the hostboot context, a service can be as elaborate as a C++ object with its own worker threads or as simple as a single function.

**Slave chip**	Any host processor chip that is not a master chip at an instantaneous moment of time.  That is, the alternate master processor chip is considered to be a slave chip when not elected as the instantaneous master processor chip.

**SPD**	Serial presence detect. SDRAM features an on-board SPD chip that contains information about the memory type, size, speed, and access time. This chip lets the computer access this information at start-up while it goes through its power-on test cycle.

**SPI**	Serial peripheral interface. Refers to a 4-wire, serial, full-duplex bus with masters and slaves. Commonly used to interface with sensors, control devices, and so on in embedded systems.

**STL**	Standard template library.

**VFS**	Virtual file system. An abstraction layer that allows applications to access data and manipulate data as files. Typically used by hostboot to execute code modules.

**VPD**	Vital product data. Information that uniquely defines system, hardware, software, and microcode elements of a processing system. The VPD is stored within SEEPROMs, where persistent FRU-specific data is stored.

**Winkle**	Special very-low power state for a processor core, where most of it, and the surrounding logic, is shut off. Because this state is destructive to core initialization settings, a core-specific winkle image containing initialization instructions must be executed by the PORE to bring the core back online. Core winkle can only occur when all hardware threads for a given core commit themselves to the winkle state.

**XSCOM**	Special, fast SCOM that allows the processor cores to directly SCOM the PIB. Processor buses must be enabled to the slave processor chips for the master processor to XSCOM them.

# Copyright and Disclaimer #

© Copyright International Business Machines Corporation 2014

IBM, the IBM logo, and ibm.com are trademarks or registered trademarks of International Business Machines Corp., registered in many jurisdictions worldwide. Other product and service names might be trademarks of IBM or other companies. A current list of IBM trademarks is available on the Web at “Copyright and trademark information” at www.ibm.com/legal/copytrade.shtml.

Linux is a registered trademark of Linus Torvalds in the United States, other countries, or both.

Other company, product, and service names may be trademarks or service marks of others.
All information contained in this document is subject to change without notice. The products described in this document are NOT intended for use in applications such as implantation, life support, or other hazardous uses where malfunction could result in death, bodily injury, or catastrophic property damage. The information contained in this document does not affect or change IBM product specifications or warranties. Nothing in this document shall operate as an express or implied license or indemnity under the intellectual property rights of IBM or third parties. All information contained in this document was obtained in specific environments, and is presented as an illustration. The results obtained in other operating environments may vary.

While the information contained herein is believed to be accurate, such information is preliminary, and should not be relied upon for accuracy or completeness, and no representations or warranties of accuracy or completeness are made.

Note: This document contains information on products in the design, sampling and/or initial production phases of development. This information is subject to change without notice. Verify with your IBM field applications engineer that you have the latest version of this document before finalizing a design.

THE INFORMATION CONTAINED IN THIS DOCUMENT IS PROVIDED ON AN “AS IS” BASIS. In no event will IBM be liable for damages arising directly or indirectly from any use of the information contained in this document.

IBM Systems and Technology Group
2070 Route 52, Bldg. 330
Hopewell Junction, NY 12533-6351

The IBM home page can be found at [ibm.com®](http://www.ibm.com/us/en/).

July 2, 2014
