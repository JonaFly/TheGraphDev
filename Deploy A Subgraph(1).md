# 部署子图

子图定义了 The Graph 将从以太坊索引哪些数据，以及它将如何存储这些数据。一旦部署，它将成为区块链数据全局图的一部分。[Graph CLI](https://github.com/graphprotocol/graph-cli)为您的子图生成代码。现在是时候将子图部署到托管的 Graph 服务了。

## 创建 Graph Explorer 帐户

在使用托管服务之前，请在 The Graph Explorer 中创建一个帐户。 为此，您需要一个[Github](https://github.com/)帐户；如果您没有，则需要先创建它。然后，导航到[Explorer](https://thegraph.com/explorer/)，单击*“Sign up with Github”*按钮并完成 Github 的授权流程。

## 存储访问令牌

创建帐户后，导航到您的 [仪表板](https://thegraph.com/explorer/dashboard)。复制仪表板上显示的访问令牌并运行`graph auth https://api.thegraph.com/deploy/ <ACCESS_TOKEN>`。这会将访问令牌存储在您的计算机上。您只需要执行一次，或者如果您曾经重新生成访问令牌。

## 创建子图

在部署子图之前，您需要在 Graph Explorer 中创建它。转到[仪表板](https://thegraph.com/explorer/dashboard) 并单击*“添加子图”*按钮并根据需要填写以下信息：

**Image**- 选择要用作子图预览图像和缩略图的图像。

**Subgraph Name**- 连同在其下创建子图的帐户名称，这还将定义`account-name/subgraph-name`用于部署和 GraphQL endpoint的样式名称。*此字段以后无法更改。*

**Account**- 在其下创建子图的帐户。这可以是个人或组织的帐户。*以后不能在帐户之间移动子图。*

**Subtitle**- 将出现在子图卡中的文本。

**Description**- 子图的描述，在子图详细信息页面上可见。

**GitHub URL** - 链接到 GitHub 上的子图存储库。

**Hide**- 打开此选项会隐藏 Graph Explorer 中的子图。

保存新子图后，您会看到一个界面，其中包含有关如何安装 Graph CLI、如何为新子图生成脚手架以及如何部署子图的帮助信息。前两个步骤已在上 [一节中介绍](https://thegraph.com/docs/define-a-subgraph)。

## 部署子图

部署您的子图会将您构建的子图文件上传`yarn build`到 IPFS，并告诉 Graph Explorer 开始使用这些文件索引您的子图。

您通过运行部署子图 `yarn deploy`

部署子图后，Graph Explorer 将切换到显示您的子图的同步状态。根据需要从历史以太坊区块中提取的数据量和事件数量，从创世区块开始，同步可能需要几分钟到几个小时。`Synced`一旦图节点从历史区块中提取了所有数据。在挖掘这些区块时，图节点将继续检查您的子图的以太坊区块。

## 重新部署子图

更改子图定义时，例如修复实体映射中的问题，请`yarn deploy` 再次运行上述命令以部署子图的更新版本。子图的任何更新都需要图节点重新索引整个子图，再次从创世块开始。

如果您之前部署的子图仍处于 status 状态`Syncing`，它将立即被新部署的版本替换。如果之前部署的子图已经完全同步，Graph Node 会将新部署的版本标记为`Pending Version`，在后台同步，并且只有在同步新版本完成后才将当前部署的版本替换为新版本。这确保您在新版本同步时有一个可以使用的子图。

### 将子图部署到多个以太坊网络

在某些情况下，您可能希望将同一个子图部署到多个以太坊网络而不复制其所有代码。主要需要解决的问题是这些网络上的合约地址不同。一种允许参数化合约地址等方面的解决方案是使用像[Mustache](https://mustache.github.io/)或[Handlebars](https://handlebarsjs.com/)这样的模板语言来生成部分代码 。

为了说明这种方法，我们假设应该使用不同的合约地址将子图部署到主网和 Ropsten。然后，您可以定义两个配置文件，为每个网络提供地址：

```
{
  "network": "mainnet",
  "address": "0x123..."
}
```

和

```
{
  "network": "ropsten",
  "address": "0xabc..."
}
```

与此同时，你可以在manifest与variable placeholders(变量占位符)替换网络名称和地址`{{network}}`，并`{{address}}`和重命名manifest例如`subgraph.template.yaml`：

```
# ...
dataSources:
  - kind: ethereum/contract
    name: Gravity
    network: mainnet
    network: {{network}}
    source:
      address: '0x2E645469f354BB4F5c8a05B3b30A929361cf77eC'
      address: '{{address}}'
      abi: Gravity
    mapping:
      kind: ethereum/events
```

为了使任一网络生成manifest，您可以向 package.json 添加两个额外的命令以及对 mustache 的依赖：

```
{
  ...
  "scripts": {
    ...
    "prepare:mainnet": "mustache config/mainnet.json subgraph.template.yaml > subgraph.yaml",
    "prepare:ropsten": "mustache config/ropsten.json subgraph.template.yaml > subgraph.yaml"
  },
  "devDependencies": {
    ...
    "mustache": "^3.1.0"
  }
}
```

要为主网或 Ropsten 部署此子图，您现在只需运行以下两个命令之一：

```
# Mainnet:
yarn prepare:mainnet && yarn deploy

# Ropsten:
yarn prepare:ropsten && yarn deploy
```

可以在[此处](https://github.com/graphprotocol/example-subgraph/tree/371232cf68e6d814facf5e5413ad0fef65144759)找到一个案例。

**注意：**这种方法也可以应用于更复杂的情况，其中需要替换的不仅仅是合约地址和网络名称，或者还需要从模板生成映射或 ABI。

## 检测子图正常运行

如果一个子图同步成功，这就证明子图将正常运行。但是，链上的new triggers(新触发器)可能会导致您的子图遇到Bug，或者由于性能问题或节点的问题，它可能会出现问题。

Graph Node 公开了一个 graphql endpoint ，您可以查询该endpoint 以检查子图的状态。在托管服务上，它位于`https://api.thegraph.com/index-node/graphql`。在本地节点上`8030`，默认情况下它在端口上可用。可以在[此处](https://github.com/graphprotocol/graph-node/blob/master/server/index-node/src/schema.graphql)找到此端点的完整架构 。这是一个检查子图当前版本状态的示例查询：

```
indexingStatusForCurrentVersion(subgraphName: "org/subgraph") {
  synced
  health
  fatalError {
    message
    block {
      number
      hash
    }
    handler
  }
  chains {
    chainHeadBlock {
      number
    }
    latestBlock {
      number
    }
  }
}
```

目前有为您提供 `chainHeadBlock` ，您可以将其与子图上的 latestBlock 进行比较，以检查它是否落后。`health`当前可以采用以下值：`healthy`如果没有发生错误，或者`failed`是否存在停止子图进度的错误。在这种情况下，您可以检查该 `fatalError`字段以获取有关此错误的详细信息。