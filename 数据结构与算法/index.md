
---  
###### 常见、基础的数据结构与算法
- 数据结构:数组、链表、栈、队列、散列表、二叉树、堆、跳表、图、Trie 树  
- 算法：递归、排序、二分查找、搜索、哈希算法、贪心算法、分治算法、回溯算法、动态规划、字符串匹配算法  
--- 
###### 时间复杂度分析
- 大``O``时间复杂度表示法：代码执行时间随数据规模增长的变化趋势,也叫作渐进时间复杂度(asymptotic time complexity),简称时间复杂度。
- 在采用大 O 标记复杂度的时候，可以忽略系数，即 O(Cf(n)) = O(f(n))
- 分析技巧
	- 只关注循环执行次数最多的一段代码
	- 加法法则：总复杂度等于量级最大的那段代码的复杂度
	- 乘法法则：嵌套代码的复杂度等于嵌套内外代码复杂度的乘积
	- 常见时间复杂度实例分析(如下图)
		- 多项式量级：随着数据规模的增长，算法的执行时间和空间占用，按照多项式的比例增长。
			- O(1):一般情况下，只要算法中不存在循环语句、递归语句，即使有成千上万行的代码，其时间复杂度也是Ο(1)。
			- O(logn)、O(nlogn): 如果一段代码的时间复杂度是 O(logn)，我们循环执行 n 遍，时间复杂度就是 O(nlogn) 了
			- O(m+n)、O(mn):代码的复杂度由两个数据的规模来决定
		- 非多项式量级：随着数据规模的增长，算法的执行时间和空间占用暴增，这类算法性能极差。只有O(2ⁿ)、O(n!)。时间复杂度为非多项式量级的算法问题叫作 NP（Non-Deterministic Polynomial，非确定多项式）问题
	![](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201214155045.png)

---
###### 空间复杂度分析
- 空间复杂度全称就是渐进空间复杂度（asymptotic space complexity），表示算法的存储空间与数据规模之间的增长关系。
- 常见的空间复杂度就是O(1)、O(n)、O(n²)

---
###### 时间、空间复杂度总结
复杂度也叫渐进复杂度，包括时间复杂度和空间复杂度，用来分析算法执行效率与数据规模之间的增长关系，可以粗略地表示，越高阶复杂度的算法，执行效率越低。常见的复杂度并不多，从低阶到高阶有：O(1)、O(logn)、O(n)、O(nlogn)、O(n2 )。
![](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201214160611.png)

---
###### 其他复杂度分析
- 最好情况时间复杂度：在最理想的情况下，执行这段代码的时间复杂度。比如*查找数组元素第一个就找到*
- 最坏情况时间复杂度:在最糟糕的情况下，执行这段代码的时间复杂度。比如*查找数组元素全部遍历以后最后一个才找到*
- 平均情况时间复杂度：加权平均时间复杂度或者期望时间复杂度。需要考虑情况的同时增加相关概率计算
- 均摊时间复杂度：均摊时间复杂度就是一种特殊的平均时间复杂度。针对有规律的有时序的一些操作。比如*连续插入数据知道数组结束以后统计总和然后重新开始*

---
###### 数组(删除、插入O(n),查找O(1))
- 数组（Array）是一种**线性表**数据结构。它用一组**连续的内存空间**，来**存储一组具有相同类型**的数据。
	- 数组索引从0开始的原因：数组是连续内存空间的数据结构，a[0]为数组的内存首地址。寻址的时候公式为：a[i].address = base_address + i * data_type_size。如果从1或者其他数开始，那么计算机需要对其做相应的减法操作，出现性能损耗问题。
	- 二维数组内存寻址：对于 m * n 的数组，a [ i ][ j ] (i < m,j < n)的地址为：address = base_address + ( i * n + j) * type_size
	- 数组支持随机访问，根据下标随机访问的时间复杂度为 O(1)。
- 多维数组寻址方法(见下图)
![多维数组寻址](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201214164747.png)

---
###### 链表(删除、插入O(1),查找O(n))
- 底层存储结构：通过“指针”将一组零散的内存块串联起来使用
- 类型
	- 单链表
	- 双链表
	- 循环链表
- 链表 VS 数组性能大比拼(见下图)
![链表 VS 数组性能大比拼](https://cdn.jsdelivr.net/gh/smallblack-xh/images@main/blob/20201214174241.png)
- 链表反转：[参考该博客](https://blog.csdn.net/weixin_43232955/article/details/107453018)
```
public class LinkList {
    private Node head;
    private Node last;
    public void add(Node node){
        Node f = head;
        head = node;
        node.setNext(f);
    }
    public void add(String data){
        Node f = head;
        Node node = new Node(null, f, data);
        head = node;
        node.setNext(f);
    }

    /**
     * 原地反转链表
     * 使用pre作为上一个接点的指针，到最后pre想等于表头
     */
    public void reverseItSelf(){
        Node pre = null;
        Node cur = head;
        Node next = null;
        while (cur!=null){
            next = cur.next();
            cur.setNext(pre);
            pre = cur;
            cur = next;
        }
        head = pre;
    }

    public Node get(){
        return head;
    }
    /**
     * 递归反转链表
     */
    public Node reverseRecursion(Node node){
        if(node == null || node.next()==null)
            return node;
        Node cur = reverseRecursion(node.next());
        node.next().setNext(node);
        node.setNext(null);
        return cur;
    }

        /**
     * 链表环检测
     */
    public void checkCircle(){
        Node fast = head;
        Node low = head;
        while (fast!=null&&fast.next()!=null){
            fast = fast.next().next();
            low = low.next();
            if(fast==low){
                System.out.println("链表存在环");
                break;
            }
        }
    }

    /**
     * @return 快慢指针查找中间数值
     */
    public String findMidData(){
        Node fast = head;
        Node slow = head;
        while (fast!=null&&fast.next()!=null){
            fast = fast.next().next();
            slow = slow.next();
        }
        if(slow!=null){
            return slow.getData();
        }
        return null;
    }

 /**
     * 查找倒数第几个
     * @param step
     * @return
     */
    public String findReciprocal(int step){
        Node fast = head;
        Node slow = head;
        for (int i=0;i<step-1;i++){
            fast = fast.next();
        }
        while (fast!=null&&fast.next()!=null){
            fast = fast.next();
            slow = slow.next();
        }
        if(slow!=null){
            return slow.getData();
        }
        return null;
    }

    /**
     * @param step 删除倒数第几个元素
     */
    public void deleReciprocal(int step){
        Node fast = head;
        Node slow = head;
        for (int i=0;i<step;i++){
            fast = fast.next();
        }
        while (fast!=null&&fast.next()!=null){
            fast = fast.next();
            slow = slow.next();
        }
        if (slow!=null) {
            slow.setNext(slow.next().next());
        }
    }

    public void print(){
        System.out.println("LinkList print start");
        Node f = head;
        while (f!=null){
            System.out.println(f.getData());
            f = f.next();
        }
        System.out.println("LinkList print end");
    }
    public void print(Node node){
        System.out.println("LinkList print start");
        Node f = node;
        while (f!=null){
            System.out.println(f.getData());
            f = f.next();
        }
        System.out.println("LinkList print end");
    }

    public static void main(String[] args) {
        LinkList linkList = new LinkList();
        linkList.add("5");
        linkList.add("4");
        linkList.add("3");
        linkList.add("2");
        linkList.add("1");
        Node node1 = linkList.get();
        linkList.print(node1);
        Node node = linkList.reverseRecursion(node1);
        linkList.print(node);
    }
}
```

---
###### 栈

---
###### 缓存淘汰策略
- 先进先出策略 FIFO（First In，First Out）
- 最少使用策略 LFU（Least Frequently Used）
- 最近最少使用策略 LRU（Least Recently Used）


---
###### leetcode题目
- 栈：20,155,232,844,224,682,496