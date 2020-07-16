![Bit of Trust](https://raw.githubusercontent.com/oSoc20/bit-of-trust/master/img/bot.png)

# Bit of Trust

**This document discribes the technical viewpoint and insights we gained during our work within
Open Summer of Code 2020. We make an attempt at defining core concepts that could serve as system
primitives. Additionally, we provide a list of open problems that could serve as potential avenues
for more research at a later point in time.**

## Introduction

To us, the students, Bit of Trust attempts to build identifiers based on the relationships between
people or organisations. This means the focus is no longer on the individuals in the system. An
example of this in a current system is your handle on Twitter, which is a unique identifier on
their platform. Instead, Bit of Trust focusses on relationships. For instance, Bob and Alice
can declare that they are friends, and to the system they could be known as `@bob-alice`,
however there are no identifiers for the individuals Bob or Alice, such as `@bob` or `@alice`. This
allows us to make explicit some of the implicit trust relationships that are currently being made
online. For example, when you store files on Google Drive, you are in an implicit trust
relationship with Google, and any files you store on Google Drive belong both to you and to
Google. They also derive information about what you like from these files and expose you as a
potential customer to an unkown amount of advertisers that you never know about.

We think this will create more awareness about who actually owns data, and we think that this model
more closely approaches the way humans think about trust. For example, in a normal human
conversations, you know who is a part of the interaction. You also know the information being
exchanged and who knows this information. In the digital world, this is not necessarily true, many
people are not aware that information they exchange with websites is also being shared with
trackers or companies.

The rest of this document discusses some aspects or building blocks that can be used in a technical
implementation, and outlines some potential problems for further technically oriented research.
Bit of Trust is still very broadly defined and most previous definition efforts have been
vision-oriented. We make an attempt at translating some of this vision into actionable technical
research problems that could be investigated.

## Core Concepts

In our version of Bit of Trust, we require a way to store descriptions or information about trust
relationships, we sometimes call them ‘trust bubbles’. We think that storing these descriptions
as an RDF file could prove to be interesting for a number of reasons.

- Descriptions will be extendible with references or statements about anything on the Web.
- We can make use of existing query languages like
  [SPARQL](https://www.w3.org/TR/rdf-sparql-query/) in order to make these descriptions queryable.
- There are powerful tools available to enforce the structure of this description, such as
  [SHACL](https://www.w3.org/TR/shacl/).
- We can make the meaning (semantics) of the data explicit and machine-interpretable, which
  improves interoperability with other systems.

In order to talk about these trust descriptions, we need to refer to them through a trust
identifier. A typical way to associate two values with each other is through use of a hash table.
But since these are usually confined to single machines, we need a way to distribute this hash
table. Examples of distributed hash tables include
[Kademlia](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf),
[Koorde](https://www.ic.unicamp.br/~celio/peer2peer/debrujin-p2p/kaashoek03koorde.pdf),
[Chord](https://pdos.csail.mit.edu/papers/ton:chord/paper-ton.pdf),
[Whanau](https://pdos.csail.mit.edu/papers/whanau-nsdi10.pdf),
[Pastry](http://rowstron.azurewebsites.net/PAST/pastry.pdf), and
[Tapestry](https://www.srhea.net/papers/tapestry_jsac.pdf). Do note, it may be best not to reinvent
the wheel here. There are numerous systems readily available that implement a DHT with many
problems typically faced in implementations already tackled. Examples of these include
[Hyperswarm](https://github.com/hyperswarm/hyperswarm) or
[libp2p](https://github.com/libp2p/js-libp2p) which contains primitives for peer-to-peer
systems. If this were to be implemented in a distributed, but not necessarily peer-to-peer
environment then a document store is also a valid option.

The last piece of the puzzle is a way to signify that people or organisations (henceforth called
actors) are a part of a given relationship, without necessarily creating a personal identifier for
them. Our solution to this is the concept of a trust shard. These are parts of a trust identifier
that actors who are part of a given relationship own. This allows them to influence the trust
relationship, such as inviting or removing actors. These shards should be seen as interchangeable
tokens, and should in no way identifiy a person, kind of like a membership card, but without this
necessarily identifying the actor in question.

### Trust Identifiers

Trust identifiers are the foundation of Bit of Trust, they provide an identifier for a
given trust relationship between certain actors. They are typically created by combining other
trust identifiers. As such, there is also a way to bootstrap these identifiers, which we handle
through trust shards.

Trust identifiers are built up like [Merkle trees](https://en.wikipedia.org/wiki/Merkle_tree),
which means they are created in the following way:

1. Assume two existing trust identifiers, these are [Blake2b-256](https://blake2.net/) hashes. For
   example, a [base64](https://en.wikipedia.org/wiki/Base64) encoded hash looks like this:
   `MLlGLgfmwWDxAnoeqYVyGvMNReedVQ41aDGICKqjzPg=`. To keep the description short we will call these
   hashes `A` and `B`.
2. Sort the hashes `A` and `B`, so we get `H1` and `H2` where `H1 < H2`.
2. We concatenate hash `H1` and `H2`, denoted by `H1 || H2`.
3. We take the [Blake2b-256](https://blake2.net) hash of the concatenation like this: `h(H1 || H2)`.
4. This resulting hash is the new trust identifier for the relationship between the trust
   identifiers `A` and `B`.

What we mean by ‘bootstrapping’ here is that the leaves of this merkle tree are infact the trust
shards, and all the subtrees inbetween are trust identifiers. One of the requirements for trust
descriptions should be that they keep track of the previous two trust relationships making up the
current trust relationship.

### Trust Shards

Trust shards are the ‘shards’ of a trust identifier, they each constitute a part of the
relationship that can be handed out to actors who are a part of the relationship being represented.
A close analogy is that of owning shares of a company, except the company is a trust relationship
and the shares are not necessarily traded for money.

Owning a trust shard of a given trust relationship enables an actor to add or remove actors from
the trust relationship. Just like being a shareholder means you can participate in board meetings
and make changes. However, the fundamental difference is that there is no notion of somebody having
more shares than someone else, everyone has an equal share in the relationship, and this also means
equal power.

### Trust Descriptions

As of now, a trust description is an RDF document in which we keep track of (or refer to) the
trust identifiers or trust shards that make up a relationship. It is currently still an open
problem what the exact structure of these documents will be, and what data they will keep
track of. One possible addition to this document could be a description of the context in which a
relationship takes place, or it could contain references to other resources on the web that the
trust relationship owns, such as a video or a document.

The validity according to a given specification can be enforced by use of
[SHACL](https://www.w3.org/TR/shacl/), and we could provide machine-interpretable semantics by use
of a [RDFS vocabulary](https://www.w3.org/TR/rdf-schema/) or [OWL
ontology](https://www.w3.org/TR/owl2-overview/).

## Research Problems

| Problem                        | Description                                            |
| ------------------------------ | ------------------------------------------------------ |
| 1: Trust Shards                | [1-trust-shards.md](/problems/1-trust-shards.md)       |
| 2: Trust Shard Exchange        | [2-trust-shard-ex.md](/problems/2-trust-shard-ex.md)   |
| 3: Trust Shard User Experience | [3-trust-shard-ux.md](/problems/3-trust-shard-ux.md)   |
| 4: Trust Shard Capabilities    | [4-trust-shard-cap.md](/problems/4-trust-shard-cap.md) |
| 5: Trust Descriptions          | [5-trust-descr.md](/problems/5-trust-description.md)   |
| 6: Security & Law Complicance  | [6-sec-and-law.md](/problems/6-sec-and-law.md)         |

You can make a pull request if you wish to add more problems to this list.

## Appendix

This section contains some brief explanations and pointers towards resources to learn more about
concepts used in the main portion of the text.

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

