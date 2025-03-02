---
title: Go的设计模式
tags:
  - Go
  - Patterns
categories: [Programming Language,Go]
toc: true
date: 2022-02-23 11:19
updated: 2022-02-23 11:19
references:
  - title: Go设计模式24-总结(更新完毕) - Mohuishou
    url: https://lailin.xyz/post/go-design-pattern.html
---

Go 语言的设计模式之美。

<!-- more -->

Go 的设计模式：❌指在 Go 中不常用

- 创建型
	- 单例模式：一个类只有一个实例
		- 饿汉：系统初始化时创建并初始化单例对象
		- 懒汉：调用实例时初始化单例对象
	- 工厂模式：使用共同接口创建对象
	- 建造者模式：与工厂不同在于，创建参数复杂的对象
	- 原型模式：利用已有对象复制的方式来创建新的对象 ❌
- 结构型
	- 代理模式：为其他对象提供⼀种代理以控制这个对象的访问 **代理访问**
	- 桥接模式：将类的抽象部分和它的实现部分分离开来，使它们可以独立地变化 **抽象实现拆分独立**
	- 装饰模式：动态地给⼀个对象添加⼀些额外的职责 **附加职责**
	- 适配器模式：将⼀个类的接口转换成用户希望得到的另⼀个接口 **转换接口**
	- 门面模式：定义⼀个高层接口，为子系统中的⼀组接口提供一个一致的外观 **对外统一接口** ❌
	- 组合模式：将对象组合成树型结构以表示”整体 - 部分”的层次结构 **树型目录结构** ❌
	- 享元模式：提供支持大量细粒度对象共享的有效方法 ❌
- 行为型
	- 观察者模式：定义对象间的⼀种⼀对多的依赖关系，当⼀个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动更新
	- 模板模式：定义⼀个操作中的算法骨架，而将⼀些步骤延迟到子类中，使得子类可以不改变⼀个算法的结构即可重新定义算法的某些特定步骤
	- 策略模式：定义⼀系列算法，把它们⼀个个封装起来，并且使它们之间可互相替换，从而让算法可以独立于使用它的用户而变化
	- 职责链模式：通过给多个对象处理请求的机会，减少请求的发送者与接收者之间的耦合。将接收对象链接起来，在链中传递请求，直到有⼀个对象处理这个请求 **传递职责**
	- 状态模式：允许⼀个对象在其内部状态改变时改变它的行为
	- 迭代器模式：提供⼀种方法来顺序访问⼀个聚合对象中的各个元素而不需要暴露该对象的内部表示
	- 访问者模式：表示⼀个作用于某对象结构中的各元素的操作，使得在不改变各元素的类的前提下定义作用于这些元素的新操作 ❌
	- 备忘录模式：在不破坏封装性的前提下，捕获⼀个对象的内部状态，并在该对象之外保存这个状态，从而可用在以后将该对象恢复到原先保存的状态 ❌
	- 命令模式：将⼀个请求封装为⼀个对象，从而可用不同的请求对客户进行参数化，将请求排队或记录请求⽇志，支持可撤销的操作 ❌
	- 解释器模式：给定⼀种语言，定义它的文法表示，并定义⼀个解释器，该解释器用来根据文法表示来解释语言中的句子 ❌
	- 中介模式：用⼀个中介对象来封装⼀系列的对象交互。它使各对象不需要显式地相互调用，从而达到低耦合，还可以独立地改变对象间的交互 ❌

## 创建型

![创建型模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231202054.png)

### 单例模式 Singleton

**一个类只允许创建一个对象（实例）**，这个类就是单例类，这种设计模式就叫单例模式
**用处：** 业务上，有些数据在系统中只应该保存一份

![单例模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231129062.png)

#### 饿汉式

> 在系统初始化的时候我们已经把对象创建好了，需要用的时候直接拿过来用就好了

类一旦加载，就把单例初始化完成，保证 `getInstance` 的时候，单例是已经存在的了。

```go
package singleton

// Singleton 饿汉式单例
type Singleton struct{}

var singleton *Singleton

func init() {
	singleton = &Singleton{}
}

// GetInstance 获取实例
func GetInstance() *Singleton {
	return singleton
}
```

#### 懒汉式（双重检测）

> 创建对象时比较懒，先不急着创建对象，在需要加载配置文件的时候再去创建

只有当调用 `getInstance` 的时候，才会去初始化这个单例。

为了实现懒汉式不可避免的需要加锁。

```go
package singleton

import "sync"

var (
	lazySingleton *Singleton
	once          = &sync.Once{}
)

// GetLazyInstance 懒汉式
func GetLazyInstance() *Singleton {
	if lazySingleton == nil {
		once.Do(func() {
			lazySingleton = &Singleton{}
		})
	}
	return lazySingleton
}
```

### 工厂模式 Factory

在工厂模式中，在创建对象时不会对客户端暴露创建逻辑，并且是通过**使用一个共同的接口来指向新创建的对象**。

![工厂模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231147355.png)

#### 简单工厂

Go 本身是没有构造函数的，一般而言采用 `NewName` 的方式创建对象/接口，当它返回的是接口的时候，其实就是简单工厂模式。

```go
package factory

// IRuleConfigParser IRuleConfigParser
type IRuleConfigParser interface {
	Parse(data []byte)
}

// jsonRuleConfigParser jsonRuleConfigParser
type jsonRuleConfigParser struct {
}

// Parse Parse
func (J jsonRuleConfigParser) Parse(data []byte) {
	panic("implement me")
}

// yamlRuleConfigParser yamlRuleConfigParser
type yamlRuleConfigParser struct {
}

// Parse Parse
func (Y yamlRuleConfigParser) Parse(data []byte) {
	panic("implement me")
}

// NewIRuleConfigParser NewIRuleConfigParser
func NewIRuleConfigParser(t string) IRuleConfigParser {
	switch t {
	case "json":
		return jsonRuleConfigParser{}
	case "yaml":
		return yamlRuleConfigParser{}
	}
	return nil
}
```

#### 工厂方法

当对象的创建逻辑比较复杂，不只是简单的 new 一下就可以，而是要组合其他类对象。

做各种初始化操作的时候，推荐使用工厂方法模式，将复杂的创建逻辑拆分到多个工厂类中，让每个工厂类都不至于过于复杂。

> 组合复用原则：优先使用组合 contains a（聚合 has a），而不是继承 is a 来达到目的
>
> 迪米特法则：⼀个对象应当对其他对象有尽可能少的了解，即不和陌生人说话
>
> - 优点：降低类之间的耦合
> - 缺点：产生大量中介类，设计变复杂

```go
// IRuleConfigParserFactory 工厂方法接口
type IRuleConfigParserFactory interface {
	CreateParser() IRuleConfigParser
}

// yamlRuleConfigParserFactory yamlRuleConfigParser 的工厂类
type yamlRuleConfigParserFactory struct {
}

// CreateParser CreateParser
func (y yamlRuleConfigParserFactory) CreateParser() IRuleConfigParser {
	return yamlRuleConfigParser{}
}

// jsonRuleConfigParserFactory jsonRuleConfigParser 的工厂类
type jsonRuleConfigParserFactory struct {
}

// CreateParser CreateParser
func (j jsonRuleConfigParserFactory) CreateParser() IRuleConfigParser {
	return jsonRuleConfigParser{}
}

// NewIRuleConfigParserFactory 用一个简单工厂封装工厂方法
func NewIRuleConfigParserFactory(t string) IRuleConfigParserFactory {
	switch t {
	case "json":
		return jsonRuleConfigParserFactory{}
	case "yaml":
		return yamlRuleConfigParserFactory{}
	}
	return nil
}
```

### 建造者模式 Builder

在 Golang 中对于创建类参数比较多的对象的时候，常见的做法是必填参数直接传递，可选参数通过传递可变的方法进行创建。

![建造者模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231158567.png)

```go
package builder

import "fmt"

// ResourcePoolConfigOption option
type ResourcePoolConfigOption struct {
	maxTotal int
	maxIdle  int
	minIdle  int
}

// ResourcePoolConfigOptFunc to set option
type ResourcePoolConfigOptFunc func(option *ResourcePoolConfigOption)

// NewResourcePoolConfig NewResourcePoolConfig
func NewResourcePoolConfig(name string, opts ...ResourcePoolConfigOptFunc) (*ResourcePoolConfig, error) {
	if name == "" {
		return nil, fmt.Errorf("name can not be empty")
	}

	option := &ResourcePoolConfigOption{
		maxTotal: 10,
		maxIdle:  9,
		minIdle:  1,
	}

	for _, opt := range opts {
		opt(option)
	}

	if option.maxTotal < 0 || option.maxIdle < 0 || option.minIdle < 0 {
		return nil, fmt.Errorf("args err, option: %v", option)
	}

	if option.maxTotal < option.maxIdle || option.minIdle > option.maxIdle {
		return nil, fmt.Errorf("args err, option: %v", option)
	}

	return &ResourcePoolConfig{
		name:     name,
		maxTotal: option.maxTotal,
		maxIdle:  option.maxIdle,
		minIdle:  option.minIdle,
	}, nil
}
```

### 原型模式 Prototype

利用已有对象（原型）进行复制（拷贝）的方式来创建新的对象，达到节省创建时间的目的。

![原型模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231203233.png)

```go
package prototype

import (
	"encoding/json"
	"time"
)

// Keyword 搜索关键字
type Keyword struct {
	word      string
	visit     int
	UpdatedAt *time.Time
}

// Clone 这里使用序列化与反序列化的方式深拷贝
func (k *Keyword) Clone() *Keyword {
	var newKeyword Keyword
	b, _ := json.Marshal(k)
	json.Unmarshal(b, &newKeyword)
	return &newKeyword
}

// Keywords 关键字 map
type Keywords map[string]*Keyword

// Clone 复制一个新的 keywords
// updatedWords: 需要更新的关键词列表，由于从数据库中获取数据常常是数组的方式
func (words Keywords) Clone(updatedWords []*Keyword) Keywords {
	newKeywords := Keywords{}

	for k, v := range words {
		// 这里是浅拷贝，直接拷贝了地址
		newKeywords[k] = v
	}

	// 替换掉需要更新的字段，这里用的是深拷贝
	for _, word := range updatedWords {
		newKeywords[word.word] = word.Clone()
	}

	return newKeywords
}
```

## 结构型

### 代理模式 Proxy

为其他对象提供⼀种代理以控制这个对象的访问。

![代理模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231208710.png)

#### 静态代理

```go
package proxy

import (
	"log"
	"time"
)

// IUser IUser
type IUser interface {
	Login(username, password string) error
}

// User 用户
type User struct {
}

// Login 用户登录
func (u *User) Login(username, password string) error {
	// 不实现细节
	return nil
}

// UserProxy 代理类
type UserProxy struct {
	user *User
}

// NewUserProxy NewUserProxy
func NewUserProxy(user *User) *UserProxy {
	return &UserProxy{
		user: user,
	}
}

// Login 登录，和 user 实现相同的接口
func (p *UserProxy) Login(username, password string) error {
	// before 这里可能会有一些统计的逻辑
	start := time.Now()

	// 这里是原有的业务逻辑
	if err := p.user.Login(username, password); err != nil {
		return err
	}

	// after 这里可能也有一些监控统计的逻辑
	log.Printf("user login cost time: %s", time.Now().Sub(start))

	return nil
}
```

### 桥接模式 Bridge

将类的抽象部分和它的实现部分分离开来，使它们可以独立地变化。

![桥接模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231210754.png)

```go
package bridge

// IMsgSender IMsgSender
type IMsgSender interface {
	Send(msg string) error
}

// EmailMsgSender 发送邮件
// 可能还有 电话、短信等各种实现
type EmailMsgSender struct {
	emails []string
}

// NewEmailMsgSender NewEmailMsgSender
func NewEmailMsgSender(emails []string) *EmailMsgSender {
	return &EmailMsgSender{emails: emails}
}

// Send Send
func (s *EmailMsgSender) Send(msg string) error {
	// 这里去发送消息
	return nil
}

// INotification 通知接口
type INotification interface {
	Notify(msg string) error
}

// ErrorNotification 错误通知
// 后面可能还有 warning 各种级别
type ErrorNotification struct {
	sender IMsgSender
}

// NewErrorNotification NewErrorNotification
func NewErrorNotification(sender IMsgSender) *ErrorNotification {
	return &ErrorNotification{sender: sender}
}

// Notify 发送通知
func (n *ErrorNotification) Notify(msg string) error {
	return n.sender.Send(msg)
}
```

### 装饰模式 Decorator

动态地给⼀个对象添加⼀些额外的职责。它提供了用子类扩展功能的⼀个灵活的替代，比派生一个子类更加灵活。

**附加职责**

![装饰模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231212129.png)

下面是一个简单的画画的例子，默认的 `Square` 只有基础的画画功能， `ColorSquare` 为他加上了颜色

```go
package decorator

// IDraw IDraw
type IDraw interface {
	Draw() string
}

// Square 正方形
type Square struct{}

// Draw Draw
func (s Square) Draw() string {
	return "this is a square"
}

// ColorSquare 有颜色的正方形
type ColorSquare struct {
	square IDraw
	color  string
}

// NewColorSquare NewColorSquare
func NewColorSquare(square IDraw, color string) ColorSquare {
	return ColorSquare{color: color, square: square}
}

// Draw Draw
func (c ColorSquare) Draw() string {
	return c.square.Draw() + ", color is " + c.color
}
```

### 适配器模式 Adapter

将⼀个类的接口转换成用户希望得到的另⼀个接口，它使原本不相容的接口得以协同⼯作。

![适配器模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231214029.png)

假设现在有一个运维系统，需要分别调用阿里云和 AWS 的 SDK 创建主机，两个 SDK 提供的创建主机的接口不一致，此时就可以通过适配器模式，将两个接口统一。

```go
package adapter

import "fmt"

// ICreateServer 创建云主机
type ICreateServer interface {
	CreateServer(cpu, mem float64) error
}

// AWSClient aws sdk
type AWSClient struct{}

// RunInstance 启动实例
func (c *AWSClient) RunInstance(cpu, mem float64) error {
	fmt.Printf("aws client run success, cpu： %f, mem: %f", cpu, mem)
	return nil
}

// AwsClientAdapter 适配器
type AwsClientAdapter struct {
	Client AWSClient
}

// CreateServer 启动实例
func (a *AwsClientAdapter) CreateServer(cpu, mem float64) error {
	a.Client.RunInstance(cpu, mem)
	return nil
}

// AliyunClient aliyun sdk
type AliyunClient struct{}

// CreateServer 启动实例
func (c *AliyunClient) CreateServer(cpu, mem int) error {
	fmt.Printf("aws client run success, cpu： %d, mem: %d", cpu, mem)
	return nil
}

// AliyunClientAdapter 适配器
type AliyunClientAdapter struct {
	Client AliyunClient
}

// CreateServer 启动实例
func (a *AliyunClientAdapter) CreateServer(cpu, mem float64) error {
	a.Client.CreateServer(int(cpu), int(mem))
	return nil
}
```

### 门面模式 Facade

定义⼀个高层接口，为子系统中的⼀组接口提供一个一致的外观，从而简化了该子系统的使用。

**对外统一接口**

![门面模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231353536.jpeg)

> 假设现在我有一个网站，以前有登录和注册的流程，登录的时候调用用户的查询接口，注册时调用用户的创建接口。为了简化用户的使用流程，我们现在提供直接验证码登录/注册的功能，如果该手机号已注册那么我们就走登录流程，如果该手机号未注册，那么我们就创建一个新的用户。

```go
package facade

// IUser 用户接口
type IUser interface {
	Login(phone int, code int) (*User, error)
	Register(phone int, code int) (*User, error)
}

// IUserFacade 门面模式
type IUserFacade interface {
	LoginOrRegister(phone int, code int) error
}

// User 用户
type User struct {
	Name string
}

// UserService UserService
type UserService struct {}

// Login 登录
func (u UserService) Login(phone int, code int) (*User, error) {
	// 校验操作 ...
	return &User{Name: "test login"}, nil
}

// Register 注册
func (u UserService) Register(phone int, code int) (*User, error) {
	// 校验操作 ...
	// 创建用户
	return &User{Name: "test register"}, nil
}

// LoginOrRegister 登录或注册
func (u UserService)LoginOrRegister(phone int, code int) (*User, error) {
	user, err := u.Login(phone, code)
	if err != nil {
		return nil, err
	}

	if user != nil {
		return user, nil
	}

	return u.Register(phone, code)
}
```

### 组合模式 Composite

将对象组合成树型结构以表示“整体 - 部分”的层次结构，使得用户对单个对象和组合对象的使用具有⼀致性。

**树形目录结构**

![组合模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231355688.png)

> 公司的人员组织就是一个典型的树状的结构，现在假设我们现在有部分，和员工，两种角色，一个部门下面可以存在子部门和员工，员工下面不能再包含其他节点。

我们现在要实现一个统计一个部门下员工数量的功能

```go
package composite

// IOrganization 组织接口，都实现统计人数的功能
type IOrganization interface {
	Count() int
}

// Employee 员工
type Employee struct {
	Name string
}

// Count 人数统计
func (Employee) Count() int {
	return 1
}

// Department 部门
type Department struct {
	Name string

	SubOrganizations []IOrganization
}

// Count 人数统计
func (d Department) Count() int {
	c := 0
	for _, org := range d.SubOrganizations {
		c += org.Count()
	}
	return c
}

// AddSub 添加子节点
func (d *Department) AddSub(org IOrganization) {
	d.SubOrganizations = append(d.SubOrganizations, org)
}

// NewOrganization 构建组织架构 demo
func NewOrganization() IOrganization {
	root := &Department{Name: "root"}
	for i := 0; i < 10; i++ {
		root.AddSub(&Employee{})
		root.AddSub(&Department{Name: "sub", SubOrganizations: []IOrganization{&Employee{}}})
	}
	return root
}
```

### 享元模式 Flyweight

提供支持大量细粒度对象共享的有效方法。

![享元模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231358380.png)

> 象棋，无论是什么对局，棋子的基本属性其实是固定的，并不会因为随着下棋的过程变化。那我们就可以把棋子变为享元，让所有的对局都共享这些对象，以此达到节省内存的目的。

```go
package flyweight

var units = map[int]*ChessPieceUnit{
	1: {
		ID:    1,
		Name:  "車",
		Color: "red",
	},
	2: {
		ID:    2,
		Name:  "炮",
		Color: "red",
	},
	// ... 其他棋子
}

// ChessPieceUnit 棋子享元
type ChessPieceUnit struct {
	ID    uint
	Name  string
	Color string
}

// NewChessPieceUnit 工厂
func NewChessPieceUnit(id int) *ChessPieceUnit {
	return units[id]
}

// ChessPiece 棋子
type ChessPiece struct {
	Unit *ChessPieceUnit
	X    int
	Y    int
}

// ChessBoard 棋局
type ChessBoard struct {
	chessPieces map[int]*ChessPiece
}

// NewChessBoard 初始化棋盘
func NewChessBoard() *ChessBoard {
	board := &ChessBoard{chessPieces: map[int]*ChessPiece{}}
	for id := range units {
		board.chessPieces[id] = &ChessPiece{
			Unit: NewChessPieceUnit(id),
			X:    0,
			Y:    0,
		}
	}
	return board
}

// Move 移动棋子
func (c *ChessBoard) Move(id, x, y int) {
	c.chessPieces[id].X = x
	c.chessPieces[id].Y = y
}
```

## 行为型

### 观察者模式 Observer

定义对象间的⼀种⼀对多的依赖关系，当⼀个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动更新。

![观察者模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231408656.png)

```go
package observer

import "fmt"

// ISubject subject
type ISubject interface {
	Register(observer IObserver)
	Remove(observer IObserver)
	Notify(msg string)
}

// IObserver 观察者
type IObserver interface {
	Update(msg string)
}

// Subject Subject
type Subject struct {
	observers []IObserver
}

// Register 注册
func (sub *Subject) Register(observer IObserver) {
	sub.observers = append(sub.observers, observer)
}

// Remove 移除观察者
func (sub *Subject) Remove(observer IObserver) {
	for i, ob := range sub.observers {
		if ob == observer {
			sub.observers = append(sub.observers[:i], sub.observers[i+1:]...)
		}
	}
}

// Notify 通知
func (sub *Subject) Notify(msg string) {
	for _, o := range sub.observers {
		o.Update(msg)
	}
}

// Observer1 Observer1
type Observer1 struct{}

// Update 实现观察者接口
func (Observer1) Update(msg string) {
	fmt.Printf("Observer1: %s", msg)
}

// Observer2 Observer2
type Observer2 struct{}

// Update 实现观察者接口
func (Observer2) Update(msg string) {
	fmt.Printf("Observer2: %s", msg)
}
```

### 模板模式 Template

定义⼀个操作中的算法骨架，而将⼀些步骤延迟到子类中，使得子类可以不改变⼀个算法的结构即可重新定义算法的某些特定步骤。

![模板模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231409069.png)

> 要做一个短信推送的系统，那么需要
>
> 1. 检查短信字数是否超过限制
> 2. 检查手机号是否正确
> 3. 发送短信
> 4. 返回状态
>
> 我们可以发现，在发送短信的时候由于不同的供应商调用的接口不同，所以会有一些实现上的差异，但是他的算法（业务逻辑）是固定的

```go
package template

import "fmt"

// ISMS ISMS
type ISMS interface {
	send(content string, phone int) error
}

// SMS 短信发送基类
type sms struct {
	ISMS
}

// Valid 校验短信字数
func (s *sms) Valid(content string) error {
	if len(content) > 63 {
		return fmt.Errorf("content is too long")
	}
	return nil
}

// Send 发送短信
func (s *sms) Send(content string, phone int) error {
	if err := s.Valid(content); err != nil {
		return err
	}

	// 调用子类的方法发送短信
	return s.send(content, phone)
}

// TelecomSms 走电信通道
type TelecomSms struct {
	*sms
}

// NewTelecomSms NewTelecomSms
func NewTelecomSms() *TelecomSms {
	tel := &TelecomSms{}
	// 这里有点绕，是因为 go 没有继承，用嵌套结构体的方法进行模拟
	// 这里将子类作为接口嵌入父类，就可以让父类的模板方法 Send 调用到子类的函数
	// 实际使用中，我们并不会这么写，都是采用组合+接口的方式完成类似的功能
	tel.sms = &sms{ISMS: tel}
	return tel
}

func (tel *TelecomSms) send(content string, phone int) error {
	fmt.Println("send by telecom success")
	return nil
}
```

### 策略模式 Strategy

定义⼀系列算法，把它们⼀个个封装起来，并且使它们之间可互相替换，从而让算法可以独立于使用它的用户而变化。

![策略模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231410116.png)

> 在保存文件的时候，由于政策或者其他的原因可能需要选择不同的存储方式，敏感数据我们需要加密存储，不敏感的数据我们可以直接明文保存。

```go
package strategy

import (
	"fmt"
	"io/ioutil"
	"os"
)

// StorageStrategy 存储策略
type StorageStrategy interface {
	Save(name string, data []byte) error
}

var strategys = map[string]StorageStrategy{
	"file":         &fileStorage{},
	"encrypt_file": &encryptFileStorage{},
}

// NewStorageStrategy NewStorageStrategy
func NewStorageStrategy(t string) (StorageStrategy, error) {
	s, ok := strategys[t]
	if !ok {
		return nil, fmt.Errorf("not found StorageStrategy: %s", t)
	}

	return s, nil
}

// FileStorage 保存到文件
type fileStorage struct{}

// Save Save
func (s *fileStorage) Save(name string, data []byte) error {
	return ioutil.WriteFile(name, data, os.ModeAppend)
}

// encryptFileStorage 加密保存到文件
type encryptFileStorage struct{}

// Save Save
func (s *encryptFileStorage) Save(name string, data []byte) error {
	// 加密
	data, err := encrypt(data)
	if err != nil {
		return err
	}

	return ioutil.WriteFile(name, data, os.ModeAppend)
}

func encrypt(data []byte) ([]byte, error) {
	// 这里实现加密算法
	return data, nil
}
```

### 职责链模式 Chain of Responsibility

通过给多个对象处理请求的机会，减少请求的发送者与接收者之间的耦合。将接收对象链接起来，在链中传递请 求，直到有⼀个对象处理这个请求。
**传递职责**

![职责链模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231411476.png)

```go
// Package chain 职责链模式
// 🌰 假设我们现在有个校园论坛，由于社区规章制度、广告、法律法规的原因需要对用户的发言进行敏感词过滤
//    如果被判定为敏感词，那么这篇帖子将会被封禁
package chain

// SensitiveWordFilter 敏感词过滤器，判定是否是敏感词
type SensitiveWordFilter interface {
	Filter(content string) bool
}

// SensitiveWordFilterChain 职责链
type SensitiveWordFilterChain struct {
	filters []SensitiveWordFilter
}

// AddFilter 添加一个过滤器
func (c *SensitiveWordFilterChain) AddFilter(filter SensitiveWordFilter) {
	c.filters = append(c.filters, filter)
}

// Filter 执行过滤
func (c *SensitiveWordFilterChain) Filter(content string) bool {
	for _, filter := range c.filters {
		// 如果发现敏感直接返回结果
		if filter.Filter(content) {
			return true
		}
	}
	return false
}

// AdSensitiveWordFilter 广告
type AdSensitiveWordFilter struct{}

// Filter 实现过滤算法
func (f *AdSensitiveWordFilter) Filter(content string) bool {
	// TODO: 实现算法
	return false
}

// PoliticalWordFilter 政治敏感
type PoliticalWordFilter struct{}

// Filter 实现过滤算法
func (f *PoliticalWordFilter) Filter(content string) bool {
	// TODO: 实现算法
	return true
}
```

### 状态模式 State

允许⼀个对象在其内部状态改变时改变它的行为。

**状态变成类**

![状态模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231412401.png)

```go
// Package state 状态模式
// 笔记请查看: https://lailin.xyz/state.html
// 这是一个工作流的例子，在企业内部或者是学校我们经常会看到很多审批流程
// 假设我们有一个报销的流程: 员工提交报销申请 -> 直属部门领导审批 -> 财务审批 -> 结束
// 在这个审批流中，处在不同的环节就是不同的状态
// 而流程的审批、驳回就是不同的事件
package state

import "fmt"

// Machine 状态机
type Machine struct {
	state IState
}

// SetState 更新状态
func (m *Machine) SetState(state IState) {
	m.state = state
}

// GetStateName 获取当前状态
func (m *Machine) GetStateName() string {
	return m.state.GetName()
}

func (m *Machine) Approval() {
	m.state.Approval(m)
}

func (m *Machine) Reject() {
	m.state.Reject(m)
}

// IState 状态
type IState interface {
	// 审批通过
	Approval(m *Machine)
	// 驳回
	Reject(m *Machine)
	// 获取当前状态名称
	GetName() string
}

// leaderApproveState 直属领导审批
type leaderApproveState struct{}

// Approval 获取状态名字
func (leaderApproveState) Approval(m *Machine) {
	fmt.Println("leader 审批成功")
	m.SetState(GetFinanceApproveState())
}

// GetName 获取状态名字
func (leaderApproveState) GetName() string {
	return "LeaderApproveState"
}

// Reject 获取状态名字
func (leaderApproveState) Reject(m *Machine) {}

func GetLeaderApproveState() IState {
	return &leaderApproveState{}
}

// financeApproveState 财务审批
type financeApproveState struct{}

// Approval 审批通过
func (f financeApproveState) Approval(m *Machine) {
	fmt.Println("财务审批成功")
	fmt.Println("出发打款操作")
}

// 拒绝
func (f financeApproveState) Reject(m *Machine) {
	m.SetState(GetLeaderApproveState())
}

// GetName 获取名字
func (f financeApproveState) GetName() string {
	return "FinanceApproveState"
}

// GetFinanceApproveState GetFinanceApproveState
func GetFinanceApproveState() IState {
	return &financeApproveState{}
}
```

### 迭代器模式 Iterator

提供⼀种⽅法来顺序访问⼀个聚合对象中的各个元素而不需要暴露该对象的内部表示。

![迭代器模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231412859.png)

```go
package iterator

// Iterator 迭代器接口
type Iterator interface {
	HasNext() bool
	Next()
	// 获取当前元素，由于 Go 1.15 中还没有泛型，所以我们直接返回 interface{}
	CurrentItem() interface{}
}

// ArrayInt 数组
type ArrayInt []int

// Iterator 返回迭代器
func (a ArrayInt) Iterator() Iterator {
	return &ArrayIntIterator{
		arrayInt: a,
		index:    0,
	}
}

// ArrayIntIterator 数组迭代
type ArrayIntIterator struct {
	arrayInt ArrayInt
	index    int
}

// HasNext 是否有下一个
func (iter *ArrayIntIterator) HasNext() bool {
	return iter.index < len(iter.arrayInt)-1
}

// Next 游标加一
func (iter *ArrayIntIterator) Next() {
	iter.index++
}

// CurrentItem 获取当前元素
func (iter *ArrayIntIterator) CurrentItem() interface{} {
	return iter.arrayInt[iter.index]
}

```

### 访问者模式 Visitor

表示⼀个作用于某对象结构中的各元素的操作，使得在不改变各元素的类的前提下定义作用于这些元素的新操作。

![访问者模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231413188.png)

```go
package visitor

import (
	"fmt"
	"path"
)

// Visitor 访问者
type Visitor interface {
	Visit(IResourceFile) error
}

// IResourceFile IResourceFile
type IResourceFile interface {
	Accept(Visitor) error
}

// NewResourceFile NewResourceFile
func NewResourceFile(filepath string) (IResourceFile, error) {
	switch path.Ext(filepath) {
	case ".ppt":
		return &PPTFile{path: filepath}, nil
	case ".pdf":
		return &PdfFile{path: filepath}, nil
	default:
		return nil, fmt.Errorf("not found file type: %s", filepath)
	}
}

// PdfFile PdfFile
type PdfFile struct {
	path string
}

// Accept Accept
func (f *PdfFile) Accept(visitor Visitor) error {
	return visitor.Visit(f)
}

// PPTFile PPTFile
type PPTFile struct {
	path string
}

// Accept Accept
func (f *PPTFile) Accept(visitor Visitor) error {
	return visitor.Visit(f)
}

// Compressor 实现压缩功能
type Compressor struct{}

// Visit 实现访问者模式方法
// 我们可以发现由于没有函数重载，我们只能通过断言来根据不同的类型调用不同函数
// 但是我们即使不采用访问者模式，我们其实也是可以这么操作的
// 并且由于采用了类型断言，所以如果需要操作的对象比较多的话，这个函数其实也会膨胀的比较厉害
// 后续可以考虑按照命名约定使用 generate 自动生成代码
// 或者是使用反射简化
func (c *Compressor) Visit(r IResourceFile) error {
	switch f := r.(type) {
	case *PPTFile:
		return c.VisitPPTFile(f)
	case *PdfFile:
		return c.VisitPDFFile(f)
	default:
		return fmt.Errorf("not found resource typr: %##v", r)
	}
}

// VisitPPTFile VisitPPTFile
func (c *Compressor) VisitPPTFile(f *PPTFile) error {
	fmt.Println("this is ppt file")
	return nil
}

// VisitPDFFile VisitPDFFile
func (c *Compressor) VisitPDFFile(f *PdfFile) error {
	fmt.Println("this is pdf file")
	return nil
}
```

### 备忘录模式 Memento

在不破坏封装性的前提下，捕获⼀个对象的内部状态，并在该对象之外保存这个状态，从而可用在以后将该对象恢复到原先保存的状态。

![备忘录模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231414592.png)

```go
// Package memento 备忘录模式
// 下面这个例子采用原课程的例子，一个输入程序
// 如果输入 :list 则显示当前保存的内容
// 如果输入 :undo 则删除上一次的输入
// 如果输入其他的内容则追加保存
package memento

// InputText 用于保存数据
type InputText struct {
	content string
}

// Append 追加数据
func (in *InputText) Append(content string) {
	in.content += content
}

// GetText 获取数据
func (in *InputText) GetText() string {
	return in.content
}

// Snapshot 创建快照
func (in *InputText) Snapshot() *Snapshot {
	return &Snapshot{content: in.content}
}

// Restore 从快照中恢复
func (in *InputText) Restore(s *Snapshot) {
	in.content = s.GetText()
}

// Snapshot 快照，用于存储数据快照
// 对于快照来说，只能不能被外部（不同包）修改，只能获取数据，满足封装的特性
type Snapshot struct {
	content string
}

// GetText GetText
func (s *Snapshot) GetText() string {
	return s.content
}
```

### 命令模式 Command

将⼀个请求封装为⼀个对象，从而可用不同的请求对客户进行参数化，将请求排队或记录请求⽇志，支持可撤销的操作。

**日志记录，可撤销**

![命令模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231415154.png)

```go
// Package command 命令模式
// Blog: https://lailin.xyz/post/command.html
// 这是示例一，采用将函数封装为对象的方式实现，
// 示例说明:
// 假设现在有一个游戏服务，我们正在实现一个游戏后端
// 使用一个 goroutine 不断接收来自客户端请求的命令，并且将它放置到一个队列当中
// 然后我们在另外一个 goroutine 中来执行它
package command

import "fmt"

// ICommand 命令
type ICommand interface {
	Execute() error
}

// StartCommand 游戏开始运行
type StartCommand struct{}

// NewStartCommand NewStartCommand
func NewStartCommand( /*正常情况下这里会有一些参数*/ ) *StartCommand {
	return &StartCommand{}
}

// Execute Execute
func (c *StartCommand) Execute() error {
	fmt.Println("game start")
	return nil
}

// ArchiveCommand 游戏存档
type ArchiveCommand struct{}

// NewArchiveCommand NewArchiveCommand
func NewArchiveCommand( /*正常情况下这里会有一些参数*/ ) *ArchiveCommand {
	return &ArchiveCommand{}
}

// Execute Execute
func (c *ArchiveCommand) Execute() error {
	fmt.Println("game archive")
	return nil
}
```

### 解释器模式 Interpreter

给定⼀种语言，定义它的文法表示，并定义⼀个解释器，该解释器用来根据文法表示来解释语言中的句子。

![解释器模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231416104.png)

```go
// Package interpreter 解释器模式
// 采用原课程的示例, 并且做了一下简化
// 假设我们现在有一个监控系统
// 现在需要实现一个告警模块，可以根据输入的告警规则来决定是否触发告警
// 告警规则支持 &&、>、< 3种运算符
// 其中 >、< 优先级比  && 更高
package interpreter

import (
	"fmt"
	"regexp"
	"strconv"
	"strings"
)

// AlertRule 告警规则
type AlertRule struct {
	expression IExpression
}

// NewAlertRule NewAlertRule
func NewAlertRule(rule string) (*AlertRule, error) {
	exp, err := NewAndExpression(rule)
	return &AlertRule{expression: exp}, err
}

// Interpret 判断告警是否触发
func (r AlertRule) Interpret(stats map[string]float64) bool {
	return r.expression.Interpret(stats)
}

// IExpression 表达式接口
type IExpression interface {
	Interpret(stats map[string]float64) bool
}

// GreaterExpression >
type GreaterExpression struct {
	key   string
	value float64
}

// Interpret Interpret
func (g GreaterExpression) Interpret(stats map[string]float64) bool {
	v, ok := stats[g.key]
	if !ok {
		return false
	}
	return v > g.value
}

// NewGreaterExpression NewGreaterExpression
func NewGreaterExpression(exp string) (*GreaterExpression, error) {
	data := regexp.MustCompile(`\s+`).Split(strings.TrimSpace(exp), -1)
	if len(data) != 3 || data[1] != ">" {
		return nil, fmt.Errorf("exp is invalid: %s", exp)
	}

	val, err := strconv.ParseFloat(data[2], 10)
	if err != nil {
		return nil, fmt.Errorf("exp is invalid: %s", exp)
	}

	return &GreaterExpression{
		key:   data[0],
		value: val,
	}, nil
}

// LessExpression <
type LessExpression struct {
	key   string
	value float64
}

// Interpret Interpret
func (g LessExpression) Interpret(stats map[string]float64) bool {
	v, ok := stats[g.key]
	if !ok {
		return false
	}
	return v < g.value
}

// NewLessExpression NewLessExpression
func NewLessExpression(exp string) (*LessExpression, error) {
	data := regexp.MustCompile(`\s+`).Split(strings.TrimSpace(exp), -1)
	if len(data) != 3 || data[1] != "<" {
		return nil, fmt.Errorf("exp is invalid: %s", exp)
	}

	val, err := strconv.ParseFloat(data[2], 10)
	if err != nil {
		return nil, fmt.Errorf("exp is invalid: %s", exp)
	}

	return &LessExpression{
		key:   data[0],
		value: val,
	}, nil
}

// AndExpression &&
type AndExpression struct {
	expressions []IExpression
}

// Interpret Interpret
func (e AndExpression) Interpret(stats map[string]float64) bool {
	for _, expression := range e.expressions {
		if !expression.Interpret(stats) {
			return false
		}
	}
	return true
}

// NewAndExpression NewAndExpression
func NewAndExpression(exp string) (*AndExpression, error) {
	exps := strings.Split(exp, "&&")
	expressions := make([]IExpression, len(exps))

	for i, e := range exps {
		var expression IExpression
		var err error

		switch {
		case strings.Contains(e, ">"):
			expression, err = NewGreaterExpression(e)
		case strings.Contains(e, "<"):
			expression, err = NewLessExpression(e)
		default:
			err = fmt.Errorf("exp is invalid: %s", exp)
		}

		if err != nil {
			return nil, err
		}

		expressions[i] = expression
	}

	return &AndExpression{expressions: expressions}, nil
}
```

### 中介模式 Mediator

用⼀个中介对象来封装⼀系列的对象交互。它使各对象不需要显式地相互调用，从而达到低耦合，还可以独立地改变对象间的交互。

**不直接引用**

![中介模式](https://picgo-1303870432.cos.ap-shanghai.myqcloud.com/img/202202231416106.png)

```go
// Package mediator 中介模式
// 采用原课程的示例，并且做了一些裁剪
// 假设我们现在有一个较为复杂的对话框，里面包括，登录组件，注册组件，以及选择框
// 当选择框选择“登录”时，展示登录相关组件
// 当选择框选择“注册”时，展示注册相关组件
package mediator

import (
	"fmt"
	"reflect"
)

// Input 假设这表示一个输入框
type Input string

// String String
func (i Input) String() string {
	return string(i)
}

// Selection 假设这表示一个选择框
type Selection string

// Selected 当前选中的对象
func (s Selection) Selected() string {
	return string(s)
}

// Button 假设这表示一个按钮
type Button struct {
	onClick func()
}

// SetOnClick 添加点击事件回调
func (b *Button) SetOnClick(f func()) {
	b.onClick = f
}

// IMediator 中介模式接口
type IMediator interface {
	HandleEvent(component interface{})
}

// Dialog 对话框组件
type Dialog struct {
	LoginButton         *Button
	RegButton           *Button
	Selection           *Selection
	UsernameInput       *Input
	PasswordInput       *Input
	RepeatPasswordInput *Input
}

// HandleEvent HandleEvent
func (d *Dialog) HandleEvent(component interface{}) {
	switch {
	case reflect.DeepEqual(component, d.Selection):
		if d.Selection.Selected() == "登录" {
			fmt.Println("select login")
			fmt.Printf("show: %s\n", d.UsernameInput)
			fmt.Printf("show: %s\n", d.PasswordInput)
		} else if d.Selection.Selected() == "注册" {
			fmt.Println("select register")
			fmt.Printf("show: %s\n", d.UsernameInput)
			fmt.Printf("show: %s\n", d.PasswordInput)
			fmt.Printf("show: %s\n", d.RepeatPasswordInput)
		}
		// others, 如果点击了登录按钮，注册按钮
	}
}
```
