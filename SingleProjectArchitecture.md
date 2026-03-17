# Single Project Architecture

## Table of Content

- [Single Project Architecture](#single-project-architecture)
  - [Table of Content](#table-of-content)
  - [Typical project structure](#typical-project-structure)

## Typical project structure

|X|Components|Packages|
|---|---|---|
|API|Controllers, DTOs & IoC Config|Swashbuckle & EF Core Design|
|Infrastructure|Persistence config & migrations|EF Core SQL|
|Domain|Entities, DbContext & services|EF Core|
|Shared Kernel|Base classes, Interfaces & Cross-cutting concerns|EF Core|
