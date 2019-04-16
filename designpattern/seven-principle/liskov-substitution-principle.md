# 里氏代换原则(Liskov Substitution Principle, LSP)

&emsp;&emsp; **里氏代换原则(Liskov Substitution Principle, LSP)**：所有引用基类（父类）的地方必须能透明地使用其子类的对象。  
&emsp;&emsp; 里氏代换原则的严格表述如下：如果对每一个类型为S的对象o1，都有类型为T的对象o2，使得以T定义的所有程序P在所有的对象o1代换o2时，程序P的行为没有变化，那么类型S是类型T的子类型。  
&emsp;&emsp; **定义拓展**：一个软件实体如果适用一个父类的话，那么就一定适用其子类，所有引用父类的地方必须能够透明地使用其子类的对象，子类对象能够替换父类对象，而程序逻辑不变。里氏代换原则告诉我们，**在软件中将一个基类对象替换成它的子类对象，程序将不会产生任何错误和异常，反过来则不成立，如果一个软件实体使用的是一个子类对象的话，那么它不一定能够使用基类对象**。例如：我喜欢动物，那我一定喜欢狗，因为狗是动物的子类；但是我喜欢狗，不能据此断定我喜欢动物，因为我并不喜欢老鼠，虽然它也是动物。
&emsp;&emsp; **隐身意义**：子类可以扩展父类的功能，但是不能修改父类的功能。

- 含义1:子类可以实现父类的抽象方法，但是不能覆盖父类的非抽象方法。
- 含义2:子类中可以增加自己特有的方法。
- 含义3:当子类方法重载父类方法时，方法的前置条件(即方法的入参、输入)要比父类方法的输入参数更宽松。
- 含义4:当子类的方法实现父类的方法时(重写/重载或实现抽象方法)，方法的后置条件(即方法的输出/返回值)要比父类更严格或相等。

里氏代换原则有众多优点：

- 约束继承泛滥，开闭原则的一种体现。
- 加强程序的健壮性，同时在变更时也可以做到非常好的兼容性，提高程序的维护性、扩展性。降低需求变更时引入的风险。

下面，我们来看一个demo。假设，长方形是正方形的基类，长方形有一个方法叫做resize(),这个方法的作用是如果宽度小于长度，就重新调整大小，使宽度大于长度一个单位。正方形也可以调用这个方法，但是却改变了自己是正方形的事实。这就违背了程序行为没有变化这一条件。

## Golang Demo

```go
package liskovsubstitution

type QuardRangle interface {
    Width() int
    Length() int
}
```

```go
package liskovsubstitution

type Rectangle struct {
    length int
    width  int
}

func (r *Rectangle) SetWidth(width int) {
    r.width = width
}

func (r Rectangle) SetLength(length int) {
    r.length = length
}

func (r Rectangle) Width() int {
    return r.width
}

func (r Rectangle) Length() int {
    return r.length
}
```

```go
package liskovsubstitution

type Square struct {
    sideLength int
}

func NewSquare(sideLength int) *Square {
    return &Square{sideLength: sideLength}
}

func (s *Square) SetSideLength(sideLength int) {
    s.sideLength = sideLength
}

func (s Square) Width() int {
    return s.sideLength
}

func (s Square) Length() int {
    return s.sideLength
}
```

```go
package liskovsubstitution

import (
    "fmt"
    "testing"
)

func Test(t *testing.T) {
    square := NewSquare(10)

    //resize(square)
}
func resize(rectangle Rectangle) {
    for rectangle.Width() <= rectangle.Length() {
        rectangle.SetWidth(rectangle.Width() + 1)
        fmt.Printf("width:%d,length:%d", rectangle.Width(), rectangle.Length())
    }
}
```

## Java Demo

```java
package tech.selinux.design.principle.liskovsubstitution;

public interface Quadrangle {
    long getWidth();
    long getLength();
}
```

```java
package tech.selinux.design.principle.liskovsubstitution;


public class Rectangle implements Quadrangle {
    private long length;
    private long width;

    @Override
    public long getWidth() {
        return width;
    }

    @Override
    public long getLength() {
        return length;
    }

    public void setLength(long length) {
        this.length = length;
    }

    public void setWidth(long width) {
        this.width = width;
    }
}

```

```java
package tech.selinux.design.principle.liskovsubstitution;


public class Square implements Quadrangle {
    private long sideLength;

    public long getSideLength() {
        return sideLength;
    }

    public void setSideLength(long sideLength) {
        this.sideLength = sideLength;
    }

    @Override
    public long getWidth() {
        return sideLength;
    }

    @Override
    public long getLength() {
        return sideLength;
    }
}
```

```java
package tech.selinux.design.principle.liskovsubstitution;

public class Test {
    public static void resize(Rectangle rectangle){
        while (rectangle.getWidth() <= rectangle.getLength()){
            rectangle.setWidth(rectangle.getWidth()+1);
            System.out.println("width:"+rectangle.getWidth() + " length:"+rectangle.getLength());
        }
        System.out.println("resize方法结束 width:"+rectangle.getWidth() + " length:"+rectangle.getLength());
    }

//    public static void main(String[] args) {
//        Rectangle rectangle = new Rectangle();
//        rectangle.setWidth(10);
//        rectangle.setLength(20);
//        resize(rectangle);
//    }
    public static void main(String[] args) {
        Square square = new Square();
//        square.setLength(10);
//        resize(square);
    }
}
```