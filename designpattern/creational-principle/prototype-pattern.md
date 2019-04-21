# 原型模式(Prototype Pattern)

&emsp;&emsp; **原型模式(Prototype  Pattern)**：使用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。原型模式是一种对象创建型模式。
不需要知道任何创建的细节，不调用构造函数.

## 适用场景

- 类初始化消耗较多资源
- new 产生的一个对象需要非常繁琐的过程(数据准备、访问权限等)
- 构造函数比较复杂
- 循环体中生产大量的对象时

## 优点

- 比直接new一个对象性能高
- 简化创建过程

## 缺点

- 必须配备克隆方法
- 对克隆复杂对象或对克隆出的对象进行复杂改造时,容易引入风险
- 深拷贝、浅拷贝要运用得当

## Golang Demo

```go
package prototype

// 原型对象需要实现的接口
type Cloneable interface {
    Clone() Cloneable
}

type User struct {
    name string
    age  int
}

func (u User) Clone() Cloneable {
    return u
}

```

```go
package prototype

import (
    "fmt"
    "strconv"
    "testing"
)

func Test(t *testing.T) {

    user := User{}
    user.name = "init"
    usermap := make(map[string]User)
    for i := 1; i < 10; i++ {
        u := user.Clone().(User)
        u.name = strconv.Itoa(i)
        u.age = i
        usermap[strconv.Itoa(i)] = u
    }
    fmt.Println(user)
    fmt.Println(usermap)
}

```

由于golang中引入了指针，所以很巧的是，如果在返回值时，不指明是指针引用的话，就是值拷贝，因此原型模式的应用在golang中并不能得到很好的体现。也有可能是笔者学习不到位，后期会进行补充。

下面给出github上的一种实现方式。[https://github.com/senghoo/golang-design-pattern/blob/master/07_prototype/prototype.go](https://github.com/senghoo/golang-design-pattern/blob/master/07_prototype/prototype.go)

```go
package prototype

//Cloneable 是原型对象需要实现的接口
type Cloneable interface {
    Clone() Cloneable
}

type PrototypeManager struct {
    prototypes map[string]Cloneable
}

func NewPrototypeManager() *PrototypeManager {
    return &PrototypeManager{
        prototypes: make(map[string]Cloneable),
    }
}

func (p *PrototypeManager) Get(name string) Cloneable {
    return p.prototypes[name]
}

func (p *PrototypeManager) Set(name string, prototype Cloneable) {
    p.prototypes[name] = prototype
}
```

```go
package prototype

import "testing"

var manager *PrototypeManager

type Type1 struct {
    name string
}

func (t *Type1) Clone() Cloneable {
    tc := *t
    return &tc
}

type Type2 struct {
    name string
}

func (t *Type2) Clone() Cloneable {
    tc := *t
    return &tc
}

func TestClone(t *testing.T) {
    t1 := manager.Get("t1")

    t2 := t1.Clone()

    if t1 == t2 {
        t.Fatal("error! get clone not working")
    }
}

func TestCloneFromManager(t *testing.T) {
    c := manager.Get("t1").Clone()

    t1 := c.(*Type1)
    if t1.name != "type1" {
        t.Fatal("error")
    }

}

func init() {
    manager = NewPrototypeManager()

    t1 := &Type1{
        name: "type1",
    }
    manager.Set("t1", t1)
}
```

## Java Demo

**浅拷贝**,clone 的时候，并不会调用构造器。浅克隆默认引用的是同一个对象，这样是会有隐患的。

```java
package tech.selinux.design.pattern.creational.prototype;

public class Mail implements Cloneable {
  private String name;
  private String emailAddress;
  private String content;

  public Mail() {
    System.out.println("Mail Class Constructor");
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getEmailAddress() {
    return emailAddress;
  }

  public void setEmailAddress(String emailAddress) {
    this.emailAddress = emailAddress;
  }

  public String getContent() {
    return content;
  }

  public void setContent(String content) {
    this.content = content;
  }

  @Override
  public String toString() {
    return "Mail{"
        + "name='"
        + name
        + '\''
        + ", emailAddress='"
        + emailAddress
        + '\''
        + ", content='"
        + content
        + '\''
        + '}'
        + super.toString();
  }

  @Override
  protected Object clone() throws CloneNotSupportedException {
    System.out.println("clone mail object");
    return super.clone();
  }
}
```

```java
package tech.selinux.design.pattern.creational.prototype;

import java.text.MessageFormat;

public class MailUtil {
  public static void sendMail(Mail mail) {
    String outputContent = "向{0}用户,邮件地址:{1},邮件内容:{2}发送邮件成功";
    System.out.println(
        MessageFormat.format(
            outputContent, mail.getName(), mail.getEmailAddress(), mail.getContent()));
  }

  public static void saveOriginMailRecord(Mail mail) {
    System.out.println("存储originMail记录,originMail:" + mail.getContent());
  }
}

```

```java
package tech.selinux.design.pattern.creational.prototype;

public class Test {
  public static void main(String[] args) throws CloneNotSupportedException {
    Mail mail = new Mail();
    mail.setContent("初始化模板");
    System.out.println("初始化mail:" + mail);
    for (int i = 0; i < 10; i++) {
      Mail mailTemp = (Mail) mail.clone();
      mailTemp.setName("姓名" + i);
      mailTemp.setEmailAddress("姓名" + i + "@selinux.tech");
      mailTemp.setContent("恭喜您，此次中奖了");
      MailUtil.sendMail(mailTemp);
      System.out.println("克隆的mailTemp:" + mailTemp);
    }
    MailUtil.saveOriginMailRecord(mail);
  }
}

```

**深克隆**，深克隆也需要对clone方法进行重写。对于引用类型一定要注意是否需要深克隆。

```java
package tech.selinux.design.pattern.creational.prototype.clone;

import java.util.Date;

public class User implements Cloneable {
  private String name;
  private Date birthday;

  public User(String name, Date birthday) {
    this.name = name;
    this.birthday = birthday;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public Date getBirthday() {
    return birthday;
  }

  public void setBirthday(Date birthday) {
    this.birthday = birthday;
  }

  @Override
  protected Object clone() throws CloneNotSupportedException {
    User user = (User) super.clone();
    // 深克隆
    user.birthday = (Date) user.birthday.clone();
    return user;
  }

  @Override
  public String toString() {
    return "User{" + "name='" + name + '\'' + ", birthday=" + birthday + '}' + super.toString();
  }
}

```

```java
package tech.selinux.design.pattern.creational.prototype.clone;

import java.util.Date;

public class Test {
  public static void main(String[] args) throws CloneNotSupportedException {
    Date birthday = new Date(0L);
    User user = new User("佩奇", birthday);
    User user1 = (User) user.clone();
    System.out.println(user);
    System.out.println(user1);

    user.getBirthday().setTime(666666666666L);

    System.out.println(user);
    System.out.println(user1);
  }
}

```

**通过抽象类的方式实现原型**,如果实际业务中能够进行合理的抽象的话，可以使用下面的方式。

```java
package tech.selinux.design.pattern.creational.prototype.abstractprototype;

public abstract class A implements Cloneable {
  @Override
  protected Object clone() throws CloneNotSupportedException {
    return super.clone();
  }
}
```

```java
package tech.selinux.design.pattern.creational.prototype.abstractprototype;

public class B extends A {
  public static void main(String[] args) throws CloneNotSupportedException {
    B b = new B();
    b.clone();
  }
}
```
