# 2. Trust Shard Exchange

## Problem Description

**How can we implement invite-based trust shards that are revokable?**

We should be able to add somebody into a trust relationship by giving them a shard. This process is
called ‘inviting’ somebody to the trust relationship. However, these trust shards should also be
revokable. For instance, if an actor in the trust relationship loses their shard, it should be
possible to revoke the shard again.

## Possible Approach: Git-based Relationships

If we structure relationships as a Merkle tree, we come close to the concept of git. In git,
commits are connected through a directed acyclic graph, with each commit pointing to the previous
commit. It could also happen that two different commits point to the same commit, which is a
concept called branching. A possible way to implement revocation is therefore to create a new
relationship where the invalid trust shard is no longer included, but pointing to the previous
state of the trust relationship.

However, the users within the trust relationship that do not
decide to revoke the trust shard will still have a relationship including it, but it will be
branched off from the original. In essence we split the relationship into two distinct
relationships each with or without the given trust shard, and pointing to the previous state.

