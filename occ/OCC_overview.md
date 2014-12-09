# OCC Overview #

**O**n **C**hip **C**ontroller is a PPC 405 hard real-time subsystem embedded within the IBM POWER8 processor.  There is one OCC per POWER processor chip.  OCC hardware logic and associated real-time subsystem firmware provides the following:

- Keep the system thermally safe by monitoring memory and processor core temperatures 
- Keep the system power safe by monitoring total system power and quick power drop line

See [OCC Firmware Interface Specification](https://github.com/open-power/docs/blob/master/occ/OCC_OpenPwr_FW_Interfaces.pdf) for more information.


## OCC Software Architecture ##

![](http://i.imgur.com/rpVv1zB.png?1)

###**AMEC**:###
  **A**utonomic **M**anagement of **E**nergy **C**omponent is the heart of the OCC firmware and utilizes the capabilities of the hardware to continuously monitor temperature and power.AMEC processes system data and populates sensors (e.g. system power and processor temperature).  This data is used to feed the control algorithms that make the decision of when and how to actuate.  This actuation takes place via different mechanisms such as changing the maximum Pstate and memory throttling.  Once a control loop has made a decision of where the system should be, it generates a vote.  All votes from all control loops are then tallied and the winner gets to drive the system.  In order to accommodate all these functions, a state machine regulated by time needs to be defined inside AMEC.  The AMEC state machine consists of eight states that are executed on consecutive ticks, where each tick happens every 250us meaning each state will get executed every 2ms.  Each state can be further sub-divided into further sub-states.  For instance:

- Having four sub-states means that each will get executed every 8ms
- Having eight sub-states means that each will get executed every 16ms

On any given tick, only one of all these sub-states will get executed.  If there is a need for a longer time delay in the execution of certain functions, a given sub-state could be broken down into additional sub-states.  

Functions that need to be executed on every 250us tick will be called outside the state machines.

The following diagram illustrates the flow of the state machine.  

- There must be at minimum, a smh_tbl for the 'states'  
- All substate & sub-substate tables are optional.  This option allows for substates & sub-substates while still keeping SRAM space savings, as well as real time loop performance in mind.  
- NULL pointers are always allowed to indicate no function to run for a given state, or to indicate no substate/sub-substate for the current state/substate.

![](http://i.imgur.com/RJaXFMR.png?1)

  
###**GPE**:###
  **G**eneral **P**urpose **E**ngine programs are compiled as part of the OCC FW build by using the GNU Assembler.  The compiled images will be stored in the OCC SRAM space.  GPE has full access to OCC SRAM tank.  GPE programs can take one 32 bit argument when they are called.  This can be used to pass a pointer to a struct or any other args that need to be passed.  They can run on PORE-GPE0 or PORE-GPE1.  GPE0 is used to read power data from the APSS  and processor core data.  GPE1 is used to read memory temperatures.

###**APSS**:###
  **A**nalog **P**ower **S**ubsystem **S**weep provides real time power measurements of voltage rails.  The OCC reads the APSS by sending commands over the SPI bus.
  

## Data Driven Design ##

The OCC is designed to have a separation of data and software.  The data is sent from the MRW files by the HTMGT (Host Thermal Management) component of Hostboot to the OCC.  See [Hostboot Programmers
Guide](https://github.com/open-power/docs/blob/master/hostboot/HostBoot_PG.md) for more details on data driven design.

## Compiling OCC ##

OCC firmware uses [buildroot](http://buildroot.uclibc.org/about.html) as its build infrastructure to pull in a variety of required packages such as cross compilers, flash image generators, and source code.

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

December 2014
