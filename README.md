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

All data (including packet headers) is sent as 9 bit bytes. The last bit should be the opposite of whatever the last bit in the data byte is. In case of length corruption, this helps to identify when the slave device is done sending a message. 

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

Determine what devices exist

Packet xfer for retransmission and error checking

1. Packet sent from master to request "does device X" exist, where X is 0 to 128
2. Wait for ACK
3. Packet sent from slave to indicate presence and any additional information
4. Master moves on to next slave ID


# Packet format

Packets have a specific format and can transmit up to 255 bytes at a time. If the length field is set to 256, then 256 bytes are transmitted, but also this tells master that there is additional information that is waiting to be sent from the slave. This can also be used from the master, though is probably less useful for slaves to know. 

Format:

0. Clock calibration pulses (not recorded)
0. Type (top 4 bits)
0. checksum of length and type (4 bits)
0. Length in bytes (1 byte)
0. data (variable)
0. attempt number (byte) 
0. CRC checksum of entire packet (not counting calibration) (1 byte) (excluded if data length is 0)
0. release channel to low for 1 cycle (so that next clock sync packet is identifiable)

The various types of packets determine the data format

0x00: broadcast plain data

Plain data, broadcast for every device to make use of. If the top bit of this type is set, then it isconsidered non-durable, and does not need ACK or Error responses

0x01: (from master) Device query

A device query packet. To save tranmission and initialization times, this is cut as short as possible. The length field is set to the device ID being requested, and the data has a length of 0. IF the bottom bit of the Type byte is set, it is treated as a device query packet. 

0x02: (from slave only) Query response

A message to the master to indicate that the device exists. Length can optionally be a non-zero value to indicate additional information: (TBD)

0x0E: (from master only) Slave OK To Send

Another special packet where the length field is actually the device ID. This is a message from the master to tell a host that it should send whatever data it needs. Non durable

0x0D: Acknowledge

A special packet that drops everything down to a more simplistic format. 

0. type (4 bits)
1. checksum of type (4 bits)

This should be sent in a few different ways

1. When a slave sends any message, the master should send an ACK afterwards
2. When a master sends a message to a particular slave, the slave should send an ACK
3. When a slave sends a message directed to another particular slave, the master should send an ACK, and afterwards the receiving device should send an ACK
4. When the master sends a message to all slaves, to ensure that the message was not corrupted, an ack should be sent from the lowest device ID on the network. 

ACK is not to be sent if the top bit of the type is set, to indicate non-durable. It is recommended that if a durable packet is sent and has errors, the master should request a resend. 

Non durable

0x0F: Error

A special packet that indicates an error was encountered with the last packet. This should only be sent by a slave that was sent a message particularly for it's ID, or by the master. The top bit of this type is set to indicate that it is non-durable. If there is an error receiving an error packet, then well. there's problems that are complicated. 

It uses the same format as ACK

0x03: Message to

Sends a message to a particular device ID. Can be sent from and to either master or slave. Length must be greater than 1 and the first byte of the data is th device ID

0x04: Change protocol (master only)

Indicates that it wil attempt to change the communication speed or other options(TBD). This is performd by the master sending 4 ACK packets complete with clock calibration prefixes. After this process, the master will set the line HIGH for 9 cyles, and then wait for 9 cycles at the OLD SPEED. If during this time any device can't handle this new speed, then it should set the line high for at least 4 cycles during this time. 

This is an optional feature of the protocol due to significant additional complexity
