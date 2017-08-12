---
author : "杨承龙"
date : "2017-07-07T14:05:14+08:00"
draft : false
title : "cobra"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
# cobra

cobra是一个命令行框架, 它可以生成和解析命令行。 Cobra具有非常干净的界面和简单的设计，而不需要不必要的构造函数或初始化方法。

用Cobra命令构建的应用程序被设计成尽可能方便用户使用。可以在命令之前或之后放置flag。可以使用短的和长的flag。命令甚至不需要完全键入，help是自动生成的，它是应用程序使用help命令或--help flag时调用的特定命令.

### 概念

Cobra是建立在命令、参数结构之上的。

Commands代表行动，Args是事物，Flags表示对行为的修饰。

比较好的应用程序使用起来就像句子一样。用户将知道如何使用应用程序因为他们本来就知道如何使用它。

一般模式就像下面一样：

```
 VERB NOUN --ADJECTIVE. or APPNAME COMMAND ARG --FLAG
```

一些好的现实世界的例子可以更好地说明这一点。

在下面的例子中，server 是一个命令， port是一个参数

```shell
hugo server --port=1313
```

### Commands

命令是应用程序的中心点。应用程序支持的每个交互都将包含在命令中。命令可以拥有子命令并且可选地运行操作。

命令具有以下结构：

```go
type Command struct {
    Use string // The one-line usage message.
    Short string // The short description shown in the 'help' output.
    Long string // The long message shown in the 'help <this-command>' output.
    Run func(cmd *Command, args []string) // Run runs the command.
}
```

### flags

flags是修改命令行为的一种方式。Cobra完全兼容POSIX命令行模式以及flags。

flag的功能由pflag库提供，它是flag标准库的一个fork，保持相同的接口的同时兼容POSIX标准。

### 使用

Cobra的工作原理是创建一组命令，然后将它们组织成一棵树。命令树树定义应用程序的结构。

一旦每个命令都定义了相应的flag，那么树将被分配给最终执行的command。

### Cobra提供的功能

- 简易的子命令行模式，如 app server， app fetch等等
- 完全兼容posix命令行模式
- 嵌套子命令subcommand
- 支持全局，局部，串联flags
- 使用Cobra很容易的生成应用程序和命令，使用cobra create appname和cobra add cmdname
- 如果命令输入错误，将提供智能建议，如 app srver，将提示srver没有，是否是app server
- 自动生成commands和flags的帮助信息
- 自动生成详细的help信息，如app help
- 自动识别-h，--help帮助flag
- 自动生成应用程序在bash下命令自动完成功能
- 自动生成应用程序的man手册
- 命令行别名
- 自定义help和usage信息
- 可选的紧密集成的[viper](http://www.ctolib.com/http://github.com/spf13/viper) apps

### 如何使用

下面简单介绍一下如何使用Cobra，基本能够满足一般命令行程序的需求，如果需要更多功能，可以研究一下源码[github](http://www.ctolib.com/https://github.com/spf13/cobra)。

## 安装

cobra的源工程地址在github.com/spf13/cobra/cobra, 同其他Go开源项目一样,我们通过go get来拉取:

```go
$ go get -u github.com/spf13/cobra/cobra
```

安装完成后，打开GOPATH目录，bin目录下应该有已经编译好的cobra或cobra.exe程序，当然你也可以使用源代码自己生成一个最新的cobra程序。

通过下面语句import到工程:

```go
import "github.com/spf13/cobra"
```

## 通过cobra命令生成工程

```shell
$ cobra init cobra-demo
Your Cobra application is ready at
/Users/yang/lib/go/src/cobra-demo
Give it a try by going there and running `go run main.go`
Add commands to it by running `cobra add [cmdname]`
```

生成后的工程目录结构如下:

```shell
$ tree cobra-demo
cobra-demo
├── LICENSE
├── cmd
│   └── root.go
└── main.go

```

cobra生成的工程, 默认文件都放在cmd目录下, 以root.go作为根命令. cobra可以嵌套[flag](https://github.com/ShevYan/GoTutor/blob/master/flag-demo), [pflag](https://github.com/ShevYan/GoTutor/blob/master/pflag-demo), [viper](https://github.com/ShevYan/GoTutor/blob/master/viper-demo)一起使用. 其中init()里面调用了initConfig(), 并可以调用pflag:

```go
func init() {
	cobra.OnInitialize(initConfig)

	// Here you will define your flags and configuration settings.
	// Cobra supports Persistent Flags, which, if defined here,
	// will be global for your application.

	RootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra-demo.yaml)")
	// Cobra also supports local flags, which will only run
	// when this action is called directly.
	RootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}
```

flag的操作中, 分为全局的flag和当前子命令的flag. 什么是子命令呢? cobra支持多级子命令, 比如: mycmd sub1 sub2, 其中mycmd是根命令, sub1是mycmd的子命令, sub2是sub1的子命令. 因此, RootCmd.PersistentFlags()是一个全局flag, RootCmd.Flags()是当前命令的flag. 由于是根命令,其实二者是相同的, 但是如果在子命令中, 二者是不同的.

下面是使用viper的地方:

```go
// initConfig reads in config file and ENV variables if set.
func initConfig() {
	if cfgFile != "" { // enable ability to specify config file via flag
		viper.SetConfigFile(cfgFile)
	}

	viper.SetConfigName(".cobra-demo") // name of config file (without extension)
	viper.AddConfigPath("$HOME")  // adding home directory as first search path
	viper.AutomaticEnv()          // read in environment variables that match

	// If a config file is found, read it in.
	if err := viper.ReadInConfig(); err == nil {
		fmt.Println("Using config file:", viper.ConfigFileUsed())
	}
}
```

cobra对象初始化:

```go
// This represents the base command when called without any subcommands
var RootCmd = &cobra.Command{
	Use:   "cobra-demo",
	Short: "A brief description of your application",
	Long: `A longer description that spans multiple lines and likely contains
demos and usage of using your application. For demo:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
// Uncomment the following line if your bare application
// has an action associated with it:
	Run: func(cmd *cobra.Command, args []string) { },
}
```

上面这个对象就是cobra的根命令对象, 默认生成的时候Run字段是被注释的, 打开以后就可以在函数里面写入自己的处理函数. 下面是运行结果:

```shell
$ go run main.go -h
A longer description that spans multiple lines and likely contains
demos and usage of using your application. For demo:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobra-demo [flags]

Flags:
      --config="": config file (default is $HOME/.cobra-demo.yaml)
  -t, --toggle[=false]: Help message for toggle
```

执行下列命令, 生成子命令:

```shell
cobra add serve
cobra add config
cobra add create -p 'configCmd'
```

之后,工程结构变为:

```shell
$ tree cobra-demo
cobra-demo
├── LICENSE
├── cmd
│   ├── config.go
│   ├── create.go
│   ├── root.go
│   └── serve.go
└──  main.go

1 directory, 8 files
```

通过上述命令,我们生成了3个子命令, config, serve, create:

```shell
$ go run main.go -h
A longer description that spans multiple lines and likely contains
demos and usage of using your application. For demo:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobra-demo [flags]
  cobra-demo [command]

Available Commands:
  config      A brief description of your command
  serve       A brief description of your command

Flags:
      --config="": config file (default is $HOME/.cobra-demo.yaml)
  -t, --toggle[=false]: Help message for toggle
```

执行config子命令

```go
$ go run main.go config
config called
```

执行config的create子命令

```go
$ go run main.go config create
create called
```

我们可以看到代码里面config是作为根命令的子命令:

```go
func init() {
	RootCmd.AddCommand(configCmd)

	// Here you will define your flags and configuration settings.

	// Cobra supports Persistent Flags which will work for this command
	// and all subcommands, e.g.:
	configCmd.PersistentFlags().String("foo", "", "A help for foo")

	// Cobra supports local flags which will only run when this command
	// is called directly, e.g.:
	configCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")

}
```

当我们输入错误时，会有相应提示

```go
$ go run main.go serv   
Error: unknown command "serv" for "cobra-demo"

Did you mean this?
        serve

Run 'cobra-demo --help' for usage.
unknown command "serv" for "cobra-demo"

Did you mean this?
        serve

exit status 1
```