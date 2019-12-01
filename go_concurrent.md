# go 并发编程

### 15  goroutine 初识

    package main
    import (
        "fmt"
        "time"
    )

    func main(){
        go pn()
        for i:=0;i<=100;i++ {
            fmt.Printf("\t主goroutine执行A%d\n",i)
        }

        time.Sleep(1*time.Second) //如果不加这个，pn()  估计打没有打印到100就结束了
        fmt.Println("main over")
    }


    func pn(){
        for i:=0;i<=100;i++ {
            fmt.Printf("子goroutine执行%d\n",i)
        }
    }



    package main
    import (
        "fmt"
        "time"
    )

    func main(){
        go nums()
        go letter()
        time.Sleep(3000*time.Millisecond)
        fmt.Println("\tmain ..over")

        //通过时间轴分析结果是： 1       a       2       3       b       4       c       5       d
    }

    func nums(){
        for i:=1;i<=5;i++ {
            time.Sleep(250*time.Millisecond)
            fmt.Printf("\t%d",i)
        }
    }

    func letter(){
        for i:='a';i<='e';i++ {
            time.Sleep(400*time.Millisecond)
            fmt.Printf("\t%c",i)
        }
    }


###  16.go的MPG模型    

### 18.临界资源安全问题，模拟售票，全局变量会出现负数的情况
	package main

	import(
		"fmt"
		"time"
		"math/rand"
	)
	var ticket =10
	func main(){
		go saleTickets("窗口1")
		go saleTickets("窗口2")
		go saleTickets("窗口3")
		go saleTickets("窗口4")
	
		time.Sleep(5*time.Second)
	}
	
	func saleTickets(name string){
		rand.Seed(time.Now().UnixNano())
		for{
			if ticket>0{
				time.Sleep(time.Duration(rand.Intn(1000))*time.Millisecond)
				fmt.Println(name,"售出",ticket)
				ticket--
			}else{
				fmt.Println(name,"卖完了")
				break
			}
		}
	}


### 19 WaitGroup同步等待组

	package main
	
	import(
		"fmt"
		"sync"
	)
	
	
	var wg sync.WaitGroup
	func main(){
		wg.Add(2)  //计数器为2  ，为0，解除阻塞
		go fun1()
		go fun2()
		
		fmt.Println("main阻塞了，")
		wg.Wait()
		fmt.Println("main 解除阻塞")
	}
	
	
	func fun1(){
		for i:=0;i<10;i++{
			fmt.Println("fun1函数",i)
		}
		wg.Done()  //wg.Add(-1)
	}
	
	func fun2(){
		defer wg.Done()
		for i:=0;i<10;i++{
			fmt.Println("fun2函数",i)
		}
	
	}
	

### 20  互斥锁

	package main
	import(
		"fmt"
		"sync"
		"math/rand"
		"time"
	)
	
	var wg sync.WaitGroup
	var mutex sync.Mutex    //互斥锁
	var nums =10
	
	func main(){
		wg.Add(4)
		go sell("窗口1")
		go sell("窗口3")
		go sell("窗口4")
		go sell("窗口5")
	
		wg.Wait()
	
		fmt.Println("main over")
	}
	
	func sell(name string){
		rand.Seed(time.Now().UnixNano())
		defer wg.Done()
	
		for{
			mutex.Lock() //上锁
			if nums>0{
				time.Sleep(time.Duration(rand.Intn(1000))*time.Millisecond)
				fmt.Println(name,"sell",nums)
				nums--
			}else{
				mutex.Unlock()
				fmt.Println(name,"sell finished.")
				break
			}
	
			mutex.Unlock() //解锁,使用完一定要解锁
		}
	}
	

### 21.读写锁
互斥锁的本质是当一个goroutine访问的时候，其他goroutine都不能访问。这样在资源同步，避免竞争的同时也降低了程序的并发性能。程序由原来的并行执行变成了串行执行。

其实，当我们对一个不会变化的数据只做“读”操作的话，是不存在资源竞争的问题的。因为数据是不变的，不管怎么读取，多少goroutine同时读取，都是可以的。

所以问题不是出在“读”上，主要是修改，也就是“写”。修改的数据要同步，这样其他goroutine才可以感知到。所以真正的互斥应该是读取和修改、修改和修改之间，读和读是没有互斥操作的必要的。

因此，衍生出另外一种锁，叫做读写锁。

读写锁可以让多个读操作并发，同时读取，但是对于写操作是完全互斥的。也就是说，当一个goroutine进行写操作的时候，其他goroutine既不能进行读操作，也不能进行写操作。

	package main
	import(
		"fmt"
		"math/rand"
		"sync"
		// "time"
	)
	
	var count int
	var rLock sync.RWMutex
	var wg sync.WaitGroup
	
	func main(){
		wg.Add(10)
	
		for i:=0;i<5;i++{
			go mywrite(i)
		}
	
		for i:=0;i<5;i++{
			go myread(i)
		}
	
		wg.Wait()
	}
	
	
	func myread(i int){
		defer wg.Done()
		rLock.RLock()
		fmt.Printf("读goroutine%d 数据 %d\n",i,count)
		defer rLock.RUnlock()
	}
	
	func mywrite(i int){
		defer wg.Done()
		rLock.Lock()
		count =rand.Intn(1000)
		fmt.Printf("写goroutine%d 数据%d\n",i,count)
		defer rLock.Unlock()
	}

### channle  缓冲  
	package main
	
	
	import(
		"fmt"
		"strconv"
	)
	
	
	func main(){
		ch1 :=make(chan int)
		fmt.Println(len(ch1),cap(ch1))
	
		ch2 := make(chan string,4)
		go send(ch2)
		for{
			data,ok := <- ch2
			if !ok{
				fmt.Println("接收完毕")
				break
			}
	
			fmt.Println(data)
		}
	}
	
	func send(ch chan string){
		for i:=0;i<10;i++{
			ch <- "子goroutine"+strconv.Itoa(i)
			fmt.Println("\t send函数")
		}
		close(ch)
	}


### 26.time包中的通道 ，NewTimer、NewTicker 和time.After

	package main
	
	import (
		"fmt"
		"time"
	)
	func main(){
		//创建一个定时器，设置时间为2S,2s后，往t通道写内容(当前时间)
		t:=time.NewTimer(2*time.Second)
		fmt.Println("当前时间：",time.Now())
	
		//2s后，往t.C写数据，有数据后，就可以读取了
		t1:= <- t.C 
		fmt.Println("t1=",t1)
	
		/*
		output:
		D:\gowork\src\demo\concurrent>go run time1.go
		当前时间： 2019-11-24 17:06:19.7921474 +0800 CST m=+0.008000401
		t1= 2019-11-24 17:06:21.7922618 +0800 CST m=+2.008114801
		*/
	}
	
	func fun2(){
		t:=time.NewTimer(1*time.Second)
		for{
			<- t.C 
			fmt.Println("时间到")
		}
	}


###  27.select语句
一个select语句用来选择哪个case中的发送或接收操作可以被立即执行。它类似于switch语句，但是它的case涉及到channel有关的I/O操作。
或者换一种说法，select就是用来监听和channel有关的IO操作，当 IO 操作发生时，触发相应的动作。语法：

//select基本用法

select {

case <- chan1:

// 如果chan1成功读到数据，则进行该case处理语句

case chan2 <- 1:

// 如果成功向chan2写入数据，则进行该case处理语句

default:

// 如果上面都没有成功，则进入default处理流程

		
	package main
	
	import(
		"fmt"
		"time"
	)
	
	func main(){
		ch1:=make(chan int)
		ch2:=make(chan int)
	
		go func(){
			time.Sleep(1*time.Second)
			ch1<-100
		}()
	
		select{
			case <-ch1:
				fmt.Println("case1..")
			case <- ch2:
				fmt.Println("case22")
			case <- time.After(1*time.Second):
				fmt.Println("time")		
		}

		//结果可能是case1.. 或者 time
	}


### 28.CSP 

### 29 反射  
	package main
	
	import(
		"fmt"
		"reflect"
	)
	
	func main(){
		var x float64 =3.4
	
		fmt.Println("type:",reflect.TypeOf(x))
		fmt.Println("value:",reflect.ValueOf(x))
	
		v := reflect.ValueOf(x)
		fmt.Println("kind is float64:",v.Kind()==reflect.Float64)
		fmt.Println("type--",v.Type())
		fmt.Println("value--",v.Float())
	}

	/*
	输出：
	D:\gowork\src\demo\fs>
	D:\gowork\src\demo\fs>go run one.go
	type: float64
	value: 3.4
	kind is float64: true
	type-- float64
	value-- 3.4
	*/


### 30.reflect对象获取接口变量信息
	
	package main
	
	import(
		"fmt"
		"reflect"
	)
	
	func main(){
		var num float64 =1.23
		//接口类型变量 ->反射类型对象
		value := reflect.ValueOf(num)
	
		fmt.Println(value)
	
		//反射类型对象--》接口类型变量
		cv := value.Interface().(float64)
		fmt.Println(cv)
	
		//反射类型对象-》接口类型变量，理解为强制转换，类型一定要完全符合
		p := reflect.ValueOf(&num)
		cp := p.Interface().(*float64)
		fmt.Println(cp)
	}

	/*
	结果：
	D:\gowork\src\demo\fs>go run two.go
	1.23
	1.23
	0xc0000120a8
	*/


### 30.reflect获取接口变量的信息
	package main
	
	import(
		"fmt"
		"reflect"
	)
	
	func main(){
		p1:=Person{"张三",20}
		fs(p1)
	}
	
	type Person struct{
		Name string
		Age int 
	}
	
	func (p Person) Show(){
		fmt.Printf("name:%s,age:%d\n",p.Name,p.Age)
	}
	
	func (p Person) Hell(msg string){
		fmt.Println("Hello",msg)
	}
	
	//反射
	func fs(obj interface{}){
		t:=reflect.TypeOf(obj)
		fmt.Println("type is:",t.Name())
		fmt.Println("kind is:",t.Kind())
	
		v:=reflect.ValueOf(obj)
		fmt.Println("all Fields:",v)
	
		/*
		获取字段
		第一步：先获取Type对象: reflect.Type   NumField()  有多少个字段
		Field(index)  第几个字段
		第二步：通过Field() 获取每一个Field字段
		第三步：Interface(),得到对应的Value
		*/
	
		for i:=0;i<t.NumField();i++{
			field :=t.Field(i)
			value:=v.Field(i).Interface() //获取对应的数值
			fmt.Printf("字段名：%s,类型：%s,字段值：%v\n",field.Name,field.Type,value)
		}
		fmt.Println("方法总数",t.NumMethod())
		//获取方法
		for i:=0;i<t.NumMethod();i++{
			m:=t.Method(i)
			fmt.Printf("方法名：%s,方法类型:%v\n",m.Name,m.Type)
		}	
	}
	
	/*
	输出：
	D:\gowork\src\demo\fs>go run three.go
	type is: Person
	kind is: struct
	all Fields: {张三 20}
	字段名：Name,类型：string,字段值：张三
	字段名：Age,类型：int,字段值：20
	方法总数 2
	方法名：Hell,方法类型:func(main.Person, string)
	方法名：Show,方法类型:func(main.Person)
	*/



### 31.通过reflect来改变实际变量的值
修改基本数据类型

	package main
	import(
		"fmt"
		"reflect"
	)
	
	func main()  {
		var n float64 =1.23
		//需要指针操作，通过reflect.VlaueOf()获取n的Value对象
		p1:=reflect.ValueOf(&n)
		nv:=p1.Elem() //获取原始值对应的反射对象
		fmt.Println("类型：",nv.Type())  //float64
		fmt.Println("是否可修改数据：",nv.CanSet())  //true
	
		//重新赋值
		nv.SetFloat(3.14)
		fmt.Println(n) //3.14
	
	}


修改结构体类型

	package main
	
	import(
		"fmt"
		"reflect"
	)
	
	func main(){
		stu:=Student{"tom",100}
		//通过反射，获取对象的数值，前提是数据可以被更改
		fmt.Printf("%T\n",stu) //main.Student
		p1:=&stu
		fmt.Printf("%T\n",p1)  //*main.Student
	
		//改变数值
		v:=reflect.ValueOf(p1)
		if v.Kind()==reflect.Ptr{
			nv :=v.Elem() //获取原始对象的反射对象
			fmt.Printf("是否可修改：%v\n",nv.CanSet()) //true
	
			f1 :=nv.FieldByName("Name")
			f1.SetString("张三")
			f2:=nv.FieldByName("Age")
			f2.SetInt(33)
	
			fmt.Println(stu)  //{张三 33}
		}
	}
	
	
	type Student struct{
		Name string
		Age int
	}



### 32. 反射对方法和函数的调用
掉用方法

	package main
	import(
		"fmt"
		"reflect"
	)
	func main(){
		p:=Person{"张三",100}
		/*
			通过反射来进行方法调用
			1.接口变量-》反射对象： Value
			2. 获取对应的方法对象： MethodByName()
			3. 将方法对象进行调用： Call()
		*/
	
		value:=reflect.ValueOf(p)
		fmt.Printf("kind:%s,type:%s\n",value.Kind(),value.Type()) //kind:struct,type:main.Person
	
		m1:=value.MethodByName("Show")
		fmt.Printf("kind:%s,type:%s\n",m1.Kind(),m1.Type())  //kind:func,type:func()
	
		//没有参数，进行调用
		m1.Call(nil)  //name: 张三  age: 100
	
		args :=make([] reflect.Value,0)
		m1.Call(args) //name: 张三  age: 100
	
		m2:=value.MethodByName("Msg")
		fmt.Printf("kind:%s,type:%s\n",m2.Kind(),m2.Type())  //kind:func,type:func(string)
		arg2:=[]reflect.Value{reflect.ValueOf("反射例子")}
		m2.Call(arg2)//hello  反射例子
	
		m3:=value.MethodByName("Num")
		fmt.Printf("kind:%s,type:%s\n",m3.Kind(),m3.Type())  //kind:func,type:func(int, int, string)
		arg3:=[]reflect.Value{reflect.ValueOf(100),reflect.ValueOf(200),reflect.ValueOf("我的")}
		m3.Call(arg3)  //100 200 我的
	}
	
	type Person struct{
		Name string
		Age int
	}
	
	func (p Person) Show(){
		fmt.Println("name:",p.Name," age:",p.Age)
	}
	
	func (p Person) Msg(m string){
		fmt.Println("hello ",m)
	}
	
	func (p Person) Num(i,j int,k string){
		fmt.Println(i,j,k)
	}


调用函数
	
	package main
	
	import (
		"fmt"
		"strconv"
		"reflect"
	)
	
	func main(){
		/*
		通过反射调用函数，思路
		1.函数-》反射对象，Value
		2.kind->func
		3.Call()
		*/
	
		f1:=fun1 
		value:=reflect.ValueOf(f1)
		fmt.Printf("kind:%s,type:%s\n",value.Kind(),value.Type())  //kind:func,type:func()
	
		value2:=reflect.ValueOf(fun2)
		fmt.Printf("kind:%s,type:%s\n",value2.Kind(),value2.Type()) //kind:func,type:func(int, string)
	
		v3:=reflect.ValueOf(fun3)
		fmt.Printf("kind:%s,type:%s\n",v3.Kind(),v3.Type())  //	kind:func,type:func(int, string) string
		
		//通过反射调用函数
		value.Call(nil) //我是无参的fun1
		value2.Call([]reflect.Value{reflect.ValueOf(100),reflect.ValueOf("zch")})  //我是有参的fun2
	
		rsv:=v3.Call([]reflect.Value{reflect.ValueOf(11),reflect.ValueOf("tom")})
		fmt.Printf("%T\n",rsv) //[]reflect.Value
		fmt.Println(len(rsv))  //1
		fmt.Printf("kind:%s,type:%s\n",rsv[0].Kind(),rsv[0].Type())  //kind:string,type:string
	
		s:=rsv[0].Interface().(string)
		fmt.Println(s)  //tom11
		fmt.Printf("%T\n",s)  //string
	}
	
	func fun1(){
		fmt.Println("我是无参的fun1")
	}
	
	func fun2(i int,s string){
		fmt.Println("我是有参的fun2")
	}
	
	func fun3(i int,s string)string{
		fmt.Println("我是有参的fun3")
		return s+strconv.Itoa(i)
	}
	
	
	
	
### strconv学习
	package main
	
	import (
		"fmt"
		"strconv"
	)
	func main() {
		//string->int
		i,_:=strconv.Atoi("3")  //3
	
		fmt.Println(i)
	
		//int->string
		fmt.Println("a"+strconv.Itoa(32))  //a32
	
		//parse类函数用于转换字符串为给定类型的值：ParseBool()、ParseFloat()、ParseInt()、ParseUint()。
	
		b,_:=strconv.ParseBool("true")
	
		fmt.Println(b)  //true
	
		f,_:=strconv.ParseFloat("3.1415", 64)
		fmt.Println(f)  //3.1415
	
		i1,_:=strconv.ParseInt("-42",10,64)
		fmt.Println(i1)  //-42
	
		//将给定类型格式化为string类型：FormatBool()、FormatFloat()、FormatInt()、FormatUint()。
		s1:=strconv.FormatBool(false)
		fmt.Println(s1)  //false
	
		s2:=strconv.FormatFloat(3.1415, 'E', -1, 64)
		fmt.Println(s2)  //	3.1415E+00
	 
		s3:=strconv.FormatInt(-42, 16)  //表示将-42转换为16进制数，转换的结果为-2a。
	
		fmt.Println(s3)  //-2a
	
	
	}
	
	
学习连接：https://www.cnblogs.com/f-ck-need-u/p/9863915.html
Go不会对数据进行隐式的类型转换，只能手动去执行转换操作。

简单的转换操作
转换数据类型的方式很简单。

valueOfTypeB = typeB(valueOfTypeA)
例如：

1
2
3
4
5
// 浮点数
a := 5.0

// 转换为int类型
b := int(a)
Go允许在底层结构相同的两个类型之间互转。例如：

1
2
3
4
5
6
7
8
9
10
11
// IT类型的底层是int类型
type IT int

// a的类型为IT，底层是int
var a IT = 5

// 将a(IT)转换为int，b现在是int类型
b := int(5)

// 将b(int)转换为IT，c现在是IT类型
c := IT(b)
但注意：

不是所有数据类型都能转换的，例如字母格式的string类型"abcd"转换为int肯定会失败
低精度转换为高精度时是安全的，高精度的值转换为低精度时会丢失精度。例如int32转换为int16，float32转换为int
这种简单的转换方式不能对int(float)和string进行互转，要跨大类型转换，可以使用strconv包提供的函数
strconv
strconv包提供了字符串与简单数据类型之间的类型转换功能。可以将简单类型转换为字符串，也可以将字符串转换为其它简单类型。

这个包里提供了很多函数，大概分为几类：

字符串转int：Atoi()
int转字符串: Itoa()
ParseTP类函数将string转换为TP类型：ParseBool()、ParseFloat()、ParseInt()、ParseUint()。因为string转其它类型可能会失败，所以这些函数都有第二个返回值表示是否转换成功
FormatTP类函数将其它类型转string：FormatBool()、FormatFloat()、FormatInt()、FormatUint()
AppendTP类函数用于将TP转换成字符串后append到一个slice中：AppendBool()、AppendFloat()、AppendInt()、AppendUint()
还有其他一些基本用不上的函数，见官方手册：go doc strconv或者https://golang.org/pkg/strconv/。

当有些类型无法转换时，将报错，返回的错误是strconv包中自行定义的error类型。有两种错误：

1
2
var ErrRange = errors.New("value out of range")
var ErrSyntax = errors.New("invalid syntax")
例如，使用Atoi("a")将"a"转换为int类型，自然是不成功的。如果print输出err信息，将显示：

strconv.Atoi: parsing "a": invalid syntax
string和int的转换
最常见的是字符串和int之间的转换：

1.int转换为字符串：Itoa()

1
2
// Itoa(): int -> string
println("a" + strconv.Itoa(32))  // a32
2.string转换为int：Atoi()

func Atoi(s string) (int, error)
由于string可能无法转换为int，所以这个函数有两个返回值：第一个返回值是转换成int的值，第二个返回值判断是否转换成功。

1
2
3
4
5
6
7
8
9
// Atoi(): string -> int
i,_ := strconv.Atoi("3")
println(3 + i)   // 6

// Atoi()转换失败
i,err := strconv.Atoi("a")
if err != nil {
    println("converted failed")
}
Parse类函数
Parse类函数用于转换字符串为给定类型的值：ParseBool()、ParseFloat()、ParseInt()、ParseUint()。

由于字符串转换为其它类型可能会失败，所以这些函数都有两个返回值，第一个返回值保存转换后的值，第二个返回值判断是否转换成功。

1
2
3
4
b, err := strconv.ParseBool("true")
f, err := strconv.ParseFloat("3.1415", 64)
i, err := strconv.ParseInt("-42", 10, 64)
u, err := strconv.ParseUint("42", 10, 64)
ParseFloat()只能接收float64类型的浮点数。

ParseInt()和ParseUint()有3个参数：

1
2
func ParseInt(s string, base int, bitSize int) (i int64, err error)
func ParseUint(s string, base int, bitSize int) (uint64, error)
bitSize参数表示转换为什么位的int/uint，有效值为0、8、16、32、64。当bitSize=0的时候，表示转换为int或uint类型。例如bitSize=8表示转换后的值的类型为int8或uint8。

base参数表示以什么进制的方式去解析给定的字符串，有效值为0、2-36。当base=0的时候，表示根据string的前缀来判断以什么进制去解析：0x开头的以16进制的方式去解析，0开头的以8进制方式去解析，其它的以10进制方式解析。

以10进制方式解析"-42"，保存为int64类型：

i, _ := strconv.ParseInt("-42", 10, 64)
以5进制方式解析"23"，保存为int64类型：

1
2
i, _ := strconv.ParseInt("23", 5, 64)
println(i)    // 13
因为5进制的时候，23表示进位了2次，再加3，所以对应的十进制数为5*2+3=13。

以16进制解析23，保存为int64类型：

1
2
i, _ := strconv.ParseInt("23", 16, 64)
println(i)    // 35
因为16进制的时候，23表示进位了2次，再加3，所以对应的十进制数为16*2+3=35。

以15进制解析23，保存为int64类型：

1
2
i, _ := strconv.ParseInt("23", 15, 64)
println(i)    // 33
因为15进制的时候，23表示进位了2次，再加3，所以对应的十进制数为15*2+3=33。

Format类函数
将给定类型格式化为string类型：FormatBool()、FormatFloat()、FormatInt()、FormatUint()。

1
2
3
4
s := strconv.FormatBool(true)
s := strconv.FormatFloat(3.1415, 'E', -1, 64)
s := strconv.FormatInt(-42, 16)
s := strconv.FormatUint(42, 16)
FormatInt()和FormatUint()有两个参数：

1
2
func FormatInt(i int64, base int) string
func FormatUint(i uint64, base int) string
第二个参数base指定将第一个参数转换为多少进制，有效值为2<=base<=36。当指定的进制位大于10的时候，超出10的数值以a-z字母表示。例如16进制时，10-15的数字分别使用a-f表示，17进制时，10-16的数值分别使用a-g表示。

例如：FormatInt(-42, 16)表示将-42转换为16进制数，转换的结果为-2a。

FormatFloat()参数众多：

func FormatFloat(f float64, fmt byte, prec, bitSize int) string
bitSize表示f的来源类型（32：float32、64：float64），会据此进行舍入。

fmt表示格式：'f'（-ddd.dddd）、'b'（-ddddp±ddd，指数为二进制）、'e'（-d.dddde±dd，十进制指数）、'E'（-d.ddddE±dd，十进制指数）、'g'（指数很大时用'e'格式，否则'f'格式）、'G'（指数很大时用'E'格式，否则'f'格式）。

prec控制精度（排除指数部分）：对'f'、'e'、'E'，它表示小数点后的数字个数；对'g'、'G'，它控制总的数字个数。如果prec 为-1，则代表使用最少数量的、但又必需的数字来表示f。

Append类函数
AppendTP类函数用于将TP转换成字符串后append到一个slice中：AppendBool()、AppendFloat()、AppendInt()、AppendUint()。

Append类的函数和Format类的函数工作方式类似，只不过是将转换后的结果追加到一个slice中。

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // 声明一个slice
    b10 := []byte("int (base 10):")
    
    // 将转换为10进制的string，追加到slice中
    b10 = strconv.AppendInt(b10, -42, 10)
    fmt.Println(string(b10))

    b16 := []byte("int (base 16):")
    b16 = strconv.AppendInt(b16, -42, 16)
    fmt.Println(string(b16))
}
输出结果：

1
2
int (base 10):-42
int (base 16):-2a
 
	
	
	
	
	