# PHP面试大全

**php是一种专注于web「脚本语言」**

#### 1.给定一个数跟数组，将小于等于该数的数组元素放在左边，将大于该数的数组元素放在右边
##### php版
    <?php

    function fun1($arr,$num){

        $x =0;
    
        for($i=0;$i<count($arr);$i++){

            if($arr[$i]<=$num){
           
                $tmp = $arr[$i];
                $arr[$i] = $arr[$x];
                $arr[$x] = $tmp;
                $x++;
            }
        }

        return $arr;
     }

    $arr = [10,6,1,3,5,9,15,20,-1];
    $arr = fun1($arr,8);
    print_r($arr);

#### Java 版
    import java.util.*;
    public class a3{
        public static void main(String[] args){
            int arr[] = {10,1,2,8,7,4,3,6};
            smallLefBigRight(arr,5);  
            System.out.println(Arrays.toString(arr));  
        }

        public static void smallLefBigRight(int [] arr,int num){
            int x = -1;
            for(int i=0;i<arr.length;i++){
                if(arr[i]<=num){
                    x++;
                    int tmp = arr[i];
                    arr[i] = arr[x];
                    arr[x] = tmp;
                }
            }
        }
    }


### 2.二叉树（中旬）实现。
根节点自定义，比根小的放左边，比根大的放右边，下面是java版，PHP版的类似
    public class Test{
        public static void main(String[] args){
            NodeTree tree = new NodeTree();
            tree.add(8); //根节点
            tree.add(5); //左节点
            tree.add(4); 
            tree.add(6);

            tree.add(10);
            tree.add(9);
            tree.add(11);

            tree.show();//结果： 4 > 5 > 6 > 8 > 9 > 10 > 11 >
     }

    }

    //节点类
    class Node{
        Node left; //左节点
        int data; //节点值
        Node right; //又节点

        public Node(int data){
            this.data = data;
        }

        //增加节点，比根节点大的放右边，比跟节点小的放左边
        public void addNode(int data){
            Node node = new Node(data);
            if(this.data>data){  //比根节点小
                if(this.left == null){
                    this.left = node;
                }else{
                    this.left.addNode(data);   
                }
                
            }else{ //比根节点大
                if(this.right == null){
                    this.right = node;
                } else{
                    this.right.addNode(data);
                }  
            }
        }

        //遍历节点
        public void showNode()
        {
            if(this.left != null){
                this.left.showNode();
            }
            System.out.print(this.data+" > ");

            if(this.right != null){
                this.right.showNode();
            }
        }
    }

    //操作节点的类
    class NodeTree{
        Node root;  //最顶部的根节点，只有一个

        //添加节点
        public void add(int data){
            if(this.root == null){
                this.root = new Node(data);
            }else{
                this.root.addNode(data);    
            }
        }

        //遍历节点
        public void show(){
            this.root.showNode();
        }
    }


 ### Java 双向链表实现
 会双向链表，单向链表也会了。一个节点分三部分，存放上个节点区域，存放下个节点区域，节点值
    public class Test{
        public static void main(String[] args){
            NodeList n = new NodeList();
        //添加
        n.add("a");
        n.add("b");
        n.add("c");
        n.add("d");
        System.out.println("添加后长度是："+n.size);
        n.show();

        n.delete(1);
        System.out.println("\n 删除第2个后长度是："+n.size);
        n.show();
        System.out.println("\n 修改第2个节点值为222：");
        n.update(1, "22");
        n.show();

        System.out.println("\n 获得第2个节点值："+n.get(1));

        System.out.println("往第2个节点前插入xy后长度:"+n.size);
        n.insert(1, "xy");
        n.show();

        /*  结果
            D:\workspace\phpdemo\rbmq\three>java Test
            添加后长度是：4
            a > b > c > d >
            删除第2个后长度是：3
            a > c > d >
            修改第2个节点值为222：
            a > 22 > d >
            获得第2个节点值：22
            往第2个节点前插入xy后长度:3
            a > xy > 22 > d >
            D:\workspace\phpdemo\rbmq\three>
        */

        }
    }


    //节点类
    class Node{
        Node prev;  //上一个节点
        Object data;
        Node next;  //下一个节点

        public Node(Object data){
            this.data = data;
        }
    }

    //节点操作类，链表
    class NodeList{
        Node start; //头节点
        Node end;   //尾巴节点，当只有一个节点时，头节点和尾部节点是同一个
        int size; //链表长度

        //添加,往尾部添加
        public void add(Object data){
            Node node = new Node(data);
            if(this.start == null){//创建第一个节点
                this.start = node;
                this.end = this.start;
            }else{
                //下面的顺序不要乱
                this.end.next = node;
                node.prev = this.end;
                this.end = node;   
            }

            this.size++;   //链表长度加1
        }

        //删除指定位置节点，从0开始，第一个节点，索引为0，以此类推,这里没有做严格判断索引值
        public void delete(int index){
            Node tmp = this.start;
            for(int i=0;i<index;i++){
                tmp = tmp.next;
            }
            //tmp 就是要删除的节点，需要记录tmp的上一个节点，下一个节点，上一个指向下一个，下一个指向上一个就行
            Node up = tmp.prev;
            Node down = tmp.next;
            
            up.next = down;
            down.prev = up;

            this.size--;
        }

        //修改指定节点的值，比较简单
        public void update(int index,Object data){
            Node tmp = this.start;
            for(int i=0;i<index;i++){
                tmp = tmp.next;
            }

            tmp.data = data;   //修改了
        }

        //获得指定节点的值
        public Object get(int index){
            Node tmp = this.start;
            for(int i=0;i<index;i++){
                tmp = tmp.next;
            }
        
            return tmp.data;
        }

        //往指定位置前插入
        public void insert(int index,Object data){
            Node node = new Node(data);
            Node tmp = this.start;

            for(int i=0;i<index;i++){
                tmp = tmp.next;
            }

            //下面的代码顺序不能变，往tmp前面插入，需要知道tmp的上一个节点
            Node up = tmp.prev;
            up.next = node; 
            node.prev = up;

            node.next = tmp;
            tmp.prev = node;

            this.size++;
        }


        //遍历节点
        public void show()
        {
            Node tmp = this.start;
            for(int i=0;i<this.size;i++){
                System.out.print(tmp.data+" > ");
                tmp = tmp.next;
            }
        }
    }

