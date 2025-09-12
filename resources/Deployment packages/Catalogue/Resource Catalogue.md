===

## Description


The EOSC Beyond Resource Catalogue solution utilizes an [open-source web based cataloging application](https://docs.sandbox.eosc-beyond.eu/Service%20Portfolio/Resource%20Catalogue/Resource%20Catalogue/) to manage and organize a diverse range of resources (providers, services, catalogs). Its source code is available on GitHub, licensed under Apache 2.0 software license. Additionally, the resource catalogue can seamlessly be integrated with the EOSC Beyond AAI service.

## Code

The Resource Catalogue consists of two main components - the frontend and the backend. These components are hosted in separate code repositories on GitHub:

- Resource Catalogue Backend - [https://github.com/madgeek-arc/resource-catalogue](https://github.com/madgeek-arc/resource-catalogue) 
- Resource Catalogue Frontend - [https://github.com/madgeek-arc/resource-catalogue-ui](https://github.com/madgeek-arc/resource-catalogue-ui)

The backend implements a REST API, while the frontend consumes it and offers an easy-to-use web interface. 


Both the frontend and the backend can be installed as standalone applications by compiling the code from scratch and running the relevant Node.JS and Java application servers, respectively. Alternatively, Docker container definitions are available in the repositories, allowing Docker image to be built.

- [Dockerfile](https://github.com/madgeek-arc/resource-catalogue/blob/master/Dockerfile) for Resource Catalogue Backend
- [Dockerfile](https://github.com/madgeek-arc/resource-catalogue-ui/blob/master/Dockerfile) for Resource Catalogue Frontend

## Dependencies

The Resource Catalogue relies on the following components and they need to be already deployed before the installation of the catalogue itself:

- ActiveMQ Classic - message queue, [https://activemq.apache.org/components/classic/](https://activemq.apache.org/components/classic/)
- Elasticsearch - NoSQL database, [https://www.elastic.co/elasticsearch](https://www.elastic.co/elasticsearch) 
- PostgreSQL - relational database, [https://www.postgresql.org/](https://www.postgresql.org/)
- Redis - key/value caching database, [https://redis.io/](https://redis.io/)
- An optional reverse proxy to terminate SSL connections

More details about the software dependencies can be obtained from the [README file](https://github.com/madgeek-arc/resource-catalogue) placed in the code repository for the backend, as well as from the example [application.properties](https://github.com/madgeek-arc/resource-catalogue/blob/master/resource-catalogue-service/src/main/resources/application.properties) file, showing the default configuration options used by the software. 

## Database Setup

The Resource Catalogue requires a PostgreSQL database. No extra PostgreSQL extensions are required. Detailed instructions on how to deploy a PostgreSQL database from scratch in a Docker environment are available in the [PostgreSQL section](#PostgreSQL) of the [Deployment Guidelines](#Deployment-Guidelines).

After the initial startup of the application, additional database seeding needs to be performed using the Resource Catalogue's API. More details about this are available in the [Data Restore and Database Seeding](#Data-Restore-and-Database-Seeding) section.

## Configuration Files and Examples

### Deploying and Configuring Docker

Docker is used for building and instantiating the container environments for each component. Instructions for installing Docker on popular GNU/Linux distributions are available on the [official Docker website](https://docs.docker.com/engine/install/). 

After a successful Docker installation, a dedicated Docker networks are created:

```bash
docker network create eosc
docker network create traefik # if traefik is to be used as a reverse proxy
```

The `catalogue` dedicated Docker network will allow for each dependency to be deployed using a separate docker-compose file, while still sharing the same network. Whether to define all dependencies within a single docker-compose file or separate ones is a matter of preference. This guide will assume that each dependency is defined in a separate docker-compose file.

The necessary directory tree can then be created using:

```
cd ~/
mkdir activemq-classic elasticsearch postgresql redis resource-catalogue traefik
```

### Example: Using Traefik as a Reverse Proxy

Traefik can be used as a reverse proxy on the same Docker host where the Resource Catalogue would be deployed and can take care of automatic certificate issuing and renewals via its native support for Let's Encrypt. This guides assumes that Traefik will be used so that it is possible to deploy a fully functioning Resource Catalogue just by following the instructions in the document. However, any other reverse proxy can also be utilized, deployed either on the Docker host itself or on a different host. In cases where a different reverse proxy is used, all references to Traefik can be omitted from the configuration files.

```bash
mkdir -p /opt/docker-data/traefik/acme/
touch /opt/docker-data/traefik/acme/acme.json
```

```yaml=
services:
  traefik:
    image: 'traefik:v2.11'
    container_name: 'traefik'
    command:
      - '--log.level=INFO'
      - '--entrypoints.web.address=:80'
      - '--entryPoints.websecure.address=:443'
      - '--entrypoints.web.http.redirections.entryPoint.to=websecure'
      - '--entrypoints.web.http.redirections.entryPoint.scheme=https'
      - '--providers.docker.exposedbydefault=false'
      - '--providers.docker=true'
      - '--api.dashboard=true'
      - '--api=true'
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.email=john.doe@example.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '/var/run/docker.sock:/var/run/docker.sock:ro'
      - '/opt/docker-data/traefik/acme/acme.json:/letsencrypt/acme.json'
    labels:
      - 'traefik.enable=true'
      - 'traefik.docker.network=traefik'
    restart: 'unless-stopped'
    networks:
      traefik: {}
networks:
  traefik:
    external: true
```

In the example above, the email on line `#16` needs to be customized to point to a real e-mail address used for registering to Let's Encrypt's ACME server.

Once the container is started using `docker compose up -d`, virtual hosts for the reverse proxy are defined using labels in the respective `docker-compose.yml` file of the service, as is the case with the labels for the [Resource Catalogue](#Resource-Catalogue1).

### Data Restore and Database Seeding

The resource catalogue, via its `/api/restore` endpoint allows backup files to be restored and the database to be populated (seeded) with initial content. 

This is a necessary step after the initial deployment so that the application becomes fully functional.

A restore can be done using:

```
curl --location 'https://cat.example.eu/api/restore/' --header 'Cookie: SESSION=SECRET_SESSION_COOKIE' --form 'datafile=@"/tmp/restore.zip"'
```

The `SECRET_SESSION_COOKIE` string in the above example needs to be replaced by a session cookie obtained after login to the catalogue using a web browser. The session cookie can be extracted using the `Developer Tools` option in any modern web browser.

The `restore.zip` file that needs to be restored to seed the database can be downloaded from the following link - [restore.zip](https://owncloud.finki.ukim.mk/index.php/s/AZqtQ0fYN9K1DKo).

## Deployment Guidelines

The reference Resource Catalogue deployment, along with all of its dependencies, is done using Docker containers which greatly simplifies not only the initial installation process, but also later updates and backups. Ubuntu 24.04 is used as the base operating system, but the installation steps should be similar for other GNU/Linux distributions as well.

To improve readability, the necessary deployment steps for each of the components are given below, in their own dedicated subsections.

### ActiveMQ Classic

Official Docker images are available for ActiveMQ Classic on Docker Hub - [apache/activemq-classic](https://hub.docker.com/r/apache/activemq-classic). 

The ActiveMQ Classic docker-compose file (which can be stored in `~/docker-compose/activemq-classic/docker-compose.yml`), is given below:

```yaml
services:
  activemq-classic:
    container_name: activemq-classic
    image: apache/activemq-classic:5.18.7
    volumes:
      - /opt/docker-data/activemq-classic/config/jetty-realm.properties:/opt/apache-activemq/conf/jetty-realm.properties:ro
    networks:
      eosc: {}
networks:
  eosc:
    external: true
```

The contents of `/opt/docker-data/activemq-classic/config/jetty-realm.properties` which is mapped read-only is:

```
# Defines users that can access the web (console, demo, etc.)
# username: password [,rolename ...]
admin: ADMIN_CHANGE_ME, admin
user: USER_CHANGE_ME, user
```

The `ADMIN_CHANGE_ME` and `USER_CHANGE_ME` strings above can be replaced by a randomly generated password. 

The container can be started by executing:

```bash
cd ~/docker-compose/activemq-classic
docker compose up -d
```

Logs can be monitored using `docker compose logs -f`. 

### Elasticsearch

Official Docker images are available for Elasticsearch on Docker Hub - [elasticsearch](https://hub.docker.com/_/elasticsearch).

The Elasticsearch docker-compose file (which can be stored in `~/docker-compose/elasticsearch/docker-compose.yml`), is given below:

```yaml
services:
  elasticsearch:
    container_name: elasticsearch
    hostname: elasticsearch
    image: elasticsearch:7.17.28
    volumes:
      - /opt/docker-data/elasticsearch/data:/usr/share/elasticsearch/data
    environment:
      - cluster.name=elasticsearch
      - bootstrap.memory_lock=true
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    networks:
      eosc: {}
networks:
  eosc:
    external: true
```

The necessary directory structure in `/opt/docker-data/elasticsearch/data`, mapped to `/usr/share/elasticsearch/data` will be created automatically upon the first start of the container and no further manual action is needed.

The container can be started by executing:

```bash
cd ~/docker-compose/elasticsearch
docker compose up -d
```

Logs can be monitored using `docker compose logs -f`. 

### PostgreSQL

Official Docker images are available for PostgreSQL on Docker Hub - [postgres](https://hub.docker.com/_/postgres).

The PostgreSQL docker-compose file (which can be stored in `~/docker-compose/postgresql/docker-compose.yml`), is given below:

```yaml
services:
  postgresql:
    container_name: postgresql
    hostname: postgresql
    image: postgres:16
    environment:
      POSTGRES_USER: 'catalogue'
      POSTGRES_PASSWORD: 'CHANGE_ME'
    volumes:
      - /opt/docker-data/postgresql/data:/var/lib/postgresql/data
    networks:
      eosc: {}
networks:
  eosc:
    external: true
```

The necessary directory structure in `/opt/docker-data/postgresql/data`, mapped to `/var/lib/postgresql/data` will be created automatically upon the first start of the container and no further manual action is needed.

The values for the environment variables `POSTGRES_USER` and `POSTGRES_PASSWORD` can be customized as desired.

The container can be started by executing:

```bash
cd ~/docker-compose/postgresql
docker compose up -d
```

Logs can be monitored using `docker compose logs -f`.

### Redis

The bitnami-legacy images are used to deploy Redis - [bitnamilegacy/redis](https://hub.docker.com/r/bitnamilegacy/redis). However, the official Redis docker images can also be used instead - [redis](https://hub.docker.com/_/redis). 

The Redis docker-compose file (which can be stored in `~/docker-compose/redis/docker-compose.yml`), is given below:

```yaml
services:
  redis:
    container_name: redis
    hostname: redis
    image: bitnamilegacy/redis:7.0
    environment:
      REDIS_PASSWORD: 'CHANGE_ME'
      ALLOW_EMPTY_PASSWORD: 'no'
      REDIS_DATABASES: '32'
    volumes:
      - /opt/docker-data/redis:/bitnami
    networks:
      eosc: {}
networks:
  eosc:
    external: true
```

The necessary directory structure in `/opt/docker-data/redis`, mapped to `/bitnami` will be created automatically upon the first start of the container and no further manual action is needed.

The value for the environment variable `REDIS_PASSWORD` can be set to a random password.

The container can be started by executing:

```bash
cd ~/docker-compose/redis
docker compose up -d
```

Logs can be monitored using `docker compose logs -f`. 

### Resource Catalogue

Both the backend and the frontend components for the Resource Catalogue are defined in a single docker-compose.yml file, available below.

```yaml
services:
  resource-catalogue:
    image: resource-catalogue:v5.1.0
    container_name: resource-catalogue
    env_file: .env
    networks:
      traefik: {}
      eosc: {}
  resource-catalogue-ui:
    image: resource-catalog-ui:v5.1.0
    container_name: resource-catalogue-ui
    networks:
      traefik: {}
      eosc: {}
    environment:
      - PLATFORM_API_ENDPOINT=http://resource-catalogue:8080
      - FAQ_API_ENDPOINT=http://example.com # edit FAQ endpoint
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.resourcemanager.rule=Host(`catalogue.example.eu`)'
      - 'traefik.http.routers.resourcemanager.tls=true'
      - "traefik.http.routers.resourcemanager.tls.certresolver=letsencrypt"
      - 'traefik.docker.network=traefik'
      - 'traefik.http.services.resourcemanager.loadbalancer.server.port=80'

networks:
  traefik:
    external: true
  eosc:
    external: true
```

The above example assumes that Traefik will be used as a reverse proxy and will handle SSL certificate issuing (via a Let's Encrypt HTTP challenge) and later SSL termination. 

If an existing reverse proxy needs to be used, then port 80 in the `resource-catalogue-ui` container spec in the docker-compose file can be mapped to a host port:

```yaml
  resource-catalogue-ui:
  ...
    ports:
      - 8081:80
  ...
```

Finally, the existing reverse proxy should be configured to forward traffic to port `8081`. 

All of the required environment variables are stored in a `.env` file next to the `docker-compose.yml`. If it does not exist, create the file and customize the parameters as needed.

```env=
# .env
# PostgreSQL Datasource Configuration
REGISTRY_DATASOURCE_CONFIGURATION_MAXIMUM_POOL_SIZE=10
REGISTRY_DATASOURCE_DRIVER_CLASS_NAME=org.postgresql.Driver
REGISTRY_DATASOURCE_USERNAME=resource_catalogue
REGISTRY_DATASOURCE_PASSWORD=CHANGE_ME # change required
REGISTRY_DATASOURCE_URL=jdbc:postgresql://postgresql:5432/resource_catalogue # change required

# JPA & Hibernate Settings for PostgreSQL
REGISTRY_JPA_PROPERTIES_HIBERNATE_ALLOW_UPDATE_OUTSIDE_TRANSACTION=true
REGISTRY_JPA_PROPERTIES_HIBERNATE_DIALECT=org.hibernate.dialect.PostgreSQLDialect
REGISTRY_JPA_PROPERTIES_HIBERNATE_ENABLE_LAZY_LOAD_NO_TRANS=true
REGISTRY_JPA_PROPERTIES_HIBERNATE_FORMAT_SQL=false
REGISTRY_JPA_PROPERTIES_HIBERNATE_HBM2DDL_AUTO=update
REGISTRY_JPA_PROPERTIES_HIBERNATE_SHOW_SQL=false

#########################
## Server Properties   ##
#########################

DYNAMIC_PROPERTIES_PATH=
SERVER_PORT=8080
SERVER_SERVLET_CONTEXT_PATH=/api
LOGGING_LEVEL_ROOT=INFO

#########################
## Spring Properties   ##
#########################

SPRING_PROFILES_ACTIVE=beyond
SPRING_SERVLET_MULTIPART_MAX_FILE_SIZE=50MB
SPRING_SERVLET_MULTIPART_MAX_REQUEST_SIZE=50MB
SPRING_JPA_OPEN_IN_VIEW=false
SPRING_MAIN_ALLOW_BEAN_DEFINITION_OVERRIDING=true

SPRING_DATA_REDIS_HOST=redis
SPRING_DATA_REDIS_PORT=6379
SPRING_DATA_REDIS_PASSWORD=CHANGE_ME # change required

SPRING_SECURITY_OAUTH2_CLIENT_PROVIDER_EOSC_ISSUER_URI=https://aai.ni4os.eu/auth/realms/ni4os # change required
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_EOSC_CLIENT_ID=resource-manager # change required
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_EOSC_CLIENT_NAME=resource-manager # change required
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_EOSC_CLIENT_SECRET=CHANGE_ME # change required
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_EOSC_SCOPE=openid,email,profile
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_EOSC_REDIRECT_URI=https://catalogue.example.eu/api/login/oauth2/code/eosc # change required
SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_ISSUER_URI=https://aai.ni4os.eu/auth/realms/ni4os # change required

SPRINGDOC_API_DOCS_PATH=/api-docs
SPRINGDOC_GROUP_CONFIGS_0_GROUP=resource-catalogue
SPRINGDOC_GROUP_CONFIGS_0_DISPLAY_NAME=1.Resource Catalogue
SPRINGDOC_GROUP_CONFIGS_0_PACKAGES_TO_SCAN=gr.uoa.di.madgik.resourcecatalogue
SPRINGDOC_GROUP_CONFIGS_1_GROUP=dynamic-catalogue
SPRINGDOC_GROUP_CONFIGS_1_DISPLAY_NAME=2.Dynamic Catalogue
SPRINGDOC_GROUP_CONFIGS_1_PACKAGES_TO_SCAN=gr.uoa.di.madgik.catalogue
SPRINGDOC_GROUP_CONFIGS_2_GROUP=registry
SPRINGDOC_GROUP_CONFIGS_2_DISPLAY_NAME=3.Registry
SPRINGDOC_GROUP_CONFIGS_2_PACKAGES_TO_SCAN=gr.uoa.di.madgik.registry
SPRINGDOC_SWAGGER_UI_DOCEXPANSION=none
SPRINGDOC_SWAGGER_UI_OPERATIONSSORTER=method
SPRINGDOC_SWAGGER_UI_SYNTAXHIGHLIGHT_ACTIVATED=false
SPRINGDOC_SWAGGER_UI_TAGSSORTER=alpha
SPRINGDOC_SWAGGER_UI_TRYITOUTENABLED=true
springdoc.swagger-ui.server-url=https://catalogue.example.eu # change required
springdoc.open-api.servers[0].url=https://catalogue.example.eu # change required
#########################
## Registry Properties ##
#########################

FQDN=catalogue.example.eu # change required
REGISTRY_HOST=http://localhost:8080/api

# Elasticsearch
REGISTRY_ELASTICSEARCH_URIS=elasticsearch:9200
REGISTRY_ELASTICSEARCH_USERNAME=
REGISTRY_ELASTICSEARCH_PASSWORD=

# ActiveMQ
REGISTRY_JMS_HOST=activemq-classic
REGISTRY_JMS_PREFIX=registry
REGISTRY_JMS_USERNAME=user
REGISTRY_JMS_PASSWORD=CHANGE_ME # change required

# AMS
CATALOGUE_JMS_AMS_HOST=
CATALOGUE_JMS_AMS_KEY=
CATALOGUE_JMS_AMS_PROJECT=
CATALOGUE_JMS_AMS_ENABLED=false

#########################
## Catalogue Settings  ##
#########################

CATALOGUE_ID=resource-catalogue
CATALOGUE_NAME=Resource Catalogue
CATALOGUE_HOMEPAGE=https://catalogue.example.eu # change required
CATALOGUE_VERSION=@project.version@
CATALOGUE_ADMINS=admin1@example.eu,admin2@example.eu # change required
CATALOGUE_ONBOARDING_TEAM=admin1@example.eu # change required
CATALOGUE_LOGIN_REDIRECT=https://catalogue.example.eu # change required
CATALOGUE_LOGOUT_REDIRECT=https://catalogue.example.eu # change required

catalogue.resources.adapter.id-prefix=adapter
catalogue.resources.deployable-service.id-prefix=deployable_service
catalogue.resources.configuration-template.id-prefix=configuration_template
catalogue.resources.configuration-template-instance.id-prefix=configuration_template_instance
catalogue.resources.datasource.id-prefix=datasource
catalogue.resources.helpdesk.id-prefix=helpdesk
catalogue.resources.interoperability-record.id-prefix=interoperability_record
catalogue.resources.monitoring.id-prefix=monitoring
catalogue.resources.provider.id-prefix=provider
catalogue.resources.resource-interoperability-record.id-prefix=resource_interoperability_record
catalogue.resources.service.id-prefix=service
catalogue.resources.tool.id-prefix=tool
catalogue.resources.training_resource.id-prefix=training_resource
catalogue.resources.vocabulary-curation.id-prefix=vocabulary_curation

# Email Settings
CATALOGUE_EMAILS_ENABLED=false
CATALOGUE_EMAILS_ADMIN_NOTIFICATIONS=false
CATALOGUE_EMAILS_PROVIDER_NOTIFICATIONS=false
CATALOGUE_EMAILS_REGISTRATION_EMAILS_TO=
CATALOGUE_EMAILS_HELPDESK_EMAILS_TO=
CATALOGUE_EMAILS_HELPDESK_EMAILS_CC=
CATALOGUE_EMAILS_MONITORING_EMAILS_TO=
CATALOGUE_EMAILS_RESOURCE_CONSISTENCY_NOTIFICATIONS=false
CATALOGUE_EMAILS_RESOURCE_CONSISTENCY_EMAILS_TO=
CATALOGUE_EMAILS_RESOURCE_CONSISTENCY_EMAILS_CC=

# Mailer Settings
CATALOGUE_MAILER_HOST=mail.example.eu # change required
CATALOGUE_MAILER_FROM=catalogue@example.eu # change required
CATALOGUE_MAILER_USERNAME=catalogue@mrezhi.net # change required
CATALOGUE_MAILER_PASSWORD=CHANGE_ME # change required
CATALOGUE_MAILER_PORT=587
CATALOGUE_MAILER_PROTOCOL=smtp
CATALOGUE_MAILER_AUTH=true
CATALOGUE_MAILER_SSL=false
```

All of the parameters above that need to be customized have a comment `# change required` next to them. The `.env` file already includes the necessary properties for integrating with the [NI4OS AAI](https://wiki.ni4os.eu/index.php/AAI_guide_for_SPs). Integration with any other OIDC compliant identity provider is also possible. 

## Documentation

Additional documentation about the Resource Catalogue can be found in the following locations:

- [Beyond Resource Catalogue](https://docs.sandbox.eosc-beyond.eu/Service%20Portfolio/Resource%20Catalogue/Resource%20Catalogue/)
- [Resource Catalogue GitHub Repository](https://github.com/madgeek-arc/resource-catalogue)
- [API Docs (Swagger)](https://providers.sandbox.eosc-beyond.eu/api/swagger-ui/index.html)

The Swagger-based API docs are available on every deployed instance of the Resource Catalogue under `/api/swagger-ui/index.html`.

## Licensing 

The Resource Catalogue backend and frontend are distributed under the [Apache 2.0 software license](https://www.apache.org/licenses/LICENSE-2.0).

The licenses of the other supporting components are as follows:

- Apache ActiveMQ Classic - [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0)
- Elasticsearch - [Elastic License 2.0](https://www.elastic.co/licensing/elastic-license)
- PostgreSQL - [PostgreSQL License](https://opensource.org/license/postgresql)
- Redis - [RSALv2](https://redis.io/legal/rsalv2-agreement/) (before 8.0, [AGPLv3](https://www.gnu.org/licenses/agpl-3.0.en.html) for later versions)

[Opensearch](https://opensearch.org/), licensed under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) can be used as an alternative to Elasticsearch.

[Valkey](https://valkey.io/), licensed under [BSD-3-Clause](https://opensource.org/license/bsd-3-clause) can be used as an alternative to Redis.