+++
title = "Dgraph入门--介绍"
+++

**欢迎开始使用Dgraph。**

[Dgraph](https://dgraph.io)是一个开源的、事务性的、分布式的、原生的Graph数据库。下面是使用Dgraph入门系列的第一篇教程。

在本教程中，我们将学习如何在Dgraph上构建以下图。

{{% load-img "/images/tutorials/1/gs-1.JPG" "The simple graph" %}}

在这个过程中，我们会了解到：

- 使用 `dgraph/standalone` docker镜像运行Dgraph。
- 使用Dgraph的UI Ratel运行以下基本操作。
 - 创建一个节点。
 - 在两个节点之间建立一个边。
 - 查询节点的情况。

你可以看看下面的配套视频。

{{< youtube u73ovhDCPQQ >}}

---

## 运行Dgraph

运行 `dgraph/standalone` docker镜像是开始使用Dgraph的最快方法。
此独立镜像仅用于快速入门目的，不建议用于生产环境。

确保[Docker](https://docs.docker.com/install/)已在你的机器上安装并运行。

现在，只需运行下面的命令，Dgraph就可以运行了。

```sh
docker run --rm -it -p 8000:8000 -p 8080:8080 -p 9080:9080 dgraph/standalone:{{< version >}}
```

### 节点和边

在本节中，我们将建立一个简单的图，其中有两个节点和一条边连接它们。

{{% load-img "/images/tutorials/1/gs-1.JPG" "The simple graph" %}}

在图数据库中，概念或实体被表示为节点。
可能是一个商品，一个交易，一个地方，或一个人，所有这些实体都是。
作为图谱数据库中的节点来表示。

一个边代表两个节点之间的关系。
上图中的两个节点代表人：`Karthic`和`Jessica`。
你还可以看到，这些节点有两个相关的属性：`name`和`age`。
节点的这些属性在Dgraph中称为 `谓语`。

Karthic跟随Jessica。他们之间的 `follows` 边代表他们的关系。
连接两个节点的边在Dgraph中也叫`谓语`。
虽然这个指向另一个节点，而不是一个字符串或整数。

`dgraph/standalone`图像设置自带有用的Dgraph UI，称为Ratel。
只要从浏览器中访问 [http://localhost:8000](http://localhost:8000) ，就能访问它。

{{% load-img "/images/tutorials/1/gs-2.png" "ratel-1" %}}

我们将使用Ratel的最新稳定版本。

{{% load-img "/images/tutorials/1/gs-3.png" "ratel-2" %}}

### 使用Ratel的突变

Dgraph中的创建、更新和删除操作称为突变。

Ratel让运行查询和突变变得更加容易。
我们会随着系列教程一直探索它的更多功能。

让我们进入 "突变 "选项卡，将以下突变粘贴到文本区域。
_先不要执行它！_。

```json
{
  "set": [
    {
      "name": "Karthic",
      "age": 28
    },
    {
      "name": "Jessica",
      "age": 31
    }
  ]
}
```

上面的查询创建了两个节点，一个对应于与`"set"`相关联的每个JSON值。
然而，它并没有在这些节点之间创建一个边。

稍微修改一下突变就会固定下来，这样就会在它们之间形成一个边缘。

```json
{
  "set": [
    {
      "name": "Karthic",
      "age": 28,
      "follows": {
        "name": "Jessica",
        "age": 31
      }
    }
  ]
}
```

{{% load-img "/images/tutorials/1/explain-query.JPG" "explain mutation" %}}

让我们来执行这个突变。点击 "运行"，轰！

{{% load-img "/images/tutorials/1/mutate-example.gif" "Query-gif" %}}

您可以在响应中看到，已经创建了两个UID（通用IDentifiers）。
响应的 `"uid"` 字段中的两个值对应于
到为 "Karthic "和 "Jessica "创建的两个节点。

### 使用has函数进行查询

现在，让我们运行一个查询来可视化我们刚刚创建的节点。
我们将使用Dgraph的`has`函数。
表达式 `has(name)` 返回所有与 `name` 关联的节点。

```sh
{
  people(func: has(name)) {
    name
    age
  }
}
```

这次进入`查询`选项卡，输入上面的查询。
然后，点击屏幕右上方的 `运行`。

{{% load-img "/images/tutorials/1/query-1.png" "query-1" %}}

Ratel渲染出一个图形可视化的结果。

只要点击任意一个节点，注意节点都会分配UID。
匹配的，我们在突变的反应中看到。

你也可以在右侧的JSON标签中查看JSON结果。

{{% load-img "/images/tutorials/1/query-2.png" "query-2" %}}

#### 了解查询

{{% load-img "/images/tutorials/1/explain-query-2.JPG" "Illustration with explanation" %}}

查询的第一部分是用户定义的函数名称。
在我们的查询中，我们将其命名为 `people`。但是，你可以使用任何其他名称。

`func`参数必须与Dgraph的内置函数相关联。
Dgraph提供了多种内置函数，`has`函数就是其中之一。
查看[查询语言指南](https://dgraph.io/docs/query-language)以了解更多关于Dgraph的其他内置函数。

查询的内部字段与SQL选择语句或GraphQL查询中的列名相似!

你可以很容易地指定你想取回哪些谓词。

```graphql
{
  people(func: has(name)) {
    name
  }
}
```

同样，你可以使用 `has` 函数找到所有带有 `年龄` 谓词的节点。

```graphql
{
  people(func: has(age)) {
    name
  }
}
```

### 灵活的模式

Dgraph不强制执行结构或模式。相反，你可以开始输入
您的数据立即，并根据需要添加约束条件。

我们来看看这个突变。

```json
{
  "set": [
    {
      "name": "Balaji",
      "age": 23,
      "country": "India"
    },
    {
      "name": "Daniel",
      "age": 25,
      "city": "San Diego"
    }
  ]
}
```

我们正在创建两个节点，而第一个节点有谓词 `name`、`age` 和 `country`。
第二个有 `姓名`、`年龄` 和 `城市`。

最初不需要模式。Dgraph创建
新的谓词出现在你的突变中。
这种灵活性可能是有益的，但如果你喜欢强迫你的
遵循给定模式的突变有以下选项
我们将在下一个教程中探讨。

## 收尾工作

在本教程中，我们学习了Dgraph的基础知识，包括如何使用
运行数据库，添加新的节点和谓词，并查询它们。

在我们收工之前，这里有一些关于接下来教程的小知识。
你知道节点的UID也可以被获取吗？
它们还可以用来在现有节点之间创建一个边!

听起来很有趣？

查看我们下一个入门系列教程[这里]({{< relref "tutorial-2/index.md" >}})。

## 帮助中心

* 在 [discuss.dgraph.io](https://discuss.dgraph.io) 提问、提出需求、讨论。
* 如果你遇到bug或有功能需求，请使用[Github Issues](https://github.com/dgraph-io/dgraph/issues)。
