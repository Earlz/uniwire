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
* Only features required is enough code space and RAM for packet decoding, and a hardware timer capable of tracking microseconds (though not necessary to be very precise)
* Only requirement for the data line is that it needs to be pulled down to ground when idle

# States

## Data format

Data is not encoded in any special way, it is just bits written to the line. No manchester encoding or anything special. Due to the clock calibration step, this should not present any problems. In dynamic environments where clockrates my change, such as in drastically different temperatures, all devices should recalibrate their clock to the master clock every X number of messages

List of states that the protocol goes through(?)

## Clock calibration

At the beginning of each message a special pulse is used to help syncronize the clock and allow each device on the network to properly communicate at right speed

This is done by using special pulses that are 1.5 cycles long. 

0. High for 1.5 cycles
0. Low for cycle
0. High for cycle
0. Low for 1.5 cycles

A device should watch these messages and synchronize to the best of it's abilities over 3 or more messages, in case of errors on the communication line. 

## Master network control

The master device can send various messages to control the network. 

### New Device Check

This will request that any devices not initialized on the network set the line to high. 

Format:

1. F0
2. 0 (pull down for 4 cycles, slaves set high if needed)

### Query Devices


## Master data




## Slave data
