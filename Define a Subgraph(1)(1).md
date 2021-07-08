# 定义子图

子图定义了 The Graph 将从以太坊索引哪些数据，以及它将如何存储这些数据。一旦部署，它将成为区块链数据全局图的一部分。

​           

![子图图形](https://thegraph.com/docs/images/define-subgraph.png)

​           

子图定义由几个文件组成：

- `subgraph.yaml`：包含*子图清单*的 YAML 文件**
- `schema.graphql`: 一个 GraphQL Schema，它定义了为你的子图存储了什么数据，以及如何通过GraphQL查询它（数据）
- `AssemblyScript Mappings`：从以太坊中的事件数据转换为Schema中定义的实体的[AssemblyScript](https://github.com/AssemblyScript/assemblyscript)代码（例如在本教程中的`mapping.ts`）

在详细了解清单文件的内容之前，您需要安装构建和部署子图所需的[Graph CLI](https://github.com/graphprotocol/graph-cli)。

## 安装The Graph CLI

Graph CLI是用JavaScript编写的，您需要安装 [yarn](https://yarnpkg.com/lang/en/docs/install/#centos-stable)或 [npm](https://www.npmjs.com/get-npm)才能使用它；以下内容假设您已经安装`yarn`。安装的详细说明`yarn` 可以在 [graph-cli repo中找到](https://github.com/graphprotocol/graph-cli/wiki/Installation-on-different-operating-systems)

一旦你完成安装`yarn`，通过运行安装 Graph CLI

```
yarn global add @graphprotocol/graph-cli
```

## 创建子图项目

该`graph init`命令可用于从任何公共以太坊网络上的现有合约或[示例子图](https://github.com/graphprotocol/example-subgraph)设置新的子图项目 。

### 从现有合约

If you already have a smart contract deployed to Ethereum mainnet or one of the testnets, bootstrapping a new subgraph from this contract can be a good way to get started.

The following command creates a subgraph that indexes all events of an existing contract. It attempts to fetch the contract ABI from Etherscan and falls back to requesting a local file path. If any of the optional arguments are missing, it takes you through an interactive form.

```
graph init \
  --from-contract <CONTRACT_ADDRESS> \
  [--network <ETHEREUM_NETWORK>] \
  [--abi <FILE>] \
  <GITHUB_USER>/<SUBGRAPH_NAME> [<DIRECTORY>]
```

The `<GITHUB_USER>` is your github user or organization name, `<SUBGRAPH_NAME>` is the name for your subgraph, and `<DIRECTORY>` is the optional name of the directory where `graph init` will put the example subgraph manifest.

The `<CONTRACT_ADDRESS>` is the address of your existing contract. `<ETHEREUM_NETWORK>` is the name of the Ethereum network that the contract lives on. `<FILE>` is a local path to a contract ABI file. Both `--network` and `--abi` are optional.

Supported networks on the Hosted Service are:

1. Ethereum Mainnet -> `mainnet`
2. Kovan -> `kovan`
3. Rinkeby -> `rinkeby`
4. Ropsten -> `ropsten`
5. Goerli -> `goerli`
6. POA-Core -> `poa-core`
7. POA-Sokol -> `poa-sokol`
8. xDAI -> `xdai`
9. Polygon/Matic -> `matic`
10. Mumbai (Matic’s testnet) -> `mumbai`
11. Fantom -> `fantom`
12. Binance Smart Chain -> `bsc`
13. Binance Smart Chain Testnet -> `chapel`
14. Clover -> `clover`
15. Avalanche -> `avalanche`
16. Avalanche's testnet -> `fuji`
17. Celo -> `celo`
18. Celo's testnet (Alfajores) -> `celo-alfajores`
19. Fuse -> `fuse`
20. Moonbeam -> `mbase`
21. Arbitrum Testnet -> `arbitrum-rinkeby`
22. Arbitrum One -> `arbitrum-one`

### 从示例子图中

The second mode `graph init` supports is creating a new project from an [example subgraph](https://github.com/graphprotocol/example-subgraph). The following command does this:

```
graph init --from-example <GITHUB_USER>/<SUBGRAPH_NAME> [<DIRECTORY>]
```

The example subgraph is based on the [Gravity contract](https://github.com/graphprotocol/example-subgraph/blob/master/contracts/Gravity.sol) by [Dani Grant](https://github.com/danigrant) that manages user avatars and emits `NewGravatar` or `UpdateGravatar` events whenever avatars are created or updated. The subgraph handles these events by writing `Gravatar` entities to the Graph Node store and ensuring these are updated according to the events. The following sections will go over the files that make up the subgraph manifest for this example.

## 子图清单

The subgraph manifest `subgraph.yaml` defines the smart contracts your subgraph indexes, which events from these contracts to pay attention to, and how to map event data to entities that Graph Node stores and allows to query. The full specification for subgraph manifests can be found [here](https://github.com/graphprotocol/graph-node/blob/master/docs/subgraph-manifest.md).

For the example subgraph, `subgraph.yaml` is:

```
specVersion: 0.0.1
description: Gravatar for Ethereum
repository: https://github.com/graphprotocol/example-subgraph
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum/contract
    name: Gravity
    network: mainnet
    source:
      address: '0x2E645469f354BB4F5c8a05B3b30A929361cf77eC'
      abi: Gravity
      startBlock: 6175244
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.1
      language: wasm/assemblyscript
      entities:
        - Gravatar
      abis:
        - name: Gravity
          file: ./abis/Gravity.json
      eventHandlers:
        - event: NewGravatar(uint256,address,string,string)
          handler: handleNewGravatar
        - event: UpdatedGravatar(uint256,address,string,string)
          handler: handleUpdatedGravatar
      callHandlers:
        - function: createGravatar(string,string)
          handler: handleCreateGravatar
      blockHandlers:
        - function: handleBlock
        - function: handleBlockWithCall
          filter:
            kind: call
      file: ./src/mapping.ts
```

The important entries to update for the manifest are:

- `description`: a human-readable description of what the subgraph is. This description is displayed by the Graph Explorer when the subgraph is deployed to the [Hosted Service](https://thegraph.com/explorer/).
- `repository`: the URL of the repository where the subgraph manifest can be found. This is also displayed by the Graph Explorer.
- `dataSources.source`: the `address` of the smart contract the subgraph sources, and the `abi` of the smart contract to use. The `address` is optional; omitting it allows to index matching events from all contracts.
- `dataSources.source.startBlock`: the optional number of the block that the data source starts indexing from. In most cases we suggest using the block in which the contract was created.
- `dataSources.mapping.entities`: the entities that the data source writes to the store. The schema for each entity is defined in the the `schema.graphql` file.
- `dataSources.mapping.abis`: one or more named ABI files for the source contract as well as any other smart contracts that you interact with from within the mappings.
- `dataSources.mapping.eventHandlers`: lists the smart contract events this subgraph reacts to and the handlers in the mapping—`./src/mapping.ts` in the example—that transform these events into entities in the store.
- `dataSources.mapping.callHandlers`: lists the smart contract functions this subgraph reacts to and handlers in the mapping that transform the inputs and outputs to function calls into entities in the store.
- `dataSources.mapping.blockHandlers`: lists the blocks this subgraph reacts to and handlers in the mapping to run when a block is appended to the chain. Without a filter, the block handler will be run every block. An optional filter can be provided with the following kinds: `call`. A `call` filter will run the handler if the block contains at least one call to the data source contract.

A single subgraph can index data from multiple smart contracts. Add an entry for each contract from which data needs to be indexed to the `dataSources` array.

The triggers for a data source within a block are ordered using the following process:

1. Event and call triggers are first ordered by transaction index within the block.
2. Event and call triggers with in the same transaction are ordered using a convention: event triggers first then call triggers, each type respecting the order they are defined in the manifest.
3. Block triggers are run after event and call triggers, in the order they are defined in the manifest.

These ordering rules are subject to change.

### 获取 ABI

The ABI file(s) must match your contract(s). There are a few ways to obtain ABI files:

- If you are building your own project, you will likely have access to your most current ABIs.
- If you are building a subgraph for a public project, you can download that project to your computer and get the ABI by using [`truffle compile`](https://truffleframework.com/docs/truffle/overview) or using `solc` to compile.
- You can also find the ABI on [Etherscan](https://etherscan.io/), but this isn't always reliable, as the ABI that is uploaded there may be out of date. Make sure you have the right ABI, otherwise running your subgraph will fail.

## GraphQL 架构

The schema for your subgraph is in the file `schema.graphql`. GraphQL schemas are defined using the GraphQL interface definition language. If you've never written a GraphQL schema, it is recommended that you check out this [primer](https://graphql.org/learn/schema/#type-language) on the GraphQL type system. Reference documentation for GraphQL schemas can be found in the [GraphQL API section](https://thegraph.com/docs/graphql-api).

## 定义实体

Before defining entities, it is important to take a step back and think about how your data is structured and linked. All queries will be made against the data model defined in the subgraph schema and the entities indexed by the subgraph. Because of this, it is good to define the subgraph schema in a way that matches the needs of your dApp. It may be useful to imagine entities as "objects containing data", rather than as events or functions.

With The Graph, you simply define entity types in `schema.graphql`, and Graph Node will generate top level fields for querying single instances and collections of that entity type. Each type that should be an entity is required to be annotated with an `@entity` directive.

### 好例子

The `Gravatar` entity below is structured around a Gravatar object and is a good example of how an entity could be defined.

```
type Gravatar @entity {
  id: ID!
  owner: Bytes
  displayName: String
  imageUrl: String
  accepted: Boolean
}
```

### 坏例子

The example `GravatarAccepted` and `GravatarDeclined` entities below are based around events. It is not recommended to map events or function calls to entities 1:1.

```
type GravatarAccepted @entity {
  id: ID!
  owner: Bytes
  displayName: String
  imageUrl: String
}

type GravatarDeclined @entity {
  id: ID!
  owner: Bytes
  displayName: String
  imageUrl: String
}
```

### 可选和必填字段

Entity fields can be defined as required or optional. Required fields are indicated by the `!` in the schema. If a required field is not set in the mapping, you will receive this error when querying the field:

```
Null value resolved for non-null field 'name'
```

Each entity must have an `id` field, which is of type `ID!` (string). The `id` field serves as the primary key, and needs to be unique among all entities of the same type.

### 内置标量类型

#### GraphQL 支持的标量

We support the following scalars in our GraphQL API:

| Type         | Description                              |
| ------------ | ---------------------------------------- |
| `Bytes`      | Byte array, represented as a hexadecimal string. Commonly used for Ethereum hashes and addresses. |
| `ID`         | Stored as a `string`.                    |
| `String`     | Scalar for `string` values. Null characters are not supported and are automatically removed. |
| `Boolean`    | Scalar for `boolean` values.             |
| `Int`        | The GraphQL spec defines `Int` to have size of 32 bytes. |
| `BigInt`     | Large integers. Used for Ethereum's `uint32`, `int64`, `uint64`, ..., `uint256` types. Note: Everything below `uint32`, such as `int32`, `uint24` or `int8` is represented as `i32`. |
| `BigDecimal` | `BigDecimal` High precision decimals represented as a signficand and an exponent. The exponent range is from −6143 to +6144. Rounded to 34 significant digits. |

#### 枚举

You can also create enums within a schema. Enums have the following syntax:

```
enum TokenStatus {
  OriginalOwner
  SecondOwner
  ThirdOwner
}
```

Once the enum is defined in the schema, you can use the string representation of the enum value to set an enum field on an entity. For example, you can set the `tokenStatus` to `SecondOwner` by first defining your entity and subsequently setting the field with `entity.tokenStatus = "SecondOwner`. The example below demonstrates what the `Token` entity would look like with an enum field:

```
type Token @entity {
  id: ID!
  name: String!
  symbol: String!
  decimals: Int!
  tokenStatus: TokenStatus!
}
```

More detail on writing enums can be found in the [GraphQL documentation](https://graphql.org/learn/schema/).

### 实体关系

An entity may have a relationship to one or more other entities in your schema. These relationships may be traversed in your queries. Relationships in The Graph are unidirectional. It is possible to simulate bidirectional relationships by defining a unidirectional relationship on either "end" of the relationship.

Relationships are defined on entities just like any other field except that the type specified is that of another entity.

#### 一对一关系

Define a `Transaction` entity type with an optional one-to-one relationship with a `TransactionReceipt` entity type:

```
type Transaction @entity {
  id: ID!
  transactionReceipt: TransactionReceipt
}

type TransactionReceipt @entity {
  id: ID!
  transaction: Transaction
}
```

#### 一对多关系

Define a `TokenBalance` entity type with a required one-to-many relationship with a `Token` entity type:

```
type Token @entity {
  id: ID!
}

type TokenBalance @entity {
  id: ID!
  amount: Int!
  token: Token!
}
```

### 反向查找

Reverse lookups can be defined on an entity through the `@derivedFrom` field. This creates a virtual field on the entity that may be queried but cannot be set manually through the mappings API. Rather, it is derived from the relationship defined on the other entity. For such relationships, it rarely makes sense to store both sides of the relationship, and both indexing and query performance will be better when only one side is stored and the other is derived.

For one-to-many relationships, the relationship should *always* be stored on the 'one' side, and the 'many' side should always be derived. Storing the relationship this way, rather than storing an array of entities on the 'many' side, will result in dramatically better performance for both indexing and querying the subgraph. In general, storing arrays of entities should be avoided as much as is practical.

#### 例子

We can make the balances for a token accessible from the token by deriving a `tokenBalances` field:

```
type Token @entity {
  id: ID!
  tokenBalances: [TokenBalance!]! @derivedFrom(field: "token")
}

type TokenBalance @entity {
  id: ID!
  amount: Int!
  token: Token!
}
```

#### 多对多关系

For many-to-many relationships, such as users that each may belong to any number of organizations, the most straightforward, but generally not the most performant, way to model the relationship is as an array in each of the two entities involved. If the relationship is symmetric, only one side of the relationship needs to be stored and the other side can be derived.

#### 例子

Define a reverse lookup from a `User` entity type to an `Organization` entity type. In the example below, this is achieved by looking up the `members` attribute from within the `Organization` entity. In queries, the `organizations` field on `User` will be resolved by finding all `Organization` entities that include the user's ID.

```
type Organization @entity {
  id: ID!
  name: String!
  members: [User!]!
}

type User @entity {
  id: ID!
  name: String!
  organizations: [Organization!]! @derivedFrom(field: "members")
}
```

A more performant way to store this relationship is through a mapping table that has one entry for each `User`/`Organization` pair with a schema like

```
type Organization @entity {
  id: ID!
  name: String!
  members: [UserOrganization]! @derivedFrom(field: "organization")
}

type User @entity {
  id: ID!
  name: String!
  organizations: [UserOrganization!] @derivedFrom(field: "user")
}

type UserOrganization @entity {
  id: ID!   # Set to `${user.id}-${organization.id}`
  user: User!
  organization: Organization!
}
```

This approach requires that queries descend into one additional level to retrieve, for example, the organizations for users:

```
query usersWithOrganizations {
  users {
    organizations { # this is a UserOrganization entity
      organization {
        name
      }
    }
  }
}
```

This more elaborate way of storing many-to-many relationships will result in less data stored for the subgraph, and therefore to a subgraph that is often dramatically faster to index and to query.

### 向模式添加注释

As per GraphQL spec, comments can be added above schema entity attributes using double quotations `""`. This is illustrated in the example below:

```
  type MyFirstEntity @entity {
    "unique identifier and primary key of the entity"
    id: ID!
    address: Bytes!
  }
```

## 定义全文搜索字段

Fulltext search queries filter and rank entities based on a text search input. Fulltext queries are able to return matches for similar words by processing the query text input into stems before comparing to the indexed text data.

A fulltext query definition includes the query name, the language dictionary used to process the text fields, the ranking algorithm used to order the results, and the fields included in the search. Each fulltext query may span multiple fields, but all included fields must be from a single entity type.

To add a fulltext query, include a `_Schema_` type with a fulltext directive in the GraphQL schema.

```
type _Schema_
  @fulltext(
    name: "bandSearch",
    language: en
    algorithm: rank,
    include: [
      {
        entity: "Band",
        fields: [
          { name: "name" },
          { name: "description" },
          { name: "bio" },
        ]
      }
    ]
  )

type Band @entity {
    id: ID!
    name: String!
    description: String!
    bio: String
    wallet: Address
    labels: [Label!]!
    discography: [Album!]!
    members: [Musician!]!
}
```

The example `bandSearch` field can be used in queries to filter `Band` entities based on the text documents in the `name`, `description`, and `bio` fields. Jump to [GraphQL API - Queries](https://thegraph.com/docs/graphql-api#queries) for a description of the Fulltext search API and for more example usage.

```
query {
  bandSearch(text: "breaks & electro & detroit") {
    id
    name
    description
    wallet
  }
}
```

### 支持的语言

Choosing a different language will have a definitive, though sometimes subtle, effect on the fulltext search API. Fields covered by a fulltext query field are examined in the context of the chosen language, so the lexemes produced by analysis and search queries vary language to language. For example: when using the supported Turkish dictionary "token" is stemmed to "toke" while, of course, the English dictionary will stem it to "token".

Supported language dictionaries:

| Code   | Dictionary |
| ------ | ---------- |
| simple | General    |
| da     | Danish     |
| nl     | Dutch      |
| en     | English    |
| fi     | Finnish    |
| fr     | French     |
| de     | German     |
| hu     | Hungarian  |
| it     | Italian    |
| no     | Norwegian  |
| pt     | Portugese  |
| ro     | Romanian   |
| ru     | Russian    |
| es     | Spanish    |
| sv     | Swedish    |
| tr     | Turkish    |

### 排名算法

Supported algorithms for ordering results:

| Algorithm     | Description                              |
| ------------- | ---------------------------------------- |
| rank          | Use the match quality (0-1) of the fulltext query to order the results. |
| proximityRank | Similar to `rank` but also includes the proximity of the matches. |

## 写映射

The mappings transform the Ethereum data your mappings are sourcing into entities defined in your schema. Mappings are written in a subset of [TypeScript](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) called [AssemblyScript](https://github.com/AssemblyScript/assemblyscript/wiki) which can be compiled to WASM ([WebAssembly](https://webassembly.org/)). AssemblyScript is stricter than normal TypeScript, yet provides a familiar syntax.

For each event handler that is defined in `subgraph.yaml` under `mapping.eventHandlers`, create an exported function of the same name. Each handler must accept a single parameter called `event` with a type corresponding to the name of the event which is being handled.

In the example subgraph, `src/mapping.ts` contains handlers for the `NewGravatar` and `UpdatedGravatar` events:

```
import { NewGravatar, UpdatedGravatar } from '../generated/Gravity/Gravity'
import { Gravatar } from '../generated/schema'

export function handleNewGravatar(event: NewGravatar): void {
  let gravatar = new Gravatar(event.params.id.toHex())
  gravatar.owner = event.params.owner
  gravatar.displayName = event.params.displayName
  gravatar.imageUrl = event.params.imageUrl
  gravatar.save()
}

export function handleUpdatedGravatar(event: UpdatedGravatar): void {
  let id = event.params.id.toHex()
  let gravatar = Gravatar.load(id)
  if (gravatar == null) {
    gravatar = new Gravatar(id)
  }
  gravatar.owner = event.params.owner
  gravatar.displayName = event.params.displayName
  gravatar.imageUrl = event.params.imageUrl
  gravatar.save()
}
```

The first handler takes a `NewGravatar` event and creates a new `Gravatar` entity with `new Gravatar(event.params.id.toHex())`, populating the entity fields using the corresponding event parameters. This entity instance is represented by the variable `gravatar`, with an `id` value of `event.params.id.toHex()`.

The second handler tries to load the existing `Gravatar` from the Graph Node store. If it does not exist yet, it is created on demand. The entity is then updated to match the new event parameters, before it is saved back to the store using `gravatar.save()`.

### 创建新实体的推荐 ID

Every entity has to have an `id` that is unique among all entities of the same type. An entity's `id` value is set when the entity is created. Below are some recommended `id` values to consider when creating new entities. NOTE: The value of `id` must be a `string`.

- `event.params.id.toHex()`
- `event.transaction.from.toHex()`
- `event.transaction.hash.toHex() + "-" + event.logIndex.toString()`

We provide the [Graph Typescript Library](https://github.com/graphprotocol/graph-ts) which contains utilies for interacting with the Graph Node store and conveniences for handling smart contract data and entities. You can use this library in your mappings by importing `@graphprotocol/graph-ts` in `mapping.ts`.

## 代码生成

In order to make working smart contracts, events and entities easy and type-safe, the Graph CLI can generate AssemblyScript types from the subgraph's GraphQL schema and the contract ABIs included in the data sources.

This is done with

```
graph codegen [--output-dir <OUTPUT_DIR>] [<MANIFEST>]
```

but in most cases, subgraphs are already preconfigured via `package.json` to allow you to simply run one of the following to achieve the same:

```
# Yarn
yarn codegen

# NPM
npm run codegen
```

This will generate an AssemblyScript class for every smart contract in the ABI files mentioned in `subgraph.yaml`, allowing you to bind these contracts to specific addresses in the mappings and call read-only contract methods against the block being processed. It will also generate a class for every contract event to provide easy access to event parameters as well as the block and transaction the event originated from. All of these types are written to `<OUTPUT_DIR>/<DATA_SOURCE_NAME>/<ABI_NAME>.ts`. In the example subgraph, this would be `generated/Gravity/Gravity.ts`, allowing mappings to import these types with

```
import {
  // The contract class:
  Gravity,
  // The events classes:
  NewGravatar,
  UpdatedGravatar,
} from '../generated/Gravity/Gravity'
```

In addition to this, one class is generated for each entity type in the subgraph's GraphQL schema. These classes provide type-safe entity loading, read and write access to entity fields as well as a `save()` method to write entities to store. All entity classes are written to `<OUTPUT_DIR>/schema.ts`, allowing mappings to import them with

```
import { Gravatar } from '../generated/schema'
```

> **Note:** The code generation must be performed again after every change to the GraphQL schema or the ABIs included in the manifest. It must also be performed at least once before building or deploying the subgraph.

Code generation does not check your mapping code in `src/mapping.ts`. If you want to check that before trying to deploy your subgraph to the Graph Explorer, you can run `yarn build` and fix any syntax errors that the TypeScript compiler might find.

If the read-only methods of your contract may revert, then you should handle that by calling the generated contract method prefixed with `try_`. For example, the Gravity contract exposes the `gravatarToOwner` method. This code would be able to handle a revert in that method:

```
  let gravity = Gravity.bind(event.address)
  let callResult = gravity.try_gravatarToOwner(gravatar)
  if (callResult.reverted) {
    log.info("getGravatar reverted", [])
  } else {
    let owner = callResult.value
  }
```

Note that a Graph node connected to a Geth or Infura client may not detect all reverts, if you rely on this we recommend using a Graph node connected to a Parity client.

In the next section, we will describe how to deploy your subgraph using the Graph Explorer.

## 数据源模板

A common pattern in Ethereum smart contracts is the use of registry or factory contracts, where one contract creates, manages or references an arbitrary number of other contracts that each have their own state and events. The addresses of these sub-contracts may or may not be known upfront and many of these contracts may be created and/or added over time. This is why, in such cases, defining a single data source or a fixed number of data sources is impossible and a more dynamic approach is needed: *data source templates*.

### 主合约数据源

First, you define a regular data source for the main contract. The snippet below shows a simplified example data source for the [Uniswap](https://uniswap.io/) exchange factory contract. Note the `NewExchange(address,address)` event handler. This is emitted when a new exchange contract is created on chain by the factory contract.

```
dataSources:
  - kind: ethereum/contract
    name: Factory
    network: mainnet
    source:
      address: '0xc0a47dFe034B400B47bDaD5FecDa2621de6c4d95'
      abi: Factory
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.2
      language: wasm/assemblyscript
      file: ./src/mappings/factory.ts
      entities:
        - Directory
      abis:
        - name: Factory
          file: ./abis/factory.json
      eventHandlers:
        - event: NewExchange(address,address)
          handler: handleNewExchange
```

### 动态创建合约的数据源模板

Then, you add *data source templates* to the manifest. These are identical to regular data sources, except that they lack a predefined contract address under `source`. Typically, you would define one template for each type of sub-contract managed or referenced by the parent contract.

```
dataSources:
  - kind: ethereum/contract
    name: Factory
    # ... other source fields for the main contract ...
templates:
  - name: Exchange
    kind: ethereum/contract
    network: mainnet
    source:
      abi: Exchange
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.1
      language: wasm/assemblyscript
      file: ./src/mappings/exchange.ts
      entities:
        - Exchange
      abis:
        - name: Exchange
          file: ./abis/exchange.json
      eventHandlers:
        - event: TokenPurchase(address,uint256,uint256)
          handler: handleTokenPurchase
        - event: EthPurchase(address,uint256,uint256)
          handler: handleEthPurchase
        - event: AddLiquidity(address,uint256,uint256)
          handler: handleAddLiquidity
        - event: RemoveLiquidity(address,uint256,uint256)
          handler: handleRemoveLiquidity
```

### 实例化数据源模板

In the final step, you update your main contract mapping to create a dynamic data source instance from one of the templates. In this example, you would change the main contract mapping to import the `Exchange` template and call the `Exchange.create(address)` method on it to start indexing the new exchange contract.

```
import { Exchange } from '../generated/templates'

export function handleNewExchange(event: NewExchange): void {
  // Start indexing the exchange; `event.params.exchange` is the
  // address of the new exchange contract
  Exchange.create(event.params.exchange)
}
```

> **Note:** A new data source will only process the calls and events for the block in which it was created and all following blocks, but will not process historical data, i.e., data that is contained in prior blocks.
>
> If prior blocks contain data relevant to the new data source, it is best to index that data by reading the current state of the contract and creating entities representing that state at the time the new data source is created.

### 数据源上下文

Data source contexts allow passing extra configuration when instantiating a template. In our example, let's say exchanges are associated with a particular trading pair, which is included in the `NewExchange` event. That information can be passed into the instantiated data source, like so:

```
import { Exchange } from '../generated/templates'

export function handleNewExchange(event: NewExchange): void {
  let context = new DataSourceContext()
  context.setString("tradingPair", event.params.tradingPair)
  Exchange.createWithContext(event.params.exchange, context)
}
```

Inside a mapping of the `Exchange` template, the context can then be accessed:

```
import { dataSource } from "@graphprotocol/graph-ts"

let context = dataSource.context()
let tradingPair = context.getString("tradingPair")
```

There are setters and getters like `setString` and `getString` for all value types.

## 起始块

The `startBlock` is an optional setting that allows you to define from which block in the chain the data source will start indexing. Setting the start block allows the data source to skip potentially millions of blocks that are irrelevant. Typically, a subgraph developer will set `startBlock` to the block in which the smart contract of the data source was created.

```
dataSources:
 - kind: ethereum/contract
   name: ExampleSource
   network: mainnet
   source:
     address: '0xc0a47dFe034B400B47bDaD5FecDa2621de6c4d95'
     abi: ExampleContract
     startBlock: 6627917
   mapping:
     kind: ethereum/events
     apiVersion: 0.0.2
     language: wasm/assemblyscript
     file: ./src/mappings/factory.ts
     entities:
       - User
     abis:
       - name: ExampleContract
         file: ./abis/ExampleContract.json
     eventHandlers:
       - event: NewEvent(address,address)
         handler: handleNewEvent
```

> **Note:** The contract creation block can be quickly looked up on Etherscan: 1. Search for the contract by entering its address in the search bar. 2. Click on the creation transaction hash in the `Contract Creator` section. 3. Load the transaction details page where you'll find the start block for that contract.

## 呼叫处理程序

While events provide an effective way to collect relevant changes to the state of a contract, many contracts avoid generating logs to optimize gas costs. In these cases, a subgraph can subscribe to calls made to the data source contract. This is achieved by defining call handlers referencing the function signature and the mapping handler that will process calls to this function. To process these calls, the mapping handler will receive an `ethereum.Call` as an argument with the typed inputs to and outputs from the call. Calls made at any depth in a transaction's call chain will trigger the mapping, allowing activity with the data source contract through proxy contracts to be captured.

Call handlers will only trigger in one of two cases: when the function specified is called by an account other than the contract itself or when it is marked as external in Solidity and called as part of another function in the same contract.

> **Note:** Call handlers are not supported on Rinkeby and Ganache. Call handlers depend on the Parity tracing API and neither Rinkeby nor Ganache support this (Rinkeby is Geth-only).

### 定义调用处理程序

To define a call handler in your manifest simply add a `callHandlers` array under the data source you would like to subscribe to.

```
dataSources:
  - kind: ethereum/contract
    name: Gravity
    network: mainnet
    source:
      address: '0x731a10897d267e19b34503ad902d0a29173ba4b1'
      abi: Gravity
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.2
      language: wasm/assemblyscript
      entities:
        - Gravatar
        - Transaction
      abis:
        - name: Gravity
          file: ./abis/Gravity.json
      callHandlers:
        - function: createGravatar(string,string)
          handler: handleCreateGravatar
```

The `function` is the normalized function signature to filter calls by. The `handler` property is the name of the function in your mapping you would like to execute when the target function is called in the data source contract.

### 映射函数

Each call handler takes a single parameter that has a type corresponding to the name of the called function. In the example subgraph above, the mapping contains a handler for when the `createGravatar` function is called and receives a `CreateGravatarCall` parameter as an argument:

```
import { CreateGravatarCall } from '../generated/Gravity/Gravity'
import { Transaction } from '../generated/schema'

export function handleCreateGravatar(call: CreateGravatarCall): void {
  let id = call.transaction.hash.toHex()
  let transaction = new Transaction(id)
  transaction.displayName = call.inputs._displayName
  transaction.imageUrl = call.inputs._imageUrl
  transaction.save()
}
```

The `handleCreateGravatar` function takes a new `CreateGravatarCall` which is a subclass of `ethereum.Call`, provided by `@graphprotocol/graph-ts`, that includes the typed inputs and outputs of the call. The `CreateGravatarCall` type is generated for you when you run `graph codegen`.

## 块处理程序

In addition to subscribing to contract events or function calls, a subgraph may want to update its data as new blocks are appended to the chain. To achieve this a subgraph can run a function after every block or after blocks that match a predefined filter.

### 支持的过滤器

```
filter:
  kind: call
```

*定义的处理程序将针对每个块调用一次，该块包含对定义处理程序的合同（数据源）的调用。*

块处理程序没有过滤器将确保处理程序被每个块调用。对于每种过滤器类型，数据源只能包含一个块处理程序。

```
dataSources:
  - kind: ethereum/contract
    name: Gravity
    network: dev
    source:
      address: '0x731a10897d267e19b34503ad902d0a29173ba4b1'
      abi: Gravity
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.2
      language: wasm/assemblyscript
      entities:
        - Gravatar
        - Transaction
      abis:
        - name: Gravity
          file: ./abis/Gravity.json
      blockHandlers:
        - handler: handleBlock
        - handler: handleBlockWithCallToContract
          filter:
            kind: call
```

### 映射函数

映射函数将接收一个`ethereum.Block`作为其唯一参数。与事件的映射函数一样，该函数可以访问商店中现有的子图实体，调用智能合约并创建或更新实体。

```
import { ethereum } from '@graphprotocol/graph-ts'

export function handleBlock(block: ethereum.Block): void {
  let id = block.hash.toHex()
  let entity = new Block(id)
  entity.save()
}
```

## 匿名事件

如果您需要在 Solidity 中处理匿名事件，可以通过提供事件的主题 0 来实现，如示例所示：

```
eventHandlers:
  - event: LogNote(bytes4,address,bytes32,bytes32,uint256,bytes)
    topic0: "0xbaa8529c00000000000000000000000000000000000000000000000000000000"
    handler: handleGive
```

只有在签名和主题 0 都匹配时才会触发事件。默认情况下，`topic0`等于事件签名的哈希值。

## IPFS 固定

将 IPFS 与以太坊结合的一个常见用例是将数据存储在 IPFS 上，这些数据在链上维护成本太高，并在以太坊合约中引用 IPFS 哈希。

给定这样的 IPFS 哈希，子图可以使用`ipfs.cat`和从 IPFS 读取相应的文件`ipfs.map`。然而，为了可靠地做到这一点，需要将这些文件固定在 IPFS 节点上，图节点索引子图连接到该节点。在[托管服务](https://thegraph.com/explorer/)的情况下，这是 <https://api.thegraph.com/ipfs/>。

为了让子图开发人员更容易做到这一点，Graph 团队编写了一个工具，用于将文件从一个 IPFS 节点传输到另一个节点，称为 [ipfs-sync](https://github.com/graphprotocol/ipfs-sync)。

## 特点：非致命错误

默认情况下，已同步子图上的索引错误将导致子图失败并停止同步。通过忽略引发错误的处理程序所做的更改，子图也可以配置为在出现错误时继续同步。这使子图作者有时间更正他们的子图，同时继续针对最新块提供查询，尽管由于导致错误的错误，结果可能会不一致。请注意，某些错误仍然总是致命的，要成为非致命错误，必须知道错误是确定性的。

启用非致命错误需要在子图清单上设置以下功能标志：

```
features:
  - nonFatalErrors
```

查询还必须选择通过`subgraphError`参数查询具有潜在不一致的数据 。还建议查询`_meta`以检查子图是否跳过错误，如示例所示：

```
foos(first: 100, subgraphError: allow) {
  id
}

_meta {
  hasIndexingErrors
}
```

如果子图遇到错误，该查询将同时返回数据和带有消息的 graphql 错误`"indexing_error"`，如本示例响应中所示：

```
"data": {
    "foos": [
        {
          "id": "fooId"
        }
    ],
    "_meta": {
        "hasIndexingErrors": true
    }
},
"errors": [
    {
        "message": "indexing_error"
    }
]
```

## 嫁接现有子图

当子图首次部署时，它在相应链的创世块（或`startBlock`每个数据源定义的）开始索引事件在某些情况下，重用来自现有子图的数据并开始索引是有益的后来块。这种索引模式称为*嫁接*。例如，嫁接在开发过程中非常有用，可以快速解决映射中的简单错误，或者在现有子图失败后暂时使其重新工作。

当子图清单在顶层`subgraph.yaml`包含一个`graft`块时，子图被嫁接到基础子图上 ：

```
description: ...
graft:
  base: Qm...      # Subgraph ID of base subgraph
  block: 7345624   # Block number
```

当`graft`部署其清单包含块的子图时，图节点将复制`base`子图的数据直到并包括给定的数据`block` ，然后继续从该块开始索引新的子图。基本子图必须存在于目标图节点实例上，并且必须至少索引到给定的块。由于此限制，嫁接应仅在开发期间或紧急情况下使用，以加快生成等效的非嫁接子图。

由于嫁接副本而不是索引基础数据，因此将子图获取到所需块比从头开始索引要快得多，尽管对于非常大的子图，初始数据复制仍可能需要几个小时。在初始化嫁接子图时，图节点将记录有关已复制的实体类型的信息。

嫁接的子图可以使用与基础子图不同的 GraphQL 模式，而只是与其兼容。它本身必须是一个有效的子图模式，但可以通过以下方式偏离基本子图的模式：

- 它添加或删除实体类型
- 它从实体类型中删除属性
- 它为实体类型添加了可为空的属性
- 它将不可为空的属性变成可以为空的属性
- 它为枚举添加值
- 它添加或删除接口
- 它改变了实现接口的实体类型
