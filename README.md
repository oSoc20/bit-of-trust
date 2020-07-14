# Bit of Trust (BoT) Protocol

**Disclaimer** - **This file is a draft, nothing written here is final or represents an end
product.**

## Introduction

The BoT protocol attempts to build identifiers based on the relationships between people. This
means the focus is no longer on the individuals in the system. An example of this in a current
system is your handle on Twitter, which is a unique identifier on their platform. Instead, the BoT
protocol focusses on relationships. For instance, Bob and Alice can declare that they are friends,
and to the system they could be known as `bob-and-alice`, however there are no identifiers for the
individuals Bob or Alice on their own.

We start with a discussion of the system model behind this protocol, that means we will describe
the data we store, and how we store it. The fundamental part of this system are relations, they are
stored in [RDF](https://www.w3.org/TR/rdf-primer/) documents since they are extensible and allow us
to define machine-interpretable semantics. We can access these RDF documents by associating them
with a trust identifier, which represents a handle for a given relationship. A natural way to store
mappings is a hash table, but this limits the scalability of the system to a single machine and we
need much more scalability. Therefore we can opt for a Decentralised Hash Table (DHT) instead, this
has the added benefit of reducing the amount of centralised components in the system.

Following this explanation we discuss the protocol itself, and the syntax and semantics of the
messages that will be exchanged between users. This description is supposed to provide enough
information to implement a working version of the protocol using your technology of choice.

## System Model

Bit of Trust requires a way to store trust descriptions (RDF documents) identified
by trust identifiers. As mentioned before, this really lends itself well to a
(distributed) hash table approach. Ideally, a distributed implementation should meet the following
requirements:

1. The maximal route length between two servers is `O(log(n))` where `n` represents the number of
   nodes in the system.
2. The system is fault tolerant to a certain degree when servers fail, leave, and join.
3. The system should be scalable enough to support thousands or millions of nodes.
4. The nodes in the system collectively make up the system, without central coordination.
5. We need a way to map keys to values, in this case we are mapping trust identifiers
   to trust descriptions. We will explain these concepts in their respective sections.

This seems like a tall order, but most Distributed Hash Table (DHT) systems can satisfy these
requirements. Examples include
[Kademlia](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf),
[Koorde](https://www.ic.unicamp.br/~celio/peer2peer/debrujin-p2p/kaashoek03koorde.pdf),
[Chord](https://pdos.csail.mit.edu/papers/ton:chord/paper-ton.pdf),
[Whanau](https://pdos.csail.mit.edu/papers/whanau-nsdi10.pdf),
[Pastry](http://rowstron.azurewebsites.net/PAST/pastry.pdf), and
[Tapestry](https://www.srhea.net/papers/tapestry_jsac.pdf).

Do note, it may be best not to reinvent the wheel here. There are numerous systems readily
available that implement a DHT with many problems typically faced in implementations already
tackled. Examples of these include [Hyperswarm](https://github.com/hyperswarm/hyperswarm) or
[libp2p](https://github.com/libp2p/js-libp2p).

### Hash Tables

Hash tables operate by associating or mapping ‘keys’ to ‘values’ by use of a hash function. For
example, if we want to map names to file paths a typical hash table might look like
this.

| Key      | Value                |
| -------- | -------------------- |
| Brussels | `/data/brussels.ttl` |
| Ghent    | `/data/ghent.ttl`    |
| Hasselt  | `/data/hasselt.ttl`  |
| Mons     | `/data/mons.ttl`     |
| Namur    | `/data/namur.ttl`    |

However, it is implemented like a contiguous region of memory in which we calculate the position
of a value based on its key. Hash functions are one-way functions that allow you to calculate a
(nearly) unique same-length number for each piece of data you can think of. If we calculate the
`crc32` hash of `Brussels` we get `h('Brussels') = 8b5c2f3b`. This number can be used to index into
an array (region of memory) at the location where we are storing the value accompanying this
specific key.

### Distributed Hash Tables (DHTs)
The idea behind distributed hash tables is that we can distribute this table over multiple
machines, without using a central coordinator. Most DHT systems need to specify the following
things in order to operate.
1. Specify a shared ‘key space’ for both servers and values. That is, what is the finite set of keys
   that we can map to? (e.g. all numbers from 0 to 15 million)
2. Connect servers in the network to each other in such a way that each server can be reached using
   a bounded number of lookups.
3. Develop a strategy to map items to certain servers inside of the network.
4. Deal with ways to let systems join and leave at will, without losing data.

In the interest of brevity, we will not tread into detail on how DHTs are implemented, as this
would warrant the size of a book chapter or research paper. A good place to start is the [Chord
paper](https://pdos.csail.mit.edu/papers/ton:chord/paper-ton.pdf), as this is arguably the most
accessible.

### Trust Identifiers

Trust identifiers are the foundation of the Bit of Trust protocol, they provide an identifier for a
given trust relationship between certain actors. These identifiers are typically created by
combining other trust identifiers. As such, there is also a way to bootstrap these identifiers
which is described in the protocol section.

Trust identifiers are built up like [Merkle trees](https://en.wikipedia.org/wiki/Merkle_tree),
which means they are created in the following way:

1. Assume two existing trust identifiers, these are [Blake2b-256](https://blake2.net/) hashes. For
   example, a [base64](https://en.wikipedia.org/wiki/Base64) encoded hash looks like this:
   `MLlGLgfmwWDxAnoeqYVyGvMNReedVQ41aDGICKqjzPg=`. To keep the description short we will call these
   hashes `A` and `B`.
2. We concatenate hash `A` and `B`, denoted by `A || B`.
3. We take the [Blake2b-256](https://blake2.net) hash of the concatenation like this: `h(A || B)`.
4. This resulting hash is the new trust identifier for the relationship between the trust
   identifiers `A` and `B`.

Each of these trust identifiers has an accompanying RDF document in the hash table. These documents
encode a [directed acyclic graph (DAG)](https://en.wikipedia.org/wiki/Directed_acyclic_graph)
describing all of the previous hashes (subtrees) that were used in the creation of this identifier.
The leaves of the Merkle tree are what we call ‘Trust shards’, and are required for the
bootstrapping process. Note that the amount of actors in a trust relationship can be determined by
counting how many trust shards a trust identifier has (i.e. counting the leaves of the Merkle
tree).

### Trust Shards

Trust shards are the ‘shards’ of a trust identifier, they each constitute a part of the
relationship that can be handed out to actors who are a part of the relationship being represented.
A close analogy is that of owning shares of a company, except the company is a trust relationship
and the shares are not necessarily traded for money.

Owning a trust shard of a given trust relationship enables an actor to invite new actors into the
trust relationship. It should also be possible for this actor to remove other shard holders from
the system through a majority vote, but the exact mechanism for this is still an **open problem**.

We currently represent trust shards by use of asymmetric cryptography (see [QR
challenge-response](/qr-challenge-response.md)), the public key of the trust shard is used as the
building block in trust identifiers, and the owner of the shard has the accompanying private key.
We do this to prevent people from claiming ownership of a shard, when they do not own it. It is
still an **open problem** whether we can remove the use of asymmetric cryptography in favour of a
different method of proving ownership.

Another **open problem** is to come up with a method for exchanging trust shards between different
actors, that is, move the trust shard from one owner to another owner. As well as enabling users to
give up their shard when they no longer wish to be a part of a trust relationship.

The last **open problem** we consider is how users can prove ownership of a shard from any
computing device they wish to use. This is an affordance provided by current username and password
systems that we wish to support as well, but it is still unsure to which degree this would be
possible without making people remember things. This is a potentially big part of being user
friendly, in this same vein, this also means that any use of asymmetric cryptography should be
invisible to actors using the system (i.e. they do not need to know how to operate a terminal to
generate and store keys), and it should be impossible to identify actors through the public key of
their trust shard.

### Trust Descriptions

As of now, a trust description is an RDF document in which we keep track of (or refer to) the
trust identifiers or trust shards that make up a relationship. It is currently still an **open
problem** what the exact structure of these documents will be, and what data they will keep
track of. One possible addition to this document could be a description of the context in which a
relationship takes place, or it could contain references to other resources on the web that the
trust relationship owns, such as a video or a document.

The validity according to our future specification will be enforced by use of
[SHACL](https://www.w3.org/TR/shacl/), and we will provide machine-interpretable semantics by use
of a [RDFS vocabulary](https://www.w3.org/TR/rdf-schema/) or [OWL
ontology](https://www.w3.org/TR/owl2-overview/), which is yet to be decided upon.

## Protocol

```
TODO
```
