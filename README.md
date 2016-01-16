# uniwire
A communication protocol based on PJON, but with clock calibration and using single-master mode

## Goals

These are what this should accomplish:

* Compensate and properly handle poor and inaccurate clocks, such as the internal oscillators of the AVR series
* Operate with somewhat predicatable amounts of latency, at the expense of raw data transfer speed
* Be decently operatable using a platform's hardware interrupts on a "value changed" type event 
* Only one single wire and a common ground required for all communications
* Support up to 128 devices on a single network
* Support dynamic device IDs (if possible without collisions?)

## States

List of states that the protocol goes through(?)

### Clock calibration

### Master request for data

### Slave send data

