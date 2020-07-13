# Bit of Trust (BoT) Protocol

**Disclaimer** - **This file is a draft, nothing written here is final or represents an end product.**

## Introduction

The BoT protocol attempts to build identifiers based on the relationships between people. This means the focus is no longer on the individuals in the system. An example of this in a current system is your handle on Twitter, which is a unique identifier on their platform. Instead, the BoT protocol focusses on relationships. For instance, Bob and Alice can declare that they are friends, and to the system they could be known as `bob-and-alice`, however there are no identifiers for the individuals Bob or Alice on their own. 

> **Comment:** In the current description of the protocol, this does not exclude the use of individual identifiers as an implementation detail, but these are never public facing or addressable. See the subsection on ‘Device Identifiers’ for more details.

We first discuss the system model that this protocol can operate on. That is, we are assuming an underlying Distributed Hash Table (DHT) in which we keep track of ‘trust identifiers’ and relate them to an RDF document describing the relationship in greater detail. This document contains a number of ‘device identifiers’. In the current version, these are public keys owned by individuals partaking in the relationship. Their primary use is to act as a lookup mechanism to retrieve the relationships somebody is part of. A secondary use, is their ability to act as a way to prove ownership over your device.

> **Comment:** In the ideal case, we would not have any device identifiers in the first place. This is still a way to address individuals which is exactly what we are trying to avoid. If we do not have any device identifiers, we lack a way to talk about membership. This means we can not stop people from claiming they are part of a trust relationship when they are not, and there is no way to revoke their ability to use the trust identifier. This is discussed furter in the subsection on ‘Device Identifiers’.

In the second part we discuss the protocol itself, and the syntax and semantics of the messages that will be exchanged between users. This is supposed to provide enough information to implement a working version of the protcol using your technology of choice.

## System Model

Bit of Trust is designed to operate in distributed systems without a centralised coordinating server, sometimes called peer-to-peer systems or decentralised systems. For a brief explanation of the intuition behind this we refer to [A Brief introduction to (De)centralised Software Systems](https://gist.github.com/tbaccaer/fb3b687581d56b030baefe253c19fbc8). The protocol requires a number of requirements from the system in order to operate.

1. The maximal route length between two servers is `O(log(n))` where `n` represents the number of nodes in the system.
2. The system is fault tolerant to a certain degree when servers fail, leave, and join.
3. The system should be scalable enough to support thousands or millions of nodes.
4. The nodes in the system collectively make up the system, without central coordination.
5. We need a way to map keys to values, in this case we are mapping trust relationship identifiers to device identifiers. We will explain these concepts in their respective sections.

This seems like a tall order, but most Distributed Hash Table (DHT) systems can satisfy these requirements. Examples include [Kademlia](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf), [Koorde](https://www.ic.unicamp.br/~celio/peer2peer/debrujin-p2p/kaashoek03koorde.pdf), [Chord](https://pdos.csail.mit.edu/papers/ton:chord/paper-ton.pdf), [Whanau](https://pdos.csail.mit.edu/papers/whanau-nsdi10.pdf), [Pastry](http://rowstron.azurewebsites.net/PAST/pastry.pdf), and [Tapestry](https://www.srhea.net/papers/tapestry_jsac.pdf).

Do note, we do not specify which algorithm or existing service is used here, it is completely up to the implementor to pick a specific technology such as [Hyperswarm](https://github.com/hyperswarm/hyperswarm) or [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) for their specific usecase. We are just declaring the environment we are expecting to work in.

### Hash Tables

Hash tables operate by associating or mapping ‘keys’ to ‘values’ by use of a hash function. For example, if we want to map server names to `IPv4` addresses a typical hash table might look like this to us.

| Key      | Value              |
| -------- | ------------------ |
| Brussels | `129.138.111.255`  |
| Ghent    | `152.55.236.11`    |
| Hasselt  | `18.136.158.247`   |
| Mons     | `77.221.211.14`    |
| Namur    | `240.252.88.246`   |

However, it is implemented like a contigiuous region of memory in which we calculate the position of a value based on its key. Hash functions are one-way functions that allow you to calculate a (nearly) unique same-length number for each piece of data you can think of. If we calculate the `crc32` hash of `Brussels` we get `h('Brussels') = 8b5c2f3b`. This number can be used to index into an array (region of memory) at the location where we are storing the value accompanying this specific key.

### Distributed Hash Tables (DHTs)
The idea behind distributed hash tables is that we can distribute this table over multiple machines, without using a central coordinator. Most DHT systems need to specify the following things in order to operate.
1. Specify a shared ‘keyspace’ for both servers and values. That is, what is the finite set of keys that we can map to? (e.g. all numbers from 0 to 15 million)
2. Connect servers in the network to each other in such a way that each server can be reached using a bounded number of lookups.
3. Develop a strategy to map items to certain servers inside of the network.
4. Deal with ways to let systems join and leave at will, without losing data.

In the interest of brevity, we will not tread into detail how DHTs are implemented here, as this would warrant the size of a book chapter or research paper. A good place to start is the [Chord paper](https://pdos.csail.mit.edu/papers/ton:chord/paper-ton.pdf), as this is the easiest one to understand.

### Storage Model

The hash table approximately looks like the table below, where the keys are trust relationships and the device identifiers are stored inside a document.

| Key                                      | Value            |
| ---------------------------------------- | ---------------- |
| a2ca2c385c0a3542a668afdcf76df54ec73d9a28 | `a2ca2c385.json` |

The file `a2ca2c385.json` looks like this:

```json
{
    "trust-relationship-id": "a2ca2c385c0a3542a668afdcf76df54ec73d9a28",
    "members": [
        "MIGeMA0GCSqGSIb3DQEBAQUAA4GMADCBiAKBgHOW59RNlahdDht7F5an8ZI6Ik/AHPPyKkVWCHzR+c9djlabD7U3/128h28NQYtXW0TkXuLeg0FLsEqM99TAyibz6dpPbNH3JyWLl8zlY1AFRy57zBzA07k8YhN7zBLPDiiHlS61UoV6aXJuPkbqawdcKwdi5vX6pYkSFlKTTpcDAgMBAAE=",
        "MIGeMA0GCSqGSIb3DQEBAQUAA4GMADCBiAKBgGWKvTO10snnoBIKr0eELCRuhUjNaPihNTDShut3cajBpYxfGkVKjsP5bBY4NJuKacq5kGrWEQ7m0T+cHSeEAzD3a4xH7ymlka5WAAro02cguTYsm3V+c7+oxePd5mQeKGOhtrMIKA/Pgwr8Pd9pdV1y/svhD7a2FlfPAa+YogsjAgMBAAE="       
    ]
}
```

### Device Identifiers

Device identifiers are based on asymmetric cryptography (more specifically: ECC?). Each user generates a random private key and derives a public key from it. The public key is used as their 'device identifier' on the network. They can use their private key to prove ownership of their public key.

Users are responsible for the safe-guarding of their private key, both from attackers and from being lost, and for distributing it across their devices.

Note that a public (or private) key *alone* does not contain or point to any personally identifiable information (PII).

TODO: elaborate further

### Trust Identifiers

Trust relationships are relationships between a group of at least two 'devices'. A device is introduced to the network by setting up a trust relationship with an existing member.

Each relationship can be identified by a hash of two concatenated hashes, each of which being either the identifier of an existing trust relationship (a hash of a subtree), or the hash of a device identifier.

They are thus structured as a Merkle tree, with hashes of device identifiers in the leaf nodes.

Using hashes as identifiers makes it possible to identify (point at, talk about, ...) one's latest known version (generation) of a whole tree without each device needing to store the whole tree.

The hash function to be used is (SHA-3? BLAKE2?), a cryptographic hash function.

TODO: elaborate further

### Context

Human relationships can mean different things to different people. For example, Alice might want to connect with Bob in the context of friendship, but Bob only considers Alice an acquaintance.

Because of this inherent ambiguity, no trust relationship in the system is tied to any specific context.

TODO: elaborate further

## Protocol

TODO
