# 向量化数据库与openAI的关联逻辑

## 开始说明：

开始前 需要先了解 qdrant 数据库，以及 openai embedding

## 相关文档：

[openai embedding文档](https://platform.openai.com/docs/guides/embeddings/what-are-embeddings)

[qdrant API 文档 所有后端均可使用](https://qdrant.github.io/qdrant/redoc/index.html#tag/points/operation/upsert_points)

[qdrant API Quick-start](https://qdrant.tech/documentation/quick-start/#add-points)

## 所用工具：

openai embedding

## 所用数据库：

`qdrant`

## 插入逻辑

1. 使用 openAI 的嵌入方法 (https://api.openai.com/v1/embeddings) 将数据转化为向量

2. 获取到嵌入方法返回的 embedding 的值，插入 points 到 某个 collection 中

3. 外部准备部分数据 准备写入到 payload 中，可以如下面的例子

   ```json
   [
       {
           "title": "问题1",
       	"text": "回答1"
       },
       {
           "title": "问题2",
       	"text": "回答2"
       }
   ]
   ```

   以上例子的 `title` 和 `text` 可以随意更改，但后面的数据必须规范 这里我把 `title` 作为 问题，`text`作为回答

   后面的point里的内容应该是以下的状态

   ```json
   {
       "id": 1,
       "vector":[
           // 返回的embedding 
       ],
       payload: {
           "title": "问题1",
       	"text": "回答1"
       }
   }
   ```

## 查询逻辑

1. 还是使用 openAI 的嵌入方法 (https://api.openai.com/v1/embeddings) 将数据转化为向量

2. 获取到嵌入方法返回的 embedding 的值，查询同个collection里的内容（将 embedding 代入 vector 中），配置好 limit，建议为3，配置 params ，可以将 `hnsw_ef` 配置为128（准确值），`exact` 默认为 false（无近似搜索/精确搜索/无模糊搜索）

3. 循环遍历搜索出来的内容，如果内容与提问相符则让gpt回答，前面可以使用 system,assistant,user 来对gpt的下一步回答进行一个微调

   1. 关键提示词：

      ```text
      使用以下段落来回答问题，如果段落内容不相关就返回未查到相关信息："1.问题1:回答1\n2.问题2:回答2"
      ```

## 参考仓库

[document.ai](https://github.com/GanymedeNil/document.ai.git)



