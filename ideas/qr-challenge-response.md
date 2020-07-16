# QR challenge-response idea

## Rationale

In the current protocol, an attacker can intercept the messages being
exchanged when setting up a trust relationship, and enter the trust
shard into an unwanted trust relationship. By employing cryptographic
signatures, MITM (man-in-the-middle) attacks of this sort can be
prevented.

## General description

1. Bob (verbally) asks Alice if she wants to enter a trust relationship
2. Alice accepts (verbally)
3. Bob generates a 'challenge':
    ```js
    QR({ "base_hash": "<base_hash>" })
    ```
4. Alice scans the QR code and generates a 'response' for Bob:
    ```js
    QR({
         "trust_hash": "<trust_hash>",
         "trust_shard": "<trust_shard>",
         "signature": "<signature>"
    })
    ```
5. (Bob announces the new trust relationship to the network?)

## Notes

* `base_hash` is a hash of the trust shard Bob uses to enter the
  relationship, when setting up an entirely new relationship, or the
  identifier of an existing relationship of which Bob is a member.
* `trust_hash` is the hash of the newly established trust relationship,
  i.e. hash(`base_hash` + hash(Alice's trust shard))
* `trust_shard` is the trust shard Alice uses to enter the relationship
* `signature` is a (Base64-encoded) cryptographic signature for
  `trust_hash` signed with the private key associated with Alice's trust
  shard

* Because Alice trusts Bob, she shouldn't need to know any more details
  about `base_hash`, but she can connect to the network to get a full
  description if she wishes to. If this is desirable, it might be useful
  if challenges include the trust shard of the party initiating the
  challenge and a signature of `base_hash` with their private key.

* To keep signatures small, as they have to be stored permanently, using
  ECC (elliptic-curve cryptography) is recommended, e.g. ECDSA with
  curve P-256, or EdDSA (Ed25519 or Ed448).
