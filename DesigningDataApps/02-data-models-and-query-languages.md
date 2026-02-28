# 2. Data Models and Query Languages
english | [russian](02-data-models-and-query-languages.ru.md)

**TL;DR:** A comparison of relational, document, and graph models, the reasons for NoSQL, and why declarative query languages matter for evolution and optimization.

---

## Relational Model Versus Document Model

- The relational model is strong at relationships and ad‑hoc queries (joins, aggregates).
- Documents are convenient for aggregates and for reading a “whole object” in one request.

## The Birth of NoSQL

- Drivers: horizontal scaling, high availability, schema flexibility, specialized workloads.

## The Object-Relational Mismatch

- The gap between application objects and tables: nesting, collections, inheritance.
- ORMs help, but can hide expensive N+1 patterns and unnecessary joins.

## Many-to-One and Many-to-Many Relationships

- M:1 and M:N relationships are easier to express in relational databases; in documents you often need references and extra queries.

## Are Document Databases Repeating History?

- Historically there were navigational models; SQL won because of declarative queries and optimizers.

### The network model

- Pointer navigation: fast for pre‑defined paths, but handles changing requirements poorly.

### The relational model

- Declarative queries separate “what” from “how”; the optimizer chooses the plan.

### Comparison to document databases

- Documents bring back the convenience of aggregates/locality, but are harder for arbitrary relationships.

## Relational Versus Document Databases Today

- Systems are converging (JSON in SQL, transactions/indexes in NoSQL), but trade‑offs remain.

### Which data model leads to simpler application code?

- Aggregates → documents are often simpler; complex relationships/invariants → relational is often simpler.

### Schema flexibility in the document model

- Schema‑on‑read speeds up changes, but pushes data quality checks into the application.

### Data locality for queries

- Nesting improves locality; but updating large documents and partial reads can become expensive.

### Convergence of document and relational databases

- Hybrid features reduce the gap, but don’t eliminate differences in optimization and consistency models.

## Query Languages for Data

- Declarative languages enable optimization and robustness to change.

### Declarative Queries on the Web

- SQL/CSS selectors are examples of declarative style; they are easier to optimize and safer.

### MapReduce Querying

- MapReduce scales, but is verbose and inconvenient for interactive queries.

## Graph-Like Data Models

- Graphs are convenient when relationships are “first class”: traversals, recommendations, dependencies.

### Property Graphs

- Nodes/edges with arbitrary properties; convenient for traversal queries.

### The Cypher Query Language

- Declarative path patterns (pattern matching) on top of property graphs.

### Graph Queries in SQL

- You can model graphs with tables, but recursive traversals are often harder and more expensive.

### Triple-Stores and SPARQL

- RDF stores facts as (subject, predicate, object); SPARQL queries the graph using patterns.

#### The semantic web

- The idea: linked data and shared ontologies across domains.

#### The RDF data model

- Triples as a universal fact format.

#### The SPARQL query language

- A pattern/filter language for RDF graphs.

### The Foundation: Datalog

- Facts + inference rules; good for recursive relationships and explainable queries.

## Summary

- Choose the model based on your data structure and query types; declarative style makes evolution easier.
