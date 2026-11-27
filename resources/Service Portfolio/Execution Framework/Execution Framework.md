# Execution Framework

## EOSC Beyond Deployment Service

The EOSC Deployment Service, implemented through the [Infrastructure Manager](https://im.egi.eu?target=_blank) (IM), is an open-source service to deploy complex applications on multiple Cloud back-ends ([AWS](https://aws.amazon.com?target=_blank), [OpenStack](https://www.openstack.org?target=_blank), etc.). It performs virtualized infrastructure provision, automatic software deployment, and lifecycle management of these applications. It supports the [OASIS TOSCA](https://www.oasis-open.org?target=_blank) (Topology and Orchestration Specification for Cloud Applications) standard for application architecture description.

| Infrastructure Manager    |                                           |
| :------------------------ | :---------------------------------------- |
| Service URL               | <https://deploy.sandbox.eosc-beyond.eu>   |
| Integration documentation | <https://imdocs.readthedocs.io/en/latest> |
| User guides               | <https://imdocs.readthedocs.io/en/latest> |
| Technology documentation  |                                           |
| Owner                     | UPV                                       |

## EOSC ADIM API

ADIM is the reference implementation of the [EOSC Application Deployment Management (ADM) API](https://github.com/EGI-Federation/eosc-application-deployment-manager). It exposes a federation-facing interface for application discovery, allocation management, and deployment lifecycle operations, and delegates application deployment to the Infrastructure Manager (IM).

| ADIM                      |                                                                         |
| :------------------------ | :---------------------------------------------------------------------- |
| Service URL               | <https://deploy.sandbox.eosc-beyond.eu/adim>                            |
| Integration documentation | <https://zenodo.org/records/20596323>                                   |
| User guides               |                                                                         |
| Technology documentation  |                                                                         |
| Owner                     | UPV                                                                     |
