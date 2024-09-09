<p align="center">
  <big><strong>Cardano GraphQL</strong></big>
</p>

<p align="center">
  <img width="200" src=".github/images/cardano-logo.png"/>
</p>

[![CI][img_src_CI]][workflow_CI]

<hr/>

## Overview

Cross-platform, _typed_, and **queryable** API for Cardano. The project contains multiple [packages] for composing 
GraphQL services to meet specific application demands, and a [docker compose stack] serving the included 
[cardano-graphql-server Dockerfile], the extended [hasura Dockerfile], [ogmios], [cardano-node]. The [schema] is defined in
native `.graphql`, and used to generate a [TypeScript package for client-side static typing]. A mutation is available to 
submit a signed and serialized transaction to the local node.
 
[Apollo Server] exposes the NodeJS execution engine over a HTTP endpoint, and includes support for open source metrics
via Prometheus, and implementing operation filtering to deny unexpected queries. Should you wish to have more control
over the server, or stitch the schema with an existing service, consider importing the executable schema from the 
`@cardano-graphql/api-*` packages only.

**GraphQL** is a query language and execution environment with server and client implementations across many programming
languages. The language can be serialized for network transmission, schema implementations hashed for assurance, and is
suited for describing most domains.
 
**TypeScript** (and JS) has the largest pool of production-ready libraries, developers, and interoperability in the
GraphQL and web ecosystem in general. TypeScript definitions for the schema, generated by [GraphQL Code Generator], are
[available on npm].

**[Ogmios]** is a protocol translation service written in Haskell running on top of cardano-node. It offers a JSON
interface through WebSockets and enables applications to speak Ouroboros' client mini-protocols via remote procedure
calls.

## Getting Started
Check the [releases] for the latest version.
``` console
git clone \
  --single-branch \
  --branch 8.2.2 \
  --recurse-submodules \
  https://github.com/cardano-foundation/cardano-graphql.git \
  && cd cardano-graphql
```

### Up
Choose **one** of the following:

#### A) Build and Run via Docker Compose
Boot the [docker compose stack] using a convention for container and volume scoping based on the network, as well as
optionally hitting the remote cache to speed up the build. The containers are detached, so you can terminate the log
console session freely. See [Docker Compose docs] to tailor for your use-case

##### Node snapshots
If you want to speed up the node syncing processes you can use a snapshot. The snapshot is a compressed file that contains the database of a node at a certain block.
You can download snapshots using Mithril, which are available for mainnet, preprod, and preview networks.
A detailed explanation can be found [here](https://mithril.network/doc/manual/getting-started/bootstrap-cardano-node)
<details open>
  <summary><i>mainnet</i></summary>
Get the most recent weekly snapshot link [here](https://update-cardano-mainnet.iohk.io/cardano-db-sync/index.html#13.5/), and set it as `RESTORE_SNAPSHOT` below, or omit if you wish to sync from genesis.

> **Disclaimer:** The Chainfollower environment variables are currently mandatory.
> Otherwise the Token registry will get stuck. 
> We will provide a fix as soon as possible.

``` console
DOCKER_BUILDKIT=1 \
COMPOSE_DOCKER_CLI_BUILD=1 \
CHAIN_FOLLOWER_START_SLOT=23068800 \
CHAIN_FOLLOWER_START_ID=a650a3f398ba4a9427ec8c293e9f7156d81fd2f7ca849014d8d2c1156c359b3a \
RESTORE_SNAPSHOT=https://update-cardano-mainnet.iohk.io/cardano-db-sync/13.5/db-sync-snapshot-schema-13.5-block-10781364-x86_64.tgz \
docker compose up -d --build &&\
docker compose logs -f
```
</details>

<details>
  <summary><i>preprod</i></summary>

``` console
DOCKER_BUILDKIT=1 \
COMPOSE_DOCKER_CLI_BUILD=1 \
NETWORK=preprod \
API_PORT=3101 \
HASURA_PORT=8091 \
OGMIOS_PORT=1338 \
POSTGRES_PORT=5433 \
docker compose -p preprod up -d --build &&\
docker compose -p preprod logs -f
```

</details>

<details>
  <summary><i>preview</i></summary>

``` console
DOCKER_BUILDKIT=1 \
COMPOSE_DOCKER_CLI_BUILD=1 \
NETWORK=preview \
API_PORT=3102 \
HASURA_PORT=8092 \
OGMIOS_PORT=1339 \
POSTGRES_PORT=5434 \
docker compose -p preview up -d --build &&\
docker compose -p preview logs -f
```

</details>

<details>
  <summary><i>sanchonet</i></summary>

``` console
DOCKER_BUILDKIT=1 \
COMPOSE_DOCKER_CLI_BUILD=1 \
NETWORK=sanchonet \
API_PORT=3102 \
HASURA_PORT=8092 \
OGMIOS_PORT=1339 \
POSTGRES_PORT=5434 \
docker compose -p preview up -d --build &&\
docker compose -p preview logs -f
```
</details>

#### B) Pull and Run via Docker Compose
Pull images from Docker Hub and run using a convention for container and volume scoping based on the network. The
containers are detached, so you can terminate the log console session freely. See [Docker Compose docs] to tailor for
your use-case.

<details open>
  <summary><i>mainnet</i></summary>

``` console
export NETWORK=mainnet &&\
docker compose up -d &&\
docker compose logs -f
```
</details>

<details>
  <summary><i>preprod</i></summary>

``` console
export NETWORK=preprod &&\
API_PORT=3101 \
HASURA_PORT=8091 \
OGMIOS_PORT=1338 \
POSTGRES_PORT=5433 \
docker compose -p ${NETWORK} up -d &&\
docker compose -p ${NETWORK} logs -f
```

</details>

<details>
  <summary><i>preview</i></summary>

``` console
export NETWORK=preview &&\
API_PORT=3102 \
HASURA_PORT=8092 \
OGMIOS_PORT=1339 \
POSTGRES_PORT=5434 \
docker compose -p ${NETWORK} up -d &&\
docker compose -p ${NETWORK} logs -f
```

</details>

### Down
The following commands will not remove volumes, however should you wish to do so, append `-v`

<details open>
  <summary><i>mainnet</i></summary>

``` console
docker compose down
```
</details>

<details>
  <summary><i>preprod</i></summary>

``` console
docker compose -p preprod down
```

</details>

<details>
  <summary><i>preview</i></summary>

``` console
docker compose -p preview down
```

</details>

### Use global Token Metadata Registry
The public Token metadata registry has a limit of daily requests, this can lead to long sync times, when resyncing from scratch.
If it's still needed to run with the global environment it's possible by removing the `token-metadata-registry` from `docker-compose.yml`.
And start it with for Mainnet:
```
METADATA_SERVER_URI="https://tokens.cardano.org" docker compose up -d
```
For other networks:
```
METADATA_SERVER_URI="https://metadata.world.dev.cardano.org"
```

### Check Cardano DB sync progress
Use the GraphQL Playground in the browser at http://localhost:3100/graphql:
> **_Note_** This Query is not available in early Era's of Cardano. Check Points of Interest here: [Link](https://ogmios.dev/mini-protocols/local-chain-sync/#points-of-interest) 
``` graphql 
{ cardanoDbMeta { initialized syncPercentage }}
```
or via command line:
``` console
curl \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"query": "{ cardanoDbMeta { initialized syncPercentage }}"}' \
  http://localhost:3100/graphql
```
:information_source: _Wait for `initialized` to be `true` to ensure the epoch dataset is complete. After the first sync
you may need to restart the services using `docker compose restart cardano-graphql` if the GraphQL server isn't
running._

### Query the full dataset
```graphql
{ cardano { tip { number slotNo epoch { number } } } }
```
``` console
curl \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"query": "{ cardano { tip { number slotNo epoch { number } } } }"}' http://localhost:3100/graphql
```
### :tada:
``` json
{ "data": { "cardano": { "tip": { "number": 4391749, "slotNo": 4393973, "epoch": { "number": 203 } } } } }
```

For more information, have a look at the [Wiki :book:].

## How to install (Linux / Docker)

### Docker

See [Using Docker].

### From Source 

See [Building].

## Documentation

| Link                                                                                               | Audience                                                     |
| ---                                                                                                | ---                                                          |
| [API Documentation]                                                                                | Users of the Cardano GraphQL API                             |
| [Wiki :book:]                                                                                      | Anyone interested in the project and our development process |
| [Example Queries - Cardano DB Hasura]        | Users of the Cardano DB Hasura API                             |

<hr/>

<p align="center">
  <a href="https://github.com/cardano-foundation/cardano-graphql/blob/master/LICENSE"><img src="https://img.shields.io/github/license/cardano-foundation/cardano-graphql.svg?style=for-the-badge" /></a>
</p>

[img_src_CI]: https://github.com/cardano-foundation/cardano-graphql/workflows/CI/badge.svg
[workflow_CI]: https://github.com/cardano-foundation/cardano-graphql/actions?query=workflow%3ACI
[packages]: ./packages
[docker compose stack]: ./docker-compose.yml
[Docker Compose docs]: https://docs.docker.com/compose/
[cardano-graphql-server Dockerfile]: ./Dockerfile
[hasura Dockerfile]: ./packages/api-cardano-db-hasura/hasura/Dockerfile
[ogmios]: https://ogmios.dev/getting-started/docker/
[cardano-node]: https://github.com/IntersectMBO/cardano-node
[schema]: ./packages/api-cardano-db-hasura/schema.graphql
[TypeScript package for client-side static typing]: ./packages/client-ts/README.md
[Apollo Server]: https://www.apollographql.com/docs/apollo-server/
[GraphQL Code Generator]: https://graphql-code-generator.com
[available on npm]: https://www.npmjs.com/package/cardano-graphql-ts
[Ogmios]: https://ogmios.dev/
[releases]: https://github.com/cardano-foundation/cardano-graphql/releases
[Wiki :book:]: https://github.com/cardano-foundation/cardano-graphql/wiki
[Using Docker]: https://github.com/cardano-foundation/cardano-graphql/wiki/Docker
[Building]: https://github.com/cardano-foundation/cardano-graphql/wiki/Building
[API Documentation]: https://cardano-foundation.github.io/cardano-graphql
[Example Queries - Cardano DB Hasura]: ./packages/api-cardano-db-hasura/src/example_queries
