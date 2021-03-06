# restdocs详细教程-进阶篇1

本章内容：

1. 进一步了解 spring-restdocs的使用
2. 完成一个常见api文档的排版效果
3. 侧边栏的配置
4. 自定义文档内容的编写
5. post参数提交
6. 请求和响应参数的描述展示排版


## 实现左侧目录功能

新增 src/docs/asciidoc/fun1_summary.adoc 文件，内容如下

```
= API列表
:toc: left

.fun api列表

operation::fun1[]
```

注意顶部的 `:toc: left` left后面有空格，这个就是 asciidoctor 语法；该配置官网链接
https://asciidoctor.org/docs/user-manual/#setting-attributes-on-a-document

再次运行 gradle asciidoctor 命令；打开 build/asciidoc/html5/fun1_summary.html 文件
![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_111635.png)

注意：如果你发现出现了如下的异常；

```
(ArgumentError) asciidoctor: FAILED: F:/dev/project/mrcode/example/spring/restdocs/spring-restdocs-example/src/docs/asciidoc/fun1_summary.adoc: Failed to load AsciiDoc document - asciidoctor: FAILED: <stdin>: Failed to load AsciiDoc document - invalid byte sequence in UTF-8
```

那么你的内容有可能是这样的

```
= API列表
:toc: left

== fun api列表  

operation::fun1[]
```


`== fun api列表`     这里的问题，我目前不太清楚为什么不能在该文件中写== 加中文，但是在后面的include其他文件的时候  其他文件中却可以写

目录已经出来了，但是效果并不好看。继续配置

## 左侧目录优化

先来看一张效果图
![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_112129.png)

这个图和我们的目标差不多了；

* 有侧边目录
* 请求地址
* 请求参数 （该api中没有参数所以没有显示）
* 响应参数

新增两个文件；目录结构如下


```bash
|-- docs
    |- asciidoc
        |- api_list.adoc
        |- fun1.adoc
        |- fun1_summary.adoc
        |- fun1_summary_2.adoc
```

api_list.adoc 入口文件，汇总所有需要到一个html中的配置

```
= API列表
:toc: left

include::./fun1_summary_2.adoc[]
```

include 指令，不用多说，引用了当前目录下的fun2自定义内容版本文件


fun1_summary_2.adoc 对fun1的内容进行定制

```
== fun1 API

=== API地址

include::{snippets}/fun1/curl-request.adoc[]

.HTTP request:

include::{snippets}/fun1/http-request.adoc[]

.HTTP response:

include::{snippets}/fun1/http-response.adoc[]
```

> {snippets} 忘记哪里来的了。好像是 转换插件里面的预定变量。指向了测试用例代码片断所在目录

上面文件内容百分之98的内容都是根据 asciidoctor 官网文档 编写

## 为 post 请求输出文档

![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_123049.png)

效果图如上；

**定义controller**

```java
    /**
     * 接收一个对象，填充部分数据后 再返还该对象
     * @param book
     * @return
     */
    @PostMapping("/fun3")
    public Book fun3(Book book) {
        book.setAuthors(new String[]{"张三丰", "张4丰", "张5丰"});
        return book;
    }
```

编写测试用例

```java
    /**
     * 增加请求参数 和 响应自定义pojo对象
     * @throws Exception
     */
    @Test
    public void fun3() throws Exception {
//        Book book = new Book();
//        book.setName("《spring resdocs进阶篇》");
//        book.setPrice(13.2);

        mockMvc.perform(
                MockMvcRequestBuilders.post("/fun3")
                        // 表单提交是不能提交一个对象的，只能提交kv的形式
                        .param("name", "《spring resdocs进阶篇》")
                        .param("price", "13.2")
                        // 请求类型也就是 平时开发的 form表单提交
                        // jquery ajax 提交的form表单
                        // contentType 就是这个
//                        .contentType(MediaType.APPLICATION_FORM_URLENCODED)
        )
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print())  // 打印请求响应详细日志，可以在控制台看到详细的日志信息
                .andDo(MockMvcRestDocumentation
                               .document("fun3"));
    }
```
增加 src/docs/asciidoc/fun3.adoc 文件；内容和 src/docs/asciidoc/fun1_summary_2.adoc 中的类似；

在 src/docs/asciidoc/api_list.adoc 文件中增加 对 fun3.adoc 的引用

api_list.adoc 内容如下

```
= API列表
:toc: left

include::./fun1_summary_2.adoc[]

include::./fun3.adoc[]
```

仔细看下，其实现在做的 对于adoc的操作并不多了，只是 restdocs api 和 测试用例的编写比较多了；

但是这个api没有请求参数的字段描述，响应字段描述也没有，那么接下来继续

## 给文档添加请求响应字段的描述
先来看效果图

![](/assets/image/spring/spring_restdocs_asciidoctor/snipaste_20180720_130600.png)


首先增加测试用例，并且需要在测试用例中对请求响应信息的描述

```java
    /**
     * 增加请求参数 和 响应自定义pojo对象; 对post参数增加请求响应字段的描述
     * 对应的官网文档地址
     * https://docs.spring.io/spring-restdocs/docs/2.0.2.RELEASE/reference/html5/#documenting-your-api-request-parameters
     * @throws Exception
     */
    @Test
    public void fun3_1() throws Exception {
        mockMvc.perform(
                MockMvcRequestBuilders.post("/fun3")
                        .param("name", "《spring resdocs进阶篇》")
                        .param("price", "13.2")
        )
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcRestDocumentation
                               .document("fun3_1",
                                         // post 的 APPLICATION_FORM_URLENCODED 和 get url中的参数 都可以用该方法获取
                                         RequestDocumentation.requestParameters(
                                                 RequestDocumentation.parameterWithName("name").description("书籍名称"),
                                                 RequestDocumentation.parameterWithName("price").description("书籍价格")),
                                         PayloadDocumentation.responseFields(
                                                 PayloadDocumentation.fieldWithPath("name").description("书籍名称"),
                                                 PayloadDocumentation.fieldWithPath("price").description("书籍价格"),
                                                 // 这个api对于响应字段描述不完全的话会报错的
                                                 // 可以尝试注释掉 authors的描述看看效果
                                                 PayloadDocumentation.fieldWithPath("authors").description("书籍价格")
                                         )
                               )
                );
    }
```

注意：这个api如果对于响应的字段没有描述完整将会报错，比如 把 响应字段的 authors 描述去掉，则会报错

```java
org.springframework.restdocs.snippet.SnippetException: The following parts of the payload were not documented:
{
  "authors" : [ "张三丰", "张4丰", "张5丰" ]
}
```

运行测试用例之后，比之前的方法多生成了2个代码片断，可以看出来了上面api配置的会生成对应的 adoc 文件；
我们要做的就是在自定义排版里面把这两个文件引用进来

```
request-parameters.adoc
response-fields.adoc
```

src/docs/asciidoc/fun3_1.adoc

```
== fun3_1 API

.AP地址

include::{snippets}/fun3_1/curl-request.adoc[]

.HTTP request

include::{snippets}/fun3_1/http-request.adoc[]

.Request fields
include::{snippets}/fun3_1/request-parameters.adoc[]

.HTTP response

include::{snippets}/fun3_1/http-response.adoc[]

.Response fields
include::{snippets}/fun3_1/response-fields.adoc[]

```

本节 restdocs详细教程-进阶篇1 结束；下一节 进阶篇2；继续深入


