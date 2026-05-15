# Marketplace 

## Description

The EOSC Beyond Marketplace solution utilizes an [open-source web application](https://docs.sandbox.eosc-beyond.eu/Service%20Portfolio/User%20Space/User%20Space/) to provide discovery and access to various research supporting services. The Marketplace can be used to advertise resources from local, national, and international providers. Its source code is available on [GitHub](https://github.com/cyfronet-fid/whitelabel-marketplace), licensed under [GNU General Public License v3.0 software license](https://github.com/cyfronet-fid/whitelabel-marketplace/blob/development/LICENSE). Additionally, the resource catalogue can seamlessly be integrated with the EOSC Beyond AAI service.

## Code  

To run the Marketplace application, two main components need to be deployed:

- The web application, providing the web UI and the REST API. 
- The worker component, executing background jobs asynchronously.

Both components are run from the same codebase and the source code is available at [https://github.com/cyfronet-fid/whitelabel-marketplace](https://github.com/cyfronet-fid/whitelabel-marketplace).

The two components can be installed as standalone applications by compiling the code from scratch and running the Ruby application server. Alternatively, a [Docker container definition (Dockerfile)](https://github.com/cyfronet-fid/whitelabel-marketplace/blob/development/Dockerfile) is available in the repository, allowing a Docker image to be built.

## Dependencies  

The Marketplace application relies on the following dependencies which need to be available before the actual Marketplace deployment can take place:

- PostgreSQL - relational database, [https://www.postgresql.org/](https://www.postgresql.org/)
- Elasticsearch - NoSQL database, [https://www.elastic.co/elasticsearch](https://www.elastic.co/elasticsearch)
- Redis - key/value caching database, [https://redis.io/](https://redis.io/)
- ActiveMQ Classic - message queue, [https://activemq.apache.org/components/classic/](https://activemq.apache.org/components/classic/)
- Docker - container runtime, [https://www.docker.com/](https://www.docker.com/)
- An optional reverse proxy to terminate SSL connections, e.g. Traefik, [https://traefik.io/traefik](https://traefik.io/traefik)

The rest of this deployment package reuses the environment setup conducted as part of the [EOSC Beyond Resource Catalogue](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/). The steps for the deployment of the individual dependencies remain the same between the two applications and can be freely reused:

- [Docker](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#deploying-and-configuring-docker)
- [Traefik Reverse Proxy](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#example-using-traefik-as-a-reverse-proxy)
- [PostgreSQL](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#postgresql)
- [Elasticsearch](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#elasticsearch)
- [Redis](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#redis)
- [ActiveMQ Classic](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#activemq-classic)

## Database Setup

The Marketplace requires a PostgreSQL database. No extra PostgreSQL extensions are needed. After [deploying PostgreSQL](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#postgresql) in a Docker environment, an extra database can be created using the following commands:

```bash
docker exec -it postgresql /bin/bash
psql -U catalogue

create database marketplace;
create user marketplace with encrypted password 'postgresql_secret';
ALTER DATABASE marketplace OWNER TO marketplace;
```

The new database will be automatically seeded during the first start of the Marketplace application. Any new database migrations will also be automatically applied on startup for future version upgrades of the Marketplace due to the execution of the `./bin/rails db:migrate` command on every container startup (part of the `command` parameter of the `docker-compose.yml` file available below).

For ActiveMQ a new user can be created by appending such a line in the `jetty-realm.properties` file (more details available as part of the [deployment guide for ActiveMQ](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#activemq-classic)):

```
username: mysecret, user
```

The first element is the username, the second is the password, and the third is the role name.

## Configuration Files and Examples  

The rest of this deployment package focuses on the deployment of the Marketplace component and assumes that all required dependencies have already been deployed in the environment, as per the instructions provided in the [Dependencies](#dependencies) section.

The reference Marketplace deployment is done using Docker containers which greatly simplifies not only the initial installation process, but also later updates and backups. Ubuntu 24.04 is used as the base operating system, but the installation steps should be similar for other GNU/Linux distributions as well.

### Marketplace Deployment

To deploy the Marketplace in a Docker environment a Docker image needs to be built first using these steps:

1. Clone the [cyfronet-fid/whitelabel-marketplace](https://github.com/cyfronet-fid/whitelabel-marketplace) repository.
2. Checkout a release-tag of the desired version of the Marketplace to be deployed, otherwise the Docker image will be built based on the latest commits available on the `development` (default) branch.
3. Build the image with `docker build`.
4. Deploy the web application and the worker component using a `docker-compose.yml` manifest file.

```
git clone https://github.com/cyfronet-fid/whitelabel-marketplace.git
cd whitelabel-marketplace
git tag # list all available Git tags
git checkout $tag_name # switch to the desired tag (recommended)
docker build -t marketplace:$tag_name . # build a new Docker image with the name `marketplace` and a tag matching the name of the previously used Git tag
```

Once a Docker image has been built, a corresponding `docker-compose.yml` file can be written, orchestrating the deployment of the two components (make sure to replace `$tag_name` with the appropriate tag used during the `docker build`):

```yml
services:
  web:
    image: marketplace:$tag_name # change with proper tag used to build the image
    command: bash -c "./bin/rails db:migrate && ./bin/rake searchkick:reindex:all && bundle exec puma"
    container_name: marketplace-web
    env_file:
      - marketplace.env
    volumes:
      - /opt/docker-data/marketplace/media:/marketplace/media # change with proper path to persist media files
    networks:
      traefik: {}
      eosc: {}
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.marketplace.rule=Host(`frontoffice.example.eu`)' # change with appropriate URL
      - 'traefik.http.routers.marketplace.tls=true'
      - "traefik.http.routers.marketplace.tls.certresolver=letsencrypt"
      - 'traefik.docker.network=traefik'
      - 'traefik.http.services.marketplace.loadbalancer.server.port=3000'

  worker:
    image: marketplace-custom:$tag_name
    command: bash -c "bundle exec sidekiq"
    container_name: marketplace-worker
    env_file:
      - marketplace.env
    volumes:
      - /opt/docker-data/marketplace/media:/marketplace/media # change with proper path to persist media files
    networks:
      eosc: {}
networks:
  traefik:
    external: true
  eosc:
    external: true
```

The Dockerized deployment of the Marketplace relies on the presence of the `traefik` and `eosc` Docker networks which act as shared networks between the various dependencies (PostgreSQL, Elasticsearch, Redis, Traefik...). Their creation is explained in the [Docker deployment section](https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#deploying-and-configuring-docker).

To improve the readability of the `docker-compose.yml` file, all environment variables have been externalized to a dedicated file named `marketplace.env` and referenced using the `env_file` parameter in the `docker-compose.yml`.

```ini
SECRET_KEY_BASE=secret_key # replace with a random string

STORAGE_DIR=/marketplace/media

ROOT_URL=https://frontoffice.example.eu # same value as the Traefik label added in `docker-compose.yml`

DATABASE_URL=postgres://marketplace:postgresql_secret@postgresql/marketplace # update according to PostgreSQL deployment
ELASTICSEARCH_URL=elasticsearch
DISABLE_DATABASE_ENVIRONMENT_CHECK=1
RAILS_SERVE_STATIC_FILES=1
REDIS_URL=redis://:redis_secret@redis:6379/10 # update according to Redis deployment

CHECKIN_HOST=aai.ni4os.eu
CHECKIN_ISSUER_ENDPOINT=auth/realms/ni4os
CHECKIN_AUTHORIZE_ENDPOINT=/auth/realms/ni4os/protocol/openid-connect/auth
CHECKIN_TOKEN_ENDPOINT=/auth/realms/ni4os/protocol/openid-connect/token
CHECKIN_USERINFO_ENDPOINT=/auth/realms/ni4os/protocol/openid-connect/userinfo
CHECKIN_JWK_ENDPOINT=/auth/realms/ni4os/protocol/openid-connect/certs
CHECKIN_IDENTIFIER=marketplace
CHECKIN_SECRET=checkin_secret # update according to obtained Check-In secret
CHECKIN_ISSUER_URI=https://aai.ni4os.eu/auth/realms/ni4os
CHECKIN_SCOPE=openid,profile,email

# Update with appropriate SMTP server values
SMPT_ADDRESS=mail.example.eu:587
SMPT_USERNAME=marketplace@example.eu
SMPT_PASSWORD=email_password
FROM_EMAIL=marketplace@example.eu

ASSET_HOST=frontoffice.ni4os.eu # same value as the Traefik label added in `docker-compose.yml`. No https:// prefix required.
ASSET_PROTOCOL=https

# ONLY FOR JIRA INTEGRATIONS
MP_JIRA_USERNAME=
MP_JIRA_PASSWORD=
MP_JIRA_URL=https://jira.egi.eu
MP_JIRA_ISSUE_TYPE_ID=10204
MP_JIRA_WF_TODO=10309
MP_JIRA_WF_IN_PROGRESS=3
MP_JIRA_WF_WAITING_FOR_RESPONSE=10310
MP_JIRA_WF_DONE=10311
MP_JIRA_WF_REJECTED=10103
MP_JIRA_WEBHOOK_SECRET=
MP_JIRA_PROJECT=EOSCSOPR
MP_JIRA_FIELD_Order_reference=customfield_10254
MP_JIRA_FIELD_CI_Name=customfield_10225
MP_JIRA_FIELD_CI_Surname=customfield_10226
MP_JIRA_FIELD_CI_Email=customfield_10227
MP_JIRA_FIELD_CI_DisplayName=customfield_10228
MP_JIRA_FIELD_CI_EOSC_UniqueID=customfield_10229
MP_JIRA_FIELD_CI_Institution=customfield_10243
MP_JIRA_FIELD_CI_Department=customfield_10244
MP_JIRA_FIELD_CI_SupervisorName=customfield_10248
MP_JIRA_FIELD_CI_SupervisorProfile=customfield_10249
MP_JIRA_FIELD_CP_CustomerTypology=customfield_10250
MP_JIRA_FIELD_CP_ReasonForAccess=customfield_10251
MP_JIRA_FIELD_CI_DepartmentalWebPage=customfield_10245
MP_JIRA_FIELD_SELECT_VALUES_CP_CustomerTypology_single_user=10187
MP_JIRA_FIELD_SELECT_VALUES_CP_CustomerTypology_research=10188
MP_JIRA_FIELD_SELECT_VALUES_CP_CustomerTypology_private_company=10189
MP_JIRA_FIELD_SO_1=customfield_10711
MP_JIRA_FIELD_CP_ScientificDiscipline=customfield_10706
MP_JIRA_FIELD_SO_ProjectName=customfield_10705
MP_JIRA_FIELD_CP_UserGroupName=customfield_10252
MP_JIRA_FIELD_CP_ProjectInformation=customfield_10253
MP_JIRA_FIELD_CP_INeedAVoucher=customfield_10716
MP_JIRA_FIELD_SELECT_VALUES_CP_INeedAVoucher_true=10705
MP_JIRA_FIELD_SELECT_VALUES_CP_INeedAVoucher_false=10706
MP_JIRA_FIELD_CP_VoucherID=customfield_10710
MP_JIRA_FIELD_CP_Platforms=customfield_10708
MP_JIRA_FIELD_SELECT_VALUES_SO_OfferType_normal=10906
MP_JIRA_FIELD_SELECT_VALUES_SO_OfferType_open_access=10907
MP_JIRA_FIELD_SELECT_VALUES_SO_OfferType_catalog=10908
MP_JIRA_FIELD_SO_OfferType=customfield_10906

# Update with appropriate values if ReCaptcha is used
RECAPTCHA_SITE_KEY=
RECAPTCHA_SECRET_KEY=

#optional sentry integrations
SENTRY_DSN=

PROFILE_4_ENABLED=true
EXTERNAL_LANDING_PAGE=false

MP_WHITELABEL=true
MP_ENABLE_EXTERNAL_SEARCH=false
ENABLE_COMMONS=false

RAILS_LOG_LEVEL=debug

MP_STOMP_CLIENT_NAME=NI4OSClient
MP_STOMP_LOGIN=user # update with ActiveMQ user, as declared in jetty-realm.properties, more details available on https://docs.sandbox.eosc-beyond.eu/Deployment%20Packages/Catalogue/Resource%20Catalogue/#activemq-classic
MP_STOMP_PASS=stomp_secret # update with ActiveMQ secret
MP_STOMP_HOST=activemq-classic
MP_STOMP_DESTINATION=registry
MP_STOMP_SSL=false

MP_IMPORT_EOSC_REGISTRY_URL=https://cat.example.eu/api # URL to an existing deployed of the EOSC Beyond Resource Catalogue, to support data fetching, details available on 
MP_IMPORT_TOKEN=my_token # access token for the EOSC Beyond Resource Catalogue. Can be imported by logging in to the Resource Catalogue, and extracting the AccessToken cookie.
FEDERATION_API_BASE_URL=https://msa.docker-fid.grid.cyf-kr.edu.pl/api/v1/services
```

The above `marketplace.env` file includes comments to help out with the modification of the appropriate environment variables. 

`MP_IMPORT_EOSC_REGISTRY_URL` and `MP_IMPORT_EOSC_REGISTRY_URL` are optional, and only needed if service sync needs to be enabled between the EOSC Beyond Resource Catalogue and the Marketplace. The necessary access token (`MP_IMPORT_TOKEN`) can be obtained manually by logging in to the Resource Catalogue, and then opening developer tools in the browser to extract the content of the `AccessToken` cookie.

Finally, before starting the containers the necessary directories backing the volumes need to be created and proper permissions assigned:

```bash
mkdir -p /opt/docker-data/marketplace/media
chown -R 100:101 /opt/docker-data/marketplace
```

## Verification  

Once the `docker-compose.yml` and `marketplace.env` files have been created, the container can be started with:

```
docker compose up -d
```

Logs can be viewed at any time using:

```
docker compose logs -f
```

The set URL should become accessible via a web browser, where the homepage of the Marketplace should be visible.

In case of any issues, the first troubleshooting step is to check whether any tables have been created in the PostgreSQL database (whether the database seed has succeeded):

```
docker exec -it postgresql /bin/bash
psql -U marketplace -d marketplace

# preview tables
\dt
```

### Data Fetching from Resource Catalogue

A data fetching job from the Resource Catalogue (assuming that `MP_IMPORT_EOSC_REGISTRY_URL` and `MP_IMPORT_TOKEN` have been set) can be initiated by:

```
docker exec -it marketplace-web /bin/bash
./bin/jms-subscriber 
rake import:all -v
```

In case the import is successful, the `services` table will get populated with the appropriate entries and they will also be visible from the Marketplace Web UI.

```
docker exec -it postgresql /bin/bash
psql -U marketplace -d marketplace

\x # enable extended display
select * from services limit 10;
```

### Promoting a User to an Admin

A user can be promoted to a Marketplace admin using these commands:

```
docker exec -it marketplace-web /bin/bash
rails c
u=User.find_by(email: "user@example.eu")
u.update(roles_mask: 7)
```

## Licensing  

The Marketplace software is distributed under the [GNU General Public License v3.0 software license](https://github.com/cyfronet-fid/whitelabel-marketplace/blob/development/LICENSE).

The licenses of the other supporting components are as follows:

- Apache ActiveMQ Classic - [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0)
- Elasticsearch - [Elastic License 2.0](https://www.elastic.co/licensing/elastic-license)
- PostgreSQL - [PostgreSQL License](https://opensource.org/license/postgresql)
- Redis - [RSALv2](https://redis.io/legal/rsalv2-agreement/) (before 8.0, [AGPLv3](https://www.gnu.org/licenses/agpl-3.0.en.html) for later versions)
- Traefik - [MIT License](https://github.com/traefik/traefik/blob/master/LICENSE.md)

[Opensearch](https://opensearch.org/), licensed under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) can be used as an alternative to Elasticsearch.

[Valkey](https://valkey.io/), licensed under [BSD-3-Clause](https://opensource.org/license/bsd-3-clause) can be used as an alternative to Redis.