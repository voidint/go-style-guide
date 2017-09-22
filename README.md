# Go代码规范(beta)
## 目录
- [前提](#前提)
- [代码规范](#代码规范)
	- [import](#import)
	- [return](#return)
	- [不建议使uint](#不建议使uint)
	- [变量声明](#变量声明)
	- [变量导出](#变量导出)
	- [变量命名](#变量命名)
	- [package命名](#package命名)
	- [interface命名](#interface命名)
	- [源代码文件命名](#源代码文件命名)
	- [error处理](#error处理)
	- [error文本内容](#error文本内容)
	- [panic与error](#panic与error)
	- [字符串拼接](#字符串拼接)
	- [receiver类型](#receiver类型)
	- [控制结构](#控制结构)
- [参考](#参考)

## 前提
- 代码经由[gofmt](https://github.com/golang/go/tree/master/src/cmd/gofmt)或[goimports](https://github.com/golang/tools/tree/master/cmd/goimports)工具格式化
- 代码已通过[golint](https://github.com/golang/lint)或[gometalinter](https://github.com/alecthomas/gometalinter)工具检查

## 代码规范
### import
不要使用相对路径，使用绝对路径。
##### Don't
``` go
import ../config
```
##### Do
``` go
import github.com/voidint/gbb/config
```

### return
##### Don't
``` go
func run() (n int, err error) {
	// ...
	return
}
```

##### Do
``` go
func run() (n int, err error) {
	// ...
	return n, err
}
```

### 不建议使uint
需要使用整形数据类型情况下，多数都可以直接使用`int`，而不建议使用`uint`。标准库及其他第三方的类库，在类似的使用场景下多使用`int`。那么，此种`随大流`可以免去不必要的数据类型转换。

##### Do
``` go
type Person struct{
	Age int
}
```

##### Don't
``` go
type Person struct{
	Age uint
}
```

### 变量声明
空slice的声明
##### Don't
``` go
s := []string{} // 声明并初始化变量
```
##### Do
``` go
var s []string // 声明
```

函数内的变量声明，推荐使用`:=`的简短赋值语句。
##### Don't
``` go
func saySomething() {
	var msg = getMsg()
	fmt.Println(msg)
}
```
##### Do
``` go

func saySomething() {
	msg := getMsg()
	fmt.Println(msg)
}
```


### 变量导出
函数、方法、结构体、结构体属性等导出与否，由使用场景决定，但秉承着`权限最小化`的原则，能不导出就不要导出。

##### Don't
``` go
type Student struct{
	Name string
}

stu := Student{
	Name: "voidint",
}

func main(){
	fmt.Printf("student name: %s\n", stu.Name)
}
```

##### Do
``` go
type Student struct{
	name string
}

func NewStudent(name string) *Student{
	return &Student{name: name}
}

func (stu Student) Name() string{
	return stu.name
}

func main(){
	fmt.Printf("student name: %s\n", NewStudent("voidint").Name())
}
```

### 变量命名
使用`驼峰式`的变量命名方式。
##### Don't
``` go
var	student_name string
const LOG_LEVEL = "info"
```

##### Do
``` go
var	studentName string
const LogLevel = "info"
```

专有名词的缩写应保持字母大小写一致。具体是全部大写还是全部小写，应由实际使用场景决定。若变量可以不导出，那就全部小写，反之全部大写。
##### Don't
``` go
var (
	Cpu string 
	Id int
)
```

##### Do
``` go
var (
	CPU string 
	id int
)
```

形参变量和函数返回值变量一律小写字母开头。
##### Don't
``` go
func add(X, Y int) (Z int) {
	return X + Y
}
```

##### Do
``` go
func add(x, y int) (z int) {
	return x + y
}
```

为函数的多个返回值命名。在`godoc`生成的文档中，带有返回值的函数声明更利于理解。
##### Don't
``` go
func SplitHostPort(hostport string) (string, string, error){
...
}
```

##### Do
``` go
func SplitHostPort(hostport string) (host, port string, err error){
...
}
```

### package命名
包名由小写字母组成，不要使用下划线或者混合大小写。
##### Don't
``` go
package busyBox
```

##### Do
``` go
package busybox
```

所有对包内的引用都应该使用包名去访问，因此包内的名称引用可以去掉包名这个标识。
##### Don't
``` go
package chubby

type ChubbyFile struct{
}
```

##### Do
``` go
package chubby

type File struct{
}
```

### interface命名
理想情况下，接口命名以`er`结尾。
##### Don't
``` go
type IRead interface{
}
```

##### Do
``` go
type Reader interface{
}
```


### 源代码文件命名
遵循简洁的原则，对于包内的源代码文件命名，一般情况下无需再携带包信息。
##### Don't
``` go
// service/network_service.go
package service
```

##### Do
``` go
// service/network.go
package service
```


### error处理
一般情况下，都需要对函数的返回值error进行逻辑判断。

如果函数返回的错误确实可以不处理，那么请用`_`明确告知他人`并非是忘记处理error而是将error丢弃`。丢弃error，本身也是一种处理方式。
##### Don't
``` go

```

##### Do
``` go
scores := map[string]int{
	"jim": 8,
	"jerry": 7,
	"tom": 3,
}

b,_ := json.Marshal(scores) // TODO 换个更好的例子
fmt.Printf("%s", b)
```

### error文本内容
仅能使用英文系字符且以小写字母开头
##### Don't
``` go
var (
	ErrInvalidIP = errors.New("无效的IP")
	ErrInvalidMacAddr = errors.New("Invalid mac address")
)
```

##### Do
``` go
var (
	ErrInvalidIP = errors.New("invalid ip")
	ErrInvalidMacAddr = errors.New("invalid mac address")
)
```

不要以标点符号结尾
##### Don't
``` go
var (
	ErrInvalidIP = errors.New("invalid ip!")
)
```

##### Do
``` go
var (
	ErrInvalidIP = errors.New("invalid ip")
)
```

### panic与error
在一般业务代码中，建议使用`error`而非`panic`。

##### Don't
``` go
func (repo model.Repo) GetUserByName(name string)(user *model.User){
	if name == ""{
		panic("user name can't be empty")
	}
	...
}
```
##### Do
``` go
var ErrEmptyUserName = errors.New("user name can't be empty")

func (repo model.Repo) GetUserByName(name string)(user *model.User, err error){
	if name == ""{
		return nil, ErrEmptyUserName
	}
	...
}
```

有时候为了降低函数的错误处理成本，可以把函数的`error`返回值移除。而函数一旦发生错误，则发生`panic`。这种场景要求函数名称以`Must`开头，并且在注释中明确告知此函数的`副作用`。

##### Do
``` go
package regexp

// MustCompile is like Compile but panics if the expression cannot be parsed.
// It simplifies safe initialization of global variables holding compiled regular
// expressions.
func MustCompile(str string) *Regexp {
  	regexp, error := Compile(str)
  	if error != nil {
  		panic(`regexp: Compile(` + quote(str) + `): ` + error.Error())
  	}
  	return regexp
}
```

对于那些可以被认为是`绝对不可能发生的事`，可谨慎使用`panic`。

##### Do
``` go
type Season int

const (
	Spring Season = iota
	Summer
	Fall
	Winter
)

func SeasonName(season Season) string {
	switch season {
	case Spring:
		return "Spring"
	case Summer:
		return "Summer"
	case Fall:
		return "Fall"
	case Winter:
		return "Winter"
	}
	panic("unreachable") 
}
```

- 对于`error`、`panic`模棱两可的情况，就选`error`。

### 字符串拼接
多个字符串拼接时不要使用`+`，建议使用`fmt.Sprintf`或者`bytes.Buffer`。
##### Don't
``` go
func (stu Student) String() string {
	return stu.Num + " " + stu.Name + " " + stu.Age
}
```

##### Do
``` go
func (stu Student) String() string {
	return fmt.Sprintf("%s %s %d", stu.Num, stu.Name, stu.Age)
}

// OR

func (stu Student) String() string {
	var buf bytes.Buffer
	buf.WriteString(stu.Num)
	buf.WriteString(" ")
	buf.WriteString(stu.Name)
	buf.WriteString(" ")
	buf.WriteString(stu.Age)
	return buf.String()
}
```

### receiver类型
如果receiver是map,func,chan，不使用指针。

如果receiver是slice,当方法不会重组或重新分配切片，不使用指针。

如果方法需要改变receiver,必须使用指针。

当receiver是包含锁或同步字段时，必须使用指针以避免复制。

对于大的结构体或数组，指针更加的高效。

当外面的改动必须影响到原始的receiver时，必须使用指针。

最后，如果怀疑，那么请使用指针。

### 控制结构
将`变量作用域`控制在最小范围内。
##### Don't
``` go
err := file.Chmod(0664)
if err != nil {
    return err
}
```

##### Do
``` go
if err := file.Chmod(0664); err != nil {
    return err
}
```

应尽早return结束代码块。尽量不要将语句嵌入代码块中，以避免层次过深，增加复杂度。
##### Don't
``` go
func gender(female bool) (desc string){
	if female {
		return "female"
	} else {
		return "male"
	}
}
```

##### Do
``` go
func gender(female bool) (desc string){
	if female {
		return "female"
	}
	return "male"
}
```

判断结构中比较布尔类型变量
##### Don't
``` go
func gender(female bool) (desc string){
	if female == true {
		return "female"
	}
	return "male"
}
```

##### Do
``` go
func gender(female bool) (desc string){
	if female{
		return "female"
	}
	return "male"
}
```

遍历slice时应尽量使用下标访问元素，避免不必要的内存拷贝。
##### Don't
``` go
type BigStruct struct{
	ID int
}

items := []BigStruct{
	{ID: 0},
	{ID: 1},
}

for _, item := range items{ // 每次都会将下标元素拷贝至变量item
	fmt.Println(item.ID)
}
```

##### Do
``` go
type BigStruct struct{
	ID int
}

items := []BigStruct{
	{ID: 0},
	{ID: 1},
}

for i := range items{ // 通过下标访问slice中的元素，无拷贝
	fmt.Println(items[i].ID)
}
```



## 参考
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Go Code Review Comments译文](https://github.com/wangming1993/issues/issues/42)
- [Go Styleguide](https://github.com/bahlo/go-styleguide)
- [Golang编码规范](https://segmentfault.com/a/1190000000464394)