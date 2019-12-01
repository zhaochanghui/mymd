# go 进阶

### 6.IO读操作

    package main

    import (
        "fmt"
        "os"
        "io"
    )

    func main() {
        //打开文件
        fileName :="D:/gowork/src/demo/aa/a.txt"   //文件内容： abcdefghij
        file,err := os.Open(fileName)
        if err !=nil{
            fmt.Println("err:",err)
            return
        }

        //3关闭文件
        defer file.Close()

        //2 读取文件
        /*
        bs :=make([]byte,4,4)
        n,err := file.Read(bs)
        fmt.Println(err)
        fmt.Println(n)
        fmt.Println(string(bs))

        n,err=file.Read(bs)
        fmt.Println(err)
        fmt.Println(n)
        fmt.Println(string(bs))

        n,err= file.Read(bs)
        fmt.Println(err)
        fmt.Println(n)
        fmt.Println(string(bs))
        */

        bs :=make([]byte,4,4)   //这里一般是1024，这里是测试，因为写为4
        n :=1
        for{
            n,err = file.Read(bs)
            if n==0 || err == io.EOF {
                fmt.Println("读到了文件的结尾，结束读取操作。。")
                break
            }

            fmt.Println(string(bs[:n]))
        }

        }


### 7.IO写操作
    package main

    import (
        "fmt"
        "os"
        
    )
    func main(){
        //写入数据
        fileName := "D:/gowork/src/demo/testfile/a3.txt"

        //打开  os.Open(name)  以只读模式打开   os.OpenFile可以指定模式，不存在则创建之类的
        file,err := os.OpenFile(fileName,os.O_CREATE|os.O_WRONLY|os.O_APPEND,os.ModePerm)
        if err != nil {
            fmt.Println(err)
            return 
        }

        //关闭文件
        defer file.Close()

        file.WriteString("\n")
        //写入数据
        bs :=[]byte{71,72,73,74,75,76}
        n,err := file.Write(bs)  //也可以只写入一部分 bs[:2], n表示写入多少个byte
        fmt.Println(n)
        fmt.Println(err)

        file.WriteString("\n")
        n,err = file.WriteString("helloworld")   //写入字符串
        fmt.Println(n)
        fmt.Println(err)

        file.WriteString("\n")
        n,err= file.Write([] byte("today"))  //字符串转为byte 切片再写入
        fmt.Println(n)  
        fmt.Println(err)

    }


### 8.复制文件三种方法
    package main

    import(
        "os"
        "io"
        "fmt"
        "io/ioutil"
    )
    func main(){
        //复制文件,推荐用copy,当然其他的也行，根据实际情况来
        oldfile :="D:/gowork/src/demo/testfile/source/a.jpg"
        //oldfile :="D:/gowork/src/demo/testfile/a.txt"
        newfile := "D:/gowork/src/demo/testfile/dest/a.jpg"
        //total,err :=copyFile1(oldfile,newfile)
        //total,err :=copyFile2(oldfile,newfile)
        total,err :=copyFile3(oldfile,newfile)

        fmt.Println(total)
        fmt.Println(err)
    }

    //返回复制的总数量，错误
    func copyFile1(oldf,newf string) (int , error){
        file1,err := os.Open(oldf)
        if err !=nil{
            return 0, nil
        }

        file2,err := os.OpenFile(newf,os.O_WRONLY|os.O_CREATE,os.ModePerm)
        if err !=nil{
            return 0,nil
        }

        defer file1.Close()
        defer file2.Close()

        //读写
        bs := make([]byte,1024,1024)

        n := -1  //读取的数量
        total :=0
        for{
            n,err = file1.Read(bs)
            if err==io.EOF || n==0 {
                fmt.Println("复制完毕")
                break
            }else if err != nil {
                fmt.Println("报错。。")
                return total,err
            }

            total +=n
            file2.Write(bs[:n]) //读到多少就写多少
        }

        return total,nil
    }


    //第二种复制的文件的方法，copy, 返回int64,错误
    func copyFile2(oldf,newf string) (int64,error){
        file1,err:=os.Open(oldf)
        if err != nil{
            return 0,err
        }

        file2,err :=os.OpenFile(newf,os.O_WRONLY|os.O_CREATE,os.ModePerm)
        if err!=nil{
            return 0,err
        }

        defer file1.Close()
        defer file2.Close()

        return io.Copy(file2,file1)
    }


    //通过io/ioutil包复制，这个要注意，大文件不推荐用，因为是一次性读取和写入
    func copyFile3(oldf,newf string) (int,error){
        bs,err := ioutil.ReadFile(oldf)
        if err!=nil{
            return 0,err
        }

        err = ioutil.WriteFile(newf,bs,0777)
        if err!=nil{
            return 0,err 
        }
        return len(bs),nil
    }


### 9.golang Seek,断点续传（先学seek,断点续传后面补充）
习惯了php中的seek和tell，转到golang时突然发现只有Seek发现，tell方法不见了。google了一下，发现了tell的实现方法：

File.Seek(0, os.SEEK_CUR) 或者File.Seek(0,1) 参考

解释：

先来看下Seek方法

func (f *File) Seek(offset int64, whence int) (ret int64, err error)

跳转到文本中的某处，并返回此处的偏移量

File.Seek(0, os.SEEK_CUR) #跳转到当前位置（位置不变）

这样就很好理解了。

    f,_:=os.Open("a.txt")
    //从头开始，文件指针偏移100
    f.Seek(100,0) 
    buffer:=make([]byte,1024)
    // Read 后文件指针也会偏移
    _,err:=f.Read(buffer)
    if err!=nil{
        fmt.Println(nil)
        return
    }
    // 获取文件指针当前位置
    cur_offset,_:=f.Seek(0,os.SEEK_CUR)
    fmt.Printf('current offset is %d\n', cur_offset)
    
往文件末尾追加内容:
Seek()查到文件末尾的偏移量
WriteAt()则从偏移量开始写入

    package main
    import (
        "os"
        "fmt"
    )

    func main(){
        f,err := os.OpenFile("D:/gowork/src/demo/testfile/a3.txt",os.O_WRONLY,0777)
        if err != nil{
            fmt.Println("出错。。",err)
            return
        }else{
            n,_ := f.Seek(0,os.SEEK_END)
            _,err= f.WriteAt([]byte("中国人"),n)
            if err!=nil{
                fmt.Println(err)
            }
        }

        defer f.Close()

    }    

 最简单的方式：
    f, err := os.OpenFile(fileName, os.O_WRONLY|os.O_APPEND, 0666)   

 ###  10.bufio  
    
    package main
        import(
            "os"
            "fmt"
            "bufio"
        )

    /**
    bufio包实现了带缓冲的IO,他封装了io.Reader和io.Writer对象
    然后创建了另一个对象Reader或Writer实现了相同的接口，但是
    增加缓冲功能
    Write方法首先会判断写入的数据长度是否大于设置的缓冲长度，如果小于，则会将数据copy到缓冲中；当数据长度大于缓冲长度时，如果数据特别大，则会跳过copy环节，直接写入文件。其他情况依然先会将数据拷贝到缓冲队列中，然后再将缓冲中的数据写入到文件中。
    添加Flush()方法，将缓存的数据写入到文件中。
    参考： https://www.jb51.net/article/155950.htm
    */
    func main(){
	//带缓冲的写入
	fileName := "D:/gowork/src/demo/testfile/a3.txt"
	f,err :=os.OpenFile(fileName,os.O_CREATE|os.O_RDWR,0777)
	if err!=nil{
		fmt.Println(err)
		return 
	}

	defer f.Close()

	str := []byte("走进新时代")
	nw :=bufio.NewWriterSize(f,1024)
        if _,err :=nw.Write(str); err!=nil{
            fmt.Println("写入出错",err)
            return 
        }

        if err = nw.Flush(); err!=nil{
            fmt.Println("flush error",err)
            return 
        }

        fmt.Println("写入成功")
    }

    //没有缓冲功能的Write(os包中)方法，它会将数据直接写到文件中。
    func nocach(){
        fileName := "D:/gowork/src/demo/testfile/a3.txt"
        f,err := os.OpenFile(fileName,os.O_CREATE|os.O_RDWR,0777)
        if err!=nil{
            fmt.Println("error:",err)
            return
        }

        defer f.Close()

        str:=[]byte("世界，你好")
        if _,err = f.Write(str); err !=nil{
            fmt.Println(err)
        }

        fmt.Println("成功写入")
    }


### 11.ioutil包使用

    package main

    import(
        "fmt"
        "net/http"
        "io/ioutil"
    )
    func main(){
        fun3()
    }


    /*ioutil.ReadAll
    func ReadAll(r io.Reader) ([]byte, error)
    从r读取数据直到遇到错误或者EOF,返回读到的数据，一个成功的调用返回的err==nil
    */
    func getHtml(){
        url :="http://www.baidu.com"
        resp, err := http.Get(url)
        if err!=nil{
            fmt.Println(err)
            return
        }

        defer resp.Body.Close()
        body,err := ioutil.ReadAll(resp.Body)
        if err !=nil{
            fmt.Println(err)
            return 
        }

        fmt.Println(string(body))
    }


    /*
    func ReadFile(filename string) ([]byte, error)
    从以参数为文件名的文件中读取文件内容。成功调用返回err == nil.因为ReadFile读取整个文件，所以它不会将Read中的EOF视为要报告的错误。
    */
    func fun1(){
        fileName:= "D:/gowork/src/demo/testfile/a3.txt"
        str,err := ioutil.ReadFile(fileName)
        if err!=nil{
            fmt.Println("读取出错：",err)
            return 
        }

        fmt.Printf("file content:%s",string(str))
    }

    /*
    func WriteFile(filename string, data []byte, perm os.FileMode) error
    WriteFile将数据写入由filename命名的文件。 如果该文件不存在，则WriteFile使用权限perm创建它; 否则WriteFile会在写入之前截断它。
    */
    func fun2(){
        fileName:= "D:/gowork/src/demo/testfile/a4.txt"
        if err :=ioutil.WriteFile(fileName,[]byte("hello world"),0777);err !=nil{
            fmt.Println(err)
        }

        fmt.Println("write file success..")
    }

    /*
    func ReadDir(dirname string) ([]os.FileInfo, error)
    ReadDir读取由dirname命名的目录，并返回按filename排序的目录条目列表。
    */
    func fun3(){
        filedir:= "D:/gowork/src"
        files,err := ioutil.ReadDir(filedir)
        if err!=nil{
            fmt.Println(err)
            return 
        }

        for _,file :=range files {
            fmt.Println(file.Name())
        }
    }


func TempDir(dir, prefix string) (name string, err error)
TempDir在目录dir中创建一个新的临时目录，其名称以prefix开头，并返回新目录的路径。 如果dir是空字符串，TempDir将使用临时文件的默认目录（请参阅os.TempDir）。 同时调用TempDir的多个程序将不会选择相同的目录。 调用者有责任在不再需要时删除目录.
func TempFile(dir, pattern string) (f *os.File, err error)
TempFile在目录dir中创建一个新的临时文件，打开文件进行读写，并返回生成的* os.File。 文件名是通过获取模式并在末尾添加随机字符串生成的。 如果pattern包含“”，则随机字符串将替换最后一个“”。 如果dir是空字符串，则TempFile使用临时文件的默认目录（请参阅os.TempDir）。 同时调用TempFile的多个程序不会选择相同的文件。 调用者可以使用f.Name（）来查找文件的路径名。 当不再需要时，调用者有责任删除该文件。


### 12.ioutil遍历文件夹

    package main
    import (
        "fmt"
        "io/ioutil"
        "log"
    )

    func main(){
        filedir:= "D:/gowork/src"
        listFiles(filedir,0)
    }

    //遍历文件，带层级
    func listFiles(dir string,deep int) {
        s:="|--"
        for i:=0;i<deep;i++ {
            s ="|  "+s
        }
        files ,err := ioutil.ReadDir(dir)
        if err!=nil{
            log.Fatal(err)
        }

        for _,f := range files{
            fileName := dir+"/"+f.Name()
            fmt.Printf("%s%s\n",s,fileName)

            if f.IsDir(){
                listFiles(fileName,deep+1)
            }
        }
    }


### iota
    package main
    
    import "fmt"
    
    const (
    	a =iota
    	b
    	c
    	d
    )
    
    
    const(
    	d1,d2 = iota+1,iota+2
    	d3,d4  = iota+1,iota+2
    )
    
    //网盘
    const(
    	_ = iota
    	KB = 1 << (10*iota)
    	MB = 1<<(10*iota)
    	GB=1<<(10*iota)
    	TB=1<<(10*iota)
    	PB=1<<(10*iota)
    )
    
    func main(){
    	fmt.Println(d1)  //1
    	fmt.Println(d2)  //2
    	fmt.Println(d3)  //2
    	fmt.Println(d4)  //3
    
    	fmt.Println(KB)  //1024
    	fmt.Println(MB)  //1048576
    	fmt.Println(GB)  //1073741824
    	fmt.Println(TB)   //1099511627776
    	fmt.Println(PB)  //1125899906842624
    }
    
    
    
### 字符串

