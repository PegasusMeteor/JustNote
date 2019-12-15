# 组合模式(Composite Pattern)

&emsp;&emsp;  **组合模式(Composite Pattern)**：组合多个对象形成树形结构以表示具有“整体—部分”关系的层次结构。组合模式对单个对象（即叶子对象）和组合对象（即容器对象）的使用具有一致性，组合模式又可以称为“整体—部分”(Part-Whole)模式，它是一种对象结构型模式。

## 适用场景

- 希望客户端可以忽略组合对象与单个对象的差异时
- 处理一个树形结构

## 优点

- 清楚地定义分层次的复杂对象，表示对象的全部或部分层次
- 让客户端忽略了层次的差异，方便对整个层次结构进行控制
- 简化客户端代码
- 符合开闭原则

## 缺点

- 限制类型时比较复杂
- 使设计变得更加抽象

下面我们引出一个应用场景，假设，我们有一个课程目录，如何使用组合模式在实现这个目录呢？

## Golang Demo

```go
package composite

type CatalogComponent interface {
    add(catalogComponent CatalogComponent)
    remove(catalogComponent CatalogComponent)
    Name(catalogComponent CatalogComponent) string
    Price(catalogComponent CatalogComponent) float32
    print()
}
```

```go
package composite

import (
    "fmt"
)

type Course struct {
    name  string
    price float32
}

func (c *Course) add(catalogComponent CatalogComponent) {
    panic("implement me")
}

func (c *Course) remove(catalogComponent CatalogComponent) {
    panic("implement me")
}

func NewCourse(name string, price float32) *Course {
    return &Course{name: name, price: price}
}
func (c *Course) Name(catalogComponent CatalogComponent) string {
    return c.name
}

func (c *Course) Price(catalogComponent CatalogComponent) float32 {
    return c.price
}

func (c *Course) print() {

    fmt.Printf("Course Name:%v, Price:%v \n", c.name, c.price)
}

```

```go
package composite

import (
    "fmt"
)

type CourseCatalog struct {
    items []CatalogComponent
    name  string
    level int
}

func (c *CourseCatalog) remove(catalogComponent CatalogComponent) {
    panic("implement me")
}

func (c *CourseCatalog) Price(catalogComponent CatalogComponent) float32 {
    panic("implement me")
}

func NewCourseCatalog(name string, level int) *CourseCatalog {
    return &CourseCatalog{
        items: []CatalogComponent{},
        name:  name,
        level: level,
    }

}

func (c *CourseCatalog) add(catalogComponent CatalogComponent) {

    c.items = append(c.items, catalogComponent)
}

func (c *CourseCatalog) Name(catalogComponent CatalogComponent) string {
    return c.name
}

func (c *CourseCatalog) print() {
    fmt.Println(c.name)
    for _, catalogComponent := range c.items {
        if c.level != 0 {
            for i := 0; i < c.level; i++ {
                fmt.Print("  ")
            }
        }
        catalogComponent.print()
    }

}

```

```go
package composite

import (
    "testing"
)

func Test(t *testing.T) {

    var linuxCourse CatalogComponent = NewCourse("Linux Course", 11)
    var windowsCourse CatalogComponent = NewCourse("Windows Course", 11)

    var javaCourseCatalog CatalogComponent = NewCourseCatalog("Java Course catalog", 2)

    var javaCourse1 CatalogComponent = NewCourse("Java Course Ⅰ", 55)
    var javaCourse2 CatalogComponent = NewCourse("Java Course Ⅱ", 66)
    var designPattern CatalogComponent = NewCourse("Java design pattern", 77)

    javaCourseCatalog.add(javaCourse1)
    javaCourseCatalog.add(javaCourse2)
    javaCourseCatalog.add(designPattern)

    mainCourseCatalog := NewCourseCatalog("Course catalog", 1)
    mainCourseCatalog.add(linuxCourse)
    mainCourseCatalog.add(windowsCourse)
    mainCourseCatalog.add(javaCourseCatalog)

    mainCourseCatalog.print()
}

```

## Java Demo

```java
package tech.selinux.design.pattern.structural.composite;

/** 定义一个抽象类， 子类需要将所有的操作进行重写 */
public abstract class CatalogComponent {

  public void add(CatalogComponent catalogComponent) {
    throw new UnsupportedOperationException("不支持添加操作");
  }

  public void remove(CatalogComponent catalogComponent) {
    throw new UnsupportedOperationException("不支持删除操作");
  }

  public String getName(CatalogComponent catalogComponent) {
    throw new UnsupportedOperationException("不支持获取名称操作");
  }

  public double getPrice(CatalogComponent catalogComponent) {
    throw new UnsupportedOperationException("不支持获取价格操作");
  }

  public void print() {
    throw new UnsupportedOperationException("不支持打印操作");
  }
}

```

```java
package tech.selinux.design.pattern.structural.composite;

public class Course extends CatalogComponent {
  private String name;
  // 目录层级
  private double price;

  public Course(String name, double price) {
    this.name = name;
    this.price = price;
  }

  @Override
  public String getName(CatalogComponent catalogComponent) {
    return this.name;
  }

  @Override
  public double getPrice(CatalogComponent catalogComponent) {
    return this.price;
  }

  @Override
  public void print() {
    System.out.println("Course Name:" + name + " Price:" + price);
  }
}

```

```java
package tech.selinux.design.pattern.structural.composite;

import java.util.ArrayList;
import java.util.List;

public class CourseCatalog extends CatalogComponent {
  private List<CatalogComponent> items = new ArrayList<CatalogComponent>();
  private String name;
  private Integer level;

  public CourseCatalog(String name, Integer level) {
    this.name = name;
    this.level = level;
  }

  @Override
  public void add(CatalogComponent catalogComponent) {
    items.add(catalogComponent);
  }

  @Override
  public String getName(CatalogComponent catalogComponent) {
    return this.name;
  }

  @Override
  public void remove(CatalogComponent catalogComponent) {
    items.remove(catalogComponent);
  }

  @Override
  public void print() {
    System.out.println(this.name);
    for (CatalogComponent catalogComponent : items) {
      if (this.level != null) {
        for (int i = 0; i < this.level; i++) {
          System.out.print("  ");
        }
      }
      catalogComponent.print();
    }
  }
}

```

```java
package tech.selinux.design.pattern.structural.composite;

public class Test {
  public static void main(String[] args) {
    CatalogComponent linuxCourse = new Course("Linux Course", 11);
    CatalogComponent windowsCourse = new Course("Windows Course", 11);

    CatalogComponent javaCourseCatalog = new CourseCatalog("Java Course catalog", 2);

    CatalogComponent javaCourse1 = new Course("Java Course Ⅰ", 55);
    CatalogComponent javaCourse2 = new Course("Java Course Ⅱ", 66);
    CatalogComponent designPattern = new Course("Java design pattern", 77);

    javaCourseCatalog.add(javaCourse1);
    javaCourseCatalog.add(javaCourse2);
    javaCourseCatalog.add(designPattern);

    CatalogComponent mainCourseCatalog = new CourseCatalog("Course catalog", 1);
    mainCourseCatalog.add(linuxCourse);
    mainCourseCatalog.add(windowsCourse);
    mainCourseCatalog.add(javaCourseCatalog);

    mainCourseCatalog.print();
  }
}

```

---

### 补充另一个版本的Java/Scala Demo 以及源码解析

---

## Java Demo_

## Scala Demo

## UML_

## 源码解析
