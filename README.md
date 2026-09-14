# Keycloak IAM Lab

A local, reproducible lab environment for hands-on work with Identity and
Access Management. The stack runs Keycloak against PostgreSQL in Docker
Compose, with the full configuration kept in version control so the
environment can be rebuilt from scratch at any time.

This is a learning environment, not a production setup.

## Learning objectives

- OAuth 2.0 and OpenID Connect flows in practice, starting with Authorization Code Flow and PKCE
- Role, group and client scope modelling, and how it surfaces in access tokens
- Identity federation and single sign-on against an external identity provider
- Automated user and group provisioning through the SCIM API
- Fine-grained authorization and the boundary between what Keycloak covers and where identity governance begins
- Auditability, through persisted admin and user events

## Prerequisites

Docker Desktop or Docker Engine with the Compose plugin. No other runtime is
required, the Keycloak image ships its own Java runtime.

Keycloak's hostname is pinned to `keycloak` (see Design decisions below), so
that name also needs to resolve on the host machine:

```bash
echo "127.0.0.1 keycloak" | sudo tee -a /etc/hosts
```

## Quick start

```bash
cp .env.example .env      # set the passwords
docker compose config     # validate syntax and resolved variables
docker compose up -d
docker compose logs --tail 80 keycloak
```

| Endpoint | URL |
| --- | --- |
| Admin console | http://keycloak:8080 |
| Readiness probe | http://127.0.0.1:9000/health/ready |
| Metrics | http://127.0.0.1:9000/metrics |
| Grafana | http://localhost:3000 |

Stop with `docker compose stop`, remove the containers with
`docker compose down`. Note that `docker compose down -v` also removes the
volume and therefore all realms, users and events.

## Structure

```
.
├── docker-compose.yml    stack definition, single source of truth
├── .env.example          template for local variables, versioned
├── .env                  local values including secrets, not versioned
├── realms/               realm exports, imported on startup
└── docs/                 lab journal and notes
    └── adr/               architecture decision records
```

## Design decisions

**PostgreSQL instead of the embedded database.** The development mode default
keeps state inside the container, which is lost whenever the container is
recreated. An external database mirrors a realistic deployment and lets realm
configuration, users and audit events survive container rebuilds.

**Secrets in an untracked environment file.** Credentials live in `.env`, which
is excluded from version control. The committed `.env.example` documents which
variables exist without exposing values. Compose fails with an explicit message
if a required variable is missing, rather than starting with an empty password.

**Pinned image version.** The Keycloak version is pinned through a variable so
results stay comparable across sessions and upgrades become a deliberate step
rather than a side effect of a restart.

**Realm configuration as code.** Realms are exported to `realms/` and imported
on startup, which makes the authorization model reviewable, diffable and
reproducible on any machine. Full exports may contain client secrets, so files
holding secrets use the `.secret.json` suffix and are excluded from version
control.

**Management endpoints separated.** Health and metrics are exposed on the
management port rather than alongside the application, matching how the server
is intended to be operated behind a reverse proxy.

**Fixed hostname for Keycloak.** Any connected client needs the discovery
document, issuer and endpoint URLs Keycloak returns to resolve identically
whether the caller is the browser or another container. `KC_HOSTNAME` pins
that to the name `keycloak`, which is why the same name also has to resolve
to `127.0.0.1` on the host (see Prerequisites).

## Status

Stack running on Keycloak 26.7 with PostgreSQL 17 and the SCIM API preview
feature enabled. Realm `iam-lab` exists with a role and group model and a
confidential client (`demo-app`) using Authorization Code Flow with PKCE for
manual protocol testing. Grafana is the first connected application,
authenticating the same way and mapping Keycloak realm roles to its own
Admin, Editor and Viewer roles
(see [ADR 0001](docs/adr/0001-grafana-als-erste-client-anwendung.md)).

Next step is exporting the realm to `realms/` as versioned configuration.

Progress notes are kept in [docs/lab-journal.md](docs/lab-journal.md).
Architecture decisions are documented as ADRs in [docs/adr](docs/adr).