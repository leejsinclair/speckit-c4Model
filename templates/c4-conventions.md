# C4 Diagram Conventions

Rules for every diagram this extension produces. Read this file before writing or
checking any diagram.

## Levels

| Level | Diagram | Mermaid type | Shows | Lives in |
|-------|---------|--------------|-------|----------|
| 1 | System context | `C4Context` | The system as one box, the people who use it, the external systems it talks to | Project: `context.md` |
| 2 | Container | `C4Container` | Deployable/runnable units inside the system (apps, services, databases, queues) | Project: `containers.md` |
| 3 | Component | `C4Component` | Major building blocks inside ONE container | Feature: `architecture.md` |
| - | Sequence | `sequenceDiagram` | One flow through containers or components over time | Feature: `architecture.md` |
| - | Entity relationship | `erDiagram` | Persistent entities, attributes, keys, relationships | Project: `erd.md`; Feature: `architecture.md` |

C4 level 4 (code) is not produced. The source code is the level 4 view.

## C4 rules

- **One level per diagram.** A context diagram never shows containers; a container
  diagram never shows components. Zoom in with a new diagram instead.
- **One container per component diagram.** Neighbouring containers and external
  systems appear only as context around the boundary (`Container`, `ContainerDb`,
  `System_Ext` outside the `Container_Boundary`).
- **Names are stable identifiers.** An element has the same alias and the same label
  in every diagram and document it appears in. Aliases are `lowerCamelCase`
  (`orderApi`, `ordersDb`); labels are the human name (`"Order API"`).
- **Every container and component states its technology** in the technology
  argument (`"TypeScript, Fastify"`, `"PostgreSQL 16"`). Use the choices recorded in
  `plan.md` Technical Context. Write `"TBD"` rather than guessing.
- **Every element has a one-line description** of its responsibility, not its
  implementation.
- **Every relationship is labelled** with a verb phrase in the direction of the arrow
  (`"Places orders using"`, `"Reads from and writes to"`) and, between containers or
  components, the protocol or mechanism (`"JSON/HTTPS"`, `"SQL/TCP"`, `"AMQP"`).
- **Relationships point from the initiator to the target**, not in the direction of
  data flow.
- **No orphans.** Every element takes part in at least one relationship.
- **A database is a container**, not a component. Tables belong in the ER diagram.
- **Keep it readable.** If a diagram passes roughly 15 elements, split it (per
  bounded context, or per container for component views).

## Mermaid C4 syntax

Mermaid's C4 support is marked experimental. Use only this subset, which is stable
across renderers including GitHub:

```text
Person(alias, "Label", "Description")
Person_Ext(alias, "Label", "Description")
System(alias, "Label", "Description")
System_Ext(alias, "Label", "Description")
SystemDb_Ext(alias, "Label", "Description")
System_Boundary(alias, "Label") { ... }
Container(alias, "Label", "Technology", "Description")
ContainerDb(alias, "Label", "Technology", "Description")
ContainerQueue(alias, "Label", "Technology", "Description")
Container_Ext(alias, "Label", "Technology", "Description")
Container_Boundary(alias, "Label") { ... }
Component(alias, "Label", "Technology", "Description")
Rel(from, to, "Label")
Rel(from, to, "Label", "Technology")
BiRel(a, b, "Label")
UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

- Start each diagram with its type keyword, then a `title` line.
- Do not use sprites, tags, `AddElementTag`, links, or `UpdateElementStyle`.
- Do not put double quotes, parentheses or commas inside a label or description.
- Layout follows declaration order. Declare people first, then the system or
  boundary, then external systems, then all `Rel` lines.

## Sequence diagram rules

- One diagram per flow. A flow is one user story's primary path, or a significant
  alternate or failure path. Name the user story in the heading above the diagram.
- **Participants are elements that exist in the C4 views.** Use the same labels.
  People are `actor`; containers, components and external systems are `participant`.
  Declare them explicitly, in left-to-right order of first use.
- Pick one level of abstraction per diagram: container-level for flows that cross
  containers, component-level for flows inside one container.
- `->>` for a synchronous call, `-->>` for its response, `-)` for an asynchronous
  message or event.
- Label every message with what is asked or returned (`POST /orders`,
  `201 Created + orderId`, `OrderPlaced event`), consistent with `contracts/`.
- Use `alt` / `else` for branches, `opt` for optional steps, `loop` for repetition,
  `par` for concurrency. Show the error path that matters, not every error.
- Use `autonumber` so steps can be referenced from prose and tasks.

## ER diagram rules

- Entity names are `UPPER_SNAKE_CASE` singular nouns (`ORDER`, `ORDER_LINE`) and map
  one-to-one to the entities in `data-model.md`.
- Each attribute is `type name` plus a key marker where it applies: `PK`, `FK`, `UK`.
  Add a quoted comment for constraints worth seeing (`"not null"`, `"ISO 4217"`).
- Use logical types (`uuid`, `string`, `int`, `decimal`, `boolean`, `datetime`,
  `json`) unless `plan.md` fixes a database, in which case use its types.
- Every relationship states cardinality on both ends and a verb label:

  ```text
  ||--||   exactly one to exactly one
  ||--o{   one to zero or more
  ||--|{   one to one or more
  }o--o{   many to many (resolve with a join entity when it carries attributes)
  ```

- Identifying relationships (child cannot exist without parent) use a solid line
  `--`; non-identifying relationships use a dashed line `..`.
- Every `FK` attribute has a matching relationship line, and every relationship has
  its `FK` attribute on the many side.
- Purely in-memory or transient objects do not belong in the ER diagram.

## Cross-references

- Links between documents are relative paths, so they work on GitHub and in editors.
- A feature's `architecture.md` links to its `spec.md`, `plan.md` and
  `data-model.md`, and to the project-level `context.md`, `containers.md` and
  `erd.md`.
- A new or changed element in a feature diagram is listed in that file's
  **Architecture Impact** table, which is the input for updating the project-level
  views.
