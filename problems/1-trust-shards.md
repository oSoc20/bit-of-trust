# 1. Trust Shards

## Problem Description

**Can we come up with a trust shard system that fulfills the following requirements?**

- **Non-PII** It should not contain any personally identifiable information (PII).
- **Invite-based** The shard should be sendable to a new actor who wishes to join the trust
  relationship (this is the topic of [2-trust-shard-ex.md](/problems/2-trust-shard-ex.md)).
- **Revokability** It has to support being revoked by an actor inside the trust relationship. Users
  should not be responsible for 'never losing' their shard. When a token is lost, other members of
  the trust relation should be able to mark it as invalid. And they can hand out a new shard to
  this user (this is the topic of [2-trust-shard-ex.md](/problems/2-trust-shard-ex.md)).
- **User-friendly** Ideally, users should not have to remember passwords to use a shard (this is
  the topic of [3-trust-shard-ux.md](/problems/3-trust-shard-ux.md)).

As you can see there is quite a bit of overlap with the other problems, so the idea of this problem
is to come up with an internal representation of trust shards that can support the requirements
mentioned above.

## Possible Approach: Asymmetric Cryptography

Our original attempt was to use asymmetric cryptography (see [QR
challenge-response](/qr-challenge-response.md)), since we believed that the core problem
was that you should be able to prove ownership of a trust shard. The public key of the trust shard
is used as the building block in trust identifiers, and the owner of the shard has the accompanying
private key. We do this to prevent people from claiming ownership of a shard, when they do not own
it.

We envisioned it as a sort of challenge-response mechanism, whereby a peer in the network can ask a
holder of a trust shard to prove that they actually own it. This mechanism would work as follows:

1. The peer presents a ‘question’ or ‘challenge’ and encrypts it using the shard's public key, so
   only the holder of the shard's private key would be able to read the challenge.
2. The holder receives this message and decrypts it using their private key
3. The holder solves the challenge and (assuming a secure connection) sends the answer to the
   challenge back to peer.
4. The holder has now proven that they own the shard.

This idea is based on [challenge-response
authentication](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication).
