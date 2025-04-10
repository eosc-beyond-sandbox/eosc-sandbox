# AAI

## EOSC Beyond Core Infrastructure Proxy

The EOSC Core Infrastructure Proxy is responsible for connecting EOSC Core Services. It is composed of the following components: 

- The Service Proxy component, a multi-protocol service provider proxy, supports OAuth2, OpenID Connect, and SAML2 for connecting to services. It is connected to the Identity Proxy in the Identity Hub via OpenID Connect to authenticate users in a consistent way. 
- The Access Management component is responsible for implementing policies based on entitlements and resource capabilities pertaining to the connected services. This information typically includes group membership and roles available via the Identity Hub and Infrastructure Proxy, for controlling access to the connected services. 
- The Service Management component provides a web interface through which Service Owners of the EOSC Core Services can manage the lifecycle of their services.

| Core Infrastructure Proxy |                                                                                                                                                          |
| :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Service URL               | <https://core-proxy.sandbox.eosc-beyond.eu/auth/realms/core>                                                                                             |
| Integration documentation | [Service Registration in the EOSC Core Innovation Sandbox via the Core Infrastructure Proxy](registering-services-with-the-core-infrastructure-proxy.md) |
| User guides               |                                                                                                                                                          |
| Technology documentation  |                                                                                                                                                          |
| Owner                     | GRNET                                                                                                                                                    |

## EOSC Beyond Identity Hub

The Identity Hub is a multi-protocol authentication proxy supporting OAuth2, OpenID Connect, and SAML2, responsible for user authentication and providing consistent user information to services in the EOSC Federated AAI. It is composed of the following components:

- The Identity Proxy is a logical component that provides a consistent way for the EOSC Node services to authenticate users.
- The Service Proxy is the logical component responsible for connecting services.
- The Service Management component provides a web interface through which Service Owners of the connected services can manage the lifecycle of their services. This simplifies the registration and reconfiguration of services, minimizing operational and management efforts.
- The Account Registry is responsible for maintaining the user accounts, their attributes, and the user personal groups, ensuring that all users have the required attributes in the account profiles. 
- Access Management is responsible for managing entitlements and resource capabilities pertaining to the connected services. This information typically includes group membership and roles retrieved by the Account Registry, for controlling access to the connected services.

| Identity Proxy            |                                                   |
| :------------------------ | :------------------------------------------------ |
| Service URL               | <https://proxy.staging.beyond.eosc-federation.eu> |
| Integration documentation |                                                   |
| User guides               |                                                   |
| Technology documentation  |                                                   |
| Owner                     | GEANT                                             |

## EOSC Beyond Federated AAI Connector

The EOSC Federated AAI Connector is responsible for registering, maintaining, and publishing the trust anchors and the associated metadata for all the entities in the EOSC Node, and provides common horizontal functionalities. It fetches, validates, and stores information about all connected services and identity providers. It exposes an MDQ API.

| Federated AAI Connector   |                                                 |
| :------------------------ | :---------------------------------------------- |
| Service URL               | <https://mdx.staging.beyond.eosc-federation.eu> |
| Integration documentation |                                                 |
| User guides               |                                                 |
| Technology documentation  |                                                 |
| Owner                     | GEANT                                           |
