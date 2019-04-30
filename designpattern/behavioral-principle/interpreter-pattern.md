# 解释器模式(Interpreter Pattern)

**解释器模式(Interpreter Pattern)**:给定一个语言，定义它的文法的一种表示。并定义一个解释器，这个解释器使用该表示来解释语言中的句子。简单来说，就是为了解释一种语言，而为语言创建的解释器。

## 适用场景

- 某个特定类型问题发生频率足够高（例如日志解析,计算器）

## 优点

- 语法由很多类表示，容易改变以及扩展此"语言"

## 缺点

- 当语法规则数目太多时，增加了系统复杂度

在实际的开发工作中，解释器一般会由编程语言本身提供的库来实现，我们一遍不需要太多关心。但是我还是引入一种应用场景来学习一下，就是一个加减计算器。  

下面从 [golang-design-pattern](https://github.com/senghoo/golang-design-pattern/tree/master/19_interpreter)引入的一个例子。  

## Golang Demo

```go
package interpreter

import (
    "strconv"
    "strings"
)

type Node interface {
    Interpret() int
}

type ValNode struct {
    val int
}

func (n *ValNode) Interpret() int {
    return n.val
}

type AddNode struct {
    left, right Node
}

func (n *AddNode) Interpret() int {
    return n.left.Interpret() + n.right.Interpret()
}

type MinNode struct {
    left, right Node
}

func (n *MinNode) Interpret() int {
    return n.left.Interpret() - n.right.Interpret()
}

type Parser struct {
    exp   []string
    index int
    prev  Node
}

func (p *Parser) Parse(exp string) {
    p.exp = strings.Split(exp, " ")

    for {
        if p.index >= len(p.exp) {
            return
        }
        switch p.exp[p.index] {
        case "+":
            p.prev = p.newAddNode()
        case "-":
            p.prev = p.newMinNode()
        default:
            p.prev = p.newValNode()
        }
    }
}

func (p *Parser) newAddNode() Node {
    p.index++
    return &AddNode{
        left:  p.prev,
        right: p.newValNode(),
    }
}

func (p *Parser) newMinNode() Node {
    p.index++
    return &MinNode{
        left:  p.prev,
        right: p.newValNode(),
    }
}

func (p *Parser) newValNode() Node {
    v, _ := strconv.Atoi(p.exp[p.index])
    p.index++
    return &ValNode{
        val: v,
    }
}

func (p *Parser) Result() Node {
    return p.prev
}

```

```go
package interpreter

import "testing"

func TestInterpreter(t *testing.T) {
    p := &Parser{}
    p.Parse("1 + 2 + 3 - 4 + 5 - 6")
    res := p.Result().Interpret()
    expect := 1
    if res != expect {
        t.Fatalf("expect %d got %d", expect, res)
    }
}

```

## Java Demo

```java
package tech.selinux.design.pattern.behavioral.interpreter;

public interface Node {
  int interpret();
}

```

```java
package tech.selinux.design.pattern.behavioral.interpreter;

public class AddNode implements Node {

  Node left;

  Node right;

  public AddNode(Node left, Node right) {
    this.left = left;
    this.right = right;
  }

  @Override
  public int interpret() {
    return this.left.interpret() + this.right.interpret();
  }
}

```

```java
package tech.selinux.design.pattern.behavioral.interpreter;

public class MinNode implements Node {
  Node left;
  Node right;

  public MinNode(Node left, Node right) {
    this.left = left;
    this.right = right;
  }

  @Override
  public int interpret() {
    return this.left.interpret() - this.right.interpret();
  }
}

```

```java
package tech.selinux.design.pattern.behavioral.interpreter;

public class ValNode implements Node {
  int val;

  public ValNode(int val) {
    this.val = val;
  }

  @Override
  public int interpret() {
    return this.val;
  }
}

```

```java
package tech.selinux.design.pattern.behavioral.interpreter;

import org.apache.commons.lang3.StringUtils;

public class Parser {
  String[] exp;
  int index;
  Node prev;

  public void parse(String expression) {
    exp = StringUtils.split(expression);

    while (true) {
      if (index >= exp.length) {
        return;
      }
      System.out.println(exp[index]);
      switch (exp[index].charAt(0)) {
        case '+':
          index++;
          prev = new AddNode(prev, newValNode());
          break;
        case '-':
          index++;
          prev = new MinNode(prev, newValNode());
          break;
        default:
          prev = newValNode();
          break;
      }
    }
  }

  public Node newValNode() {

    int v = Integer.parseInt(exp[index]);
    index++;

    return new ValNode(v);
  }

  public Node result() {
    return prev;
  }
}

```

```java
package tech.selinux.design.pattern.behavioral.interpreter;

public class Test {

  public static void main(String[] args) {
    Parser parser = new Parser();
    parser.parse("1 + 2 + 3 - 4 + 5 - 6");
    int res = parser.result().interpret();

    System.out.println(res);
  }
}

```