![Bit of Trust](https://raw.githubusercontent.com/oSoc20/bit-of-trust/master/img/bot.png)

# Bit of Trust

**This document discribes the insights we gained during our work within Open Summer of Code 2020.**

## Introduction

The current state of the Web is dominated by Big Data. A small number of large technology companies
own user data. They use it in order to provide companies with a platform that can market towards
highly specific user groups. Not only is this a privacy concern, it also destroys competition. As a
small company today, it is hard to compete based on service since data is aggregated at the large
companies.

There are systems like MIT's [SOLID](https://solid.mit.edu/) that propose the idea of personal
data vault. This would mean that your personal data is stored in a location that you fully control,
and companies wanting to provide you a service query you for the data they wish to use. There is
access control which means that you get to decide who gets to use the data. Another approach to
this are peer-to-peer filesystems like [Hypercore](https://hypercore-protocol.org/), and
[IPFS](https://ipfs.io/). Each user has a number of drives in which they can put their data, and
everyone has read-only access. Only the creator of the drive has write access. This would also mean
that applications relying on personal data query you for your data instead of owning it.

There is a shortcoming with the two previous approaches, which is their reliance on data being
owned and identified by individuals. In practice, there is also data that belongs to a
relationship. An example of this is a university degree, which is a document established between an
individual and a university. Of course, you could write your own degree but its value comes from
someone you trust to be credible vouching for your abilities. Additionally, if data is in a shared
data pod, how can you trust that this data is correct? How can you see who is using your data?

Bit of Trust is what you get when relationships form the core of a system. In order to make
progress we wanted to gain a better understanding, explore the problem, and work on communication
of this idea towards the outside world. The rest of this document is structured based on
challenges. We briefly describe each challenge. For each of these, we discuss what we have tried
during Open Summer of Code, and provide some potential next steps or ideas to read into.

## Challenges

## **Story**: How do we make Bit of Trust explainable?

In order to get people on board with the project, it would be useful if there was a way to
communicate the ideas within Bit of Trust clearly. We think that the following things are important
here:

- It should be explainable in **30 to 60 seconds**, possibly in a pitch format.
- The **value** should be clear immediately (i.e. What day-to-day problem does it solve?).
- It should use **simple, clear, and to the point** language.
- It should be understandable for **non-technical** audiences.

Having spent close to a month on this project, we realise this is a very tall order. But, in order
to motivate more people to work on the project, this is a really valuable challenge to resolve.

### What have we tried? What went wrong?

Weekly pitching sessions are a part of Open Summer of Code. These are about one minute blocks of
time that each team has to communicate their project, their progress, and what they have learned to
the other teams. These were very valuable moments for us, since it really forced us to find the
essence of what Bit of Trust is about, without being too vague. This had varying success however,
since our understanding of Bit of Trust got refined many times over the course of the project.
We will go over the ideas that stuck with us, or seemed to work when conveying concepts.

Our use of analogies was valuable in order to make a non-technical audience understand what we
were working on. However, it was tricky to convey some concepts with the right accuracy. During one
of our first pitches we used the ‘bubbles’ analogy, in which we equated trust relationships to
bubbles of people allowed to visit you during the corona virus pandemic. The problem with this, is
that it is specific to a point in time, and other countries outside of Belgium may have used
different terminology.

During our third pitch, we decided to introduce some terminology in order make it easier to talk
about the project. The first term we introduced was ‘trust identifier’ which we defined as the
means with which people can refer to a trust relationship. In order to convey the shared ownership
of these trust relationships we invented ‘trust shards’. These are essentially an equally sized
piece or component of a trust relationship used to prove membership. And finally, the log where we
keep track of which trust shards make up a trust identifier, would be noted down in a trust
description. Of course, it is already clear that this terminology hints at a certain implementation
choice, which pinpoints one of the difficulties about this challenge. It is hard to explain
something at a high-level, without keeping in mind possible implementations.

One of the use cases we used to pitch the project was about identity recovery. In most online
systems, forgetting your password means you have to ask the website you are authenticating against
to send you a ‘recovery email’. This will allow you to choose a new password. But, what if you also
forget the password of your personal email? With Bit of Trust we could model what happens in a
non-digital environment. Some people leave a spare key with someone they trust, and you can ask
this person to come and open the door for you. To translate this to a digital environment, we could
authenticate someone based on the people they trust.

`TODO: Include the Demo Day pitch video here`

### What are potential next steps?

A potential story around Bit of Trust could be framing it as a mechanism for making implicit trust
relationships more explicit. For example, in a normal human conversations, you know who is a part
of the interaction. You also know the information being exchanged and who knows this information.
In the digital world, this is not necessarily true, many people are not aware that information they
exchange with websites is also being shared with trackers or companies. When you store files on
Google Drive, you are in an implicit trust relationship with Google, and any files you store on
Google Drive belong both to you and to Google. They also derive information about what you like
from these files and expose you as a potential customer to an unknown amount of advertisers that
you never know about.

In order to really turn this story into something engaging, we think that it still lacks some way
to easily explain how we will achieve this explicitness. Of course, this seems to be dependent on
an implementation. One way would be to log access to the people interacting with the data, and note
down their reason for accessing it. We could have some form of access control based on that, to
prevent people from interacting, or we could allow the interaction and create a fork of the shared
data. These are some potential explanations, but they remain rather vague.

## **Naming**: How can we refer to relationships?

In order to talk about relationships digitally, it would be useful if we have a way to talk about
them. That is, we have a need to identify or refer to a given relationship. Though, there are a few
points we have to keep in mind when doing this.

- The name **can not personally identify** any member of the relationship.
- It should be **human-readable**, that is, do not expect users to remember long hashes along the
  lines of `a8e9bd0f4df60ed5246a1b1f53d51a1feaeb1315266f769ac218436f12fda830`.
- Identifiers should be built **recursively**, this essentially equates to being able to use the
  name of a relationship to construct another name for the aggregate of these two relationships.

We should prevent personal identification for privacy reasons. Human readability is important
because people are accustomed to referring to a ‘handle’ they use on a given platform. This makes
it easier to share and refer to the relationship.

### What have we tried? What went wrong?

#### Can we represent names by hashes?

One potential model for naming users came from our idea of ‘Trust Identifiers’ which is based on
the idea of a [Merkle tree](https://en.wikipedia.org/wiki/Merkle_tree). Since we want names to
support being built ‘recursively’, this seemed like a natural fit at first.

We briefly explain how the process of building a name using Merkle trees could work:
1. Assume two existing relationships, these are represented by hashes. For example, an encoded hash
   looks like this: `5a80571818aee6f7fe7b800179d0b905eb6595f4`. To keep the description short
   we will call these hashes `A` and `B`.
2. Sort the hashes `A` and `B`, so we get `H1` and `H2` where `H1 < H2`.
3. We concatenate hash `H1` and `H2`, denoted by `H1 || H2`.
4. We take the hash of the concatenation like this: `h(H1 || H2)`.
5. This resulting hash is the new name for the relationship between the relationships `A` and `B`.

While this approach prevents personal identification, and manages to allow for recursive
construction, there are some problems with this approach:

1. Hashes are not human readable
2. There is a **bootstrapping** problem, what are the leaves of the Merkle tree?.

#### Can we represent names by random words?

Another model, is to name trust relationships through a random list of nouns and adjectives. For
example, a name could look like `6-sad-squid-snuggle-softly` or `8-mad-orcs-stomp-loudly`. This
idea comes from an [Asana blogpost](https://blog.asana.com/2011/09/6-sad-squid-snuggle-softly/) and
some accompanying npm libraries such as [Greg](https://github.com/linus/greg/) and
[human-readable-ids-js](https://www.npmjs.com/package/human-readable-ids).

There are also a number of problems here:

1. The name space depends on the used naming scheme, which means that we could run into a
   scalability issue later down the line.
2. This method does not recursively construct the name.

#### Why not let users pick their own name?

A critical reader may note that most platforms online today let users pick their own names or
handles. The reason why this does not work here, is because we want to prevent users from exposing
their personal identity through their relationship naming. For instance, if Alice and Bob want to
create a digital relationship, they could choose to call their relationship `Alice-Bob`. This would
expose both of their personal identities. A relationship name should only be meaningful to the
people inside the relationship, and not to the outside world.

### What are potential next steps?

#### Improving upon the hashing approach

Hashes not being human readable may be overcome with some form of reference system, which is
similar to [what Git does](https://git-scm.com/book/en/v2/Git-Internals-Git-References). The
bootstrapping problem is a rather large issue though. One of our ideas was to use our concept of
‘trust shards’ to represent the leaves of the Merkle tree. These would be implemented using
asymmetric cryptography, that is, the public key would be the leaf, and the private key would be
held by the owner of the shard. The main idea being, we needed a way to prove that somebody owns a
leaf of the Merkle tree.

We envisioned this mechanism as a sort of challenge-response mechanism, whereby a peer in the
network can ask a holder of a trust shard to prove that they actually own it. This mechanism would
work as follows:

1. The peer presents a ‘question’ or ‘challenge’ and encrypts it using the shard's public key, so only
   the holder of the shard's private key would be able to read the challenge.
2. The holder receives this message and decrypts it using their private key.
3. The holder solves the challenge and (assuming a secure connection) sends the answer to the
   challenge back to peer.
4. The holder has now proven that they own the shard.

This idea is based on [challenge-response
authentication](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication). There may
be different avenues to explore here. The essence is that we need some form of token that can serve
as a bootstrap from which the rest of the Merkle tree can be built.

#### Improving upon the random words approach

`<TODO>`

#### Looking for more alternatives

`<TODO>`

## **Onboarding**: How are relationships established?

`<TODO>`

### What have we tried? What went wrong?

`<TODO>`

### What are potential next steps?

`<TODO>`

## **Evolution**: How do relationships evolve over time?

`<TODO>`

### What have we tried? What went wrong?

`<TODO>`

### What are potential next steps?

`<TODO>`

## **Capabilities**: What are people able to do with a relationship?

`<TODO>`

- Members outside of the trust relationship **can not derive the personal identity**
  of someone inside.

### What have we tried? What went wrong?

`<TODO>`

### What are potential next steps?

`<TODO>`

