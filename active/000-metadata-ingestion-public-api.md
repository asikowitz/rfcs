- Start Date: 2024-11-18
- RFC PR: (after opening the RFC PR, update this with a link to it and update the file name)
- Implementation PR(s): \<None>

# Python SDK Public API

## Summary

This RFC proposes a standardized approach for maintaining and communicating the public API surface
of the `acryl-datahub` Python package, ensuring clear boundaries between public and internal
interfaces while avoiding disruptive changes to the existing package structure.

## Motivation

- SDK maintainers need to clearly communicate what parts of the codebase are part of the public API
- Users need to easily identify which classes and functions they can rely on
- Breaking changes to public APIs require careful versioning and migration paths

## Requirements

- Provide clear distinction between public and internal APIs, for both maintainers and users
- Avoid mass restructuring of existing code, for users who rely on the current package structure
- Support semantic versioning and maintain backwards compatibility for public APIs
- Work with standard Python tooling (IDEs, mypy, documentation generators)

### Extensibility

- Should support adding automated validation of public API:
  - Checking for proper documentation and type hints
  - Requiring new version for backwards-incompatible changes

## Detailed design

Define `__all__` in the root `__init__.py` of the package, i.e. `metadata-ingestion/src/datahub/__init__.py`.
This file will contain imports of all public classes and functions, and will be the only file that users
should import from. For all inner modules, set `__all__` to an empty list, until we are ready to expose them
as part of the public API. We must also call this out in the release notes and documentation.

## Basic example

```python
# metadata_ingestion/src/datahub/__init__.py
from datahub.ingestion.graph.client import DatahubClientConfig, DataHubGraph
from datahub.ingestion.api.source import Source, SourceReport

__all__ = ["DatahubClientConfig", "DataHubGraph", "Source", "SourceReport"]
```

## How we teach this

This affects those who fork the `acryl-datahub` Python package, or use it as a dependency in their own projects.
We should update the release notes, along with the `metadata-ingestion/` docs files
`README.md`, `as-a-library.md`, and `developing.md` to call out the new convention.

## Drawbacks

The primary drawback is that it's not immediately obvious which classes and functions are part of the public API:
users must know to import from `acryl-datahub` and not from other modules. Users do not always read
documentation, so this could lead to confusion and accidental use of internal APIs. It is also not fully
standard practice in Python to put all imports at the root level, so it may be unfamiliar to some users.

## Alternatives

- Mark internal APIs with a leading underscore
  * Pros
    + Simple and widely understood convention
  * Cons:
    + Requires renaming the majority of existing classes and methods
    + Contributors can forget to add the underscore, leading to accidental exposure of internal APIs
    + Discovery of public APIs is not easy
- All modules are default public, with `__all__` defined in each
  * Pros
    + Standard convention in Python
    + Import paths describe the import's location
  * Cons:
    + Contributors can forget to add new classes and methods to `__all__`
    + Discovery of public API is not as easy
- Organize public APIs in a separate `public` module, importing everything from the internal modules
  * Pros
    + Clearer separation between public and internal APIs
    + Easier to discover public APIs
    + No need to rename existing classes and methods
  * Cons:
    + Less standard convention in Python
    + Will require deprecation of the `public` module if we ever want to move to the standard convention
    + The `public` module may become bloated as we add more public APIs
- Add a decorator to mark public classes and functions
  * Pros
    + Clear distinction between public and internal APIs, in the code itself
    + No need to rename existing classes and methods
  * Cons:
    + Requires contributors to remember to add the decorator
    + Discovery of public APIs is not easy
    + Not standard practice in Python

## Rollout / Adoption Strategy

We need to clearly communicate that users should only import from `acryl-datahub` and not from any other module. 

## Future Work

- Add automated validation of public API
- Add more public classes and functions to the public API
