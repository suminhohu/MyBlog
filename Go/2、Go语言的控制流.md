在学习到 Go 的控制流的时候，会发现分支判断和循环相对较少，循环只有 for 循环，没有其他语言那些 while、do while 等；分支语句只有 if 和 switch，也没有三元操作符，并且有一点要注意和其他语言的不同，这三者的条件表达式都不需要括号，且允许初始化表达式。

## if
我们通过一个栗子看看

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
  // 使用时间作为随机种子保证每次随机得到的数不一样
	rand1 := rand.New(rand.NewSource(time.Now().UnixNano()))
	a := rand1.Intn(10)
	fmt.Println("a =", a)
	if b := 4; a > b {
	    fmt.Println("a > b")
	} else if a < b {
	    fmt.Println("a < b")
	} else {
	    fmt.Println("a = b")
    }
    fmt.Println("b=", b) // 编辑器会报错
}
// 暂时只看 if 条件判断内容，这里我们随机生成了一个 10 以内的整数，然后和 b 做比较
// 这里在 if 外打印 b 会报错说明 b 的作用域只是在当前的 if 块中
```

if 语句和其他语言大同小异，刚开始有点不习惯没有括号，而且没有三元运算会显得啰嗦很多；但是一个好处的地方就是条件判断语句里面允许声明一个变量。

## switch 
日常开发中我们通常会把多个条件判断使用 switch 或者映射表来解决面条代码问题，这里看一下 switch 如何解决这个问题。

```go
package main

import "fmt"

func main() {
    fmt.Println(prize1(60))
    fmt.Println(prize2(60))
}

// 值匹配
func prize1(score int) string {
    switch score / 10 {
        case 0, 1, 2, 3, 4, 5:
            return "差"
        case 6, 7:
            return "及格"
        case 8:
            return "良"
        default:
            return "优"
        }
}

// 表达式匹配
func prize2(score int) string {
    // 注意 switch 后面什么也没有
    switch {
        case score < 60:
            return "差"
        case score < 80:
            return "及格"
        case score < 90:
            return "良"
        default:
            return "优"
    }
}
```

从上面的代码我们可以看出，每一个 case 都没有 break，其实是 switch 默认给每个 case 最后带有 break，匹配成功后自动跳出，而不继续执行其他的 case，如果我们想继续执行，Go 也给我们提供了一个关键字 fallthrough 来继续执行后续的 case。

```go
a := 5
switch a {
    case 4:
        fmt.Println("a <= 4")
        fallthrough
    case 5:
        fmt.Println("a <= 5")
        fallthrough
    case 6:
        fmt.Println("a <= 6")
        fallthrough
    case 7:
        fmt.Println("a <= 7")
        fallthrough
    case 8:
        fmt.Println("a <= 8")
        fallthrough
    default:
        fmt.Println("default")
}
// 打印结果
a <= 6
a <= 7
a <= 8
default
```

## for
在 Go 语言中只有一种循环语句会不会不方便呀？其实用了之后发现还好，Go 语言虽然没有提供 while 等语句，不过都可以通过 for 循环的形式来模拟。

```go
// 标准 for 循环
for i := 0; i < 10; i++ {
    fmt.Println("i = ", i)
}
```

```go
// while 循环
for true {
    fmt.Println("while循环")
}
```

```go
//  无限循环
for {
    fmt.Println("无限循环")
}
 
```

以上是 for 循环的三种不同形式，for 循环配合 range 关键字通常用于遍历，我们通过下面一个栗子看一下。

```go
package main

import "fmt"

func main() {
    var s = []int{1,2,3,4,5}
	for i := 0; i < len(s); i++ {
	    fmt.Println(i, s[i])
	}
	fmt.Println("----------")
	for index, value := range s {
	    fmt.Println(index, value)
	}
}
// 打印结果
0 1
1 2
2 3
3 4
4 5
----------
0 1
1 2
2 3
3 4
4 5
```

通过这两种形式我们都可以很方便的遍历数据，得到我们想要的结果。


## 跳转语句 break、continue、goto
在 Go 语言当中，有以上三个跳转语句，和其他语句有所不同的是，在 Go 里面 break 和 continue 可以配合标签跳出多层循环，goto 是调整执行位置，在 Go 语言当中同样是 `不推荐` 使用。这三个关键字都可以配合标签使用，标签必须区分大小写。

```go

func main() {
LABEL:
    for {
        for i := 0; i < 10; i++ {
            if i > 4 {
                break LABEL
            }
        }
    }
    fmt.Println("break 跳出多层循环")
}
// 使用 break 配合标签可以跳出多层循环
```

如果这里 break 改成 continue，结果会不一样

```go

func main() {
LABEL:
    for {
        for i := 0; i < 10; i++ {
            if i > 4 {
                continue LABEL
            }
        }
    }
    fmt.Println("continue 跳出多层循环")
}
// 这里使用 continue 配合标签并不会跳出多层循环，如果想要跳出我们
// 需要改一下代码结构

func main() {
LABEL:
    for i := 0; i < 10; i++ {
        for {
            continue LABEL
        }
    }
    fmt.Println("continue 跳出多层循环")
}
// 将有限循环放到外面，这样就可以跳出多层了，只要确保最外层是有限循环
```

尽管不推荐使用 goto，但是这里如果换成 goto 呢，是否可以跳出多层循环？
```go

func main() {
LABEL:
    for {
        for i := 0; i < 10; i++ {
            if i > 4 {
                goto LABEL
            }
        }
    }
    fmt.Println("goto 跳出多层循环")
}
// 我们发现这里也不可以跳出多层循环，稍微改动一下代码结构就可以

func main() {
    for {
        for i := 0; i < 10; i++ {
            if i > 4 {
                goto LABEL
            }
        }
    }
LABEL:
    fmt.Println("goto 跳出多层循环")
}
```

## 本章小结
到这里 Go 语言的控制流语句基本就讲完了，在 Go 语言中控制流也相对简单，没有其他语言那么多种类，减少了语法学习成本，不过简单归简单，也要注意它的不同之处。接下来我们来学一学 Go 语言的数组吧~

## 导航
+ 上一节：[Go语言的变量与类型](./1、Go语言的变量与类型.md)
+ 下一节：[Go语言的数组](./3、Go语言的数组.md)