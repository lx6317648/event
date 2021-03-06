# Event 

[![GoDoc](https://godoc.org/github.com/gookit/event?status.svg)](https://godoc.org/github.com/gookit/event)
[![Build Status](https://travis-ci.org/gookit/event.svg?branch=master)](https://travis-ci.org/gookit/event)
[![Coverage Status](https://coveralls.io/repos/github/gookit/event/badge.svg?branch=master)](https://coveralls.io/github/gookit/event?branch=master)
[![Go Report Card](https://goreportcard.com/badge/github.com/gookit/event)](https://goreportcard.com/report/github.com/gookit/event)

> **[EN README](README.md)**

Go 实现的轻量级的事件管理、调度工具库

- 支持自定义定义事件对象
- 支持对一个事件添加多个监听器
- 支持设置事件监听器的优先级，优先级越高越先触发
- 支持根据事件名称前缀 `PREFIX.*` 来进行一组事件监听.
  - 注册`app.*` 事件的监听，触发 `app.run` `app.end` 时，都将同时会触发 `app.*` 事件
- 支持使用通配符 `*` 来监听全部事件的触发
- 完善的单元测试，单元覆盖率 `> 95%`

## GoDoc

- [godoc for github](https://godoc.org/github.com/gookit/event)

## 主要方法

- `On(name string, listener Listener, priority ...int)` 注册事件监听
- `AddSubscriber(sbr Subscriber)`  订阅，支持注册多个事件监听
- `Fire(name string, params M) (error, Event)` 触发事件
- `MustFire(name string, params M) Event`   触发事件，有错误则会panic
- `FireEvent(e Event) (err error)`    根据给定的事件实例，触发事件
- `FireBatch(es ...interface{}) (ers []error)` 一次触发多个事件

## 快速使用

```go
package main

import (
	"fmt"
	
	"github.com/gookit/event"
)

func main() {
	// 注册事件监听器
	event.On("evt1", event.ListenerFunc(func(e event.Event) error {
        fmt.Printf("handle event: %s\n", e.Name())
        return nil
    }), event.Normal)
	
	// 注册多个监听器
	event.On("evt1", event.ListenerFunc(func(e event.Event) error {
        fmt.Printf("handle event: %s\n", e.Name())
        return nil
    }), event.High)
	
	// ... ...
	
	// 触发事件
	// 注意：第二个监听器的优先级更高，所以它会先被执行
	event.MustFire("evt1", event.M{"arg0": "val0", "arg1": "val1"})
}
```

## 编写事件监听

- 使用匿名函数

```go
package mypgk

import (
	"fmt"
	
	"github.com/gookit/event"
)

var fnHandler = func(e event.Event) error {
	fmt.Printf("handle event: %s\n", e.Name())
    return nil
}

func Run() {
    // register
    event.On("evt1", event.ListenerFunc(fnHandler), event.High)
}
```

- 使用结构体方法

> 实现接口 `event.Listener`

```go
package mypgk

import (
	"fmt"
	
	"github.com/gookit/event"
)

type MyListener struct {
	// userData string
}

func (l *MyListener) Handle(e event.Event) error {
	e.Set("result", "OK")
	return nil
}
```

## 同时注册多个事件监听

> 实现接口 `event.Subscriber`

```go
package mypgk

import (
	"fmt"
	
	"github.com/gookit/event"
)

type MySubscriber struct {
	// ooo
}

func (s *MySubscriber) SubscribedEvents() map[string]interface{} {
	return map[string]interface{}{
		"e1": event.ListenerFunc(s.e1Handler),
		"e2": event.ListenerItem{
			Priority: event.AboveNormal,
			Listener: event.ListenerFunc(func(e Event) error {
				return fmt.Errorf("an error")
			}),
		},
		"e3": &MyListener{},
	}
}

func (s *MySubscriber) e1Handler(e event.Event) error {
	e.Set("e1-key", "val1")
	return nil
}
```

## 编写自定义事件

```go
package mypgk 

import (
	"fmt"
	
	"github.com/gookit/event"
)

type MyEvent struct{
	event.BasicEvent
	customData string
}

func (e *MyEvent) CustomData() string {
    return e.customData
}
```

使用：

```go
e := &MyEvent{customData: "hello"}
e.SetName("e1")
event.AddEvent(e)

// add listener
event.On("e1", event.ListenerFunc(func(e event.Event) error {
   fmt.Printf("custom Data: %s\n", e.(*MyEvent).CustomData())
   return nil
}))

// trigger
event.Fire("e1", nil)
// OR
// event.FireEvent(e)
```

## LICENSE

**[MIT](LICENSE)**
