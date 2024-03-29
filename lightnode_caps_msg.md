# Adaptive nodes capabilities protocol (SWIP PIONEER CANDIDATE)

## SWIP SERIAL

1

## STATUS

Approved

## PROBLEM

Adaptive node operation means a node should be able to change the capabilities it has according to changes in available resources.

The peers of the node may need to adjust their behavior towards the node when these capabilities change. Therefore, a protocol is required to inform the peers of changes in capabilities.

## PROPOSAL

Upon initial connection, capabilities can be transmitted already within the handshake message.

Capabilities are combinational. Some combination may make more sense than other. Interpreting which combinations are feasible and which are not should be handled by respective nodes, and not be hardwired in the protocol.

For this reason, a bit vector seems the most logical choice. The bit vector can represent the adaptive node operation state within the node, and be embedded into the protocol message as-is.

Since the capabilities may have to be set from different system modules, the network module should provide an API for registering them. The following structure is proposed:

* the first byte is the identifier of the module (0x00 is bzz, and built in)
* the second byte is the length `n` of the bit vector containing the flags
* the last `n` bytes is the bit vector containing the flags


For the individually capability flags we will initially be choosing from, we can see an example of two distinct byte pairs representing tematic categories:

```
BZZ/SYNC: 			// (registered by default)
- retrieve		0x00020001
- push			0x00020002
- retrieve relay	0x00020010
- push sync relay	0x00020020
- storer node		0x00028000	// the node participates as storer for area of responsibility

PSS:
- receive	0x010101
- send		0x010102
- relay		0x0l0110
```

We define a new protocol message for the `bzz` protocol called `Capabilities`. This message contains one field holding all module capability bit vectors.

```
Capabilities {
	Module [][]byte
}
```

This protocol message can be sent at any time to inform peers of _changes_ to the node's capabilities. 

Furthermore, the bzz handshake message itself should also embed this message instead of the `Light` field that is currently there. The `HandshakeMsg` message this becomes:

```
HandshakeMsg {
	Version		uint64
	NetworkID	uint64
	Addr		*BzzAddr
	Mode		Capabilities
}
```

The network module API can merely consist of:

```
RegisterCapabilityModule(id byte)	// called during setup, before node is started
SetCapability(id byte, flags byte) 	// turns on flags set to 1
RemoveCapability(id byte, flags byte)	// turne off flags set to 1
```

## COMPATIBILITY

The version number of the bzz protocol will have to be incremented after this change is implemented.

We hereby abandon the notion of categorical "full" and "light" nodes completely. Currently no functional light mode operation is implemented in Swarm. In the interim, to emulate the `Light` field in the bzz handshake, capabilities for `BZZ/SYNC` (builtin id `0x00`) should simply be set to `0x80` for `Light = false` and `0x00` for `Light = true`

## AUTHORS

Louis Holbrook - @nolash

## COPYRIGHT WAIVER

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/)
