# Adapters

An **Adapter** is a software component (typically a library, wrapper, or connector) that helps a service interact with EOSC services or implement EOSC interoperability guidelines.

Adapters typically:

- Wrap EOSC APIs
- Simplify integration with EOSC services
- Translate between standards or metadata formats
- Provide client libraries for common programming languages
- Implement functionality aligned with EOSC Interoperability Framework (IF) Guidelines

Examples include:

- Python or Java wrappers for EOSC service APIs
- Metadata translators (e.g., between EOSC profiles and DCAT)
- Libraries for publishing monitoring or accounting metrics

Adapters are **optional components**: EOSC services must remain usable without them.

---

Adapters are created at the [Providers Dashboard](https://providers.sandbox.eosc-beyond.eu), and offered through the [Discovery Hub](https://search.userspace.sandbox.eosc-beyond.eu/search/adapter?q=*).

Before publishing an Adapter, ensure it satisfies the following requirements. _Those requirements are represented as mandatory fields in the Providers Dashboard publication workflow._

## Open Source

Adapters must be **open-source software**.

Recommended licenses include:

- Apache License 2.0
- MIT
- BSD
- ...any [SPDX license](https://spdx.org/licenses/)

The license must be clearly declared in the repository.


## Public Code Repository

The Adapter source code must be available in a **public repository**, for example:

- GitHub
- GitLab
- Bitbucket
- Institutional repositories

The repository should contain:

- Source code
- README documentation
- License file
- Usage instructions
- Release information


## Documentation

Each Adapter must provide clear documentation, including:

- Description of the Adapter
- Target EOSC service(s)
- Associated EOSC Interoperability Guideline
- Installation instructions
- Example usage
- API documentation (if applicable)

Documentation is typically provided in the repository README or a documentation website (eg, RTD).


## Link to an EOSC Interoperability Guideline

Every Adapter are be associated with an EOSC Interoperability Framework (IF) Guideline.

The guideline defines:

- the interface
- the protocol
- the interoperability rule implemented by the Adapter

> In case a suitable Guideline is not yet available, you can associate your Adapter with its respective EOSC Service.

- [Catalogue Adapters](Research Product Catalogue/Catalogue.md)
- [Accounting Adapters](Accounting/Accounting.md)
- [Horizontal Adapters](Horizontal/Horizontal.md)
- [PID Adapters](PID/PID.md)
- [Messaging Adapters](Messaging/Messaging.md)
- [Monitoring Adapters](Monitoring/Monitoring.md)
- [Helpdesk Adapters](Helpdesk/Helpdesk.md)
