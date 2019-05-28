# 中介者模式(Mediator Pattern)

**中介者模式(Mediator Pattern)**：用一个中介对象（中介者）来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

## 适用场景

- 系统中对象之间存在复杂的引用关系,产生的相互依赖关系结构混乱且难以理解。
- 交互的公共行为，如果需要改变行为则可以增加新的中介者类。

## 优点

- 将一对多转化成了一对一、降低了程序复杂度
- 类之间解耦

## 缺点

- 中介者过多，导致系统复杂

接下来，我们引入一种应用场景。在工作中，我们会处于某个小组中。小组内的成员之间需要进行互相的沟通和交流。然后我们会有一个小组的工作群，我们在里面进行工作沟通和交流。这个小组群，其实就是我们的中介者。

## Golang Demo

定义一个函数直接调用来模拟静态方法。

```go
package mediator

import (
    "fmt"
)

type Member struct {
    name string
}

func (m *Member) Name() string {
    return m.name
}

func NewMember(name string) *Member {
    return &Member{name: name}
}

func (m *Member) sendMessage(message string) {
    ShowMessage(m, message)
}

// 定义一个函数直接调用来模式静态方法
func ShowMessage(member *Member, message string) {
    fmt.Println("2019-05-28" + "  [" + member.Name() + "]  " + message)
}

```

```golang
package mediator

func ExampleMediator() {
    peagsus := NewMember("Peagsus")
    meteor := NewMember("Meteor")
    ShowMessage(peagsus, "hello")
    ShowMessage(meteor, "world")
    //Output:
    //2019-05-28  [Peagsus]  hello
    //2019-05-28  [Meteor]  world
}

```

## Java Demo

我们定义一下小组的成员

```java
package tech.selinux.design.pattern.behavioral.mediator;

public class Member {
  private String name;

  public Member(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public void sendMessage(String message) {
    WorkGroup.showMessage(this, message);
  }
}
```

```java
package tech.selinux.design.pattern.behavioral.mediator;

import java.util.Date;

public class WorkGroup {

  public static void showMessage(Member member, String message) {
    System.out.println(new Date().toString() + "  [" + member.getName() + "]  " + message);
  }
}
```

```java
package tech.selinux.design.pattern.behavioral.mediator;

public class Test {
  public static void main(String[] args) {
    Member peagsus = new Member("Pegasus");
    Member meteor = new Member("Meteor");
    peagsus.sendMessage("hello");
    meteor.sendMessage("world");
  }
}
```

## UML

UML 太简单了，就不贴图了。