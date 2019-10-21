# 迪米特法则(Law of Demeter, LoD)

&emsp;&emsp; **迪米特法则(Law of  Demeter, LoD)**：一个对象应该对其他对象保持最少的了解，又叫最少知道原则。

&emsp;&emsp; 迪米特法则要求我们在设计系统时，应该尽量减少对象之间的交互，如果两个对象之间不必彼此直接通信，那么这两个对象就不应当发生任何直接的相互作用，如果其中的一个对象需要调用另一个对象的某一个方法的话，可以通过第三者转发这个调用。简言之，就是通过引入一个合理的第三者来降低现有对象之间的耦合度。

&emsp;&emsp; 迪米特法则强调**不要和“陌生人”说话、只与你的直接朋友通信**，在迪米特法则中，对于一个对象，其朋友包括以下几类：

- 当前对象本身(this)；
- 以参数形式传入到当前对象方法中的对象；
- 当前对象的成员对象；
- 如果当前对象的成员对象是一个集合，那么集合中的元素也都是朋友；
- 当前对象所创建的对象。

## Golang Demo

```go
package demeter

type Member struct {
}

func NewMember() *Member {
    return &Member{}
}
```

```go
import (
    "container/list"
    "fmt"
)

type TeamLeader struct {
}

func NewTeamLeader() *TeamLeader {
    return &TeamLeader{}
}

func (TeamLeader) checkNumberOfMember() {
    l := list.New()
    l.Init()
    for i := 0; i < 20; i++ {
        l.PushBack(NewMember())
    }
    fmt.Println(l.Len())
}

```

```go
package demeter

type Boss struct {
}

func NewBoss() *Boss {
    return &Boss{}
}

func (Boss) commandCheckNumber(leader *TeamLeader) {

}

```

```go
package demeter

import "testing"

func Test(t *testing.T) {
    boss := NewBoss()
    boss.commandCheckNumber(NewTeamLeader())
}
```

## Java Demo

```java
package tech.selinux.design.principle.demeter;

public class Boss {
  public void commandCheckNumber(TeamLeader teamLeader) {
    teamLeader.checkNumberOfMember();
  }
}
```

```java
package tech.selinux.design.principle.demeter;

public class Member {}

```

```java
package tech.selinux.design.principle.demeter;

import java.util.ArrayList;
import java.util.List;

public class TeamLeader {
  public void checkNumberOfMember() {
    List<Member> memberList = new ArrayList<Member>();
    for (int i = 0; i < 20; i++) {
      memberList.add(new Member());
    }
    System.out.println("组内成员数量" + memberList.size());
  }
}
```

```java
package tech.selinux.design.principle.demeter;

public class Test {
  public static void main(String[] args) {
    Boss boss = new Boss();
    TeamLeader teamLeader = new TeamLeader();
    boss.commandCheckNumber(teamLeader);
  }
}
```

## Scala Demo

## UML

这里的类结构关系比较简单，就不上图了。
