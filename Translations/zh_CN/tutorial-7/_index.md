+++
title = "开始使用Dgraph--社会图谱上的模糊搜索。"
+++

**欢迎来到第七篇Dgraph入门教程。**

在[上一篇教程]({{< relref "tutorial-6/index.md" >}})中，我们学习了关于
在Dgraph中，通过对推文的建模，在社会图上建立高级文本搜索。
作为一个例子，我们使用 `fulltext` 和 `trigram` 索引来查询推文，并实现了对推文的查询。

在本教程中，我们将继续探索Dgraph的字符串查询功能
使用[第五节]({{< relref "tutorial-5/index.md" >}})中的twitter模型的能力。
和[第六节]({{< relref "tutorial-6/index.md" >}})教程。特别是：
我们将使用Dgraph的 `twitter username` 搜索功能来实现
模糊搜索函数。

附带的教程视频将于近期推出，敬请关注。
[我们的YouTube频道](https://www.youtube.com/channel/UCghE41LR8nkKFlR3IFTRO4w)。

---

在深入研究之前，我们先来回顾一下我们是如何在推文中建模的
前两篇教程。

{{% load-img "/images/tutorials/5/a-graph-model.jpg" "tweet model" %}}

我们用三条真实生活中的例子推文作为样本数据集，并存储了
他们在Dgraph中以上述图形为模型。

如果你跳过前面的教程，这里也是样本数据集。
复制下面的 mutation，进入 mutation 选项卡，点击运行。

```json
{
  "set": [
    {
      "user_handle": "hackintoshrao",
      "user_name": "Karthic Rao",
      "uid": "_:hackintoshrao",
      "authored": [
        {
          "tweet": "Test tweet for the fifth episode of getting started series with @dgraphlabs. Wait for the video of the fourth one by @francesc the coming Wednesday!\n#GraphDB #GraphQL",
          "tagged_with": [
            {
              "uid": "_:graphql",
              "hashtag": "GraphQL"
            },
            {
              "uid": "_:graphdb",
              "hashtag": "GraphDB"
            }
          ],
          "mentioned": [
            {
              "uid": "_:francesc"
            },
            {
              "uid": "_:dgraphlabs"
            }
          ]
        }
      ]
    },
    {
      "user_handle": "francesc",
      "user_name": "Francesc Campoy",
      "uid": "_:francesc",
      "authored": [
        {
          "tweet": "So many good talks at #graphqlconf, next year I'll make sure to be *at least* in the audience!\nAlso huge thanks to the live tweeting by @dgraphlabs for alleviating the FOMO😊\n#GraphDB ♥️ #GraphQL",
          "tagged_with": [
            {
              "uid": "_:graphql"
            },
            {
              "uid": "_:graphdb"
            },
            {
              "hashtag": "graphqlconf"
            }
          ],
          "mentioned": [
            {
              "uid": "_:dgraphlabs"
            }
          ]
        }
      ]
    },
    {
      "user_handle": "dgraphlabs",
      "user_name": "Dgraph Labs",
      "uid": "_:dgraphlabs",
      "authored": [
        {
          "tweet": "Let's Go and catch @francesc at @Gopherpalooza today, as he scans into Go source code by building its Graph in Dgraph!\nBe there, as he Goes through analyzing Go source code, using a Go program, that stores data in the GraphDB built in Go!\n#golang #GraphDB #Databases #Dgraph ",
          "tagged_with": [
            {
              "hashtag": "golang"
            },
            {
              "uid": "_:graphdb"
            },
            {
              "hashtag": "Databases"
            },
            {
              "hashtag": "Dgraph"
            }
          ],
          "mentioned": [
            {
              "uid": "_:francesc"
            },
            {
              "uid": "_:dgraphlabs"
            }
          ]
        },
        {
          "uid": "_:gopherpalooza",
          "user_handle": "gopherpalooza",
          "user_name": "Gopherpalooza"
        }
      ]
    }
  ]
}
```

_注：如果你是Dgraph的新手，而且这是你第一次运行 mutation ，我们强烈建议在继续之前阅读[系列的第一篇教程]({{< relref "tutorial-1/index.md" >}})。_

现在你应该有一个包含推文、用户和标签的图表。
并准备好让我们去探索。

{{% load-img "/images/tutorials/5/x-all-tweets.png" "tweet graph" %}}

_注：如果你很想知道我们是如何在Dgraph中建模的，请参考[第五篇教程]({{< relref "tutorial-5/index.md" >}})。_

在向大家展示模糊搜索的操作之前，我们先来了解一下什么是模糊搜索以及它的工作原理。

## 模糊搜索

在产品或用户名上提供搜索功能，如果不存在完全匹配，则需要搜索与字符串最接近的匹配。
这个功能可以帮助你获得相关的结果，即使有一个错别字，或者用户没有根据它存储的确切名称进行搜索。
这正是模糊搜索的作用：它比较字符串值，并返回最近的匹配值。
因此，它非常适合我们在`twitter用户名`上实现搜索的用例。

模糊搜索的功能是基于Dgraph中存储的用户名的值与搜索字符串之间的`Levenshtein距离`。

[`Levenshtein距离`](https://en.wikipedia.org/wiki/Levenshtein_distance)是一个定义两个字符串之间紧密程度的度量。
两个词之间的 `Levenshtein距离` 是将一个词改为另一个词所需的最小单字符编辑次数（插入、删除或替换）。

例如，字符串 `book` 和 `back` 之间的 `Levenshtein距离` 是2。
2的值是合理的，因为通过改变两个字符，我们把 `book` 改为 `back`。

现在你已经了解了什么是模糊搜索，以及它能做什么。
接下来，我们来学习如何在Dgraph中的字符串断定上使用它。

## 在Dgraph中实现模糊搜索

要在Dgraph中对字符串谓词使用模糊搜索，首先要设置`trigram`索引。

进入 "模式 "选项卡，在 `user_name` 谓词上设置 `trigram` 索引。

在设置了 `user_name` 谓词的 `trigram` 索引后，你可以使用Dgraph的
内置函数 `match` 来运行模糊搜索查询。

以下是`match`函数的语法：`match(predicate, search string, distance)`。

[match函数](https://dgraph.io/docs/query-language/#fuzzy-matching)需要三个参数。

1. 用于查询的字符串谓词的名称。
2. 用户提供的搜索字符串
3. 一个整数，表示前两个参数之间的最大 `Levenshtein距离` 。
这个值应该大于0。例如，当有一个8的整数时，返回谓词。
距离值小于或等于8的。

对 `distance` 参数使用更大的值可能会匹配更多的字符串谓词。
但它的结果也不太准确。

在使用`match`函数之前，我们先来获取数据库中存储的用户名列表。

```graphql
{
    names(func: has(user_name)) {
        user_name
    }
}
```

{{% load-img "/images/tutorials/7/e-names.png" "tweet graph" %}}

从结果可以看出，我们有四个用户名：`Gopherpalooza`。
`Karthic Rao`、`Francesc Campoy` 和`Dgraph Labs`。

首先，我们将 `Levenshtein 距离` 参数设置为3，我们希望看到Dgraph返回的是
所有与所提供的搜索字符串相距三个或更少距离的`username`谓词。

然后，我们将第二个参数，即用户提供的搜索字符串设置为`graphLabs`。

进入查询选项卡，粘贴下面的查询并点击运行。

```graphql
{
    user_names_Search(func: match(user_name, "graphLabs", 3)) {
        user_name
    }
}
```

{{% load-img "/images/tutorials/7/h-one.png" "first query" %}}

我们得到了一个积极的匹配!
因为搜索字符串 `graphLabs` 与谓词的距离为2。
值的 `Dgraph Labs`，所以我们在搜索结果中看到它。

如果您有兴趣了解更多关于如何寻找列文斯丁的距离
两个字符串之间，[这里有一个有用的网站](https://planetcalc.com/1721/)。

让我们再次运行上面的查询，但这次我们将使用搜索字符串`graphLab`代替。
进入查询选项卡，粘贴下面的查询并点击运行。 

```graphql
{
    user_names_Search(func: match(user_name, "graphLab", 3)) {
        user_name
    }
}
```

{{% load-img "/images/tutorials/7/i-two.png" "first query" %}}

我们仍然在 `user_name` 谓词中得到了一个积极的匹配，其值为 `Dgraph Labs` ！这是因为搜索字符串 `graphLab` 与谓词之间有三个距离。
这是因为搜索字符串 `graphLab` 与谓词的距离为3，所以我们在搜索结果中看到它。

在这种情况下，搜索字符串 `graphLab` 和 `Dgraph Labs` 之间的 `Levenshtein距离` 为3，因此匹配。

在查询的最后一次运行中，让我们将搜索字符串改为`Dgraph`，但保留以下内容
Levenshtein 距离为3。

```graphql
{
    user_names_Search(func: match(user_name, "Dgraph", 3)) {
        user_name
    }
}
```

{{% load-img "/images/tutorials/7/j-three.png" "first query" %}}

现在，你不再看到Dgraph Labs出现在搜索结果中，因为距离太远了。
词 `Dgraph` 和 `Dgraph Labs` 之间大于3.但根据正常
的理由，你自然会期望Dgraph Labs出现在搜索中。
结果，而使用Dgraph作为搜索字符串。

这是基于 `Levenshtein距离` 算法的模糊搜索的缺点之一。
随着距离参数值的降低，模糊搜索的有效性也会降低。
并且它还会随着字符串谓词中包含的字数的增加而减少。

因此，不建议在字符串谓词上使用模糊搜索，因为这些字符串谓词
可以包含许多词，例如，存储`blog posts`值的谓词。
`bio`、`product description`等。因此，使用模糊搜索的理想人选是
谓词，如 `names`、`zipcodes`、`places`，其中字符串中的字数为
谓词一般会在1-3之间。

另外，根据使用情况，调整 `距离` 参数对以下情况至关重要
模糊搜索的有效性。

## 模糊搜索评分，因为是你要求的。

在Dgraph，我们致力于提高分布式Graph的全方位能力。
数据库。作为我们最近改进数据库功能的努力之一，我们采取了以下措施
注意到其中一人的[Github上的请求](https://github.com/dgraph-io/dgraph/issues/3211)。
我们的社区成员整合了一个基于`tf-idf`分数的文本搜索。这种整合将
进一步增强了Dgraph的搜索功能。

我们已经在产品路线图中优先解决了这个问题。
我们想借此机会向我们的帮助我们把产品做得更好社区的用户表示感谢
。

## 总结

模糊搜索是一种简单而有效的搜索技术，适用于广泛的用例。
除了现有的查询和搜索字符串谓词的特征外，还增加了
基于`tf-idf`的搜索将进一步提高Dgraph的能力。

这标志着我们探索字符串索引及其查询的三个连续教程的结束。
使用推文的图模型。

查看我们下一个入门系列教程[这里]({{< relref "tutorial-8/index.md" >}})。

记得点击下面的 "加入我们的社区 "按钮，并订阅我们的通讯。
以获得最新的教程，直接发送到您的收件箱。

## 需要帮助

* 请使用[discuss.dgraph.io](https://discuss.dgraph.io)进行提问、功能请求和讨论。
* 如果你遇到bug或有功能需求，请使用[Github Issues](https://github.com/dgraph-io/dgraph/issues)。
