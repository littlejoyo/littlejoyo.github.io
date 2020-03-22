---
title: Go:内置json包的应用
date: 2020-03-21 15:50:25
categories:
  - [Go]
tag:
    - Go
    - json
---

- Go内置了对json处理的包，提供json对象和Go对象的转换方法

- Marshal()：Go数据对象 -> json数据

- UnMarshal()：Json数据 -> Go数据对象

<!-- more -->

> Github issues:https://github.com/littlejoyo/Blog/issues/

# 1.Go json包

```go
func Marshal(v interface{}) ([]byte, error)
func Unmarshal(data []byte, v interface{}) error
```

- Marshal() : 将Go数据对象转化为json数据

- UnMarshal()：将Json数据转化为Go数据对象

# 2.生成json数据

Marshal()和MarshalIndent()函数可以将数据封装成json数据。

- struct、slice、array、map都可以转换成json

- struct转换成json的时候，只有**字段首字母大写**的才会被转换

- map转换的时候，key必须为string

- 封装的时候，如果是指针，会追踪指针指向的对象进行封装


## 2.1 struct转化json

下面是一个Go结构体，要将这段struct数据转换成json，只需使用Marshal()即可。

```go
type Student struct {
    Id      int
    Name    string
    Age     int
}
```

利用Marshal()转换为json数据

```go
stu := &Student{1, "Joyo", "18"}
b, err := json.Marshal(stu)
if err != nil {
    fmt.Println(nil)
}
```

- Marshal()返回的是一个`[]byte`类型，现在变量b就已经存储了一段[]byte类型的json数据

- 将`[]byte`转化为string，`fmt.Println(string(b))`

输出结果：

```json
{"Id":1,"Name":"Joyo","Age":"18"}
```

- 可以在封装成json的时候进行"美化"

- 使用MarshalIndent()即可自动添加前缀(前缀字符串一般设置为空)和缩进


```go
c,err := json.MarshalIndent(stu,"","\t")
if err != nil {
    fmt.Println(nil)
}
fmt.Println(string(c))
```

输出结果：

```json
{
    "Id": 1,
    "Name": "Joyo",
    "Age": "18"
}
```

## 2.2 map和slice转换为json

- 除了struct，array、slice、map结构都能解析成json

- 但是map解析成json的时候，key必须只能是string，这是json语法要求的

```go
// slice -> json
s := []string{"a", "b", "c"}
d, _ := json.MarshalIndent(s, "", "\t")
fmt.Println(string(d))

// map -> json
m := map[string]string{
    "a":"aa",
    "b":"bb",
    "c":"cc",
}
e,_ := json.MarshalIndent(m,"","\t")
fmt.Println(string(e))
```

输出结果：

```json
[
    "a",
    "b",
    "c"
]
{
    "a": "aa",
    "b": "bb",
    "c": "cc"
}
```

# 3.使用struct tag辅助构建json

- struct能被转换的字段都是首字母大写的字段

- 但如果想要在json中使用小写字母开头的key，可以使用struct的tag来辅助反射

```go
type Student struct {
    Id      int    `json:"ID"`
    Name string    `json:"name"`
    Age  string    `json:"age"`
}

stu := &Post{
    1,
    "Joyo",
    "18",
    }

p, _ := json.MarshalIndent(stu, "", "\t")
fmt.Println(string(p))
```

输出结果：

```json
{
    "ID": 2,
    "name": "Joyo",
    "age": "18",
}
```

> 根据输出结果可知，上面的json字段首字母跟tag设置的一致

使用struct tag的时候，几个注意点：

- 1.tag中标识的名称将称为json数据中key的值

- 2.tag可以设置为`json:"-"`来表示本字段不转换为json数据，即使这个字段名首字母大写

- 3.如果想要json key的名称为字符"-"，则可以特殊处理`json:"-,"`，也就是加上一个逗号

- 4.如果tag中带有`,omitempty`选项，那么如果这个字段的值为0值，即false、0、""、nil等，这个字段将不会转换到json中

- 5.如果字段的类型为bool、string、int类、float类，而tag中又带有`,string`选项，那么这个字段的值将转换成json字符串

示例：

```go
type Post struct {
    Id      int      `json:"ID,string"`
    Content string   `json:"content"`
    Author  string   `json:"author"`
    Label   []string `json:"label,omitempty"`
}
```

# 4.解析json数据到已知结构的struct

- json数据可以解析到struct或空接口interface{}中(也可以是slice、map等)。

例如，下面是一段json数据：

```json
{
    "id": 1,
    "content": "hello world",
    "author": {
        "id": 2,
        "name": "userA"
    },
    "published": true,
    "label": [],
    "nextPost": null,
    "comments": [{
            "id": 3,
            "content": "good post1",
            "author": "userB"
        },
        {
            "id": 4,
            "content": "good post2",
            "author": "userC"
        }
    ]
}
```

分析下这段json数据：

- 顶层的大括号表示是一个匿名对象，映射到Go中是一个struct，假设这个struct名称为Post

- 顶层大括号里的都是Post结构中的字段，这些字段因为都是json数据，所以必须都首字母大写，同时设置tag，tag中的名称小写

- 其中author是一个子对象，映射到Go中是另一个struct，在Post中这个字段的名称为Author，假设名称和struct名称相同，也为Author

- label是一个数组，映射到Go中可以是slice，也可以是array，且因为json array为空，所以Go中的slice/array类型不定，比如可以是int，可以是string，也可以是`interface{}`，对于这里的示例来说，我们知道标签肯定是string

- nextPost是一个子对象，映射到Go中是一个struct，但因为json中这个对象为null，表示这个对象不存在，所以无法判断映射到Go中struct的类型。

- comment是子对象，且是数组包围的，映射到Go中，是一个slice/array，slice/array的类型是一个struct

分析之后，对应地去构建struct和struct的tag就很容易了。如下，是根据上面分析构建出来的数据结构：

```go
type Post struct {
    ID        int64         `json:"id"`       
    Content   string        `json:"content"`  
    Author    Author        `json:"author"`   
    Published bool          `json:"published"`
    Label     []string      `json:"label"`    
    NextPost  *Post         `json:"nextPost"` 
    Comments  []*Comment    `json:"comments"` 
}

type Author struct {
    ID   int64  `json:"id"`  
    Name string `json:"name"`
}

type Comment struct {
    ID      int64  `json:"id"`     
    Content string `json:"content"`
    Author  string `json:"author"` 
}
```

- 前面在介绍构建json数据的时候说明过，指针会进行追踪，所以这里反推出来的struct中使用指针类型是没问题的。

- 于是，解析上面的json数据到Post类型的对象中，获取到对象就能获取到数据。

- 假设这个json数据存放在a.json文件中。代码如下：

```go
func main() {
    // 打开json文件
    fh, err := os.Open("a.json")
    if err != nil {
        fmt.Println(err)
        return
    }
    defer fh.Close()
    // 读取json文件，保存到jsonData中
    jsonData, err := ioutil.ReadAll(fh)
    if err != nil {
        fmt.Println(err)
        return
    }
    
    var post Post
    // 解析json数据到post中
    err = json.Unmarshal(jsonData, &post)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(post)
    fmt.Println(post.id)
    fmt.Println(post.content)
    fmt.Println(post.author.name)
    fmt.Println(post.published)
}
```

输出结果：

```go
{1 hello world {2 userA} true [] <nil> [0xc042072300 0xc0420723c0]}
1
hello world
userA
true
```

# 5.解析json到interface(结构未知)

- 上面是已知json数据结构的解析方式，如果json结构是未知的或者结构可能会发生改变的情况，则解析到struct是不合理的。

- 这时可以解析到空接口`interface{}`或`map[string]interface{}`类型上，这两种类型的结果是完全一致的。

- 解析到`interface{}`上时，Go类型和JSON类型的对应关系如下:

```go
 JSON类型             Go类型                
---------------------------------------------
JSON objects    <-->  map[string]interface{} 
JSON arrays     <-->  []interface{}          
JSON booleans   <-->  bool                   
JSON numbers    <-->  float64                
JSON strings    <-->  string                 
JSON null       <-->  nil     
```

例如：读取json并解析到未知结构的interface

```go
func main() {
    // 读取json文件
    fh, err := os.Open("a.json")
    if err != nil {
        fmt.Println(err)
        return
    }
    defer fh.Close()
    jsonData, err := ioutil.ReadAll(fh)
    if err != nil {
        fmt.Println(err)
        return
    }
    
    // 定义空接口接收解析后的json数据
    var unknown interface{}
    // 或：map[string]interface{} 结果是完全一样的
    err = json.Unmarshal(jsonData, &unknown)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(unknown)
}
```

输出结果：

```go
map[nextPost:<nil> comments:[map[id:3 content:good post1
author:userB] map[id:4 content:good post2 author:userC]]
id:1 content:hello world author:map[id:2 name:userA] published:true label:[]]
```

- 上面输出了map结构。这是显然的，因为类型对应关系中已经说明了，json object解析到Go interface的时候，对应的是map结构。

- 如果将上面输出的结构进行一下格式化，得到的将是类似下面的结构：

```json
map[
    nextPost:<nil>
    comments:[
        map[
            id:3
            content:good post1
            author:userB
        ]
        map[
            id:4
            content:good post2
            author:userC
        ]
    ]
    id:1
    content:hello world
    author:map[
        id:2
        name:userA
    ]
    published:true
    label:[]
]
```

- 获取到了map结构后我们就可以通过key来获取value。

- 可以从这个map中去判断类型、取得对应的值。

- 但是如何判断类型？可以使用类型断言：

```go
func main() {
    // 读取json数据
    fh, err := os.Open("a.json")
    if err != nil {
        fmt.Println(err)
        return
    }
    defer fh.Close()
    jsonData, err := ioutil.ReadAll(fh)
    if err != nil {
        fmt.Println(err)
        return
    }
    
    // 解析json数据到interface{}
    var unknown interface{}
    err = json.Unmarshal(jsonData, &unknown)
    if err != nil {
        fmt.Println(err)
        return
    }

    // 进行断言，并switch匹配
    m := unknown.(map[string]interface{})
    for k, v := range m {
        switch vv := v.(type) {
        case string:
            fmt.Println(k, "type: string\nvalue: ", vv)
            fmt.Println("------------------")
        case float64:
            fmt.Println(k, "type: float64\nvalue: ", vv)
            fmt.Println("------------------")
        case bool:
            fmt.Println(k, "type: bool\nvalue: ", vv)
            fmt.Println("------------------")
        case map[string]interface{}:
            fmt.Println(k, "type: map[string]interface{}\nvalue: ", vv)
            for i, j := range vv {
                fmt.Println(i,": ",j)
            }
            fmt.Println("------------------")
        case []interface{}:
            fmt.Println(k, "type: []interface{}\nvalue: ", vv)
            for key, value := range vv {
                fmt.Println(key, ": ", value)
            }
            fmt.Println("------------------")
        default:
            fmt.Println(k, "type: nil\nvalue: ", vv)
            fmt.Println("------------------")
        }
    }
}
```

输出结果：

```go
comments type: []interface{}
value:  [map[id:3 content:good post1 author:userB] map[author:userC id:4 content:good post2]]
0 :  map[id:3 content:good post1 author:userB]
1 :  map[id:4 content:good post2 author:userC]
------------------
id type: float64
value:  1
------------------
content type: string
value:  hello world
------------------
author type: map[string]interface{}
value:  map[id:2 name:userA]
name :  userA
id :  2
------------------
published type: bool
value:  true
------------------
label type: []interface{}
value:  []
------------------
nextPost type: nil
value:  <nil>
------------------
```

# 6.解析、创建json流

除了可以直接解析、创建json数据，还可以处理流式数据。

- type Decoder解码json到Go数据结构
- type Encoder编码Go数据结构到json

示例：

```go
const jsonStream = `
    {"Name": "Ed", "Text": "Knock knock."}
    {"Name": "Sam", "Text": "Who's there?"}
    {"Name": "Ed", "Text": "Go fmt."}
    {"Name": "Sam", "Text": "Go fmt who?"}
    {"Name": "Ed", "Text": "Go fmt yourself!"}
`
type Message struct {
    Name, Text string
}
dec := json.NewDecoder(strings.NewReader(jsonStream))
for {
    var m Message
    if err := dec.Decode(&m); err == io.EOF {
        break
    } else if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%s: %s\n", m.Name, m.Text)
}
```

输出结果：

```go
Ed: Knock knock.
Sam: Who's there?
Ed: Go fmt.
Sam: Go fmt who?
Ed: Go fmt yourself!
```

再例如，从标准输入读json数据，解码后删除名为Name的元素，最后重新编码后输出到标准输出。

```go
func main() {
    dec := json.NewDecoder(os.Stdin)
    enc := json.NewEncoder(os.Stdout)
    for {
        var v map[string]interface{}
        if err := dec.Decode(&v); err != nil {
            log.Println(err)
            return
        }
        for k := range v {
            if k != "Name" {
                delete(v, k)
            }
        }
        if err := enc.Encode(&v); err != nil {
            log.Println(err)
        }
    }
}
```

# 微信公众号

> 扫一扫关注Joyo说公众号，共同学习和研究开发技术。

![weixin-a](https://i.loli.net/2020/01/11/HQT8NMsmDhIkXZv.png)