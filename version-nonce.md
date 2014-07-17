# Preamble
BIP:
Title: Extra nonce in block header (vNonce)
Author: Timo Hanke 
Discussions-To:
Status:
Type: Standards Track
Created:
Post-History:
Replaces:
Superseded-By:
Resolution:
# Abstract
There are incentives for miners to find cheap, non-standard ways to generate new work, which are not necessarily in the best interest of the protocol.
In order to reduce these incentives this proposal re-assigns 15 bits from the version field of the block header to a new extra nonce field called _vNonce_.
# Copyright
# Specification
The block version number field in the block header is reduced in size from 32 bits to 16 bits, occupying now only the first two bytes of the header.
The third and fourth byte in the block header are assigned to the new extra nonce field (vNonce) inside the block header. 
However, the most significant bit of the vNonce (as a little-endian number) is always zero and the vNonce is effectively 15 bits long.  
For informational purposes the version number of blocks containing a non-zero vNonce is set to 3.  
# Motivation
The motivation of this proposal is to provide miners with a cheap constant-complexity method to create new work that does not require altering the transaction tree.

Furthermore, the motivation is to protect the version and timestamp fields in the block header from abuse.
# Rationale
Traditionally, the extra nonce is part of the coinbase field of the generation transaction, which is always the very first transaction of a block.
After incrementing the extra nonce the minimum amount of work a miner has to do to re-calculate the block header is a) to hash the coinbase transaction and b) to re-calculate the left-most branch of the merkle tree all the way to the merkle root.
This is necessary overhead any miner has to do besides hashing the block header itself.
We shall call the process that leads to a new block header from the same transaction set the _pre-hashing_.

First it should be noted that the relative cost of pre-hashing in the whole mining process depends on the block size.
(At least this is true if pre-hashing is done in the traditional way described above.) 
This may create the unwanted incentive for miners to keep the block size small. 
However, this is not the main motivation for the current proposal.

While the block header is hashed by ASICs, pre-hashing typically happens on a CPU because of the greater flexibility required.
Consequently, as ASIC cost per hash performance drops the relative cost of pre-hashing in the whole mining process increases.
This creates an incentive for miners to find cheaper ways to create new work than by means of pre-hashing.
An example of this currently happening is the on-device rolling of the timestamp into the future.
These ways of creating new work are unlikely to be in the best interest of the protocol.
For example, rolling the timestamp faster than the real time is unwanted (more so on faster blockchains).

The version number in the block header is a possible target for alteration with the goal of cheaply creating new work.
Currently, blocks with arbitrarily large version numbers get relayed and are accepted by the network.
As this is unwanted behaviour, there should not exist any incentive for a miner to abuse the version number in this way. 

The solution is to reduce the range of version numbers from 2^32 to 2^16 and to declare the freed two bytes of the block header as legitimate space for an extra nonce.
This will economically reduce the incentive for a miner to abuse the shortened version number by a factor of 2^16.
# Backwards Compatibility
The proposal is implemented without causing or requiring any blockchain forks.

Old nodes (before this BIP) accept all block headers whose first 4 bytes represent a little-endian integer (the version number) greater or equal to 2.
However, since the 4 version bytes are interpreted as a signed integer the most significant bit, which lives in the 4th byte, must be 0.
If it is not then the version number will be interpreted as smaller than 2 (because it is negative) and the block will be rejected.
Therefore we require that the most significant bit of the vNonce (as a little-endian number) is unset.
New nodes explicitly reject blocks which have the highest bit of vNonce set.
This makes the vNonce effectively 15 bits long.  

Old nodes will accept blocks created by new nodes with any 15 bit vNonce,
so old and new nodes can co-exist without any fork happening.
When a majority of the last 100 blocks has a non-zero vNonce then old nodes will create an update alert to the user.

The new block version number 3 is introduced only for informational purposes to make the block creator's intention clearly visible.
There is no code to phase out version 2 blocks.
Strictly speaking it is not necessary to introduce a new version number for this BIP at all. 
# Reference Implementation
See github:

Previously the data type of the version was $int$ (signed).
Now the data type of version is $unsigned short$ so that the full 16 bits are available for version numbers.
The data type of vNonce is also $unsigned short$. 

The reference implementation creates new block headers by incrementing vNonce in an inner loop and extraNonce in an outer loop. 
These block headers are served through the GBT protocol.
# Final implementation

