# A p2p database idea

This is a draft. It is meant to gather feedback and find possible collaborators. _Pull requests & GitHub issues are welcome!_

## Intro
I'd like to explore the idea of creating a schema-based peer-to-peer database with explicit causality links. The schema would be rich enough to include all the necessary data validation rules (including causal dependencies between entities, if necessary).

Applications would configure with whom the database should be kept in sync, either in the form of `<crypto key, channel>` abstract pairs (for maximum flexibility), or a mesh + epidemic gossip solution provided by default (for convenience), and then use the database as if it were local.

The database would run over an operational log encoded as a Merkle DAG, as described [here](https://research.protocol.ai/blog/2019/a-new-lab-for-resilient-networks-research/PL-TechRep-merkleCRDT-v0.1-Dec30.pdf). Familiar data types would be built operationally, as in [OrbitDB](https://github.com/orbitdb/orbit-db) or [Hyper Hyper Space](https://www.hyperhyperspace.org), and would be the building blocks for schema definitions.

The schema definitions should be sufficient for ensuring both data validity and coherence between replicas. If -for example- a certain capability must be granted before an action can be taken, it should _not_ be possible, because of differences in the propagation of changes, for the action to be deemed valid in some replicas and invalid in others.

I'd like to explore _pseudo-transactions_, as used in Hyper Hyper Space's [causal types](https://github.com/hyperhyperspace/hyperhyperspace-core/tree/master/src/data/collections/causal), as a solution to this issue. The end result should resemble a temporally-bound foreign key (or _foreign condition_ perhaps, for generality).

## Schemas

A schema-definition language would be used, and schemas would be stored in text files. They could be formatted to a given standard and hashed, and be content-addressed.

Schemas would loosely follow the tables-and-rows found in RDBMs, defining named collections of objects of a given type (the fields in each object would be typed, similar to columns in a table). Types would not be required to be normalized, and objects could contain other collections or rich data types. The objective is providing something that is both familiar and easy to use.

As long as they can be defined in an eventually-consistent setting, constraints should be expressible in schemas. In particular, user access rights and capability systems should be easy to use within the database.

### Schema updates

It should be possible to define a new schema by indicating a set of changes from a previous one. These would define if and how the new or modified fields should be populated, and any other changes the data in the old schema needs to undergo to be valid in the new one.

Schema updates could also be useful to upgrade the database itself to a new version.
 

## Architecture

The database should be highly modular, providing just two main modules:

- **Storage backend**: an optimized local store for replica sync & data validation.
- **Sync protocol**: an implementation of the sync protocl, that would work over an abstract secure channel.

To operate a database instance, its storage backend would need to be opened and passed to the sync module. Either the specific peers or a mesh implementation (a default one should be provided) would need to be provided as well. Each instance of the store would come with its hash-identified schema spec, and automatic migration of old instances would be supported.

### PKI

All permissions in the system would stem from the use of cryptographic keypairs. These should be pluggable, and the operator of the database should be able to bring his own crypto.

### Event stream

The storage backend would provide an event stream of all the changes in the database _(this would be different from the raw operation log, containing instead their actual effects, if any)_. The event stream could be used to keep the state of other components in sync with the database (be it a UI, or a more traditional storage system thorugh an [adapter](### Adapters)). A mechanism for checkpoinitng should be provided, enabling the event stream to be resumed from the last event that was ingested by the client.

### Adapters

While the storage backend could be used directly by the application, a series of adapters should be provided that would allow the usage of more traditional storage systems, that would be kept in-sync with the backend automatically: in-memory, relational, graph- and document-based, etc.

Writing directly to the adapter could be a challenge. The simplest option would be to always write in the storage backend, using it as a source-of-truth, but schemas for writable adapters could be devised.

### Custom Data Types

It would be nice if the operator of the database could define their own operational data types, based on the op-log.

## Spec

The sync protocol should be as simple as possible, and a working spec would be developed hand-in-hand with the implementation.

## Implementation choices

Given how prominent **rust** has become as a system development language, it'd be used for the default implementation. JavaScript and WASM versions could be compiled for use in brower-based environments, and native versions for Mac, Windows, Android and Linux.


