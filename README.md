# Go代码规范草案
## 目录

## 前提
- 代码经由`gofmt`或`goimports`工具格式化
- 代码已通过[golint](https://github.com/golang/lint)工具检查

## 代码规范
### 声明空的slice
#### Don't
``` go
s := []string{}
```
#### Do
``` go
var s []string
```

### 除非性能敏感且必须，否则不建议使用uint
需要整形数据类型情况下，多数都可以直接使用`int`。标准库及其他第三方的类库，在类似的使用场景下多使用`int`。那么，此种`随大流`可以免去不必要的数据类型转换。

#### Do
``` go
type Person struct{
	Age int
}
```

#### Don't
``` go
type Person struct{
	Age uint
}
```

### 错误处理
一般情况下，都需要对函数的返回值error进行逻辑判断。

如果函数返回的错误确实可以不处理，那么请用`_`明确告知他人`并非是忘记处理error而是将error丢弃`。丢弃error，本身也是一种处理方式。
#### Don't
TODO 

#### Do
``` go
scores := map[string]int{
	"jim": 8,
	"jerry": 7,
	"tom": 3,
}

b,_ := json.Marshal(scores) // TODO 换个更好的例子
fmt.Printf("%s", b)
```

### 错误字符串
仅能使用英文系字符且以小写字母开头
#### Don't
``` go
var (
	ErrInvalidIP = errors.New("无效的IP")
	ErrInvalidMacAddr = errors.New("Invalid mac address")
)
```

#### Do
``` go
var (
	ErrInvalidIP = errors.New("invalid ip")
	ErrInvalidMacAddr = errors.New("invalid mac address")
)
```

不要以标点符号结尾
#### Don't
``` go
var (
	ErrInvalidIP = errors.New("invalid ip!")
)
```

#### Do
``` go
var (
	ErrInvalidIP = errors.New("invalid ip")
)
```

### 谨慎使用panic
在一般业务代码中，建议使用`error`而非`panic`。

#### Don't
``` go
func (repo model.Repo) GetUserByName(name string)(user *model.User){
	if name == ""{
		panic("user name can't be empty")
	}
	...
}
```
#### Do
``` go
var ErrEmptyUserName = errors.New("user name can't be empty")

func (repo model.Repo) GetUserByName(name string)(user *model.User, err error){
	if name == ""{
		return nil, ErrEmptyUserName
	}
	...
}
```

对于那些可以被认为是`绝对不可能发生的事`，可谨慎使用`panic`。

#### Do
``` go
func week(n int)string{
	switch n{
	case 0:
		return "Sunday"
	case 1:
		return "Monday"
	case 2:
		return "Tuesday"
	case 3:
		return "Wednesday"
	case 4:
		return "Thursday"
	case 5:
		return "Friday"
	case 6:
		return "Saturday"
	}
	panic("unreachable")
}
```

- 对于`error`或`panic`模棱两可的情况，不用犹豫，选`error`。


## 参考
- [Go Styleguide](https://github.com/bahlo/go-styleguide)