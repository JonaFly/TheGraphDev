- 子图定义了 The Graph 将从以太坊索引哪些数据，以及它将如何存储这些数据。一旦部署，它将成为区块链数据全局图的一部分。

  ​           

  ![子图图形](https://thegraph.com/docs/images/define-subgraph.png)

  ​           

  子图定义由几个文件组成：

  - `subgraph.yaml`：包含*子图清单*的 YAML 文件**
  - `schema.graphql`: 一个 GraphQL 模式，它定义了为你的子图存储了什么数据，以及如何通过 GraphQL 查询它
  - `AssemblyScript Mappings`：从以太坊中的事件数据转换为模式中定义的实体的[AssemblyScript](https://github.com/AssemblyScript/assemblyscript)代码（例如`mapping.ts`在本教程中）

  在详细了解清单文件的内容之前，您需要安装 构建和部署子图所需的[Graph CLI](https://github.com/graphprotocol/graph-cli)。

  ## 安装图形 CLI

  Graph CLI 是用 JavaScript 编写的，您需要安装 [yarn](https://yarnpkg.com/lang/en/docs/install/#centos-stable)或 [npm](https://www.npmjs.com/get-npm)才能使用它；假设您有`yarn`以下内容。安装的详细说明`yarn` 可以在 [graph-cli repo中找到](https://github.com/graphprotocol/graph-cli/wiki/Installation-on-different-operating-systems)

  完成后`yarn`，通过运行安装 Graph CLI

  ```
  yarn global add @graphprotocol/graph-cli
  ```

  ## 创建子图项目

  该`graph init`命令可用于从任何公共以太坊网络上的现有合约或[示例子图](https://github.com/graphprotocol/example-subgraph)设置新的子图项目 。

  ### 从现有合同

  如果您已经将智能合约部署到以太坊主网或其中一个测试网，那么从该合约引导一个新的子图可能是一个很好的入门方式。

  以下命令创建一个子图，用于索引现有合约的所有事件。它尝试从 Etherscan 获取合约 ABI 并回退到请求本地文件路径。如果缺少任何可选参数，它将引导您完成交互式表单。

  ```
  graph init \
    --from-contract <CONTRACT_ADDRESS> \
    [--network <ETHEREUM_NETWORK>] \
    [--abi <FILE>] \
    <GITHUB_USER>/<SUBGRAPH_NAME> [<DIRECTORY>]
  ```

  该`<GITHUB_USER>`是你的GitHub用户或组织名称，`<SUBGRAPH_NAME>`是名称为您子，并且`<DIRECTORY>`是所在的目录的名称（可选）`graph init`将把例如子清单。

  该`<CONTRACT_ADDRESS>`是你的现有合同的地址。 `<ETHEREUM_NETWORK>`是合约所在的以太坊网络的名称。`<FILE>`是合同 ABI 文件的本地路径。这两个`--network`和 `--abi`是可选的。

  托管服务支持的网络包括：

  1. 以太坊主网 -> `mainnet`
  2. 科万-> `kovan`
  3. 林克比-> `rinkeby`
  4. 罗普斯滕 -> `ropsten`
  5. 歌尔莉-> `goerli`
  6. POA-核心 -> `poa-core`
  7. POA-Sokol -> `poa-sokol`
  8. xDAI -> `xdai`
  9. 多边形/Matic -> `matic`
  10. 孟买（Matic 的测试网）-> `mumbai`
  11. 幻影 -> `fantom`
  12. 币安智能链 -> `bsc`
  13. 币安智能链测试网 -> `chapel`
  14. 三叶草-> `clover`
  15. 雪崩 -> `avalanche`
  16. Avalanche 的测试网 -> `fuji`
  17. 塞洛-> `celo`
  18. Celo 的测试网 (Alfajores) -> `celo-alfajores`
  19. 保险丝-> `fuse`
  20. 月光-> `mbase`
  21. 仲裁测试网 -> `arbitrum-rinkeby`
  22. 套路一-> `arbitrum-one`

  ### 从示例子图中

  第二种模式`graph init`支持从[示例子图](https://github.com/graphprotocol/example-subgraph)创建一个新项目 。以下命令执行此操作：

  ```
  graph init --from-example <GITHUB_USER>/<SUBGRAPH_NAME> [<DIRECTORY>]
  ```

  示例子图基于[Dani Grant](https://github.com/danigrant)的[Gravity 合约](https://github.com/graphprotocol/example-subgraph/blob/master/contracts/Gravity.sol) ，该[合约](https://github.com/graphprotocol/example-subgraph/blob/master/contracts/Gravity.sol)管理用户头像并在创建或更新头像时发出或发送事件。子图通过将实体写入图节点存储并确保根据事件更新这些实体来处理这些事件。以下部分将介绍构成此示例的子图清单的文件。`NewGravatar``UpdateGravatar``Gravatar`

  ## 子图清单

  子图清单`subgraph.yaml`定义了您的子图索引的智能合约，要注意来自这些合约的哪些事件，以及如何将事件数据映射到 Graph Node 存储并允许查询的实体。可以在[此处](https://github.com/graphprotocol/graph-node/blob/master/docs/subgraph-manifest.md)找到子图清单的完整规范 。

  对于示例子图，`subgraph.yaml`是：

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

  要为清单更新的重要条目是：

  - `description`：子图是什么的人类可读的描述。当子图部署到[托管服务](https://thegraph.com/explorer/)时，图资源管理器会显示此描述。
  - `repository`：可以在其中找到子图清单的存储库的 URL。这也由 Graph Explorer 显示。
  - `dataSources.source`:`address`智能合约的子图来源，以及`abi`要使用的智能合约的 。该`address`是可选的; 省略它允许索引所有合约中的匹配事件。
  - `dataSources.source.startBlock`：数据源开始索引的块的可选编号。在大多数情况下，我们建议使用创建合约的区块。
  - `dataSources.mapping.entities`：数据源写入存储的实体。每个实体的架构在`schema.graphql`文件中定义。
  - `dataSources.mapping.abis`：源合约以及您从映射中与之交互的任何其他智能合约的一个或多个命名 ABI 文件。
  - `dataSources.mapping.eventHandlers`：列出该子图响应的智能合约事件以及映射`./src/mapping.ts`中的处理程序——在示例中——将这些事件转换为商店中的实体。
  - `dataSources.mapping.callHandlers`: 列出该子图响应的智能合约函数和映射中的处理程序，这些函数将输入和输出转换为函数调用到商店中的实体。
  - `dataSources.mapping.blockHandlers`: 列出这个子图响应的块和映射中的处理程序，当一个块被附加到链时运行。如果没有过滤器，块处理程序将在每个块中运行。可以提供具有以下种类的可选过滤器：`call`. 甲`call`如果块包含至少一个呼叫到数据源合同滤波器将运行该处理程序。

  单个子图可以索引来自多个智能合约的数据。为需要将数据索引到`dataSources`数组的每个合约添加一个条目 。

  块内数据源的触发器使用以下过程排序：

  1. 事件和调用触发器首先按块内的交易索引排序。
  2. 同一事务中的事件和调用触发器使用约定排序：首先事件触发器然后调用触发器，每种类型都遵循它们在清单中定义的顺序。
  3. 块触发器按照它们在清单中定义的顺序在事件和调用触发器之后运行。

  这些订购规则可能会发生变化。

  ### 获取 ABI

  ABI 文件必须与您的合同相符。获取ABI文件有以下几种方式：

  - 如果您正在构建自己的项目，则可能可以访问最新的 ABI。
  - 如果您正在为公共项目构建子图，您可以将该项目下载到您的计算机并通过使用[`truffle compile`](https://truffleframework.com/docs/truffle/overview) 或使用`solc`编译来获取 ABI 。
  - 您也可以在[Etherscan](https://etherscan.io/)上找到 ABI ，但这并不总是可靠的，因为在那里上传的 ABI 可能已过时。确保您拥有正确的 ABI，否则运行您的子图将失败。

  ## GraphQL 架构

  子图的架构在文件中`schema.graphql`。GraphQL 模式是使用 GraphQL 接口定义语言定义的。如果您从未编写过 GraphQL 模式，建议您查看有关 GraphQL 类型系统的这本 [入门](https://graphql.org/learn/schema/#type-language)书。GraphQL 模式的参考文档可以在 [GraphQL API 部分找到](https://thegraph.com/docs/graphql-api)。

  ## 定义实体

  在定义实体之前，退后一步考虑一下您的数据的结构和链接方式是很重要的。所有查询都将针对子图模式中定义的数据模型和子图索引的实体进行。因此，最好以符合 dApp 需求的方式定义子图模式。将实体想象为“包含数据的对象”而不是事件或函数可能会很有用。

  使用 The Graph，您只需在 中定义实体类型`schema.graphql`，Graph Node 将生成顶级字段，用于查询该实体类型的单个实例和集合。每个应该是实体的类型都需要用`@entity`指令进行注释。

  ### 好例子

  `Gravatar`下面的实体是围绕 Gravatar 对象构建的，是如何定义实体的一个很好的例子。

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

  下面的示例`GravatarAccepted`和`GravatarDeclined`实体基于事件。不建议将事件或函数调用映射到实体 1:1。

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

  实体字段可以定义为必需或可选。必填字段由`!`架构中的 指示。如果映射中未设置必填字段，则在查询该字段时会收到此错误：

  ```
  Null value resolved for non-null field 'name'
  ```

  每个实体都必须有一个`id`字段，它的类型是`ID!`（字符串）。该`id`字段作为主键，需要在所有同类型实体中唯一。

  ### 内置标量类型

  #### GraphQL 支持的标量

  我们在 GraphQL API 中支持以下标量：

  | 类型           | 描述                                       |
  | ------------ | ---------------------------------------- |
  | `Bytes`      | 字节数组，表示为十六进制字符串。通常用于以太坊哈希和地址。            |
  | `ID`         | 存储为`string`.                             |
  | `String`     | `string`值的标量。不支持空字符，会自动删除。               |
  | `Boolean`    | `boolean`值的标量。                           |
  | `Int`        | GraphQL 规范定义`Int`大小为 32 字节。              |
  | `BigInt`     | 大整数。用于以太坊的`uint32`, `int64`, `uint64`, ...,`uint256`类型。注意：下面的所有内容`uint32`，例如`int32`、`uint24`或`int8`都表示为`i32`。 |
  | `BigDecimal` | `BigDecimal`高精度小数表示为有效数和指数。指数范围是从 -6143 到 +6144。四舍五入为 34 位有效数字。 |

  #### 枚举

  您还可以在架构中创建枚举。枚举具有以下语法：

  ```
  enum TokenStatus {
    OriginalOwner
    SecondOwner
    ThirdOwner
  }
  ```

  一旦在架构中定义了枚举，您就可以使用枚举值的字符串表示在实体上设置枚举字段。例如，您可以通过首先定义您的实体然后将字段设置为`tokenStatus`来`SecondOwner`设置`entity.tokenStatus = "SecondOwner`。下面的示例演示了`Token`带有枚举字段的实体的外观：

  ```
  type Token @entity {
    id: ID!
    name: String!
    symbol: String!
    decimals: Int!
    tokenStatus: TokenStatus!
  }
  ```

  有关编写枚举的更多详细信息，请参阅[GraphQL 文档](https://graphql.org/learn/schema/)。

  ### 实体关系

  一个实体可能与您的架构中的一个或多个其他实体有关系。这些关系可能会在您的查询中遍历。图中的关系是单向的。可以通过在关系的任一“端”上定义单向关系来模拟双向关系。

  除了指定的类型是另一个实体的类型之外，关系就像任何其他字段一样在实体上定义。

  #### 一对一关系

  定义与`Transaction`实体类型具有可选一对一关系的`TransactionReceipt`实体类型：

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

  定义与`TokenBalance`实体类型具有必需的一对多关系的`Token`实体类型：

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

  可以通过`@derivedFrom` 字段在实体上定义反向查找。这会在实体上创建一个虚拟字段，该字段可以被查询但不能通过映射 API 手动设置。相反，它源自在另一个实体上定义的关系。对于这样的关系，存储关系的两边几乎没有意义，当只存储一侧并导出另一侧时，索引和查询性能都会更好。

  对于一对多关系，关系应该*始终*存储在“一”侧，并且始终应该导出“多”侧。以这种方式存储关系，而不是在“多”端存储实体数组，将显着提高索引和查询子图的性能。通常，应尽可能避免存储实体数组。

  #### 例子

  我们可以通过派生一个`tokenBalances`字段来从令牌访问令牌的余额：

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

  对于多对多关系，例如每个用户可能属于任意数量的组织，最直接但通常不是最高效的关系建模方法是作为所涉及的两个实体中的每一个的数组。如果关系是对称的，则只需存储关系的一侧，即可导出另一侧。

  #### 例子

  定义从`User`实体类型到`Organization`实体类型的反向查找。在下面的示例中，这是通过`members`从`Organization`实体内查找属性来实现的。在查询中，`organizations`字段 on`User`将通过查找`Organization`包含用户 ID 的所有实体来解析。

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

  存储这种关系的一种更高效的方法是通过一个映射表，该表为每个`User`/`Organization`对具有一个条目，其架构类似于

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

  这种方法要求查询下降到一个额外的级别来检索，例如，用户的组织：

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

  这种存储多对多关系的更精细的方法将导致为子图存储的数据更少，因此子图的索引和查询速度通常要快得多。

  ### 向模式添加注释

  根据 GraphQL 规范，可以使用双引号在模式实体属性上方添加注释`""`。下面的示例说明了这一点：

  ```
    type MyFirstEntity @entity {
      "unique identifier and primary key of the entity"
      id: ID!
      address: Bytes!
    }
  ```

  ## 定义全文搜索字段

  全文搜索查询基于文本搜索输入对实体进行过滤和排名。通过在与索引文本数据进行比较之前将查询文本输入处理到词干中，全文查询能够返回相似词的匹配项。

  全文查询定义包括查询名称、用于处理文本字段的语言词典、用于对结果进行排序的排名算法以及包含在搜索中的字段。每个全文查询可能跨越多个字段，但所有包含的字段必须来自单个实体类型。

  要添加全文查询，请`_Schema_`在 GraphQL 模式中包含带有全文指令的类型。

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

  这个例子`bandSearch`字段可以在查询过滤器中使用`Band`基于文本的文档实体`name`，`description`和`bio`领域。跳转到[GraphQL API - 查询](https://thegraph.com/docs/graphql-api#queries)全文搜索 API 的描述和更多示例用法。

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

  选择不同的语言将对全文搜索 API 产生明确但有时很微妙的影响。全文查询字段涵盖的字段在所选语言的上下文中进行检查，因此分析和搜索查询生成的词素因语言而异。例如：当使用受支持的土耳其语词典时，“token”词干为“toke”，而英语词典当然会将其词干为“token”。

  支持的语言词典：

  | 代码   | 字典    |
  | ---- | ----- |
  | 简单的  | 一般的   |
  | 达    | 丹麦语   |
  | nl   | 荷兰语   |
  | 恩    | 英语    |
  | 菲    | 芬兰    |
  | fr   | 法语    |
  | 德    | 德语    |
  | 胡    | 匈牙利   |
  | 它    | 意大利语  |
  | 不    | 挪威    |
  | 点    | 葡萄牙语  |
  | 罗    | 罗马尼亚语 |
  | 茹    | 俄语    |
  | es   | 西班牙语  |
  | SV   | 瑞典    |
  | tr   | 土耳其   |

  ### 排名算法

  支持的排序结果算法：

  | 算法   | 描述                         |
  | ---- | -------------------------- |
  | 秩    | 使用全文查询的匹配质量 (0-1) 对结果进行排序。 |
  | 邻近等级 | 类似于`rank`但也包括匹配项的接近度。      |

  ## 写映射

  映射将您的映射所获取的以太坊数据转换为您的架构中定义的实体。映射是用 称为 [AssemblyScript](https://github.com/AssemblyScript/assemblyscript/wiki)的[TypeScript](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)子集编写的， 它可以编译为 WASM ( [WebAssembly](https://webassembly.org/) )。AssemblyScript 比普通的 TypeScript 更严格，但提供了熟悉的语法。

  对于`subgraph.yaml`在下 定义的每个事件处理程序`mapping.eventHandlers`，创建一个同名的导出函数。每个处理程序都必须接受一个参数`event`，该参数的类型与正在处理的事件的名称相对应。

  在示例子图中，`src/mapping.ts`包含`NewGravatar`和`UpdatedGravatar`事件的处理程序 ：

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

  第一个处理程序接受一个`NewGravatar`事件并使用 来创建一个新`Gravatar` 实体`new Gravatar(event.params.id.toHex())`，并使用相应的事件参数填充实体字段。此实体实例由变量 表示`gravatar`，`id`值为`event.params.id.toHex()`。

  第二个处理程序尝试`Gravatar`从图形节点存储中加载现有的。如果它还不存在，则按需创建。然后更新实体以匹配新的事件参数，然后使用`gravatar.save()`.

  ### 创建新实体的推荐 ID

  每个实体都必须有一个`id`在相同类型的所有实体中是唯一的。实体的`id`值是在创建实体时设置的。以下是`id`创建新实体时要考虑的一些推荐值。注意： 的值`id`必须是`string`。

  - `event.params.id.toHex()`
  - `event.transaction.from.toHex()`
  - `event.transaction.hash.toHex() + "-" + event.logIndex.toString()`

  我们提供了 [Graph Typescript 库](https://github.com/graphprotocol/graph-ts)，其中包含与 Graph Node 存储交互的实用程序以及处理智能合约数据和实体的便利。您可以通过导入使用您的映射这个库`@graphprotocol/graph-ts`中`mapping.ts`。

  ## 代码生成

  为了使智能合约、事件和实体的工作变得简单且类型安全，Graph CLI 可以从子图的 GraphQL 模式和数据源中包含的合约 ABI 生成 AssemblyScript 类型。

  这是用

  ```
  graph codegen [--output-dir <OUTPUT_DIR>] [<MANIFEST>]
  ```

  但在大多数情况下，子图已经通过预配置`package.json`，允许您简单地运行以下之一来实现相同的目的：

  ```
  # Yarn
  yarn codegen

  # NPM
  npm run codegen
  ```

  这将为 中提到的 ABI 文件中的每个智能合约生成一个 AssemblyScript 类`subgraph.yaml`，允许您将这些合约绑定到映射中的特定地址，并针对正在处理的块调用只读合约方法。它还将为每个合约事件生成一个类，以便轻松访问事件参数以及事件源自的区块和交易。所有这些类型都写入`<OUTPUT_DIR>/<DATA_SOURCE_NAME>/<ABI_NAME>.ts`. 在示例子图中，这将是`generated/Gravity/Gravity.ts`，允许映射导入这些类型

  ```
  import {
    // The contract class:
    Gravity,
    // The events classes:
    NewGravatar,
    UpdatedGravatar,
  } from '../generated/Gravity/Gravity'
  ```

  除此之外，还为子图的 GraphQL 模式中的每个实体类型生成一个类。这些类提供类型安全的实体加载、对实体字段的读写访问以及一种`save()`将实体写入存储的方法。所有实体类都被写入`<OUTPUT_DIR>/schema.ts`，允许映射将它们导入

  ```
  import { Gravatar } from '../generated/schema'
  ```

  > **注意：**每次更改 GraphQL 架构或清单中包含的 ABI 后，都必须再次执行代码生成。在构建或部署子图之前，它也必须至少执行一次。

  代码生成不会在`src/mapping.ts`. 如果您想在尝试将子图部署到 Graph Explorer 之前进行检查，您可以运行`yarn build`并修复 TypeScript 编译器可能发现的任何语法错误。

  如果您的合约的只读方法可能会恢复，那么您应该通过调用以 为前缀的生成的合约方法来处理该问题`try_`。例如，Gravity 合约公开了该`gravatarToOwner`方法。此代码将能够处理该方法中的还原：

  ```
    let gravity = Gravity.bind(event.address)
    let callResult = gravity.try_gravatarToOwner(gravatar)
    if (callResult.reverted) {
      log.info("getGravatar reverted", [])
    } else {
      let owner = callResult.value
    }
  ```

  请注意，连接到 Geth 或 Infura 客户端的 Graph 节点可能无法检测到所有还原，如果您依赖于此，我们建议使用连接到 Parity 客户端的 Graph 节点。

  在下一节中，我们将描述如何使用 Graph Explorer 部署您的子图。

  ## 数据源模板

  以太坊智能合约中的一个常见模式是使用注册或工厂合约，其中一个合约创建、管理或引用任意数量的其他合约，每个合约都有自己的状态和事件。这些子合同的地址可能会或可能不会预先知道，并且其中许多合同可能会随着时间的推移而创建和/或添加。这就是为什么在这种情况下，定义单个数据源或固定数量的数据源是不可能的，需要一种更动态的方法：*数据源模板*。

  ### 主合约数据源

  首先，您为主合约定义一个常规数据源。下面的代码片段显示了[Uniswap](https://uniswap.io/) 交换工厂合约的简化示例数据源。注意`NewExchange(address,address)`事件处理程序。当工厂合约在链上创建新的交换合约时会发出此消息。

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

  然后，将*数据源模板*添加到清单。它们与常规数据源相同，只是它们在 `source`. 通常，您将为父合同管理或引用的每种类型的子合同定义一个模板。

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

  在最后一步中，您更新主合同映射以从模板之一创建动态数据源实例。在此示例中，您将更改主合约映射以导入`Exchange`模板并调用其`Exchange.create(address)`上的方法以开始索引新的交换合约。

  ```
  import { Exchange } from '../generated/templates'

  export function handleNewExchange(event: NewExchange): void {
    // Start indexing the exchange; `event.params.exchange` is the
    // address of the new exchange contract
    Exchange.create(event.params.exchange)
  }
  ```

  > **注意：**新数据源只会处理创建它的块和所有后续块的调用和事件，但不会处理历史数据，即包含在先前块中的数据。
  >
  > 如果先前的区块包含与新数据源相关的数据，最好通过读取合约的当前状态并在创建新数据源时创建代表该状态的实体来索引该数据。

  ### 数据源上下文

  数据源上下文允许在实例化模板时传递额外的配置。在我们的示例中，假设交易所与`NewExchange`事件中包含的特定交易对相关联。该信息可以传递到实例化的数据源中，如下所示：

  ```
  import { Exchange } from '../generated/templates'

  export function handleNewExchange(event: NewExchange): void {
    let context = new DataSourceContext()
    context.setString("tradingPair", event.params.tradingPair)
    Exchange.createWithContext(event.params.exchange, context)
  }
  ```

  在`Exchange`模板的映射中，可以访问上下文：

  ```
  import { dataSource } from "@graphprotocol/graph-ts"

  let context = dataSource.context()
  let tradingPair = context.getString("tradingPair")
  ```

  有getter和setter方法一样`setString`，并`getString`为所有的值类型。

  ## 起始块

  这`startBlock`是一个可选设置，允许您定义数据源将从链中的哪个块开始索引。设置起始块允许数据源跳过潜在的数百万个不相关的块。通常，子图开发人员将设置`startBlock`为创建数据源智能合约的块。

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

  > **注意：**合约创建区块可以在 Etherscan 上快速查找： 1. 通过在搜索栏中输入其地址来搜索合约。2. 点击`Contract Creator`栏目中的创建交易哈希。3. 加载交易详情页面，您将在其中找到该合约的起始区块。

  ## 呼叫处理程序

  虽然事件提供了一种收集合约状态相关变化的有效方法，但许多合约避免生成日志以优化 gas 成本。在这些情况下，子图可以订阅对数据源合同的调用。这是通过定义引用函数签名的调用处理程序和将处理对此函数调用的映射处理程序来实现的。为了处理这些调用，映射处理程序将接收一个`ethereum.Call`作为参数，其中包含调用的类型化输入和输出。在事务调用链中的任何深度进行的调用都将触发映射，从而允许通过代理合约捕获数据源合约的活动。

  调用处理程序只会在以下两种情况之一触发：当指定的函数被合约本身以外的帐户调用时，或者当它在 Solidity 中被标记为外部并作为同一合约中另一个函数的一部分被调用时。

  > **注意：** Rinkeby 和 Ganache 不支持呼叫处理程序。调用处理程序依赖于 Parity 跟踪 API，Rinkeby 和 Ganache 都不支持这一点（Rinkeby 仅支持 Geth）。

  ### 定义调用处理程序

  要在清单中定义调用处理程序，只需`callHandlers`在要订阅的数据源下添加一个数组。

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

  该`function`是标准化函数签名通过过滤器的调用。该 `handler`属性是在数据源合同中调用目标函数时要执行的映射中函数的名称。

  ### 映射函数

  每个调用处理程序都接受一个参数，该参数的类型与被调用函数的名称相对应。在上面的示例子图中，映射包含一个处理程序，用于`createGravatar`调用函数并接收 `CreateGravatarCall`参数作为参数：

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

  该`handleCreateGravatar`函数采用一个 new `CreateGravatarCall`，它是 的子类`ethereum.Call`，由 提供`@graphprotocol/graph-ts`，其中包括调用的类型化输入和输出。在`CreateGravatarCall`当您运行类型为您生成`graph codegen`。

  ## 块处理程序

  除了订阅合约事件或函数调用之外，子图可能希望在新块被附加到链上时更新其数据。为了实现这一点，子图可以在每个块或匹配预定义过滤器的块之后运行一个函数。

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