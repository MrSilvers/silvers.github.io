#  关于Go语言中结构体方法和接口
做Go项目的时候，遇到Go语言中的结构体方法和interface方法的实现，发现两者之间看起来很像，但用起来却不一样，经过大量的资料查阅和思考，整理和记录了关于结构体方法和interface的一些理解，希望能帮助一些人理解这些概念和用法。以下是具体的例子和例子中表现的问题  
##  一、结构体方法
首先定义一个名为Person的结构体，我们暂时可把它视为Person类
```go
type Person struct {
	name string
}
```
它只有一个name成员。在面向对象的角度来说，我们可以对name这个成员变量进行set和get，用Java里面就是定义getName和setName方法；在GO里面我们可以这样实现GetName方法：
```go
func (p Person) GetName() string {
	return p.name
}
```
类似的，我们去这样定义SetName方法
```go
func (p Person) SetName(name string) {
	p.name = name
}
```
我们写一个例子去实例化Person类并且调用GetName和SetName方法
```go
func main() {
	var Tom Person // 声明一个Person
	Tom.SetName("Tom") 
	fmt.Println(Tom.GetName())
}

```
运行代码，结果如下：
```shell
silvers@HP-ZHAN:~/Desktop$ go run Go_Understandable.go 

silvers@HP-ZHAN:~/Desktop$ 
```
结果什么也没有打印，我们先不急着解释，我们尝试把刚刚定义的SetName方法改一下：
```go
func (p *Person) SetName(name string) {
	p.name = name
}
```
和刚刚不一样的地方是我们把方法的接收者（或者叫方法的绑定对象/目标）改成了Person类型的指针，经常用C语言的人可能马上明白了刚刚为什么没有打印任何数据，下面我们继续去运行一下代码，结果如下
```shell
silvers@HP-ZHAN:~/Desktop$ go run Go_Understandable.go 
Tom
silvers@HP-ZHAN:~/Desktop$ 
```
我们SetName方法使用了Person类型的指针作为接收者，或者说SetName和Person类型的指针绑定后，返回了期望的结果。下面我对此进行一些解释：
站在面向对象的角度说，一个类的成员方法可以读/写本类的成员变量，java中，类把成员变量和方法都封装在一起，很容易的实现了对成员变量读取和修改控制，但GO里面没有类的概念，它使用结构体来模拟类的概念，但结构体里面不能直接定义方法，因此GO不能向java一样把实现成员方法和成员变量封装在一起，GO采用的是类似于“绑定”的策略，例如刚刚举例的:
`func (p Person) SetName() { p.name = name }`
这种分离式的‘成员方法’访问成员变量必然会产生变量作用域的问题，注意到实际调用结构体方法的时候，我们先声明了一个结构体变量，然后直接使用‘.’操作符调用结构体方法：
```go
	var Tom Person // 声明一个Person
	Tom.SetName("Tom") 
```
这个Tom变量的作用范围是main方法内，调用SetName方法的时候相当于把Tom当作实参传递给SetName方法，这时SetName方法的参数列表可以想象成是
`SetName(Tom, "Tom")`
换成这样就好理解了，Tom是实参，而SetName方法的修改的是形参p的name属性，这种值传递的操作不会影响到实参Tom，也就是说Tom传递到SetName方法的时候，SetName拷贝了一份Tom的内容到p变量，所以修改的操作之修改了拷贝的内容，并不会影响到实参Tom，所以我们第一次打印的时候什么也没有打印出来。
细心的朋友可能会发现了一个问题：第二次的打印的时候，我们调用SetName的形式没有变，还是：
```go
	var Tom Person // 声明一个Person
	Tom.SetName("Tom") 
```
按照刚刚的理解，SetName方法应该接收一个Person类型的指针，但我们这里调用的时候并没有指针，我们还是Person类型的Tom变量调用的方法，这样类型不就不匹配了吗？我们怎么可以用指针类型的形参接收一个非指针类型的变量？
其实我刚刚只是这样类比，试图方便大家理解，并不代表Go一定是这样实现的，但如果一定要按照我刚才的类比来解释的话，我们可以这样理解：
在使用Tom调用SetName方法的时候，Go自动为我们取了Tom的地址传递给SetName。这种取地址的操作的代价是很小的，而且可以很轻易的去实现。这时，SetName的参数列表变为:
`SetName(&Tom, "Tom")`
所以可以正确的使用。反过来，有人可能会问，如果定义的是
`func (p Person) SetName() { p.name = name }`
但是调用的时候使用指针不就可以了？如我们这样调用：
```go
	var Tom *Person = &Person{}
	Tom.SetName("Tom")
	fmt.Println(Tom.GetName())
```
运行：
```shell
silvers@HP-ZHAN:~/Desktop$ go run Go_Understandable.go 

silvers@HP-ZHAN:~/Desktop$ 
```
结果令人遗憾。一样的，我们可以这么理解：在把参数传递给SetName的时候，Go又‘聪明的’帮你把指针指向的内存空间的值取出来了，同时把值拷贝一份给形参p，所以你对Tom的修改没有生效，也没有引发类型错误。
其实站在设计者的角度看，他们希望我们使用指针接收者去控制结构体成员变量的可写操作，不需要写操作的结构体方法使用非指针类型作为接收者，或者处于节约内存考虑，可能也会定义一些使用指针类型作为接收者的方法。

##  二、关于interface
下面我们看一下go里面的interface
首先我们新定义一个interface
```go
type Student interface {
	GetClass() string
	SetClass(class string)
}
```
定义的结构体：
```go
type Person struct {
	name string
	class string
}
```
然后我们实现Student这个interface，有了上面的知识后，我们可以直接定义出get方法和恰当的set方法
```go
func (p Person) GetClass() string {
	return p.class
}

func (p *Person) SetClass(class string) {
	p.class = class
}
```
下面我们在main里面调用
```go
func main(){
	var Tom Student = Person{}
	Tom.SetClass("12")
	fmt.Println(Tom.GetClass())
}
```
接着我们运行一下代码：
```shell
silvers@HP-ZHAN:~/Desktop$ go run Go_Understandable.go 
# command-line-arguments
./Go_Understandable.go:42: cannot use Person literal (type Person) as type Student in assignment:
	Person does not implement Student (SetClass method has pointer receiver)
silvers@HP-ZHAN:~/Desktop$
```
发现报错了，查看原因：Person does not implement Student (SetClass method has pointer receiver)，意思就是Person没有实现Student接口，其中还指出了SetClass需要‘绑定’的是一个指针类型的接收者。按照第一节的理解，我们直接用结构体变量调用SetClass方法应该没有问题，如下：
```go
func main(){
	var p = Person{}
	var Tom = p
	Tom.SetClass("12")
	fmt.Println(Tom.GetClass())
}
```
结果：
```shell
silvers@HP-ZHAN:~/Desktop$ go run Go_Understandable.go 
12
silvers@HP-ZHAN:~/Desktop$
```
符合预期。但为什么换成interface就不行了呢？
再看一个调用的例子：
```go
func main(){
	var p = &Person{}
	var Tom Student = p
	Tom.SetClass("12")
	fmt.Println(Tom.GetClass())
}
```
运行结果如下：
```shell
silvers@HP-ZHAN:~/Desktop$ go run Go_Understandable.go 
12
silvers@HP-ZHAN:~/Desktop$ 
```
结果正确，我们发现两次的不同点仅仅在与调用SetClass方法的时候，调用者是一个指针还是一个interface变量。下面我尝试进行一些解释：
我们把初始化的结构体赋值给interface的时候：
`var Tom Student = Person{}`
实际上进行了一次值的拷贝，这条语句执行完后，内存里现在存在两份初始化后的Person，一个是赋值操作之前创建的Person类型的匿名变量，另一个是赋值操作拷贝的Person。调用的时候，按照第一节的类比，SetClass方法的参数列表的应该是这样的SetClass(&Tom, "12")，问题就出现在这里了：针对于拷贝的Tom（内容是Person），我们对它的任何操作都不会影响到我们最原始的Person（赋值前创建的Person类型的匿名变量），这和上一节谈到的“结构体方法通过设定接收者是指针类型还是普通结构体类型来控制结构体成员变量的写入权限”这一设计逻辑冲突，所以编译器在这里就报错了。
而使用：
`var p = &Person{}`
`var Tom Student = p`
这种情况，实际上也进行了一次拷贝操作，只是这次拷贝的内容是指向Person结构体初始化后的地址（也就是说，\*Tom和\*p的结果是一样的），所以，对于Tom的操作同样可以作用于p，符合“结构体方法通过设定接收者是指针类型还是普通结构体类型来控制结构体成员变量的写入权限”这一设计逻辑，因此这种用法可行。