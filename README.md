![Banner](https://raw.githubusercontent.com/oSoc20/bit-of-trust/master/img/banner.png)

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

![Story](https://raw.githubusercontent.com/oSoc20/bit-of-trust/master/img/story.png)

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

> **Note: Beyond this point, we discuss more possibly implementation-dependent information. It is
> strongly advised to really figure out the story part first, as this has further implications on
> what should and should not be implemented. Otherwise valuable time may be spent on implementation
> details that do not even matter to realise the value set forth in the story behind Bit of
> Trust.**

![Naming](https://raw.githubusercontent.com/oSoc20/bit-of-trust/master/img/naming.png)

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

1. The peer presents a ‘question’ or ‘challenge’ and encrypts it using the shard's public key, so
   only the holder of the shard's private key would be able to read the challenge.
2. The holder receives this message and decrypts it using their private key.
3. The holder solves the challenge and (assuming a secure connection) sends the answer to the
   challenge back to peer.
4. The holder has now proven that they own the shard.

This idea is based on [challenge-response
authentication](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication). There may
be different avenues to explore here. The essence is that we need some form of token that can serve
as a bootstrap from which the rest of the Merkle tree can be built.

#### Improving upon the random words approach

We could make the random words approach better by using it as a layer on top of some method that
does work recursively. This way we could take advantage of the human-readability. For instance, if
we were to use the hash-based approach mentioned earlier, the challenge would be in mapping the
hashes to a human-readable string. A suitable library that can do this, is called
[codenamize](https://github.com/stemail23/codenamize-js). Do note, this works deterministically so
it is best to use it on input that is already hashed in some way. If we were to use this method on
bare user information such as an IP address, the process of mapping to a human-readable string
could be reversed again, and we would expose personal information.

#### Looking for more alternatives

Of course, these may not be the only ways to name trust relationships. We could also use
[asymmetric cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) in which the
public key is used as the name for the relationship. The private keys would be held by the members
of the group. This would cause a shared private key management problem, which we will discuss in
greater detail in the section on the ‘onboarding’ challenge.

Another potential avenue is to identify trust relationships based on domain names. This means that
we accept the risk that people could personally identify themselves through their domain name
choices. The downside is that domain names cost money, and we would have to ask people to buy 
[whois protection](https://en.wikipedia.org/wiki/Domain_privacy) since their personal details would
otherwise be open for all of the internet to see. Not only this, but the process of registering a
domain name is not exactly user-friendly. A useful compromise would be to provide a handful of
existing servers, hosted by users, with which other users can register a relationship name of their
choosing. This is similar to what the decentralised social network
[Mastodon](https://joinmastodon.org/) does, except that it is based on individual identifiers.

![Onboarding](https://raw.githubusercontent.com/oSoc20/bit-of-trust/master/img/onboarding.png)

## **Onboarding**: How are relationships established?

Naming and onboarding are quite intimately connected, since the way relations are constructed
should in some way give rise to the name, and the name should mean something to the members of the
relationship. Therefore, it comes as no surprise that we have largely the same requirements.

- The way in which a relationship is constructed should **not reveal any personal information**
  during or after the process.
- The way in which people establish a relationship should be **user friendly**. (i.e. keep the
  process as short and as simple as possible)

### What have we tried? What went wrong?

The way we envisioned the user-facing aspect of onboarding is by generating a link or a QR-code on
one person's device. This person would subsequently share their QR-code or link with another
person. When this user clicks on the link or scans the code, they will be asked to confirm being
added to the trust relationship. We thought this would be the minimal interaction required.
However, this still leaves us with a gap between the user facing interface, and its implementation.
Therefore, we discuss two possible underlying implementations below in greater detail.

#### Can we use asymmetric cryptography?

We have tried two methods, the first being the usage of asymmetric cryptography. As mentioned in
the section about naming, we would represent the trust relationship by its public key, and
distribute the private keys among the members of the relationship. We could prove membership
exactly like the challenge-response based authentication we mentioned earlier. The problem however,
lies in establishing a shared key between all of the members of the relationship, without:

1. Causing a performance problem.
2. Storing public keys of users on the server, as this could be seen as information that could
   potentially personally identify someone.

One way shared keys are typically established is through some form of [Diffie-Hellman key
exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). Which is a method for
establishing a shared secret key over an untrusted communication channel. This could however
introduce some slight performance problems, which I will briefly explain (feel free to skip the
next two paragraphs if it goes over your head).

There are two major performance costs to take into account here. The first being the work each
system in the network has to do. And secondly, the communication cost, which is the cost of
transferring messages over the network. When studying distributed systems, some researchers like to
leave out the cost of systems themselves since the cost of transferring messages vastly trumps it
in most cases. In any case, if we denote the number of participants in the relationship by `n`,
then a naive circular arrangement of Diffie-Hellman key exchange with multiple parties would have a
`O(n^2)` communication cost.

**Proof** (Slightly hand-wavy). This can be seen by modelling the communication as a directed
graph, where each node is a participant, and each directed edge is a message going from one user to
another user. Because each user has to contact every other user at least once, this graph will be
a complete digraph. Denote the number of participants by `n`. We know that the complete digraph
will have `n * (n - 1)` directed edges, or messages. We can now determine that the communication
cost is `O(n^2 - n) = O(n^2)`.

This result is not necessarily show-stopping. But to put it in perspective, a relationship with
about `100` people would have to send about `100^2 = 10 000` messages every time a new key has to
be established, which is whenever someone joins or leaves (this also means that the public key
changes every time someone joins or leaves). This is still a amount of messages that can be dealt
with but it is far from ideal.

Another way to do it, is by using two layers of asymmetric encryption. In this system, every time a
new key needs to be generated, the users have to either retrieve the public keys of every user from
some server, or exchange them with every participant at runtime. The communication cost here is
`O(n)` where `n` is the number of participants, since everyone is sending a message to a single
user doing the key generation. In the case where it is stored on a server, the cost is `O(1)`, but
this is not desirable because public keys can be seen as personal information. The user who
received all of the public keys of the participants will do the following:

1. Generate a new asymmetric key pair that will be used by everyone.
2. Encrypt the private key with the public keys of every participant in the trust relationship.
3. Send all of these encrypted keys back to each of the participants.

The communication cost here is already better than the Diffie-Hellman case. However, we have to put
major trust in the fact that the person generating the new key pair is not also sending this private
key to other people who are not supposed to be part of the relationship.

#### What about software tokens?

Another approach to represent membership would be to use a software token. This could be a string
of random bytes, generated upon establishing the trust relationship. Each user gets their own
random token, and the combination (such as hashing) of both random tokens gives rise to the name
for the trust relationship. This way, the network would just consist of a bunch of random bytes,
similar to asymmetric cryptography approach. The meaning of these random bytes is only apparent to
the people who know of a given software token.

We do lose the property of being able to prove ownership compared to the asymmetric cryptography
approach. A user could potentially copy one of these tokens from the server and act like they are a
member in the relationship. This would leave us without any way to make sure if a user is who they
say they are.

Even worse, some automated script could copy all of the tokens from the server and
start faking random operations on the entire network, basically rendering the entire system
completely useless. This is somewhat similar to what is known as a
[Sybil Attack](https://en.wikipedia.org/wiki/Sybil_attack), except that it is trivially easy to do.
The attacker does not even need to create their own pseudonymous identities, they can just make use
of all existing tokens.

### What are potential next steps?

The challenges here are mostly about making the process as simple as possible while making attacks
hard. It is important that security is taken into account from the start in this process, since
retrofitting it in as an afterthought is much harder. The main concerns are about maintaining
anonymity while preventing Sybil attacks. There have been efforts at tackling this problem before,
such as [Advogato](https://en.wikipedia.org/wiki/Advogato#Trust_metric), which arguably did not
entirely succeed in its efforts.

Because we are essentially agreeing on who is a member of a trust relationship, and who is not, we
could look at these problems under the light of consensus mechanisms. These are absolutely critical
in blockchains, which have provided most of the last innovations in this area in the last few
years. Methods such as proof of stake could prove useful to look into. In short, it pays to look
towards blockchain research since they have been dealing with problems similar in nature (i.e.
dealing with byzantine fault tolerance in distributed systems).

![Evolution](https://raw.githubusercontent.com/oSoc20/bit-of-trust/master/img/evolution.png)

## **Evolution**: How do relationships evolve over time?

This challenge is about defining what happens when participants in a relationship get added or
removed from the relationship. There are two mostly obvious ways to look at it, but this doesn't
mean there may be other options. It comes down to what happens to the naming upon changes to the
relationship:

- Either, a relationship is made, and members joining and leaving have no impact on the name of the
  relationship. That is, the relationship stays the same, it is just its members that are
  interchangeable.
- Or, members join and leave relationships, but these relationship names change as members are
  removed or added. That is, the members stay the same, but the relationships are interchangeable.
  The name can change at any time when the composition changes.

Another point to consider is how participants in a relationship can remove someone else from the
relationship. There are also two options we came across here:

- Either, we allow anyone to remove anyone else at will. Doing so will create a new relationship
  excluding the target, but only for the person who decided to remove the target. The other people
  will remain in the original relationship until they decide to remove this person as well.
- Or, we implement a system of majority voting in which people get removed from the relationship
  when the majority thinks this is a good decision. There are no forks, relationships just move
  from one state to the next state in which someone is no longer included.

This could potentially include other actions, but this is discussed further in the section on
‘capabilities’.

### What have we tried? What went wrong?

#### Git-based relationships

There are two major models we have considered when thinking about this challenge. The first being
git-based relationships. If we structure relationships as a Merkle tree, we come close to the
concept of git. In git, commits are connected through a directed acyclic graph, with each commit
pointing to the previous commit. It could also happen that two different commits point to the same
commit, which is a concept called branching. A possible way to implement revocation is therefore to
create a new relationship where the invalid trust shard is no longer included, but pointing to the
previous state of the trust relationship.

However, the users within the trust relationship that do not decide to revoke the membership will
still have a relationship including that person, but it will be branched off from the original. In
essence we split the relationship into two distinct relationships each with or without the given
trust shard, and pointing to the previous state.

#### File system based relationships

The other approach is thinking of relationships as more of a database or a file system. In this
case we would have a single document in which we capture the participants. When changes occur,
people will write to this single file and change the structure of the relationship for everyone
else. Of course, this brings us to distributed consensus mechanisms again, which may require
research in order to determine the best approach for this scenario, depending on how relationships
are established.

In this case, one person removing someone from the relation will impact the relationship for
everybody else. So, it may be a better idea to implement a voting-based system if this were the
case, where the individual control over relationships is supported better by the git-based
approach.

### What are potential next steps?

It may be useful to look at existing decentralised identity protocols to see at what they are
doing with regards to relationship evolution. For instance the [Jolocom
protocol](https://jolocom.io/wp-content/uploads/2019/12/Jolocom-Whitepaper-v2.1-A-Decentralized-Open-Source-Solution-for-Digital-Identity-and-Access-Management.pdf)
makes use of the ethereum blockchain and IPFS, but is implemented in a ledger agnostic way. They
may have had to deal with much of the same problems with regards to version control of files or
identities.

Another approach is that of [BrightID](https://www.brightid.org/whitepaper) which keeps track of a
log of actions to determine a given user's ‘health’. Which is a measure of trustworthiness of the
identifier. This may be interesting to determine who can make lasting changes to the structure of a
relationship and who can not.

These two solutions remain useful to check out for other reasons as well. They faced quite a few of
the same problems. The major difference is that both Jolocom and BrightID are focussed on the
individual, just like most decentralised solutions around right now.

![Capabilities](https://raw.githubusercontent.com/oSoc20/bit-of-trust/master/img/capabilities.png)

## **Capabilities**: What are people able to do with a relationship?

What are users allowed to do when they are in a relationship? As we discussed earlier, this should
include a way to add or remove participants from the relationship. But what are the other things we
should allow? At the least, there are a number of things we have to keep in mind:

- Members outside of the trust relationship **can not derive the personal identity**
  of someone inside.
- Relationships should always have a **minimum of two participants**. This means that we either
  prevent users from leaving if the relationship has two members left, or the relationship
  disappears as soon as one of the two users leaves the two-person relationship.

### What are potential next steps?

This is very much still an open question, it really depends on which future applications the idea
will solicit in the future. It may be something interesting to define once there is a clear story
behind what day-to-day value Bit of Trust wishes to provide.

## Attribution

### Icons
- Megaphone icon made by [Fasil](https://freeicons.io/profile/722) from
  [www.freeicons.io](https://www.freeicons.io)
- Other icons (besides Bit of Trust logo and Open Summer of Code logo) made by
  [Freepik](https://www.flaticon.com/authors/freepik) from
  [www.flaticon.com](https://www.flaticon.com)
