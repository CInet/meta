# NAME

Yak-shaving the Conditional Independence Net

# SYNOPSIS

This document outlines the way to create an open database of conditional
independence structures called *CInet* at <https://conditional-independence.net>.
Documented are software prerequisites, database layout and computation goals.

# DESCRIPTION

The study of conditional independence (CI) structures is a field of discrete
mathematics where one is concerned with finite sets of CI symbols (ij|K),
consisting of two atoms i, j and a set of atoms K, over a finite ground set,
which conform to varying systems of exchange axioms. The common denominator,
or weakest such system, are the semigraphoid axioms inspired by conditional
independence of random variables.

From this formal perspective, every occurence of a "conditional independence-like"
relation in mathematics poses a *recognition* and then a *synthesis* problem:

1. Which distinctive features do the CI structures arising from
   this type of relation possess?
2. Given a CI structure which has all the features of some type,
   is it always possible to find an object of the given type which
   realizes the structure and how can we do this systematically?

Of course, one is also interested in how hard these problems are in general,
for various CI-like relations.

To give an intriguing example, the modular pairs of every matroid form
a semigraphoid, which is not completely arbitrary. And while such
semigraphoids are not always realizable by discrete random variables,
they *are* when the matroid is linear.
The combinatorial abstraction of discrete CI structures (semigraphoids)
contains the combinatorial abstraction of linear independence (matroids).
In this case, the unary relation of linear independence of a set of
vectors was "upgraded" to a ternary conditional independence relation
and suitably generalized.

With this database of CI structures, we want to list CI structures in low
dimension and catalogue their possibly hard to compute properties to aid
mathematicians seeking to test their conjectures for counterexamples more
quickly.

## Goals

To follow good scientific practice, we want to create an *[open database]*.
This means that the database should be

- Accessible: the data is available under a Creative Commons [CC BY-SA]
  license and online access is provided according to our means.
- Interoperable: the data is structured and stored in a relational
  database supporting SQL queries.
- Reproducible: all the source code required to produce a copy of the
  database is released as free software, the design and processes are
  documented.
- Transparent: the software and database are [semantically versioned],
  errata are publically posted.

[open database]: https://opendatahandbook.org/guide/en/
[CC BY-SA]: https://creativecommons.org/licenses/by-sa/4.0/
[semantically versioned]: https://semver.org/

What follows are "topics", i.e. categories of objects and the properties
we want to catalogue. Not all properties are feasible, some may only be
partially computed when the first version of the database is released.

### Semigraphoids

There are **many** semigraphoids. There is no hope to go beyond dimension 5
and even that will only be doable modulo isomorphy. Next to an encoding of
the semigraphoid itself and extremely fundamental properties like its dimension,
we want to store, as far as possible:

- `Cardinality`: how many CI statements it contains.
- `RankBag`: for each cardinality, how many CI statements of that cardinality it contains.
- `Coatomic`: whether it is a coatom in the semigraphoid lattice.
- `Gaussian`: whether it is realizable by a regular Gaussian distribution.
- `Discrete`: whether it is realizable by a discrete distribution.
- `Complete`: whether it is (Gaussian) complete in the sense of Drton-Xiao.
- `Markovian`: whether it is a Markov network.
- `Bayesian`: whether it is a Bayesian network.
- `DualId`: a pointer to the record of this object's dual.

The "whether" properties are booleans. Since some of the associated decision
problems are probably "hard", it should be expected that such columns contain
an SQL `NULL` value when the answer is unknown.

Realizabilities should be accompanied by a witness in another table.

Representatives of higher symmetry groups than isomorphy should be included
and probably be used to define materialized views which include only those
properties which are still invariant under the bigger group.

### Axiom systems

We indicate which semigraphoids fulfill which of some distinguished
axiom systems:

- `Intersectional`: whether the intersection axiom holds.
- `Graphoid`: given the semigraphoid axioms, equivalent to `Intersectional`.
- `Compositional`: whether the composition axiom holds, the dual of `Intersectional`.
- `Ascending`: given `Graphoid`, equivalent to `Markovian`.
- `Descending`: the dual of `Ascending`.
- `WeaklyTransitive`: whether weak transitivity holds.
- `Semigaussoid`: the semigraphoid axioms plus `Intersectional` and `Compositional`.
- `Gaussoid`: `Semigaussoid` plus `WeaklyTransitive`.

### Gaussoids

Gaussoids are special instances of semigraphoids related to regular Gaussian
distributions and show signs of stronger geometric behavior which allows the
definition of extra properties:

- `Orientable`: obtainable as the support of an oriented gaussoid.
- `Valuable`: obtainable as the support of a valuated gaussoid.
- `NearIdentity`: whether there exists a Gaussian realization in every ball around the identity matrix.

In addition, we store the collection of isomorphy classes of all 3-minors
of the gaussoid. <!-- use an enum, query with @> operator -->

It may even be possible to store all oriented gaussoids up to dimension 5
modulo isomorphy. <!-- about 464 million -->

### Entropy region

The investigation of discrete realizability of semigraphoids connects to
the entropy region and provides various approximations of that property:

- `Matroidal`: whether the semigraphoid consists of the modular pairs of a matroid.
- `Semimatroidal`: modular pairs of a polymatroid.
- `Simple`: obtainable as the intersection of uniform semimatroids.
- `Linear`: `Matroidal` from a vector matroid. <!-- store the field's characteristic? -->
- `Multilinear`: `Matroidal` with a subspace arrangement.
- `Algebraic`: `Matroidal` from an algebraic matroid.
- `Entropic`: equivalent to `Discrete`.
- `AlmostEntropic`: semimatroid of a limit of entropy vectors.

In every dimension, the set of polymatroids forms a rational polyhedral cone
whose generators should be stored up to isomophy. Since entropicness is not
easy to decide, we want to store several necessary properties:

- `Selfadhesive`: selfadhesivity in the sense of Matúš.
- `ZhangYeung`: Zhang-Yeung inequality.
- `Ingleton`: Ingleton inequality.

When these properties are equivalent to a conjunction of linear information
inequalities, tightness defines a supporting face of the polymatroid cone,
which in turn corresponds to an order ideal in the semimatroid lattice.
In this case, tightness is directly attributable to semigraphoids and
should be stored.

<!-- I have yet to become knowledgeable enough in the theory of information
  inequalities. See Studeny section 5.2.2 for the connection of entropies,
  multiinformation and structural imsets -->

## Implementation notes

### Database technology

The incidence of properties to objects which we intend to store fits
perfectly with relational databases. Moreover, SQL is a rather commonly
learned language allowing even complex queries. We choose PostgreSQL
for having convenient datatypes, a rich set of operators and other
helpful features (such as materialized views). It has decent support
in Perl and seems fast and stable overall.

The data is [semantically versioned] with the three version components
**major**.**minor**.**patch**. A bump of these versions means the
following:

- Patch: new data is added.
- Minor: new columns are added.
- Major: there is a backwards incompatible schema change
  or data errors are fixed.

After its creation, the database is used in a read-only fashion, except
for such controlled, versioned updates accompanied by software releases
of the tools and database build process.

The versioning of the database is implemented by having a separate
PostgreSQL database for each version. Inside each database, schemas
are used to group related tables and (materialized) views together.
This separation prevents newer releases from breaking code using an
older version of the database. Older versions are kept online as long
as available disk space permits.

Database dumps should be offered in any case.

### Software: libraries and scripts

Decision procedures for the various properties listed above range widely
in their computational complexities. Multiplied by the combinatorial
explosion in the number of semigraphoids, this requires very fast
primitives at the core of our software.

- Axiomatic properties can often be decided by a SAT solver, the AllSAT
  variant can be used to list all inhabitants of an axiomatically defined
  set of semigraphoids. We develop [libpropcalc] to serve these needs.
- Polymatroids and linear information inequalities are in the domain
  of (integer) linear programming and rational polyhedral geometry.
  [libnormaliz], [4ti2] and [SCIP] are available.
- Gaussian realizability concerns (semi)algebraic geometry: decidable
  but often out of reach in practice. General methods for discrete
  realizability are unknown to me.

[libpropcalc]: https://github.com/taboege/libpropcalc
[libnormaliz]: https://www.normaliz.uni-osnabrueck.de/
[4ti2]: https://4ti2.github.io/
[SCIP]: https://scip.zib.de/

Such heavy lifting should be entrusted to well-tested and efficient
libraries, often written in C, C++ or the like. On the other hand, there
is the task of organizing computations, collecting and distributing
results for which a stable glue language like Perl is predestined.

Any code that needs to be written by ourselves and be fast should be
done in C++ with suitable generality, interface consistency and low
serialization overhead, so that it can be accessed from Perl through
[FFI] and composed with the rest of the system.

[FFI]: https://metacpan.org/pod/FFI::Platypus

#### `CInet::Tools`

Provides all the domain-specific code to do computations with CI structures.
It starts with `CInet::Cube::Faces` to work with `(ij|K)` CI statements,
goes over propositional-logic tools to enumerate semigraphoids and gaussoids
or decide orientability, then graphical models, to polyhedral geometry and
integer optimization for CI inference on semimatroids. Bundles dependencies
on native libraries for heavy lifting, provides a coherent interface to them.

#### `CInet::DB`

Provides a wrapper for SQL queries on the online database and inspecting
catalogued properties of the results. Optionally depends on `CInet::Tools`
for unmarshalling database records into `CInet::Tools` objects for further
processing.

#### `CInet::Gen`

Contains everything required to build the current version of the database
from scratch. It depends on `CInet::Tools` and automates the computation
of objects and properties, symmetry reduction and so on. The distribution
contains `CInet::Gen::Task` packages which encapsulate computations and
declare dependencies among each other. After a topological sorting of the
dependencies, a job queue like [Minion] executes the tasks.

The intention is to provide sufficient automation to let the database
regeneration run once per major version with a pay-per-hour cloud computing
service. Minor versions and patches are handled by `CInet::Gen::Delta`s,
which are specialized tasks acting like database migrations.

Ideally, there is a test suite for the database.

[Minion]: https://metacpan.org/pod/Minion

# HISTORY

- A precursor to this project is the website [Gaussoids.de] which collects
  byproducts of computations for papers about gaussoids in the form of text
  files. The software for creating this data is messy and unpublished.
  The plain text format records the objects but no properties beyond
  "being on this list". Naturally, complex queries are unsupported.
- The design process for CInet culminating in this summary began after
  Prague Stochastics 2019. I already had a prior interest in cataloguing
  CI structures, but in particular the talk by László Csirmaz on sticky
  polymatroids opened up a whole new realm of CI research for me, making
  the collection of examples and counterexamples even more important.
- In October 2019, I applied to the RISE Germany internship program of
  the DAAD with this project. The DAAD placed Garett Cunningham with it
  before cancelling the program due to COVID-19. We agreed to pursue
  the project together anyway.

[Gaussoids.de]: https://gaussoids.de
