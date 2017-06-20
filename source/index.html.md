---
title: WeiWatchers Documentation

<!-- language_tabs: -->
  <!-- - shell -->

toc_footers:
  - <a href='mailto:support@smartcontract.com'>Need Help?</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the WeiWatchers API documentation. WeiWatchers is a service which turns changes in Ethereum blockchains into push notifications for other applications. WeiWatchers can report on Ethereum event logs or account balance changes. Additionally, you can interact with your Ethereum node through Wei Watchers, sending in transactions and querying the network.

Wei Watchers can be configured to run on any Ethereum chain, public or private, as long as it supports the JSON-RPC interface.

If you want to view the source code or build the image locally, you can do so on [GitHub](https://github.com/oraclekit/wei_watchers).  

## Installation

### Prerequisites

- Docker installed
- a running instance of Postgres, 9.4 or above
- a running instance of Ethereum with the [JSON-RPC interface](https://github.com/ethereum/wiki/wiki/JSON-RPC) enabled

### Install

```shell
docker pull smartcontract/wei_watchers
docker run -it --env-file=.env smartcontract/smartoracle rake watcher:create
```

> Which base URL would you like notifications to be sent to?

```shell
http://localhost:3000/wei_watchers_base/
```

> Watcher Key:               RANDOM_KEY_1 ...

```shell
docker run -t --env-file=.env smartcontract/wei_watchers
```

First set up your configuration, by creating a `.env` file, that has at least the variables set in the [.env.example](https://github.com/oraclekit/wei_watchers/blob/master/.env.example).

Once you have set up your database and configuration, and you are ready to initialize WeiWatchers.

Run the watcher creation command and provide the URL where you would like push notifications to be sent.

Once a watcher is successfully initialized two sets of credentials will be printed out. The first credentials are used to create new event or account subscriptions on the watcher via API. The second is the HTTP basic authorization credentials that will be used by Wei Watchers when pushing updates out to applications.

<br/>

To have a managed instance spun up for you, contact us at [support@smartcontract.com](mailto:support@smartcontract.com).


## Authentication

Wei Watchers currently uses HTTP basic authorization for authentication. There are two different types of credentials used for interacting with Wei Watchers, Watcher Credentials are used to send updates to Wei Watchers, and Notification Credentials are used by Wei Watchers to send updates to applications. The two are intentionally kept distinct, but can be made to match for convenience sake.

Watchers and their credentials are currently created and managed via the command line only. Once created, Watcher information and settings are available via API accesss.

### Watcher Credentials

```shell
curl -u watcherKey:watcherSecret
  http://localhost:3174/status
```

Watcher credentials are used to authenticate with Wei Watchers and create new notification subscriptions. Watcher credentials are required and automatically generated for you upon creating a new Watcher. They can be updated, but never removed.

Watcher credentials are used to access all API endpoints and update settings. Wathcers are set up as a multitenant system; Watcher's cannot access or update information for other Watchers.

### Notification Credentials

Notification credentials are used by Wei Watchers to authenticate with the endpoints receiving the notifications. Notificaiton credentials are not required, although they are automatically generated upon creating a new Watcher. They can be updated or removed.

## Subscriptions

There are currently two types of notifications to subscribe to, [event logs](#event-subscriptions) and [account balances](#balance-subscriptions).

# Event Subscriptions

Creating event subscriptions allows your application to have events pushed to you instead of regularly querying. Subscriptions support all the parameters of [Ethereum log filters](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_newfilter).

Subscriptions are identified and updated via a UUID that is generated on creation.

## Creation
```shell
curl -u watcherKey:watcherSecret -X POST
  -d '{address: "0x00E89e4Ace8c83E6e345d6faA771E1E8c505F350", endAt: "1813520678"}'
  http://localhost:3174/api/event_subscriptions
```
> Responds:

```json
{
  "id": "ccb04d98-44ca-4d1b-aaf2-2546ce6b56d5"
}
```

Event subscriptions are created support all the parameters of [Ethereum log filters](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_newfilter) and additionally can set a subscription expiration. All parameters are optional, but something other than the `endAt` parameter must be specified.

Upon the successful creation of an event subscription the UUID of the subscription is returned, otherwise a list of errors is returned. 

Parameter | Type | Description
---- | ----- | --------
address | string | an Ethereum contract address
endAt | integer | a Unix timestamp specifying when the subscription expires
fromBlock | integer | block number of the earliest block to query from(defaults to the genesis block)
toBlock | integer | block number of the latest block to query up to(defaults to the latest block)
topics | array | array of topic IDs, to filter down to specific types of events at an address you would like updates on

## Event Notifications

```json
{
  "address": "0x00E89e4Ace8c83E6e345d6faA771E1E8c505F350",
  "blockHash": "0x4788f8c32b1bf4b3ae10b37d94bb19f0c14747b5d5d7331eef4397b184c14741",
  "blockNumber": 974359,
  "data": "4869204d6f6d21212121212131213121212e2e2e3f0000000000000000000000",
  "logIndex": 5,
  "subscription": "ccb04d98-44ca-4d1b-aaf2-2546ce6b56d5",
  "topics": ["0x79bb13962aba40064cd5dec819f7622f541dbcb8b79961132a1993fdd900edf8"],
  "transactionHash": "0x7cdce24fa32602e9fece7fd5a695736667eed8eb828306987498337f050ced76",
  "transactionIndex": 28
}
```

Parameter | Type | Description
---- | ----- | --------
address | string | Ethereum address at which the event was logged
blockHash | integer | identifier of the block in which the event was logged
blockNumber | string | height of the blockchain when the event was logged
data | string | data that was logged by the event, represented as hex
logIndex | integer | index of the event within the transaction that triggered it
subscripiton | integer | UUID of the subscription which triggered the notification
topics | array | list of event topics that the log emitted
transactionHash | string | identifier of the transaction which logged the event
transactionIndex | integer | index of the transaction within the block which logged the event
