# GO面试大全

**go是21世纪的C语言**

#### 1.判断字符串中汉字的数量
    
    //判断一个字符串中汉字的数量
    func fun1(){
        s1:="hello中国"
        var count int
    
        for _,c :=range s1 {
            if unicode.Is(unicode.Han, c) {
                count++
            }
        }
    
        fmt.Println(count)  //2
    }


#### 2.统计单词出现的次数
    //统计单词出现的次数
    func fun2(){
    	s2:="how do you do"
    	s3:=strings.Split(s2," ")
    	m1:=make(map[string]int,10)
    
    	for _,w :=range s3{
    		if _,ok:=m1[w]; !ok{
    			m1[w] =1
    		}else{
    			m1[w]++
    		}
    	}
    
    
    	for k,v:=range m1{
    		fmt.Println(k,v)
    	}
    }
    
    /*
    输出
    how 1
    do 2
    you 1
    */
    
### 3./判断回文数，正读和倒读是一样的
    //判断回文数，正读和倒读是一样的
    func fun1(){
        s:="上海自来水来自海上"
    
        r:=make([] rune,0,len(s))
    
        for _,c:=range s{
            r = append(r,c)
        }
    
        var flag = true
        for i:=0;i<len(r)/2;i++{
    
            if r[i]!=r[len(r)-i-1] {
                flag = false
                break
            }
        }
    
        fmt.Println(flag)  //true代表是回文数
    }    