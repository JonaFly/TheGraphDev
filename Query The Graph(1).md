# 查询图表

部署子图后，访问[Graph Explorer](https://thegraph.com/explorer/dashboard)以打开一个[GraphiQL](https://github.com/graphql/graphiql)界面，您可以在其中通过发出查询和查看架构来探索为子图部署的 GraphQL API。

下面提供了一个示例，但请参阅[查询 API](https://thegraph.com/docs/graphql-api)以获取有关如何查询子图实体的完整参考。

#### 例子

此查询列出了我们的映射创建的所有计数器。由于我们只创建一个，结果将只包含我们的一个`default-counter`：

```
{
  counters {
    id
    value
  }
}
```

## 使用图形浏览器

Graph Explorer 及其 GraphQL 游乐场是探索和查询托管服务上部署的子图的有用方法。

一些主要功能详述如下： ![探索者游乐场](https://thegraph.com/docs/images/explorer-playground.png)