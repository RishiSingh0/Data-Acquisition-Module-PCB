# Data-Acquisition-Module-PCB
6 Layer 50.00mm x 50.00mm PCB handling all incoming sensor data on the submarine

# Project Description 
This project is the design and fabrication of a 6-layer PCB to serve as the main data acquisition board for a human-powered submarine. The board's primary function is to collect real-time sensor data and provide visual feedback to the pilot via an LED system. It will interface with a barometer for depth and pressure measurements, temperature/humidity sensor for monitoring internal environmental conditions. The board will also be integrated with a separate tachometer design to measure propeller RPM.  

# Schematic 
![A screenshot of the schematic](assets/Schematic.png)

# Issue regarding I2C
Some of the sensor's I²C bus was unreliable over the long cable run because I²C is a short-distance protocol. The long cable's capacitance slowed the signal's transitions, making it difficult for the receiving device to interpret the data correctly. To solve this, a local microcontroller to read the sensor data via a short I²C connection. The microcontroller then transmitted the data over the long cable using UART, which is a more robust protocol for longer distances. To further improve reliability, the UART signal was paired with Low-Voltage Differential Signaling (LVDS), which uses a pair of wires to send a signal and its inverse. This differential signaling would cancels out external noise. 



# 4 Layer Design

# 2 Layer Design

# Issues and Challenges
