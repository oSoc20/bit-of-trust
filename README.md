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
someone you trust to be credible vouching for your abilities. Addittionally, if data is in a shared
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
from these files and expose you as a potential customer to an unkown amount of advertisers that you
never know about.

In order to really turn this story into something engaging, we think that it still lacks some way
to easily explain how we will achieve this explicitness. Of course, this seems to be dependent on
an implementation. One way would be to log access to the people interacting with the data, and note
down their reason for accessing it. We could have some form of access control based on that, to
prevent people from interacting, or we could allow the interaction and create a fork of the shared
data. These are some potential explanations, but they remain rather vague.

## **Naming**: How can we refer to relationships?

`<Description>`

### What have we tried? What went wrong?

### What are potential next steps?

## **Onboarding**: How are relationships established?

### What have we tried? What went wrong?

### What are potential next steps?

## **Evolution**: How do relationships evolve over time?

### What have we tried? What went wrong?

### What are potential next steps?


