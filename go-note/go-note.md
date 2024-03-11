# Gin error question

记录在学习go web gin时遇到的错误与理解



### go-redis

```go
result, err := conn.Client.Do(conn.Ctx, "get", any1).Result()
```

```go
func (c *Client) Do(ctx context.Context, args ...interface{}) *Cmd {
    cmd := NewCmd(ctx, args...)
    _ = c.Process(ctx, cmd)
    return cmd
}
```

```go
type Cmd struct {
    baseCmd

    val interface{}
}
```

这里的val记录的是传过来的值和类型 baseCMD是基础命令

使用Result方法返回接口类型的值  和 baseCMD中错误


```go
type baseCmd struct {
    ctx    context.Context
    args   []interface{}
    err    error
    keyPos int8

    _readTimeout *time.Duration
}
```





### init

最简单的例子

```go
type Login struct {
    User     string
    Password string
}


func main() {
    r := gin.Default()
    r.GET("/paramExm1", exm1)
}

func exm1(c *gin.Context) {
    var login Login
    err := c.ShouldBind(&login)
    
    if err == nil {
       fmt.Printf("login info: %#v\n", login)
       c.JSON(http.StatusOK, gin.H{
          "user":     login.User,
          "password": login.Password,
       })
    
    } else {
       c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
    }
}
```

相当于**controller**层

**ShouldBind**方法类似于**springboot**中接受对象作为参数

就是将接收到的json请求信息 转化为**括号内结构体格式**

ifelse就是处理请求成功与否的关系了



### 数据库连接三格式

```go
	db, err := gorm.Open("mysql", "root:123456@/sql_test")
```

**直接粘贴栈溢出**

You might want to specify the protocol (like '`tcp`'), instead of `localhost` directly.
See [those examples](https://github.com/go-sql-driver/mysql#examples):

```java
user:password@tcp(localhost:5555)/dbname
```

In your case:

```java
username@tcp(localhost)/my_db
```

------

Note, if you use the default protocol (`tcp`) and host (`localhost:3306`), this could be rewritten as

```bash
user:password@/dbname
```







### missing port in address

![image-20240116230721091](./image/image-20240116230721091.png)

`gin.Default.Run()`内端口或网址有没写对

看看是不是漏了个 **:**

```go
r := gin.Default()
err := r.Run(":8080")
```





### missing go.sum entry for module providing package  < package name >

![image-20240116230721091](./image/image-20240116230721091.png)

go.mod 中文件依赖的配置问题

可能是版本不兼容 或者多个第三方包引入依赖冲突 使用 `go mod tidy` 清除现有缓存并重新下载第三方包

虽然但是还是没用 StackOverflow里找到解法  `go mod tidy -e`

![image-20240118002128606](C:\Users\chenz\AppData\Roaming\Typora\typora-user-images\image-20240118002128606.png)

[go mod tidy详细说明]: https://go.dev/ref/mod#go-mod-tidy







### git push等操作失败

`unable to access 'https://github.com/ivorysillegalss/mini-gpt.git/': Failed to connect to github.com port 443 after 21063 ms: Couldn't connect to server`

修改git配置：（其中的10809改为你电脑的端口号）

```bash
git config --global http.proxy http://127.0.0.1:10809
git config --global https.proxy http://127.0.0.1:10809
```

clash默认修改的端口号为7890







### go项目启动失败 (权限)

nohup: failed to run command '/home/go/main': Permission denied nohup: failed to run command '/home/go/main': Permission denied nohup: failed to run command '/home/go/main': Permission denied load config from file failed, 

文件权限不够 运行不了项目

```bash
chmod +x /home/go/main
```

使用以上命令来授权





### err:open ./conf/config.ini: no such file or directory panic: 

找不到配置文件





### 运行时错误 数据库错误 redis错误 有可能是没读到配置文件 导致数据库初始化失败

runtime error: invalid memory address or nil pointer dereference [signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x84e120] goroutine 1 [running]: mini-gpt/setting.initLog() D:/GoLand/GoProject/mini-gpt/setting/initialzation.go:41 +0x140 mini-gpt/setting.GetLogger(...) 

D:/GoLand/GoProject/mini-gpt/setting/initialzation.go:62 main.main() D:/GoLand/GoProject/mini-gpt/main.go:15 +0x34 time="2024-01-30T16:15:15+08:00" level=error msg="open ./logs/logfile.log: no such file or directory" time="2024-01-30T16:15:15+08:00" level=error msg="init database failed, err:%v\n"







### 回显主键

```go

type BotToStruct struct {
	BotId      int  `gorm:"primaryKey column:bot_id"` // 明确指定BotId为主键
	IsDelete   bool `gorm:"column:is_delete"`         // 导出字段，并可指定列名
	IsOfficial bool `gorm:"column:is_official"`       // 导出字段，并可指定列名
}

// 写入映射结构体对象中
func writeBotToStruct(isOfficial bool) *BotToStruct {
	return &BotToStruct{
		isDelete:   false,
		isOfficial: isOfficial,
	}
}

func CreateBot(isOfficial bool) (int, error) {
    botToStruct := writeBotToStruct(isOfficial)
//    获得一个botToStruct返回一个初始化后的
    if err := dao.DB.Table("bot").Create(botToStruct).Error; err != nil {
       return -1, err
    }
    return botToStruct.BotId, nil 
}
```

gorm中 如果需要使用到回显主键的功能 通过create之后 会自动返回到传入的对象中 可以看到我上方并没有人为设置botId的值 下方直接get是能获取到的 但是上方获取不到 估计是gorm的bug

	ID         int  `gorm:"primaryKey column:bot_id"` // 明确指定BotId为主键

botToStruct中主键改成默认的主键名字ID就可以了



**另**

Q：当使用save保存一个对象到对应的mysql数据库当中 而这个对象id字段放的是零值 若想回显主键 它会不会把零值作为id就直接赋值进数据库当中

A：在使用Go语言的GORM库进行数据库操作时，如果你在保存对象到MySQL数据库时使用了零值（例如，对于整型的`ID`字段，零值就是`0`）作为对象的`ID`，GORM的默认行为取决于你的`ID`字段是如何配置的。

**GORM利用`ID`字段来判断一个对象是否应该被插入为新记录还是更新现有记录。如果`ID`的值是其类型的零值（对于整数，这个值是`0`），GORM会将这个操作视为插入新记录。对于插入操作，如果`ID`字段被设置为自增（这是MySQL表中常见的配置），数据库会自动生成一个唯一的`ID`值，而不是使用提供的零值。**





### BindJSON和指针传递


正确
```go
	var bot dto.UpdateBotDTO
	resultDTO := dto.ResultDTO{}
	if err := c.BindJSON(&bot); err != nil {
		// 检查参数解析是否出错
		c.JSON(http.StatusBadRequest, resultDTO.FailResp(constant.AdminModifyBotError, "管理员修改机器人失败", nil))
	}
	
```
错误
```go
	var bot *dto.UpdateBotDTO	
	resultDTO := dto.ResultDTO{}
	if err := c.BindJSON(bot); err != nil {
		// 检查参数解析是否出错
		c.JSON(http.StatusBadRequest, resultDTO.FailResp(constant.AdminModifyBotError, "管理员修改机器人失败", nil))
	}

```

BindJSON方法 相当于springboot的自动映射 将前端传过来的JSON数据映射为对应结构体

可以传递映射的**结构体实例或其指针**

在第一段代码中 首先声明了结构体的实例 再传递他的指针进行映射 （此时的指针已经分配好了内存空间） 是正确的做法

在第二段代码中 直接定义了对应结构体的指针 并且传递该指针 但是此指针并没有进行初始化(未分配空间) 值是nil 所以gin并不能将其绑定为对应的结构体类型

 在go语言中 所有的指针变量若未经初始化 他们的值都是nil 并不代表对应的 具体的数据类型

初始化之后 尽管都是结构体指针，但由于指向不同类型的结构体，不同结构体的指针类型是不同的。





### Windows上编译linux的go可执行文件

```go
PS D:\GoLand\GoProject\mini-gpt> $env:GOOS="linux"
PS D:\GoLand\GoProject\mini-gpt> $env:GOARCH="amd64"
PS D:\GoLand\GoProject\mini-gpt> go build main.go  
```





### 结构体嵌套 与 匿名结构体 映射等

先看代码及错误JSON

```go
type UpdateBotDTO struct {
    *models.Bot
}

type Bot struct {
	*BotInfo
	*BotConfig
	BotId int `gorm:"primaryKey"`
	//是否已经删除
	IsDelete bool
	//是否官方bot
	IsOfficial bool
}

type BotInfo struct {
	BotId       int    `json:"bot_id"`
	Name        string `json:"bot_name"`
	Avatar      string `json:"bot_avatar"`
	Description string `json:"bot_description"`
}

type BotConfig struct {
	BotId      int    `json:"bot_id"`
	InitPrompt string `json:"init_prompt"`
	Model      string `json:"model"`
}
```
原本是无脑按照嵌套嵌进JSON中的 但是这样子一直都是传的nil 及dto没有映射到真实值 错误json如下

```json
{
	"bot":{
            "bot_info": {
            "bot_name": "translate",
            "bot_avatar": "default",
            "bot_description": "a test"
        },
        "bot_config": {
            "init_prompt": "say lvoe plz",
            "model": "gpt-3.5-turbo-instruct"
        },
      "is_delete":false,
      "is_official":false,
      "bot_id":1
	}
}
```

但是go结构体中匿名嵌套结构体的特性有点类似于继承  如上 `updateBotDTO`结构体中只有一个 `Bot`的匿名字段 代表着在外部看来 这两个类是完全一样的 并且**没有嵌套关系** 
换句话说 完全可以使用 调用Bot方式字段中的方式来调用 updateBotDTO 中 
就如
```go
if updatedBot.IsOfficial {
		beforeBot, err := models.GetOfficialBot(updatedBot.BotId)
```
同时 也可以在 `updateBotDTO` 中把匿名对象搞出来 单独处理
```go
	bot := updatedBot.Bot
	isOfficial := bot.IsOfficial
```
两者效果是一样的
一句话总结就是 使用的时候 可以按照原本的嵌套关系进行调用 也可以直接调用匿名嵌套结构体中的字段

但是 **绑定字段 Bind前端JSON** 的时候只能后者 不可以进行显式嵌套 要么就在匿名对象后方加一个 `json:"bot"`等 但是这样使用匿名就没啥意义了 综上所述 正确的JSON调用只能是下方

```json
{
	"bot_name": "translate",
	"bot_avatar": "default",
	"bot_description": "a test",
	"init_prompt": "say lvoe plz",
	"model": "gpt-3.5-turbo-instruct",
	"is_delete": false,
	"is_official": false,
	"bot_id": 0
}
```





### 反射与嵌套匿名结构体

```go
//这里的反射代码其实可以用第三方structs类进行简化
//通过反射部分更新目标结构体
updateValue := reflect.ValueOf(bot).Elem()
//这个updateValue是需要部分更新的数据的元数据集 自己理解
dataValue := reflect.ValueOf(&beforeBot).Elem()
//Elem方法是获取到结构体字段或数组切片等数据结构底层字段值的方法
//于是这里先通过结构体的指针获取 其中指向的字段的值
```

```go
for i := 0; i < updateValue.NumField(); i++ {
    field := updateValue.Type().Field(i)

    fmt.Println(field)
    //类似java中的Field类型对象
    updateFieldValue := updateValue.Field(i)
    if updateFieldValue.IsZero() {
       continue
    }
    //如果是零值则跳过 不更新
    dataFieldValue := dataValue.FieldByName(field.Name)
    //通过属性的key获取返回的值
    if dataFieldValue.IsValid() && dataFieldValue.CanSet() {
       dataFieldValue.Set(updateFieldValue)
    }
}
```

数据结构同上一bug

此段代码是 **利用反射达成部分更新** 但是一直报 `err:interface *reflect.ValueError` 经检查是因为没有进行是否结构体的判断 只对普通的字段有效



### int等类型 & float64 json映射

当将json中的数据手动映射到map中 所有的整数或浮点数都会被识别成float64类型 需转换成float64之后再根据需求转回对应类型

```java
// 对于整数
if id, ok := data["id"].(float64); ok {
    intId := int(id)
    // 使用intId
}
```



### Type和Kind的区别

前者是得到变量的类型 可以获取指定的自定义结构体的类型

后者只能获得底层的基础类型 若是结构体则是`reflect.Struct` 指针则是`reflect.ptr`

`reflect.Type` 是类型的描述符，可以用于检查变量的类型。它是通过调用 `reflect.TypeOf()` 函数来获得的。`reflect.Kind` 是一个枚举类型，表示类型的底层基础类别，用于描述类型的分类，例如 `int`、`string`、`bool` 等。您可以通过 `reflect.ValueOf().Kind()` 方法来获取变量的底层类型。

`reflect.Type` 主要用于检查变量的类型，而 `reflect.Kind` 主要用于检查变量的底层基础类别。例如，`reflect.Type` 可以用于检查变量是否为 `int` 类型，而 `reflect.Kind` 可以用于检查变量是否为 `reflect.Int` 类型。

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x int = 10
    var y string = "Hello"

    // 获取 x 的类型
    typeOfX := reflect.TypeOf(x)

    // 获取 x 的底层类型
    kindOfX := reflect.ValueOf(x).Kind()

    // 检查 x 的类型是否为 int
    if typeOfX == reflect.TypeOf(int(0)) {
        fmt.Println("x is of type int")
    }

    // 检查 x 的底层类型是否为 int
    if kindOfX == reflect.Int {
        fmt.Println("x is of kind int")
    }

    // 获取 y 的类型
    typeOfY := reflect.TypeOf(y)

    // 获取 y 的底层类型
    kindOfY := reflect.ValueOf(y).Kind()

    // 检查 y 的类型是否为 string
    if typeOfY == reflect.TypeOf("") {
        fmt.Println("y is of type string")
    }

    // 检查 y 的底层类型是否为 string
    if kindOfY == reflect.String {
        fmt.Println("y is of kind string")
    }
}
```





