# Proposal requirements at a glance
### Required:

**The proposed IP block should be a block that is useful within the context of a system-on-chip (SoC).** 

Proposals need to provide the following:

- The type of IP block
- List of I/O, including test ports
- Functional description of the IP block
- A target specification with all relevant characteristics, including average, minimum, and maximum values (as appropriate), and absolute limits.
- An outline of a test plan for validating the IP block through measurement.
- List of any equipment available for use in testing (sent separately from the proposal document)
- CVs of the designers (sent separately from the proposal document)

[ see https://opencircuitdesign.com/chipalooza/rules-4.html ]


## List of I/O

 Designers may assume availability of external resources in the form of: 
<img width="511" height="352" alt="image" src="https://github.com/user-attachments/assets/411ae8f4-911f-43a3-8d3e-737e2d2284ba" />


The harness chip will be designed with a 1.8V logic library and the standard Sky130 PDK dual 1.8V core, 3.3V periphery pad library. Analog designs which expect to operate at the voltage range of the periphery should be designed to operate at 3.3V. Designs which expect to operate at 1.8V on-chip but need to interface to the padframe for testing need to ensure overvoltage protection on their inputs and outputs


