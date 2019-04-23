# LearningGoTogether
### 一、前言
##### 1.1 名称:名称的开头必须是字母(unicode字符)或下划线,后面可以跟任意数量的字符、数字和下划线，并区分大小写。
##### 1.2 关键字(25个)
        break    default    func      interface     select
        case     defer      go        map           struct 
        chan     slse       goto      package       switch 
        cons     fallthough if        range         type
        continue for        import    return        var
        
        注释:
        引导程序整体结构的8个关键字
            package //定义包名的关键字
            import  //导入包名的关键字
            const   //常量声明的关键字
            var     //变量声明关键字
            func    //函数定义关键字
            defer   //延迟执行关键字
            go      //并发语法糖关键字
            return  //函数返回关键字
        声明复合数据结构的4个关键字
            struct  //定义结构类型关键字
            interface //定义接口类型关键字
            map       //声明或创建map关键字
            chan      //声明或创建通道类型关键字
        控制程序结构的13个关键字
            if else  //if else语句关键字
            for range break continue //for循环使用的关键字
            switch select type case default fallthough //switch 和select语句使用的关键字
            goto        //goto跳转语句关键字
            
##### 1.3 预声明常量、类型和函数
        常量: true false iota   nil
        类型: int  int8  int16  int32   int64 
             uint  uint8  uint16  uint32   uint64 uintptr
             float32 float64 complex128 complex64
              bool type rune string error
        函数: make len cap new append copy close delete
             complex real imag
             panic recover
##### 1.4 public vs private 
        如果一个实体包的函数中声明,它只在函数局部中有效。
        如果一个实体在包的函数外,它在包中有效.
        第一个单词大写，在包外也能访问。小写只能在包内访问
##### 1.5 声明
      变量 var
      常量 const
      类型 type
      函数 func
##### 1.6 变量
      var name type = expression
      ① var a int = 5
      ② var a int
         a = 5
      ③ var a, b, c = 1, 2, 3
      ④ i := 100  ( 只能在函数内部使用)
##### 1.7 类型声明
      type name underlying-type
      type myint int64
##### 1.8 包和文件
        package 包名
        import  文件路径
### 二、基本数据
##### 2.1 整数(默认值为`0`)
        int   int8   int16   int32    int64 
        uint  uint8  uint16  uint32   uint64
        rune类型是int32类型的同义词
        type类型是uint8类型的同义词
        int、uint在32位系统是32位,在64位系统是64位
        int、uint、uintptr都有别于其大小明确的相似类型的类型。就是说,int和int32是不同类型,
        尽管int天然的大小就是32位,并且int值若要当作int32使用,必须显示转换,反之亦然。
        uintptr无符号整型,其大小并不明确,但足以完整存储指针.uintptr类型仅仅用于底层编程.
##### 2.2 浮点数(默认值为`0`)
        float32 float64
##### 2.3 复数(默认值为`(0+0i)`)
        complex128 complex64
##### 2.4 布尔值(默认为`false`,值:false,true)
        bool 
##### 2.5 字符串(默认为"")  
        string 
###### 2.5.1 字符串字面量
        字符串的值可以直接写成字符串字面量,形式上就是带双引号的字节序列.
        因为go的源文件总是按UTF-8编码,并且习惯上Go的字符串会按UTF-8解读,所以
        在源码中我们可以将Unicode码点写入字符串字面量。
        在带双引号的字符串字面量中,转义序列以反斜杠(\)开始,可以将任意值的字节插入字符串中。
        \a "警告"或响铃
        \b 退格符
        \f 换页符
        \n 换行符
        \r 回车符
        \v 垂直制表符
        \' 单引号
        \" 双引号
        \\ 反斜杠
        
        const GoUsage = `
        Go is a tool  for managing Go Source code.
            Usage:
                go command [arguments]
        `
###### 2.5.2 Unicode
        Unicode它囊括了世界上所有文书体系
###### 2.5.3 UTF-8
        UTF-8以字节为单位对Unicode码点作变长编码
###### 2.5.4 字符串和字节slice
        4个标准包对字符串操作特别重要: bytes, strings, strconv, unicode
###### 2.5.5 字符串和数字的相互转换
        //todo::
##### 2.6 常量
        const pi = 3.14159
###### 2.6.1 常量生成器iota
        常量声明可以使用iota常量生成器初始化，它用于生成一组以相似规则初始化的常量，但是不用每行都写一遍初始化表达式。
        在一个const声明语句中，在第一个声明的常量所在的行，iota将会被置为0，然后在每一个有常量声明的行加一。
        const (
        	x = iota // x == 0
        	y = iota // y == 1
        	z = iota // z == 2
        	w  // 这里隐式地说w = iota，因此w == 3。其实上面y和z可同样不用"= iota"
        	)
        const v = iota // 每遇到一个const关键字，iota就会重置，此时v == 0
        
        const (
        	h, i, j = iota, iota, iota //h=0,i=0,j=0 iota在同一行值相同
        	)
        
        const (
        	a       = iota //a=0
        	b       = "B"   // B
        	c       = iota             //c=2
        	d, e, f = iota, iota, iota //d=3,e=3,f=3
        	g       = iota             //g = 4
        	)

### 三、复合数据类型
##### 3.1 数组
    数组是具有固定长度且拥有零个或者多个相同数据类型元素的序列。
###### 3.1.1 声明
    ① var a [3]int
    ② var a [3]int = [3]int{1, 2, 3}
       var a [3]int = [3]int{1, 2}
    ③ var a := [...]int{1, 2, 3}
    ④ type Currency int
       const (
            USD Currency = iota
            EUR
            GBP
            RMB
       )
       symbol := [...]string{USD: "$", EUR: "€", GBP: "￡", RMB: "￥" }
    ⑤ r := [...]int{99: -1} // 定义一个拥有100个元素的数组r
##### 3.2 切片(slice)
    slice 表示一个拥有相同类型元素的可变长度的序列。
    是有序的
###### 3.2.1 声明
    ① months := [...]string{
      		1: "January",
      		2: "February",
      		3: "March",
      		4: "April",
      		5: "May",
      		6: "June",
      		7: "July",
      		8: "Augest",
      		9: "Septemper",
      		10: "October",
      		11: "November",
      		12: "December",
      	}
      
      	summer := months[4:7]  // ["April", "May", "June"]
      
      	fmt.Println(len(summer))  // 长度为3
      	fmt.Println(cap(summer))  // 容量为9
      	
    ② var a []int
    ③ z := make([]int, 3, 10) // 声明长度未3, 容量为10的切片
###### 3.2.2 追加
    var runes []rune
    for _, r := range "Hello, 世界" {
        runes = append(runes, r) 
    }
    fmt.Printf("%q\n", runes) // "['H', 'e', 'l', 'l', 'o', ',', '世', '界']"
##### 3.3 散列(map)
    散列表是设计精妙、用途广泛的数据结构之一.
    是无序的
###### 3.3.1 声明
    ① ages := make(map[string]int)
    ② ages := map[string]int {
            "alice": 31,
            "charlie": 41,
        }
       等价于:
            ages := make(map[string]int)
            ages["alice"] = 31
            ages["charlie"] = 41
##### 3.4 结构体
     结构体是将零个或者多个任意类型的命名变量组合在一起的聚合数据类型
###### 3.4.1 声明
     ① type Employee struct {
            Id int
            Name string
            Address string
        }
        var dilbert Employee
        dilbert.Name = "zhangshan"
     
        type tree struct {
            value int
            left, right *tree
        }
###### 3.4.2 结构体字面量
       type point struct{x, y int}
       p := point{1, 2}
       
##### 3.5 json
        //todo::

### 四、函数
##### 4.1 声明
        func name(paramter-list) (result-list) {
            body
        }
##### 4.2 多返回值,返回值可以是多个
        func test() ([]int, error) {
            
        }
##### 4.3 错误
        错误一直存在
##### 4.4 错误处理策略
        ① resp, err := http.Get(url)
           if err != nil {
             return nil, err
           }
        ② doc, err := html.Parse(resp.Body)
           resp.Body.Close()
           if err != nil {
                return nil, fmt.Errorf("Parsing %s as Html: %v", url, err)
           }
        ③ if err := WaitForServer(url); err != nil {
                fmt.Fprintf(os.Stderr, "Site is down: %v\n", err)
                os.Exit(1)
           }
        ④ if err ！= nil {
             err = fmt.Errorf("parsing HTML: %s", err)
             return
           }
##### 4.5 函数变量
        func square(n int) int {return n * n}
        f := square
        fmt.Println(f(3))
        
        var f func(int) int
        f(3) //宕机:调用空函数
##### 4.6 匿名函数
        func squares() func() int {
            var x int
            return func() int {
                x++
                return x * x
            }
        }
        f := squares()
        f()   //1
        f()   //4
        f()   //9
##### 4.7 变长函数
        func sum(vals ...int) int {
            total := 0
            for _, val := range vals {
                total += val
            }
            return total
        }
##### 4.8 延迟函数调用
        resp, err := http.Get(url)
        if err != nil {
            return err
        }
        defer resp.Body.Close()
##### 4.9 宕机
        go 语言的类型系统会捕捉许多编译时错误,但有些其他的错误(比如数组越界访问或者解引用空指针)都需要在运行时进行检查.当go语言运行时检测到
        这些错误,它就会发生宕机.
        一个典型的宕机发生时,正常的程序执行会终止,goroutine中的所有延迟函数会执行,然后程序会异常退出并留下一条日志消息.日志消息包括宕机的值,这
        往往代表某种错误消息,每一个goroutine都会在宕机的时候显示一个函数调用的栈跟踪消息。通常可以借助这条日志消息来诊断问题的原因而不需要再一次运行
        该程序,因此报告一个发生宕机的程序bug时,总是会加上这条消息。
### 五、接口
    接口即约定
##### 5.1 接口定义
        type  reader interface {
            Read(p []byte) (n int, err error)
        }
##### 5.2 实现接口
        var w io.Writer
        w = os.Stdout
        
        var any interface{}
        any = true
        any = 12.34
        any = "hello"
### 六、goroutine (非抢占式)
        func main() {
        	for  i := 0; i < 1000; i++ {
        		go func(i int) {
        			for {
        				fmt.Printf("hello from " + "goroutine %d\n", i)
        			}
        		}(i)
        	}
        	time.Sleep(time.Millisecond)
        }

        func main() {
        	var a [10]int
        	for  i := 0; i < 10; i++ {
        		go func(i int) {
        			for {
        				a[i]++
        				runtime.Gosched() // 交出控制权
        			}
        		}(i)
        	}
        	time.Sleep(time.Millisecond)
        	fmt.Println(a)
        }
        
        func main() {
        	var a [10]int
        	for  i := 0; i < 10; i++ {
        		go func() {
        			for {
        				a[i]++
        				runtime.Gosched()
        			}
        		}()
        	}
        	time.Sleep(time.Millisecond)
        	fmt.Println(a)
        }
        // out of index  原因是因为闭包中的go func中的i使用的是外部的i,外部的i循环最后一次会变成10,所以越界了
        
        子程序都是协程的特例
        
        协程可能在一个线程或多个线程
        
###### 6.1 goroutine的定义
        任何函数只需加上go就能送给调度器运行
        不需要在定义时区分是否是异步函数
        调度器在合适的点进行切换
        使用-race来检测数据访问的冲突
        
###### 6.2 goroutine可能切换点
        I/O, select
        channel
        等待锁
        函数调用(有时)
        runtime.Gosched()
        注: 只是参考,不能保证切换,不能保证在其它地方不切换
        
### 七 channel
        ① 
        func chanDemo()  {
        	//var c chan int // c == nil
        	c := make(chan int)
        	c <- 1
        	c <- 2
        
        	n := <-c
        	fmt.Println(n)
        
        }
        func main() {
        	chanDemo()
        }
        // 报错: fatal error: all goroutines are asleep - deadlock!
        
        ②        
        func chanDemo()  {
        	//var c chan int // c == nil
        	c := make(chan int)
        	go func() {
        		for {
        			n := <-c
        			fmt.Println(n)
        		}
        	}()
        	c <- 1
        	c <- 2
        }
        func main() {
        	chanDemo()
        }
        // 只输出: 1  2来不及打印就退出了
        
        ③
        func worker(c chan int) {
        	for {
        		n := <-c
        		fmt.Println(n)
        	}
        }
        func chanDemo()  {
        	//var c chan int // c == nil
        	c := make(chan int)
        	go worker(c)
        	c <- 1
        	c <- 2
        	time.Sleep(time.Millisecond)
        }
        func main() {
        	chanDemo()
        }
        // 输出: 1
        //      2
        
        ④
        func worker(id int, c chan int) {
        	for {
        		fmt.Printf("Worker %d received %d\n", id, <-c)
        	}
        }
        func chanDemo()  {
        	//var c chan int // c == nil
        	c := make(chan int)
        	go worker(0, c)
        	c <- 1
        	c <- 2
        	time.Sleep(time.Millisecond)
        }
        func main() {
        	chanDemo()
        }
        // 输出:
        //      Worker 0 received 1
        //        Worker 0 received 2
        
        ⑤
        func worker(id int, c chan int) {
        	for {
        		fmt.Printf("Worker %d received %c\n", id, <-c)
        	}
        }
        
        func chanDemo()  {
        	var channels [10]chan int
        
        	for i := 0; i < 10; i++ {
        		channels[i] = make(chan int)
        		go worker(i, channels[i])
        	}
        	for i := 0; i < 10; i++ {
        		channels[i] <- 'a' + i
        	}
        	time.Sleep(time.Millisecond)
        }
        
        func main() {
        	chanDemo()
        }
        // 输出:
    // Worker 0 received a
    // Worker 2 received c
    // Worker 8 received i
    // Worker 3 received d
    // Worker 4 received e
    // Worker 7 received h
    // Worker 5 received f
    // Worker 9 received j
    // Worker 6 received g
    // Worker 1 received b
       
      ⑥
      func createWorker(id int) chan int {
      	c := make(chan int)
      	go func() {
      		for {
      			fmt.Printf("Worker %d received %c\n", id, <-c)
      		}
      	}()
      	return c
      }
      
      func chanDemo()  {
      	var channels [10]chan int
      
      	for i := 0; i < 10; i++ {
      		channels[i] = createWorker(i)
      	}
      	for i := 0; i < 10; i++ {
      		channels[i] <- 'a' + i
      	}
      	time.Sleep(time.Millisecond)
      }
      
      func main() {
      	chanDemo()
      }
      // 输出:
      //Worker 0 received a
      //Worker 2 received c
      //Worker 1 received b
      //Worker 3 received d
      //Worker 7 received h
      //Worker 5 received f
      //Worker 6 received g
      //Worker 9 received j
      //Worker 4 received e
      //Worker 8 received i
      
      
##### 7.1 声明
        var c chan int // c== nil
        c := make(chan int)
        
        c := make(chan int, 3) // buffer channel
        close(c) //关闭channel
        
        //发送方要负责close掉channel
        
        // 接收方判断channel是否关闭
        n, ok := <-c 
        if !ok {
            break
        }
        或
        for n := range c {
            fmt.Printf("received %d\n", n)
        }
        
        // 如果发的人没有close掉,收的人只收了一会，因为main函数执行完会退掉
        
        // 理论基础: communication Sequential Process(CSP)
        
        // Don't communicate by sharing memory; share memory by communicating.
        // 不要通过共享内存来通信,通过通信来共享内存
        
##### 7.2 使用channel等待任务结束
        ①、
        func createWorker(id int) worker {
        	w := worker {
        		in: make(chan int),
        		done: make(chan bool),
        	}
        	go doWorker(id, w.in, w.done)
        	return w
        }
        
        func chanDemo()  {
        	var workers [10]worker
        
        	for i := 0; i < 10; i++ {
        		workers[i] = createWorker(i)
        	}
        	for i := 0; i < 10; i++ {
        		workers[i].in <- 'a' + i
        		<-workers[i].done
        	}
        }
        
        func doWorker(id int,
        	c chan int, done chan bool) {
        		for n := range c {
        			fmt.Printf("Worker %d received %c\n",
        				id, n)
        			done <- true
        		}
        }
        
        type worker struct {
        	in chan int
        	done chan bool
        }
        
        func main() {
        	chanDemo()
        }
        // 输出:
        Worker 0 received a
        Worker 1 received b
        Worker 2 received c
        Worker 3 received d
        Worker 4 received e
        Worker 5 received f
        Worker 6 received g
        Worker 7 received h
        Worker 8 received i
        Worker 9 received j

        ②、
        func createWorker(id int) worker {
        	w := worker {
        		in: make(chan int),
        		done: make(chan bool),
        	}
        	go doWorker(id, w.in, w.done)
        	return w
        }
        
        func chanDemo()  {
        	var workers [10]worker
        
        	for i := 0; i < 10; i++ {
        		workers[i] = createWorker(i)
        	}
        	for i, worker := range workers {
        		worker.in <- 'a' + i
        	}
        	for _, worker := range workers {
        		<-worker.done
        	}
        }
        
        func doWorker(id int,
        	c chan int, done chan bool) {
        		for n := range c {
        			fmt.Printf("Worker %d received %c\n",
        				id, n)
        			done <- true
        		}
        }
        
        type worker struct {
        	in chan int
        	done chan bool
        }
        
        func main() {
        	chanDemo()
        }
        
        // 输出
        Worker 9 received j
        Worker 5 received f
        Worker 4 received e
        Worker 1 received b
        Worker 7 received h
        Worker 8 received i
        Worker 3 received d
        Worker 6 received g
        Worker 2 received c
        Worker 0 received a
        
        //注: 所有channel的发送都是阻塞式的,我发一个任务给它,必须另一端有人收
        
        ③、
        func createWorker(id int, wg *sync.WaitGroup) worker {
        	w := worker {
        		in: make(chan int),
        		wg: wg,
        	}
        	go doWorker(id, w.in, wg)
        	return w
        }
        
        func chanDemo()  {
        	var wg sync.WaitGroup
        
        	var workers [10]worker
        	for i := 0; i < 10; i++ {
        		workers[i] = createWorker(i, &wg)
        	}
        
        	wg.Add(20)
        
        	for i, worker := range workers {
        		worker.in <- 'a' + i
        	}
        	for i, worker := range workers {
        		worker.in <- 'A' + i
        	}
        
        	wg.Wait()
        }
        
        func doWorker(id int,
        	c chan int, wg *sync.WaitGroup) {
        		for n := range c {
        			fmt.Printf("Worker %d received %c\n",
        				id, n)
        			wg.Done()
        		}
        }
        
        type worker struct {
        	in chan int
        	wg *sync.WaitGroup
        }
        
        func main() {
        	chanDemo()
        }
        // 输出:
        Worker 9 received j
        Worker 0 received a
        Worker 4 received e
        Worker 8 received i
        Worker 7 received h
        Worker 2 received c
        Worker 6 received g
        Worker 5 received f
        Worker 1 received b
        Worker 2 received C
        Worker 1 received B
        Worker 0 received A
        Worker 3 received d
        Worker 3 received D
        Worker 9 received J
        Worker 4 received E
        Worker 5 received F
        Worker 6 received G
        Worker 7 received H
        Worker 8 received I
        
        ④、
        func createWorker(id int, wg *sync.WaitGroup) worker {
        	w := worker {
        		in: make(chan int),
        		done: func() {
        			wg.Done()
        		},
        	}
        	go doWorker(id, w)
        	return w
        }
        
        func chanDemo()  {
        	var wg sync.WaitGroup
        
        	var workers [10]worker
        	for i := 0; i < 10; i++ {
        		workers[i] = createWorker(i, &wg)
        	}
        
        	wg.Add(20)
        
        	for i, worker := range workers {
        		worker.in <- 'a' + i
        	}
        	for i, worker := range workers {
        		worker.in <- 'A' + i
        	}
        
        	wg.Wait()
        }
        
        func doWorker(id int, w worker) {
        		for n := range w.in {
        			fmt.Printf("Worker %d received %c\n",
        				id, n)
        			w.done()
        		}
        }
        
        type worker struct {
        	in chan int
        	done func()
        }
        
        func main() {
        	chanDemo()
        }
        
        // 输出:
        Worker 5 received f
        Worker 0 received a
        Worker 3 received d
        Worker 1 received b
        Worker 1 received B
        Worker 2 received c
        Worker 2 received C
        Worker 3 received D
        Worker 4 received e
        Worker 4 received E
        Worker 5 received F
        Worker 8 received i
        Worker 7 received h
        Worker 0 received A
        Worker 6 received g
        Worker 6 received G
        Worker 8 received I
        Worker 9 received j
        Worker 9 received J
        Worker 7 received H
        
##### 7.3 使用select进行调度
        ①
        func generator() chan int {
        	out := make(chan int)
        	go func() {
        		i := 0
        		for {
        			time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
        			out <- i
        			i++
        		}
        	}()
        	return out
        }
        
        func main() {
        	var c1, c2 = generator(), generator()
        	for {
        		select {
        		case n := <- c1:
        			fmt.Printf("Received from c1: %d\n", n)
        		case n := <- c2:
        			fmt.Printf("Received from c2: %d\n", n)
        			//default:
        			//	fmt.Println("No value received")
        		}
        	}
        }
        // 输出:
        Received from c2: 0
        Received from c1: 0
        Received from c2: 1
        Received from c1: 1
        Received from c2: 2
        Received from c1: 2
        Received from c1: 3
        Received from c2: 3
        Received from c2: 4
        
        ②
        func createWorker(id int) chan<- int {
        	c := make(chan int)
        	go worker(id, c)
        	return c
        }
        
        func worker(id int, c chan int)  {
        	for n := range c {
        		fmt.Printf("Worker %d received %d\n", id, n)
        	}
        }
        
        func generator() chan int {
        	out := make(chan int)
        	go func() {
        		i := 0
        		for {
        			time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
        			out <- i
        			i++
        		}
        	}()
        	return out
        }
        
        func main() {
        	var c1, c2 = generator(), generator()
        	w := createWorker(0)
        	for {
        		select {
        		case n := <- c1:
        			w <- n
        		case n := <- c2:
        			w <- n
        		}
        	}
        }
        // 输出:
        Worker 0 received 0
        Worker 0 received 0
        Worker 0 received 1
        Worker 0 received 1
        Worker 0 received 2
        Worker 0 received 2
        Worker 0 received 3
        Worker 0 received 3
        Worker 0 received 4
        Worker 0 received 4
        Worker 0 received 5
        
        ③
        func createWorker(id int) chan<- int {
        	c := make(chan int)
        	go worker(id, c)
        	return c
        }
        
        func worker(id int, c chan int)  {
        	for n := range c {
        		fmt.Printf("Worker %d received %d\n", id, n)
        	}
        }
        
        func generator() chan int {
        	out := make(chan int)
        	go func() {
        		i := 0
        		for {
        			time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
        			out <- i
        			i++
        		}
        	}()
        	return out
        }
        
        func main() {
        	var c1, c2 = generator(), generator()
        	var worker = createWorker(0)
        
        	n := 0
        	hasValue := false
        	for {
        		var activeWorker chan<- int
        		if hasValue {
        			activeWorker = worker
        		}
        		select {
        		case n = <- c1:
        			hasValue = true
        		case n = <- c2:
        			hasValue = true
        		case activeWorker <- n:
        			hasValue = false
        		}
        	}
        }
        // 输出:
        Worker 0 received 0
        Worker 0 received 0
        Worker 0 received 1
        Worker 0 received 1
        Worker 0 received 2
        Worker 0 received 2
        
        ③、
        func createWorker(id int) chan<- int {
        	c := make(chan int)
        	go worker(id, c)
        	return c
        }
        
        func worker(id int, c chan int)  {
        	for n := range c {
        		time.Sleep(5 * time.Second)
        		fmt.Printf("Worker %d received %d\n", id, n)
        	}
        }
        
        func generator() chan int {
        	out := make(chan int)
        	go func() {
        		i := 0
        		for {
        			time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
        			out <- i
        			i++
        		}
        	}()
        	return out
        }
        
        func main() {
        	var c1, c2 = generator(), generator()
        	var worker = createWorker(0)
        
        	n := 0
        	hasValue := false
        	for {
        		var activeWorker chan<- int
        		if hasValue {
        			activeWorker = worker
        		}
        		select {
        		case n = <- c1:
        			hasValue = true
        		case n = <- c2:
        			hasValue = true
        		case activeWorker <- n:
        			hasValue = false
        		}
        	}
        }
        // 输出:
        Worker 0 received 0
        Worker 0 received 18
        Worker 0 received 41
        Worker 0 received 66
        Worker 0 received 81
        
        ④
        func createWorker(id int) chan<- int {
        	c := make(chan int)
        	go worker(id, c)
        	return c
        }
        
        func worker(id int, c chan int)  {
        	for n := range c {
        		time.Sleep(time.Second)
        		fmt.Printf("Worker %d received %d\n", id, n)
        	}
        }
        
        func generator() chan int {
        	out := make(chan int)
        	go func() {
        		i := 0
        		for {
        			time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
        			out <- i
        			i++
        		}
        	}()
        	return out
        }
        
        func main() {
        	var c1, c2 = generator(), generator()
        	var worker = createWorker(0)
        
        	var values []int
        	tm := time.After(10 * time.Second)
        	tick := time.Tick(time.Second)
        	for {
        		var activeWorker chan<- int
        		var activeValue int
        		if len(values) > 0 {
        			activeWorker = worker
        			activeValue = values[0]
        		}
        		select {
        		case n := <- c1:
        			values = append(values, n)
        		case n := <- c2:
        			values = append(values, n)
        		case activeWorker <- activeValue:
        			values = values[1:]
        		case <-time.After(800 * time.Millisecond):
        			fmt.Println("timeout")
        		case <-tick:
        			fmt.Println("queue len = ", len(values))
        		case <-tm:
        			fmt.Println("bye")
        			return
        		}
        	}
        }
        // 输出:
        queue len =  7
        Worker 0 received 0
        queue len =  15
        Worker 0 received 0
        queue len =  21
        Worker 0 received 1
        queue len =  27
        Worker 0 received 1
        queue len =  34
        Worker 0 received 2
        queue len =  42
        Worker 0 received 2
        queue len =  50
        Worker 0 received 3
        queue len =  58
        Worker 0 received 3
        queue len =  67
        Worker 0 received 4
        bye
        
##### 7.4 传统同步机制
        ①、
        type atomicInt struct {
            value int
            lock sync.Mutex
        }
        
        func (a *atomicInt) increment() {
            a.lock.Lock()
            defer a.lock.Unlock()
            a.value++
        }
        
        func (a *atomicInt) get() int  {
            a.lock.Lock()
            defer a.lock.Unlock()
            return a.value
        }
        
        func main() {
            var a atomicInt
            a.increment()
            go func() {
                a.increment()
            }()
            time.Sleep(time.Millisecond)
            fmt.Println(a.get())
        }
        // 输出:
        2
        
        ②、
        type atomicInt struct {
        	value int
        	lock sync.Mutex
        }
        
        func (a *atomicInt) increment() {
        	fmt.Println("safe increment")
        	func () {
        		a.lock.Lock()
        		defer a.lock.Unlock()
        		a.value++
        	} ()
        }
        
        func (a *atomicInt) get() int  {
        	a.lock.Lock()
        	defer a.lock.Unlock()
        	return a.value
        }
        
        func main() {
        	var a atomicInt
        	a.increment()
        	go func() {
        		a.increment()
        	}()
        	time.Sleep(time.Millisecond)
        	fmt.Println(a.get())
        }
        // 输出:
        safe increment
        safe increment
        2