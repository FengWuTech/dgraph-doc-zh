+++
title = "Get Started - 快速开始指引"
aliases = ["/get-started-old"]
+++

{{% notice "note" %}}
这是一个快速启动指引
你可以在[这里]({{< relref "tutorials/index.md" >}})找到完整的入门使用教程.
{{% /notice %}}

## Dgraph

一开始就是为生产环境锁设计的,**Dgraph**也是一个天然的用图做后端的支持GraphQL的数据库.  它是开源的，可扩展的，分布式的，高可用的，并且拥有很好的运行速度.


Dgraph集群由不同的节点（Zero、Alpha和Ratel）组成，每个节点服务于不同的目的。

- **Dgraph Zero** 控制 Dgraph 集群，将server分配给组，
重新平衡server组之间的数据.

- **Dgraph Alpha** 用于管理数据(谓词和索引)，谓词or属性
代表两个节点之间的关系。
索引是边/谓词的一个属性标记，标记后就可以用函数来进行filter过滤查询了.

- **Ratel** 提供用户界面来执行数据查询，数据修改及元数据管理

服务搭建最少需要 1 个 zero 节点，1 个 alpha 节点即可启动一个服务。

**这里是一个带你上手和运行服务的四步向导.**

这是运行Dgraph的快速入门指南。如果想要交互式的演练，请来[这里](https://dgraph.io/tour/).

{{% notice "tip" %}}
本指南适用于Dgraph的强大查询语言[GraphQL+-]
(https://dgraph.io/docs/master/query-language/#graphql)，它是Facebook创建的一种查询语言 [GraphQL](https://graphql.org/)的变体。

您可以从中[dgraph.io/graphql](https://dgraph.io/graphql)找到开始使用GraphQL的说明{{% /notice %}}.

### 第1步： 运行 Dgraph

这里提供多种方式来安装和运行Dgraph, 你可以在这里[Download page](https://dgraph.io/downloads)找到他们. 

最简单的办法是用docker镜像 `dgraph/standalone`来启动运行 . 如果你没有用过docker，可以按照如下的介绍来安装[docker](https://docs.docker.com/install) 

_此独立image仅用于快速启动运行。不建议在生产环境中使用此方式。_

```sh
docker run --rm -it -p 8080:8080 -p 9080:9080 -p 8000:8000 -v ~/dgraph:/dgraph dgraph/standalone:{{< version >}}
```

命令将会启动一个运行着**Dgraph Alpha**、**Dgraph Zero**和**Ratel**的容器。
你会发现Dgraph数据存储在你电脑的*根目录*中名为*Dgraph*的文件夹中

{{% notice "tip" %}}通常，你可以用 `lru_mb` 参数来设置Dgraph alpha的内存大小。这只是Dgraph alpha的一个默认值，实际使用量将高于此值。建议将 lru_mb设为可用RAM的三分之一。对于独立安装，默认设置就是如此。
{{% /notice %}}


### 第2步： 运行 Mutation突变

{{% notice "tip" %}}
Dgraph运行后，您可以访问**Ratel**  [`http://localhost:8000`](http://localhost:8000)。它是基于浏览器的图形化工具，提供查询、变异等操作，并且可视化。

你既可以在命令行中执行curl，也可以通过在**Ratel**中提交突变数据来运行mutations和query等操作。{{% /notice %}}

#### 数据集
使用的数据集是一个电影图，所有的图节点包含导演、演员、流派 和 电影本身这几种类型.

#### 存储数据到图
mutation：改变Dgraph中存储的数据的操作,新增or修改。 Dgraph目前支持两种类型的数据变异：RDF和JSON。下面的RDF格式数据中存储了关于“星球大战”系列和《星际迷航》电影的前三个版本的信息。通过curl或ratelui的mutate选项卡运行改RDF数据，这将会在Dgraph中存储新增数据。

```sh
curl -H "Content-Type: application/rdf" "localhost:8080/mutate?commitNow=true" -XPOST -d $'
{
  set {
   _:luke <name> "Luke Skywalker" .
   _:luke <dgraph.type> "Person" .
   _:leia <name> "Princess Leia" .
   _:leia <dgraph.type> "Person" .
   _:han <name> "Han Solo" .
   _:han <dgraph.type> "Person" .
   _:lucas <name> "George Lucas" .
   _:lucas <dgraph.type> "Person" .
   _:irvin <name> "Irvin Kernshner" .
   _:irvin <dgraph.type> "Person" .
   _:richard <name> "Richard Marquand" .
   _:richard <dgraph.type> "Person" .

   _:sw1 <name> "Star Wars: Episode IV - A New Hope" .
   _:sw1 <release_date> "1977-05-25" .
   _:sw1 <revenue> "775000000" .
   _:sw1 <running_time> "121" .
   _:sw1 <starring> _:luke .
   _:sw1 <starring> _:leia .
   _:sw1 <starring> _:han .
   _:sw1 <director> _:lucas .
   _:sw1 <dgraph.type> "Film" .

   _:sw2 <name> "Star Wars: Episode V - The Empire Strikes Back" .
   _:sw2 <release_date> "1980-05-21" .
   _:sw2 <revenue> "534000000" .
   _:sw2 <running_time> "124" .
   _:sw2 <starring> _:luke .
   _:sw2 <starring> _:leia .
   _:sw2 <starring> _:han .
   _:sw2 <director> _:irvin .
   _:sw2 <dgraph.type> "Film" .

   _:sw3 <name> "Star Wars: Episode VI - Return of the Jedi" .
   _:sw3 <release_date> "1983-05-25" .
   _:sw3 <revenue> "572000000" .
   _:sw3 <running_time> "131" .
   _:sw3 <starring> _:luke .
   _:sw3 <starring> _:leia .
   _:sw3 <starring> _:han .
   _:sw3 <director> _:richard .
   _:sw3 <dgraph.type> "Film" .

   _:st1 <name> "Star Trek: The Motion Picture" .
   _:st1 <release_date> "1979-12-07" .
   _:st1 <revenue> "139000000" .
   _:st1 <running_time> "132" .
   _:st1 <dgraph.type> "Film" .
  }
}
' | python -m json.tool | less
```


{{% notice "tip" %}}
用curl来执行RDF/JSON文件做mutation操作时, 你可以使用curl的参数
`--data-binary @/path/to/mutation.rdf` 代替常用的 `-d $''`.
参数 `--data-binary` 允许curl忽略默认的URL-encoding编码.
{{% /notice %}}

### 第3步： 变更表结构

因为图数据只有设置为索引才可以查询，所以需要修改schema来添加索引，只有这样才可以做查询郭过滤和排序等操作.

```sh
curl "localhost:8080/alter" -XPOST -d $'
  name: string @index(term) .
  release_date: datetime @index(year) .
  revenue: float .
  running_time: int .

  type Person {
    name
  }

  type Film {
    name
    release_date
    revenue
    running_time
    starring
    director
  }
' | python -m json.tool | less
```

{{% notice "tip" %}}
在Ratel UI中提交schema时, 打开Schema页面,
点击 **Bulk Edit**, 随后把schema数据粘贴进来.
{{% /notice %}}

### 第4步： 运行查询

#### 获取所有的电影
运行如下query来获取所有的电影. 这个query查询返回所有有starring边的电影.

```sh
curl -H "Content-Type: application/graphql+-" "localhost:8080/query" -XPOST -d $'
{
 me(func: has(starring)) {
   name
  }
}
' | python -m json.tool | less
```

{{% notice "tip" %}}
你也可以在Ratel UI的query这个tab中提交GraphQL+- 语句来查询.
{{% /notice %}}

#### 获取"1980"年以后发行的所有电影
运行如下query语句来获取 "1980"年后发行的所有名字里有"Star Wars" 的电影. 可以在user interface里运行他来查看生成的一个图.

```sh
curl -H "Content-Type: application/graphql+-" "localhost:8080/query" -XPOST -d $'
{
  me(func:allofterms(name, "Star Wars")) @filter(ge(release_date, "1980")) {
    name
    release_date
    revenue
    running_time
    director {
     name
    }
    starring {
     name
    }
  }
}
' | python -m json.tool | less
```

输出

```json
{
  "data":{
    "me":[
      {
        "name":"Star Wars: Episode V - The Empire Strikes Back",
        "release_date":"1980-05-21T00:00:00Z",
        "revenue":534000000.0,
        "running_time":124,
        "director":[
          {
            "name":"Irvin Kernshner"
          }
        ],
        "starring":[
          {
            "name":"Han Solo"
          },
          {
            "name":"Luke Skywalker"
          },
          {
            "name":"Princess Leia"
          }
        ]
      },
      {
        "name":"Star Wars: Episode VI - Return of the Jedi",
        "release_date":"1983-05-25T00:00:00Z",
        "revenue":572000000.0,
        "running_time":131,
        "director":[
          {
            "name":"Richard Marquand"
          }
        ],
        "starring":[
          {
            "name":"Han Solo"
          },
          {
            "name":"Luke Skywalker"
          },
          {
            "name":"Princess Leia"
          }
        ]
      }
    ]
  }
}
```

以上即所有，总结：我们可执行4个步骤，初始化dgraph,插入一些数据,设置一个schema和查询数据.

## 你可以继续探索如下知识

- 可以在 [Clients]({{< relref "clients/index.md" >}}) 中查看
如何在自己的应用中与dgraph进行通信.
- 可以在 [Tour](https://dgraph.io/tour/) 中查看如何写dgraph的query语句.
- 更多的query查询方式可以在
[Query Language]({{< relref "query-language/index.md" >}}) 中找到.
- 可以查看 [Deploy]({{< relref "deploy/index.md" >}}) ，如果你是想运行一个dgraph的集群
  .

## 帮助中心

* 可以在 [discuss.dgraph.io](https://discuss.dgraph.io) 中讨论问题,
答疑解惑等.
* 可以使用[Github Issues](https://github.com/dgraph-io/dgraph/issues)
来提交你遇到的问题和想要的功能.
