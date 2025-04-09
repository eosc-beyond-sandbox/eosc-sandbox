# Catalogue and Marketplace

## EOSC Service Catalogue

The EOSC Service Catalogue provides data and functionality to register, maintain, administer, and share resources onboarded by various providers. It also serves as the point of reference for all EOSC Core components that add value to this information, making the data and services searchable and accessible through various tools for both researchers and end users.

EOSC Service Catalogue has two main components: **Registry** and **Provider Dashboard**.

The **Registry** offers the underlying storage functionality and the interoperability tools for the programmatic access, registration, manage (CRUD) of providers, services, and catalogs. It also offers the necessary API functionality for the interoperability of service catalogs from individual providers or aggregators (e.g., thematic or regional catalogs).

| Registry            |                                                          |
| :------------------ | :------------------------------------------------------- |
| Service URL         | <https://providers.sandbox.eosc-beyond.eu/>              |
| User guides         | <https://zenodo.org/records/8333871>                     |
| Software repository | <https://github.com/madgeek-arc/resource-catalogue-docs> |

The **Providers Dashboard** facilitates front-end functionality for registering providers and organizations that are eligible to publish their resources in the catalog. It offers features for onboarding and managing these resources effectively. Additionally, the dashboard provides representatives from provider organizations with a comprehensive view of their offerings, along with various usage statistics related to their resources. Moreover, it supports members of the onboarding team by providing tools to manage entries, oversee the onboarding process of providers, and audit the resources that have been onboarded.

| Provider Dashboard |                                             |
| :----------------- | :------------------------------------------ |
| Service URL        | <https://providers.sandbox.eosc-beyond.eu/> |
| User guides        | <https://zenodo.org/records/8333871>        |

## EOSC Research Product Catalogue

| Knowledge Graph           |                                                                                                                                                                            |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Service URL               | <https://graph.openaire.eu>                                                                                                                                                |
| User guides               |                                                                                                                                                                            |
| Integration documentation | [https://wiki.eoscfuture.eu/.../Guidelines](https://wiki.eoscfuture.eu/display/PUBLIC/1.2+EOSC+Research+Product+Catalogue%3A+Architecture+and+Interoperability+Guidelines) |

| Research Product Provider Dashboard |                                                                                                                                                                            |
| :---------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Service URL                         | <https://beta.provide.openaire.eu>                                                                                                                                         |
| Integration documentation           | [https://wiki.eoscfuture.eu/.../Guidelines](https://wiki.eoscfuture.eu/display/PUBLIC/1.2+EOSC+Research+Product+Catalogue%3A+Architecture+and+Interoperability+Guidelines) |
| User guides                         |                                                                                                                                                                            |

## EOSC Marketplace

EOSC Beyond Marketplace is a tool where researchers can discover, order, and access the services, analytical tools, data management tools, storage, and computing resources they need for their work. It is an integrated platform that allows easy access to many services for various research domains alongside data and integrated data analytics tools. Furthermore, the Marketplace offers the opportunity to advertise services and resources from local, national, and international providers, like European e-Infrastructures and Research Infrastructures, with the aim of promoting access to a broader international user group.

| Marketplace               |                                                                 |
| :------------------------ | :-------------------------------------------------------------- |
| Service URL               | <https://marketplace.sandbox.eosc-beyond.eu>                    |
| User guides               | <https://marketplace.sandbox.eosc-beyond.eu/user_documentation> |
| Integration documentation | <https://marketplace.docker-fid.grid.cyf-kr.edu.pl/api_docs>    |
| Software repository       | <https://github.com/cyfronet-fid/marketplace>                   |

## EOSC Explore

EOSC Explore is one of the key components of the EOSC Beyond initiative, providing researchers with an intuitive platform to discover, access, and integrate a wide range of scientific resources from multiple EOSC Nodes. By leveraging the EOSC Interoperability Framework, EOSC Explore ensures seamless cross-domain access and compliance with FAIR data principles, empowering researchers to compose scientific workflows, foster collaboration, and accelerate scientific discoveries within a federated Open Science ecosystem.

| Explore     |                               |
| :---------- | :---------------------------- |
| Service URL | <https://explore.openaire.eu> |

## EOSC Resource Discovery Hub

The EOSC Resource Discovery Hub provides a research-focused tool enabling users to effectively locate, manage, and reuse a range of resources within the EOSC ecosystem. Users can access highly relevant content tailored to their needs through an intuitive interface and advanced filtering options, creating a successful research experience that aligns with their workflows and expectations.

| Search Service      |                                                                 |
| :------------------ | :-------------------------------------------------------------- |
| Service URL         | <https://search.marketplace.sandbox.eosc-beyond.eu>             |
| User guides         | <https://marketplace.sandbox.eosc-beyond.eu/user_documentation> |
| Software repository | <https://github.com/cyfronet-fid/eosc-search-service>           |

## EOSC User Dashboard

EOSC User Dashboard is an AI-based - personalized to some extent - user environment to display users' personal website space. Each user can define and design his scientific profile and - based on it - receive resources tailored to their needs. The dashboard helps them find resources that match their specific fields of interest, making it more straightforward to explore the EOSC Resource Discovery Hub. It consists of an informative landing page and a personal subpage available only for authenticated users.

| User Dashboard      |                                                       |
| :------------------ | :---------------------------------------------------- |
| Service URL         | <https://my.sandbox.eosc-beyond.eu>                   |
| Software repository | <https://github.com/cyfronet-fid/eosc-user-dashboard> |

## EOSC Order Management

EOSC Order Management is a set of services, processes, and guidelines designed to facilitate a unified ordering process for resources, providing a uniform experience for users and enabling efficient resource management for providers. It provides a coherent user journey from resource discovery to procurement within the EOSC Marketplace, enhancing the user experience and supporting inter-provider communications to improve resource composability and outreach. Furthermore, it integrates various Order Management Systems through a flexible API, allowing seamless communication between the EOSC Marketplace and provider-specific Order Management Systems. This integration supports a wide range of provider use cases and enables efficient resource provisioning and order management.

| Order Handling System |                                                                                      |
| :-------------------- | :----------------------------------------------------------------------------------- |
| Service URL           | [Swagger UI](https://marketplace.sandbox.eosc-beyond.eu/api_docs/swagger/index.html) |
| User guides           | <https://marketplace.sandbox.eosc-beyond.eu/api_docs>                                |

## EOSC PID Service

The Persistent Identifiers (PID) Service offers a robust, reliable solution for the long-term identification and management of digital objects underpinned by leading European data centers and cutting-edge technology. Here are the core value propositions of the PID Service:

1. **Global Persistence and Reliability**: the PID Service ensures enduring identification of digital objects through a globally resolvable Handle System. This persistence is crucial for maintaining long-term access and usability of research data, tools, and resources.
2. **High Performance and Redundancy**: Utilizing advanced data center infrastructure and a redundancy scheme compliant with ePIC Consortium’s Quality of Service policies, the PID Service guarantees high performance and reliability. The primary-mirror federation architecture ensures that PIDs are mirrored across multiple providers, minimizing the risk of service disruption.
3. **Advanced Technology Integration**: The service is based on B2HANDLE technology, incorporating the distributed Handle System solution for efficient storage, management, and resolution of PIDs. It supports both REST API and native Handle API, enabling seamless integration with middleware applications, end-user tools, and other services.
4. **Flexible and Accessible Interfaces**: The PID Service provides both a REST API and a native Handle API, catering to diverse user needs. These interfaces facilitate easy integration and interaction with the PID service, whether for managing data objects or integrating with other digital infrastructures.
5. **Secure and Scalable Infrastructure**: Leveraging the latest advancements in virtualization and data storage technologies, the PID Service ensures secure, scalable, and efficient handling of PIDs. This infrastructure supports a wide range of applications, from catalogues and registries to training materials and guidelines.
6. **Global Resolve Capability**: PIDs are globally resolvable through the DONA Global Handle Registry network, ensuring that identifiers remain consistent and accessible across different systems and locations, regardless of changes in object location or ownership.
7. **Comprehensive Coverage**: The PID Service supports a broad spectrum of catalogues and registries, including services/providers, tools, training materials, and guidelines, enhancing the traceability and management of various digital resources.

By delivering a high-performance, reliable, and globally accessible PID service, the PID Service ensures that digital objects are effectively managed and identified over time, supporting the integrity and continuity of research and data management efforts.

| PID Service |                                 |
| :---------- | :------------------------------ |
| Service URL | <https://www.pidconsortium.net> |
| User guides | <https://doc.pidconsortium.net> |
