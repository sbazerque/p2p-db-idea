# A p2p database idea

This is a draft. It is meant to gather feedback and find possible collaborators. _Pull requests & GitHub issues are welcome!_

## Intro
I'd like to explore the idea of creating a schema-based, eventually consistent peer-to-peer database, designed specifically for distributed applications. 

Cryptographic keypairs would be used to identify users, and to verify the authenticity of their changes to the data. The set of users (identified by keypairs) with whom the databse should be kept in sync would be configured by the application (it could be a collection kept inside the database, or an external source like a peer discovery service). A mechanism for mapping keypairs and network addresses would also be necessary (sensible defaults could be provided: a DHT, a broadcast service, etc.).

The database would run over an cryptographically-secure operational log encoded as a Merkle DAG, as described [here](https://research.protocol.ai/blog/2019/a-new-lab-for-resilient-networks-research/PL-TechRep-merkleCRDT-v0.1-Dec30.pdf). Familiar data types would be built operationally, as in [OrbitDB](https://github.com/orbitdb/orbit-db) or [Hyper Hyper Space](https://www.hyperhyperspace.org), and would be the building blocks for schema definitions. In particular, all the well-known operational CRDTs can be encoded this way.

The schema language would be expressive enough to include _all_ the necessary data validation rules, including any behavioral rules that the application needs to enforce. Once these have been included in the schema definition, the application could basically forget about behavioral rule enforcing: that'd be taken care at the database level. These constraints should somehow resemble a foreign key with temporal awareness.

The replication protocol would ensure both data validity and coherence between replicas. If -for example- a certain capability must be granted before an action can be taken, it should _not_ be possible, because of differences in the propagation of changes, for the action to be deemed valid in some replicas and invalid in others.

Aiming for an always-available, eventually consistent database presents a challenge for the implementation of behavioral rules, since those will probably depend on the current state of the database (state that could have changed in other replicas, but not been fully propagated). A possible solution for this issue is the use of _pseudo-transactions_, as used in Hyper Hyper Space's [causal types](https://github.com/hyperhyperspace/hyperhyperspace-core/tree/master/src/data/collections/causal). Under this apporach, all the assumptions that must be satisfied for a given action to be considered valid are also persisted in the database, in the form of attestations that show the assumptions were satisfied in the local replica at the time the action was taken. If the assumptions are invalid in other replicas, because of propagation delays, the secure Merkle-DAG mentioned above can be used to resolve the issue deterministically. The resolution may involve a _rollback_ of some actions (they are called _pseudo_-transactions because there are no _commits_: always-available implies that there may be some actions that the replica is unaware of, that could invalidate assumptions taken locally).

## Schemas

A schema-definition language would be used, and schemas would be stored in text files. They could be formatted to a given standard and hashed, and be content-addressed.

Schemas would loosely follow the tables-and-rows found in RDBMs, defining named collections of objects of a given type (the fields in each object would be typed, similar to columns in a table). Types would not be required to be normalized, and objects could contain other collections or rich data types. The objective is providing something that is both familiar and easy to use.

Behavioral rules could be expressed using a syntax similar to that of `EXISTS` clauses in SQL. These could be nested, and would roughly correspond to attestations about database state. They would be limited to what can be defined in an eventually-consistent setting. In particular, user access rights and capability systems should be easy to use within the database.

### Schema updates

It should be possible to define a new schema by indicating a set of changes from a previous one. These would define if and how the new or modified fields should be populated, and any other changes the data in the old schema needs to undergo to be valid in the new one.

Schema updates could also be useful to upgrade the database itself to a new version.
 

## Architecture

The database should be highly modular, providing just two main modules:

- **Storage backend**: an optimized local store for replica sync & data validation.
- **Sync protocol**: an implementation of the sync protocl, that would work over an abstract secure channel.

To operate a database instance, its storage backend would need to be opened and passed to the sync module. 

### PKI

The primitives used to build the cryptographically-secure Merkle-DAG should have sensible defaults, but should be replaceable by whatever the application wants to use.

### Event stream

The storage backend would provide an event stream of all the changes in the database _(this would be different from the raw operation log, containing instead their actual effects, if any)_. The event stream could be used to keep the state of other components in sync with the database (be it a UI, or a more traditional storage system thorugh an [adapter](### Adapters)). A mechanism for checkpoinitng should be provided, enabling the event stream to be resumed from the last event that was ingested by the client.

### Database Adapters

While the storage backend could be used directly by the application, a series of adapters should be provided that would allow the usage of more traditional storage systems, that would be kept in-sync with the backend automatically: in-memory, relational, graph- and document-based, etc.

Writing directly to the adapter could be a challenge. The simplest option would be to always write in the storage backend, using it as a source-of-truth, but schemas for writable adapters could be devised.

### Mesh Network Adapters

It should be possible to run the gossip+sync protocol over any network. A sensible default implementation for running over the Internet will be provided.

### Custom Data Types

It would be nice if the operator of the database could define their own operational data types, based on the op-log.

## Spec

The sync protocol should be as simple as possible, and a working spec would be developed hand-in-hand with the implementation.

## Implementation choices

Given how prominent **rust** has become as a system development language, it'd be used for the default implementation. JavaScript and WASM versions could be compiled for use in brower-based environments, and native versions for Mac, Windows, Android and Linux.

## Credits

The idea of seeing Hyper Hyper Space's causal types as pseudo-transactions, as mentioned above, is due to José Orlicki. 

The suggestion of adding schema support to Hyper Hyper Space is due to Micah Fitch.


## Feedback

Santiago Bazerque [santi@hyperhyperspace.org](mailto:santi@hyperhyperspace.org) or GitHub issues / pull requests
