# 单一职责原则(Single Responsibility Principle, SRP)

&emsp;&emsp; **单一职责原则(Single Responsibility Principle, SRP)**：一个类只负责一个功能领域中的相应职责，或者可以定义为：就一个类而言，应该只有一个引起它变化的原因，不要存在多于一个导致类变更的原因。一个类/接口/方法只负责一项职责。  
&emsp;&emsp; 单一职责原则是高内聚低耦合的指导方针，可以降低类的复杂度，提高类的可读性，提高系统的可维护性、降低变更引起的风险。其实，通俗来理解，一个类不能太累。我们在传统的软件工程，或者老旧的系统中经常能够看到，一个代码上万行的类。这种情况可能是由于历史原因导致的，但是不可否认，维护起来难度简直太大。因此，我们在软件设计过程中，要尝试将职责进行分离，不通职责封装在不同的类中，这样就能够降低我们设计软件的复杂度了。  
&emsp;&emsp; 单一职责原则不仅仅适用于面向对象编程语言设计，只要是模块化的编程，都适用。  

## Golang Demo

接下来，我们以接口为例，来看看单一职责原则在golang编程中的体现。详细的例子可以参考下面的Java Demo.  

```go
package singleresponsibility

type CourseContent interface {
  getCourseName() string

  getCourseVideo() []byte
}
```

```go
package singleresponsibility

type CourseManager interface {
  studyCourse()

  refundCourse()
}
```

```go
package singleresponsibility

import "fmt"

type Course struct {
}

func (Course) studyCourse() {
  fmt.Println("study")
}

func (Course) refundCourse() {
  fmt.Println("refund")
}

func (Course) getCourseName() string {
  return "course name"
}

func (Course) getCourseVideo() []byte {
  return nil
}
```

## Java Demo

### 接口单一职责原则的体现  

假设我们有下面这样一个接口。这个接口里面 既有获取信息，又有对课程的的操作等行为。  

```java
package tech.selinux.design.principle.singleresponsibility;

public interface ICourse {
  String getCourseName();

  byte[] getCourseVideo();

  void studyCourse();

  void refundCourse();
}
```

我们就可以对接口中的行为进行分类，使得每个接口具有单一职责。实现类可以实现多个接口就可以了。  

```java
package tech.selinux.design.principle.singleresponsibility;

public interface ICourseContent {

  String getCourseName();

  byte[] getCourseVideo();
}
```

```java
package tech.selinux.design.principle.singleresponsibility;

public interface ICourseManager {

  void studyCourse();

  void refundCourse();
}
```

```java
package tech.selinux.design.principle.singleresponsibility;

public class CourseImpl implements ICourseManager, ICourseContent {
  @Override
  public void studyCourse() {}

  @Override
  public void refundCourse() {}

  @Override
  public String getCourseName() {
    return null;
  }

  @Override
  public byte[] getCourseVideo() {
    return new byte[0];
  }
}
```


### 类单一职责原则的体现。

假设我们有下面一个类。  

```java
package tech.selinux.design.principle.singleresponsibility;

public class Bird {
  public void mainMoveMode(String birdName) {
    if ("鸵鸟".equals(birdName)) {
      System.out.println(birdName + "用脚走");
    } else {
      System.out.println(birdName + "用翅膀飞");
    }
  }
}
```

如果后期，我们需要增加许多新的鸟类，那这个类将变的无比庞杂，所以，我们可以定义几个只有单一职责的类。

```java
package tech.selinux.design.principle.singleresponsibility;

public class FlyBird {
  public void mainMoveMode(String birdName) {
    System.out.println(birdName + "用翅膀飞");
  }
}
```

```java
package tech.selinux.design.principle.singleresponsibility;

public class WalkBird {
  public void mainMoveMode(String birdName) {
    System.out.println(birdName + "用脚走");
  }
}
```

```java
package tech.selinux.design.principle.singleresponsibility;

public class Test {
  public static void main(String[] args) {
    FlyBird flyBird = new FlyBird();
    flyBird.mainMoveMode("大雁");

    WalkBird walkBird = new WalkBird();
    walkBird.mainMoveMode("鸵鸟");
  }
}
```

## Scala Demo

## UML (接口为例)

![接口单一职责UML](images/single-responsibility-principle.png)
