# 一、基础

> **时间复杂度是一个算法流程中，最差情况下，常数操作数量的指标**
>
> **稳定性：相同大小的数字次序是否会被打乱**
>
> 1. 选择、快排、堆排不能做到稳定
> 2. 冒泡、插入、归并都可以做到（也可能不稳定）
>
> **工程中综合排序：基础类型往往是快排（因为基础类型，不需要保证原始次序，相同数值无差异）；对象类型往往是归并；如果数组量小（如小于60），往往是插排（常数项低）**
>
> **桶排序 - 计数排序、基数排序**：非基于比较的排序，与样本实际数据状况有很大关系，实际中不常用

## 1）排序

### 1. 冒泡排序

两两比较，每次排好一个数

![](D:\githubXuan\Review\Picture\012.png)

### 2. 选择排序

找出最小的数的小标，放到第一位，依次进行

![](D:\githubXuan\Review\Picture\013.png)

### 3. 插入排序

和打扑克类似；到i时，i-1都是有序的

|      情况       |   结论   |
| :-------------: | :------: |
| 最好 - 数组正序 |  O（N）  |
| 最差 - 数组逆序 | O（N^2） |

![](D:\githubXuan\Review\Picture\014.png)

### 4. 归并排序

分成左右排序，然后外排

T（N）=2*T（N/2）+O（N）结合递归部分，可知得复杂度  O（N * logN）

![](D:\githubXuan\Review\Picture\016.png)

![](D:\githubXuan\Review\Picture\017.png)

### 5. 快速排序

**用随机一个数代替最后的数，可以转为概率期望（优化）**，**长期期望为O（N*logN）**

> **空间复杂度为O（logN）** -> 用于断点，只有断点确定了，才能进行下一步【最差是O（N），左侧的也是长期期望而言】

```java
    public  static void  quickSort(int[] arr, int from, int to){
        if(arr == null || arr.length < 2)
            return;
        if(from < to){
            //优化为随机快排
            swap(arr, from + (int)(Math.Random()*(to - from + 1)),to);
            int[] p = partition(arr, from, to);
            quickSort(arr, from, p[0] - 1);
            quickSort(arr, p[1] + 1, to);
        }
    }

    //to：  拿最后一个数作为标准
    public static int[] partition(int[] arr, int from, int to){
        int less = from - 1;
        int more = to;
        while(from < more){
            if(arr[from] < arr[to])
                swap(arr,++less,from++);
            else if(arr[from] > arr[to])
                swap(arr,--more,from);
            else
                from++;
        }
        swap(arr,more,to);
        //中间相等部分的下标
        return new int[]{less + 1, more};
    }

    public static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
```

### 6.  堆排序

时间复杂度：O（N*log N）

空间复杂度：O（1）

1. 堆结构的 heap Insert 和 heapify
2. 堆结构的增大和减少
3. 如果只是建立堆的过程，时间复杂度为O（N）
4. 优先级队列结构，就是堆结构

```java
    public static void heapSort(int[] arr){
        int len = arr.length;
        if(arr == null || len < 2)
            return ;
        for(int i = 0; i < len; i++)
            heapInsert(arr,i);
        swap(arr,0,--len);
        while(len > 0){
            heapify(arr,0,len);
            swap(arr,0,--len);
        }
    }

    private static void heapify(int[] arr, int index, int size) {
        int left = index * 2 + 1;
        while(left < size){
            int largest = left + 1 < size && arr[left + 1] > arr[left] ? left + 1: left;
            largest = arr[largest] > arr[index] ? largest : index;
            if(largest == index)
                break;
            swap(arr,largest,index);
            index = largest;
            left = index * 2 + 1;
        }
    }

    private static void heapInsert(int[] arr, int index) {
        while(arr[index] > arr[(index - 1)/2]){
            swap(arr,index,(index - 1)/2);
            index = (index - 1)/2;
        }
    }

    public static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
```

## 2）递归

1. 思想：分治

2. 自己调用自己，其实是系统压栈

3. **任何递归行为都可以改为非递归** （自己来压栈就是非递归）

4. 时间复杂度通式：T（N）=a*T（n/b）+O（n^d）

   其中，根据一步情况进行看

   a：切成几块

   n/b：子过程样本量

   n^d：操作部分

   ![](D:\githubXuan\Review\Picture\015.png)

## 3）归并

## 4）堆

是一棵完全二叉树（实际结构是数组，树只是逻辑结构）

### 4.1 大根堆

任何一棵完全二叉树的子树头部是最大值

建立大根堆的时间复杂度是O（N）

> 代码见"堆排序"

### 4.2 小根堆

任何一棵完全二叉树的子树头部是最小值

> 代码见"堆排序"

## 5）满二叉树

i节点的左孩子是 2 i+1，右孩子是 2 i+2，父节点是（i-1）/2

## 6）哈希函数

1. 输入域是无尽的
2. 输出域是有尽的
3. 输入如果一样，输出肯定一样
4. 如果有两个或多个输入，可能得到同一个输出 hashcode （产生碰撞）
5. 分散比较均匀
6. 可以打乱输入规律，实现均匀分布
7. 推论：如果原均匀分布，%m后，在0 - （m-1）上也均匀分布
8. Java的哈希表，在JVM中，桶不是链表，是红黑树、二叉平衡搜索树
9. 经典案例：大文件 + 分布式 + 查重

## 7）布隆过滤器

查一个元素是否在集合中

布隆过滤器有失误率

是一个比特类型的map

和样本单个元素大小无关，和个数相关

能极大地减小空间

三大公式【了解】   搜索公司可能会问

## 8）哈希一致性

前后端负载均衡中，如增删机器，降低数据迁移的成本

思想类似于圈（实际是数组），前端机器计算出hash值，存到离hash值最近的数组

![1545637553171](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1545637553171.png)

![1545637064035](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1545637064035.png)

上面的情况，在机器少的情况下，会有 不均匀划分 的情况，可以使用虚拟节点、路由规则进行均分【增加机器时候，同时增加虚拟节点 - 如一个房间里有四瓶香水，打碎了，每瓶香水的分子均匀飘在房子里，添加依旧如此】

## 9）并查集

如有N个样本， 查找 +  合并 在O（N）及以上 ，则单次查询为O（1）

例题：20) 岛问题

## 10）前缀树

把字母放到边上 比 放节点上更合适

例题： 21）前缀树

## 11）贪心策略

1. 不要纠结验证过程
2. 善用对数器进行比较验证
3. 贪心：制定一个标准规则，样本产生优先级

## 12）动态规划

有重复计算的递归  能改成动态规划 - 无后效性

## 13）KMP

  核心思想：让比较过的前部分指导后面，以起到优化效果

1. 无后效性：记忆性算法 - 缓存

2. 根据参数数目及变化范围创建对应的n维数组

3. 在n维数组上画出 目标点  和  base点（不依赖任何节点的）

4. 根据画出的情况，给出优化

   见例子 40 ，40 中的优化如下

   ![1546767543880](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1546767543880.png)

```java
    public static int getIndexOf(String s, String m){
        if(s == null || m == null || s.length() < 1 || m.length() < 1)
            return 0;
        char[] ss = s.toCharArray();
        char[] ms = m.toCharArray();
        int[] next = getArray(ms);
        int sindex = 0;
        int mindex = 0;
        while(sindex < s.length() && mindex < m.length()){
            if(ss[sindex] == ms[mindex]){
                sindex++;
                mindex++;
            }else if(next[mindex] == -1)
                sindex++;
            else
                mindex = next[mindex];
        }
        return mindex == m.length() ? sindex - mindex : 0;
    }

    private static int[] getArray(char[] ms) {
        if(ms == null || ms.length < 1)
            return  new int[] {0};
        int[] next = new int[ms.length];
        next[0] = -1;
        next[1] = 0;
        int cur = 2;
        int cn = 0;
        while(cur < ms.length){
            if(ms[cur - 1] == ms[cn]){
                next[cur++] = ++cn;
            }else if(cn > 0){
                cn = next[cn];
            }else
                next[cur++] = 0;

        }
        for(Integer i :next)
            System.out.println(i);
        return next;
    }

    public static void main(String[] args) {
        String str = "abcabcababaccc";
        String match = "ababa";
        System.out.println("..." + getIndexOf(str, match));
    }
```

## 14）Manacher

![1545808788661](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1545808788661.png)



```java

```



## 15）BFPRT

严格O（N），和Partition区别主要在于划分值的选择策略  

[https://blog.csdn.net/qq_37480159/article/details/76713801?utm_source=blogxgwz2]

1. 分组 

2. 组内排序

3. 中位数拿出来组成N/5大小的新数组

4. 递归调用

5. 利用4得出的值进行划分

   ![1545893475953](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1545893475953.png)



```java

```



   ## 16）单调栈

见31例题

   ## 17）Morris遍历

   时间复杂度O（N）     空间复杂度相比于正常的由O（H）变成了O（1） --> 利用了空闲的二叉树空间

   ![1546343887886](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1546343887886.png)

##    18）搜索二叉树

左小右大 - 时间复杂度与高度有关  最好O（log N） 最差O（N） 

1. AVL：高度平衡，左右树高度差不超过1
2. 红黑树：头节点必为黑、叶节点必为黑、相邻不能出现连续红、任何一个链黑色节点量差不多（最长链和最短链的高度差不超过两倍）

调整的基本动作：左旋（逆时针旋转）和右旋（顺时针旋转）

例题32

---

# 二、例题

> **在实际验证过程中，要学会利用对数器** **- 不要裸奔做笔试**

## 1）打印出无序数组b在a有序数组中没出现的数

假设a数组长度为N， b数组长度为M，则

**如果b数组短 - O（M*log M）+ O（M+N）**

1. 排序b
2. 双指针滑动

**如a数组短 - O（M*logN）**

   对于b的每个数，在a中进行二分查找

【需要确定的数，才能判断哪种办法更好】

## 2）小和问题

![](D:\githubXuan\Review\Picture\018.png)

归并 -  分批次求小和

```java
   public static int getSmallSum(int[] num){
        int len = num.length;
        if(num == null  ||  len < 2)
            return 0;
        return mergeSort(num,0,len - 1);
    }

    private static int mergeSort(int[] num, int from, int end) {
        if(from == end)
            return 0;
        int mid = from + (end - from)/2;
        return mergeSort(num,from,mid) + mergeSort(num,mid + 1,end)+ merge(num,from,mid,end);
    }

    private static int merge(int[] num, int from, int mid, int end) {
        int[] help = new int[end - from + 1];
        int i = 0;
        int p1 = from;
        int p2 = mid + 1;
        int result = 0;
        while(p1 <= mid && p2 <= end){
            //与归并相比  仅仅多了result部分
            result += num[p1] < num[p2] ? num[p1]*(end - p2 + 1):0;
            help[i++] =  num[p1] < num[p2] ? num[p1++]:num[p2++];
        }
        while(p1 <= mid){
            help[i++] = num[p1++];
        }
        while(p2 <= end){
            help[i++] = num[p2++];
        }
        for(int j = from;j <= end;j++){
            num[j] = help[j - from];
        }
        return result;
    }
```

## 3）荷兰国旗问题

要求额外空间复杂度O（1），时间复杂度O（N）

> “Partition”问题

分成 小于、等于、大于  三段求解

```java
    public static int[] partition(int[] arr, int from, int to, int num){
        int less = from - 1;
        int more = to + 1;
        int current = from;
        while(current < more){
            if(arr[current] < num)
                swap(arr,++less,current++);
            else if(arr[current] > num)
                swap(arr,--more,current);
            else
                current++;
        }
        //中间相等部分的下标
        return new int[]{less + 1, more - 1 };
    }

    public static void swap(int[] arr, int i, int j){
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
```

## 4）++相邻两数桶最大值

![1545300222808](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1545300222808.png)

```java
    public static int getMax(int[] arr){
        int len = arr.length;
        if(arr == null || len < 2)
            return 0;
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        for(int i = 0; i < len; i++){
            min = min < arr[i] ? min : arr[i];
            max = max > arr[i] ? max : arr[i];
        }
        if(max == min)
            return 0;
        boolean[] hasNum = new boolean[len + 1];
        int[] minNum = new int[len + 1];
        int[] maxNum = new int[len + 1];
        for(int i = 0; i < len; i++){
            int index = bucket(arr[i],len,min,max);
            minNum[index] = hasNum[index] ? Math.min(minNum[index],arr[i]) : arr[i];
            maxNum[index] = hasNum[index] ? Math.max(maxNum[index],arr[i]) : arr[i];
            hasNum[index] = true;
        }
        int resGap = 0;
        int lastNum = maxNum[0];
        for(int i = 1; i <= len;i++){
            if(hasNum[i]){
                resGap = Math.max(resGap,minNum[i] - lastNum);
                lastNum = maxNum[i];
            }
        }
        return resGap;
    }

    public static int bucket(long num, long len, long min, long max){
        return (int)((num - min)*len/(max - min));
    }
```

## 5）数组实现队列

```java
    private int[] arr;
    private int size;
    private int start;
    private int end;

    public ArrayQueue(int len){
        if(len < 0)
            throw new  IllegalArgumentException("init size is less than 0");
        arr = new int[len];
        size = 0;
        start = 0;
        end = 0;
    }

    public Integer peek(){
        if(size == 0)
            return null;
        return arr[start];
    }

    public Integer poll(){
        if(size == 0)
            throw new ArrayIndexOutOfBoundsException();
        size--;
        int temp = start;
        start = start == arr.length - 1 ? 0 : start + 1;
        return arr[temp];
    }

    public void push(int num){
        if(size == arr.length)
            throw new ArrayIndexOutOfBoundsException();
        size++;
        arr[end] = num;
        end = end == arr.length - 1 ? 0 : end + 1;
    }
```

## 6）数组实现栈

```java
    private int[] arr;
    private int index = 0;

    public  ArrayStack(int len){
        if(len < 0)
            throw new  IllegalArgumentException("init size is less than 0");
        arr = new int[len];
    }

    public int peek(){
        if(index == 0)
            throw new ArrayIndexOutOfBoundsException();
        return arr[index - 1];
    }

    public int pop(){
        if(index == 0)
            throw new ArrayIndexOutOfBoundsException();
        return arr[--index];
    }

    public void push(int num){
        if(index == arr.length)
            throw new ArrayIndexOutOfBoundsException();
        arr[index++] = num;
    }
```

## 7）返回最小元素的栈

![1545304552433](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1545304552433.png)

```java
    Stack<Integer> dataStack = new Stack<>();
    Stack<Integer> minStack = new Stack<>();

    public void push(Integer num){
         if(minStack.isEmpty()){
             minStack.push(num);
         }else if(num < getMin()){
             minStack.push(num);
         }else
             minStack.push(getMin());
         dataStack.push(num);
    }

    public Integer pop(){
        if(dataStack.isEmpty())
            throw  new RuntimeException();
        minStack.pop();
        return dataStack.pop();
    }

    public Integer getMin(){
        if(minStack.isEmpty())
            throw  new RuntimeException();
        return minStack.peek();
    }
```

## 8）转圈打印矩阵 

给定一个整型矩阵matrix， 请按照转圈的方式打印它

例如： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 

打印结果为： 1， 2， 3， 4， 8， 12， 16， 15， 14， 13， 9，5， 6， 7， 11， 10

【要求】 额外空间复杂度为O(1) 

```java
private static void printMatrix(int[][] matrix) {
        int lr = 0;
        int lc = 0;
        int rr = matrix.length - 1;
        int rc = matrix[0].length - 1;
        while(lr <= rr && lc <= rc)
            printNum(matrix, lr++, lc++, rr--, rc--);
    }

    private static void printNum(int[][] matrix, int lr, int lc, int rr, int rc) {
        if(lr == rr){
            for(int i = lc; i <= rc; i++)
                System.out.print(" ." + matrix[lr][i]);
        }else if(lc == rc){
            for(int i = lr; i <= rr; i++)
                System.out.print(" ." + matrix[i][lc]);
        }else{
            int currentR = lr;
            int currentC = lc;
            while(currentC < rc){
                System.out.print(" ." + matrix[lr][currentC]);
                currentC++;
            }
            while(currentR < rr){
                System.out.print(" ." + matrix[currentR][rc]);
                currentR++;
            }
            while(currentC != lc){
                System.out.print(" ." + matrix[rr][currentC]);
                currentC--;
            }
            while(currentR != lr){
                System.out.print(" ." + matrix[currentR][lr]);
                currentR--;
            }
        }
    }
```

## 9）旋转正方形矩阵 

给定一个整型正方形矩阵matrix， 请把该矩阵调整成顺时针旋转90度的样子

【要求】 额外空间复杂度为O(1) 

```java
private static void rotate(int[][] matrix) {
        int lr = 0;
        int lc = 0;
        int rr = matrix.length - 1;
        int rc = matrix[0].length - 1;
        while(lr < rr)
            rotateArray(matrix,lr++,lc++,rr--,rc--);
    }

    private static void rotateArray(int[][] m, int lr, int lc, int rr, int rc) {
        int times = rc - lc;
        int tmp = 0;
        for (int i = 0; i != times; i++) {
            tmp = m[lr][lc + i];
            m[lr][lc + i] = m[rr - i][lc];
            m[rr - i][lc] = m[rr][rc - i];
            m[rr][rc - i] = m[lr + i][rc];
            m[lr + i][rc] = tmp;
        }
    }

    private static void printMatrix(int[][] matrix) {
        for(int i = 0; i < matrix.length; i++)
            for(int j = 0; j < matrix[0].length; j++)
                System.out.print(" " + matrix[i][j]);
    }
```

## 10）“之” 字形打印矩阵 

给定一个矩阵matrix， 按照“之” 字形的方式打印这个矩阵， 例如： 1 2 3 4 5 6 7 8 9 10 11 12
“之” 字形打印的结果为： 1， 2， 5， 9， 6， 3， 4， 7， 10， 11，8， 12

【要求】 额外空间复杂度为O(1) 

```java
private static void printMatrixZigZag(int[][] matrix) {
        int ar = 0;
        int ac = 0;
        int br = 0;
        int bc = 0;
        int endr = matrix.length - 1;
        int endc = matrix[0].length - 1;
        boolean reverse = false;
        while(ar <= endr){
            printNum(matrix, ar, ac, br, bc, reverse);
            ar = ac == endc ? ar + 1 : ar;
            ac = ac == endc ? ac : ac + 1;
            //格外注意此处顺序
            bc = br == endr ? bc + 1 : bc;
            br = br == endr ? br : br + 1;
            reverse = !reverse;
        }
    }

    private static void printNum(int[][] matrix, int ar, int ac, int br, int bc, boolean reverse) {
        if(reverse){
            while(ar <= br && ac >= bc)
                System.out.print("..." + matrix[ar++][ac--]);
        }else{
            while(br >= ar && bc <= ac)
                System.out.print("..." + matrix[br--][bc++]);
        }
    }
```

## 11）判断链表是否为回文结构 

给定一个链表的头节点head， 请判断该链表是否为回文结构

例如： 1->2->1， 返回true。 1->2->2->1， 返回true ； 15->6->15， 返回true。 1->2->3， 返回false

进阶： 如果链表长度为N， 时间复杂度达到O(N)， 额外空间复杂度达到O(1) 

```java
public static class Node {
        public int value;
        public Node next;

        public Node(int data) {
            this.value = data;
        }
    }

    // 利用栈进行对比
    private static boolean isPalindrome1(Node head) {
        Stack<Integer> stack = new Stack<>();
        Node res = head;
        while(res != null){
            stack.push(res.value);
            res = res.next;
        }
        while(head != null){
            if(head.value != stack.pop()){
                return false;
            }
            head = head.next;
        }
        return true;
    }

    //快指针和慢指针  需要额外空间O（1）
    private static boolean isPalindrome2(Node head) {
        if (head == null || head.next == null) {
            return true;
        }
        Node n1 = head;
        Node n2 = head;
        //需要注意的是应该用n2.next 而不是 n1.next
        while(n2.next != null && n2.next.next != null){
            n1 = n1.next;
            n2 = n2.next.next;
        }
        Node n3 = null;
        n2 = n1.next;
        n1.next = null;
        while(n2 != null){
            n3 = n2.next;
            n2.next = n1;
            n1 = n2;
            n2 = n3;
        }
        n3 = n1;
        n2 = head;
        while(n1 != null && n2 != null){
            if(n1.value != n2.value){
                return false;
            }
            n1 = n1.next;
            n2 = n2.next;
        }
        return true;
    }

    public static void printLinkedList(Node node) {
        System.out.print("Linked List: ");
        while (node != null) {
            System.out.print(node.value + " ");
            node = node.next;
        }
        System.out.println();
    }
```

## 12）++单向链表-荷兰国旗问题

给定一个单向链表的头节点head， 节点的值类型是整型， 再给定一个整数pivot。 实现一个调整链表的函数， 将链表调整为左部分都是值小于 pivot的节点， 中间部分都是值等于pivot的节点， 右部分都是值大于 pivot的节点。
除这个要求外， 对调整后的节点顺序没有更多的要求。 例如： 链表9->0->4->5-1， pivot=3。 调整后链表可以是1->0->4->9->5， 也可以是0->1->9->5->4。 总之， 满 足左部分都是小于3的节点， 中间部分都是等于3的节点（本例中这个部
分为空） ， 右部分都是大于3的节点即可。 对某部分内部的节点顺序不做要求 

进阶： 在原问题的要求之上再增加如下两个要求。
在左、 中、 右三个部分的内部也做顺序要求， 要求每部分里的节点从左 到右的顺序与原链表中节点的先后次序一致 

```java
 public static class Node {
        public int value;
        public Node next;

        public Node(int data) {
            this.value = data;
        }
    }

    //Node类型数组  转为荷兰国旗做法  但不能保证稳定性
    private static Node listPartition1(Node head, int pivot) {
        if(head == null)
            return null;
        int i = 0;
        Node current = head;
        while(current != null){
            i++;
            current = current.next;
        }
        Node[] arr = new Node[i];
        current = head;
        for(int j = 0 ; j < i; j++){
            arr[j] = current;
            current = current.next;
        }
        partition(arr,pivot);
        for(int j = 1; j < i; j++)
            arr[j - 1].next = arr[j];
        arr[i - 1].next = null;
        return arr[0];
    }

    //先断开  后连接  确保顺序
    private static Node listPartition2(Node head, int pivot) {
        Node ss = null;
        Node se = null;
        Node es = null;
        Node ee = null;
        Node ls = null;
        Node le = null;
        Node curr = head;
        Node temp = null;
        while(curr != null){
            temp = curr.next;
            curr.next = null;
            if(curr.value < pivot){
                if(ss == null){
                    ss = curr;
                    se = curr;
                }else{
                    se.next = curr;
                    se = curr;
                }
            }else if(curr.value == pivot){
                if(es == null){
                    es = curr;
                    ee = curr;
                }else{
                    ee.next = curr;
                    ee = curr;
                }
            }else{
                if(ls == null){
                    ls = curr;
                    le = curr;
                }else{
                    le.next = curr;
                    le = curr;
                }
            }
            curr = temp;
        }
        if(se != null){
            se.next = es;
            ee = ee == null ? es :ee;
        }
        if(ee != null){
            ee.next = ls;
            le = le == null ? ls : le;
        }
        return ss != null ? ss : es != null ? es : ls;
    }

    public static void partition(Node[] arr, int pivot){
        int less = -1;
        int larger = arr.length;
        int curr = 0;
        while(curr < larger){
            if(arr[curr].value < pivot){
                swap(arr,++less,curr++);
            }else if(arr[curr].value > pivot){
                swap(arr,--larger,curr);
            }else
                curr++;
        }
    }

    public static void swap(Node[] nodeArr, int a, int b) {
        Node tmp = nodeArr[a];
        nodeArr[a] = nodeArr[b];
        nodeArr[b] = tmp;
    }

    public static void printLinkedList(Node node) {
        System.out.print("Linked List: ");
        while (node != null) {
            System.out.print(node.value + " ");
            node = node.next;
        }
        System.out.println();
    }
```

## 13）复制含有随机指针节点的链表 

一种特殊的链表节点类描述如下：
public class Node { 

public int value; 

public Node next; 

public Node rand;

public Node(int data) { 

this.value = data; 

}
} 

Node类中的value是节点值， next指针和正常单链表中next指针的意义一 样， 都指向下一个节点， rand指针是Node类中新增的指针， 这个指针可 能指向链表中的任意一个节点， 也可能指向null。 给定一个由Node节点类型组成的无环单链表的头节点head， 请实现一个 函数完成这个链表中所有结构的复制， 并返回复制的新链表的头节点。 进阶：不使用额外的数据结构， 只用有限几个变量， 且在时间复杂度为 O(N)内完成原问题要实现的函数 

## 14)....

## 15)实现二叉树的先序、 中序、 后序遍历

```java
    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    private static void preOrderRecur(Node head) {
        if(head == null)
            return ;
        System.out.print(" " + head.value);
        preOrderRecur(head.left);
        preOrderRecur(head.right);
    }

    private static void inOrderRecur(Node head) {
        if(head == null)
            return;
        inOrderRecur(head.left);
        System.out.print(" " + head.value);
        inOrderRecur(head.right);
    }

    private static void posOrderRecur(Node head) {
        if(head == null)
            return;
        posOrderRecur(head.left);
        posOrderRecur(head.right);
        System.out.print(" " + head.value);
    }

    private static void preOrderUnRecur(Node head) {
        if(head == null)
            return;
        Stack<Node> stack = new Stack<>();
        stack.push(head);
        while (!stack.isEmpty()){
            Node temp = stack.pop();
            System.out.print(" " + temp.value);
            if(temp.right != null)
                stack.push(temp.right);
            if(temp.left != null)
                stack.push(temp.left);
        }
    }

    private static void inOrderUnRecur(Node head) {
        if(head == null)
            return;
        Stack<Node> stack = new Stack<>();
        while (!stack.isEmpty() || head != null){
            if(head != null){
                stack.push(head);
                head = head.left;
            }else{
                head = stack.pop();
                System.out.print(" " + head.value);
                head = head.right;
            }
        }
    }

    private static void posOrderUnRecur1(Node head) {
        if(head == null)
            return;
        Stack<Node> stack1 = new Stack<>();
        Stack<Node> stack2 = new Stack<>();
        stack1.push(head);
        while (!stack1.isEmpty()){
            Node temp  = stack1.pop();
            stack2.push(temp);
            if(temp.left != null)
                stack1.push(temp.left);
            if(temp.right != null)
                stack1.push(temp.right);
        }
        while(!stack2.isEmpty())
            System.out.print(" " + stack2.pop().value);
    }
```

## 16)在二叉树中找到一个节点的后继节点 

现在有一种新的二叉树节点类型如下：
public class Node { public int value; public Node left;
public Node right; public Node parent;
public Node(int data) { this.value = data; }
} 

该结构比普通二叉树节点结构多了一个指向父节点的parent指针。 假设有一 棵Node类型的节点组成的二叉树， 树中每个节点的parent指针都正确地指向 自己的父节点， 头节点的parent指向null。 只给一个在二叉树中的某个节点 node， 请实现返回node的后继节点的函数。 在二叉树的中序遍历的序列中， node的下一个节点叫作node的后继节点 

```java
// 左中右
private static Node getSuccessorNode(Node test) {
        if(test == null)
            return null;
        if(test.right != null)
            return getMostLeft(test.right);
        else{
            Node par = test.parent;
            while(par != null && test != par.left){
                test = par.left;
                par = par.parent;
            }
            return test;
        }
    }

    private static Node getMostLeft(Node right) {
        while(right.left != null){
            right = right.left;
        }
        return right;
    }
```

## 17）判断一棵二叉树是否是平衡二叉树 

```java
    static boolean ans = true;

    public static boolean isBalance(Node node){
        getHeight(node, 1, ans);
        return ans;
    }

    public static int getHeight(Node node, int level, boolean isok){
        if(node == null)
            return level;
        int h1 = getHeight(node.left, level + 1, isok);
        if(!isok)
            return level;
        int h2 = getHeight(node.right, level + 1, isok);
        if(!isok)
            return level;
        if(Math.abs(h1 - h2) > 1){
            ans = false;
            isok = false;
        }
        return Math.max(h1, h2);
    }
```

## 18）判断一棵树是否是搜索二叉树 

```java
//层次遍历  ！！！ 层次！！！    
public static boolean isCBT(Node head){
        if(head == null)
            return false;
        Queue<Node> queue = new LinkedList<Node>();
        queue.offer(head);
        boolean leaf = false;
        while(!queue.isEmpty()){
            Node node = queue.poll();
            if(leaf && (node.left != null || node.right != null) || (node.left == null && node.right != null))
                return false;
            if(node.left != null)
                queue.offer(node.left);
            if(node.right != null)
                queue.offer(node.right);
            else
                leaf = true;
        }
        return true;
    }
```

## 19）已知一棵完全二叉树， 求其节点的个数 

要求： 时间复杂度低于O(N)， N为这棵树的节点个数 

```java
    static  int ans = 0;

    public static int nodeNum(Node head){
        if(head == null)
            return 0;
        int rightH = 0;
        if(head.right != null)
           rightH = getMostH(head.right,0);
        int height = getMostH(head,0);
        if(rightH + 1  == height){
            ans += (1 << (height - 1) ) + nodeNum(head.right);
            System.out.println("left full" + ans);
            return ans;
        }else{
            ans += (1 << rightH ) + nodeNum(head.left);
            System.out.println("right full" + ans);
            return ans;
        }

    }

    public static int getMostH(Node head, int level){
        while(head != null){
            head = head.left;
            level++;
        }
        return level;
    }
```

## 20）岛问题

1. 将遍历过的1改为2，清晰区别
2. 如果有连的，一次性全扫完，如此只需要遍历就可以了

```java
    private static int R;
    private static int C;

    private static int countIslands(int[][] m) {
        if(m == null)
            return 0;
        R = m.length;
        C = m[0].length;
        int ans = 0;
        for(int i = 0; i < R; i++)
            for(int j = 0; j < C; j++){
                if(m[i][j] == 1){
                    ans++;
                    infect(m, i ,j);
                }
            }
        return ans;
    }

    public static void infect(int[][] m ,int row, int col){
        if(row >= R || col >= C || col < 0 || row < 0 || m[row][col] != 1)
            return;
        m[row][col] = 2;
        infect(m,row - 1,col);
        infect(m,row + 1,col);
        infect(m,row,col + 1);
        infect(m,row,col - 1);
    }
```

## 21）前缀树

```java
public static class TrieNode{
        int end;
        int path;
        TrieNode[] next;

        public TrieNode(){
            end = 0;
            path = 0;
            next = new TrieNode[26];
        }
    }

    public static class Trie{
        TrieNode head;

        public Trie(){
            head = new TrieNode();
        }

        public void insert(String t){
            if(t == null)
                return;
            char[] str = t.toCharArray();
            TrieNode node = head;
            int index;
            for(int i = 0; i < str.length; i++){
                index = str[i] - 'a';
                if(node.next[index] == null){
                    node.next[index] = new TrieNode();
                }
                node = node.next[index];
                node.path++;
            }
            node.end++;
        }

        public  int search(String t){
            if(t == null)
                return 0;
            char[] str = t.toCharArray();
            TrieNode node = head;
            int index;
            for(int i = 0; i < str.length; i++){
                index = str[i] - 'a';
                if(node.next[index] == null){
                    return 0;
                }
                node = node.next[index];
            }
            return node.end;
        }

        public void delete(String t){
            if(search(t) == 0)
                return;
            char[] str = t.toCharArray();
            TrieNode node = head;
            int index;
            for(int i = 0; i < str.length; i++){
                index = str[i] - 'a';
                if(--node.next[index].path == 0){
                    node.next[index] = null;
                    return;
                }
                node = node.next[index];
            }
            node.end--;
        }

        public int prefixNumber(String t){
            if(t == null)
                return 0;
            char[] str = t.toCharArray();
            TrieNode node = head;
            int index;
            for(int i = 0; i < str.length; i++){
                index = str[i] - 'a';
                if(node.next[index] == null){
                    return 0;
                }
                node = node.next[index];
            }
            return node.path;
        }
    }
```



## 22）并差集

```java
public static class Node {
		// whatever you like
	}

	public static class UnionFindSet {
		public HashMap<Node, Node> fatherMap;
		public HashMap<Node, Integer> sizeMap;

		public UnionFindSet() {
			fatherMap = new HashMap<Node, Node>();
			sizeMap = new HashMap<Node, Integer>();
		}

		public void makeSets(List<Node> nodes) {
			fatherMap.clear();
			sizeMap.clear();
			for (Node node : nodes) {
				fatherMap.put(node, node);
				sizeMap.put(node, 1);
			}
		}

		private Node findHead(Node node) {
			Node father = fatherMap.get(node);
			if (father != node) {
				father = findHead(father);
			}
			fatherMap.put(node, father);
			return father;
		}
		
		public boolean isSameSet(Node a, Node b) {
			return findHead(a) == findHead(b);
		}

		public void union(Node a, Node b) {
			if (a == null || b == null) {
				return;
			}
			Node aHead = findHead(a);
			Node bHead = findHead(b);
			if (aHead != bHead) {
				int aSetSize= sizeMap.get(aHead);
				int bSetSize = sizeMap.get(bHead);
				if (aSetSize <= bSetSize) {
					fatherMap.put(aHead, bHead);
					sizeMap.put(bHead, aSetSize + bSetSize);
				} else {
					fatherMap.put(bHead, aHead);
					sizeMap.put(aHead, aSetSize + bSetSize);
				}
			}
		}

	}
```

## 23）字典排序

例如输入  "b" "ba" "cf" "de" ，结果输出应为“babcfde”

```java
	// 选对合适的贪心算法很重要
	public static class MyComparator implements Comparator<String>{
        @Override
        public int compare(String o1, String o2) {
            return (o1 + o2).compareTo(o2 + o1);
        }
    }

    public static String graSort(String[] strs){
        int len = strs.length;
        if(strs == null || len == 0)
            return "";
        Arrays.sort(strs,new MyComparator());
        String ans = "";
        for(int i = 0; i < len; i++)
            ans += strs[i];
        return  ans;
    }
```

## 24）切分金条

一块金条切成两半， 是需要花费和长度数值一样的铜板的。 比如长度为20的 金条， 不管切成长度多大的两半， 都要花费20个铜板。 一群人想整分整块金 条， 怎么分最省铜板？
例如,给定数组{10,20,30}， 代表一共三个人， 整块金条长度为10+20+30=60. 金条要分成10,20,30三个部分。 如果， 先把长度60的金条分成10和50， 花费60 再把长度50的金条分成20和30，花费50 一共花费110铜板。但是如果， 先把长度60的金条分成30和30， 花费60 再把长度30金条分成10和20， 花费30 一共花费90铜板。
输入一个数组， 返回分割的最小代价 

```java
public static class MaxComparator implements Comparator<Integer>{
        @Override
        public int compare(Integer o1, Integer o2) {
            return o2 - o1;
        }
    }

    public static class MinComparator implements Comparator<Integer>{
        @Override
        public int compare(Integer o1, Integer o2) {
            return o1 - o2;
        }
    }

    private static int minCost(int[] golden) {
        int len = golden.length;
        if(len == 0 || golden == null)
            return 0;
        PriorityQueue<Integer> minQueue = new PriorityQueue<Integer>(new MinComparator());
        PriorityQueue<Integer> maxQueue = new PriorityQueue<Integer>(new MaxComparator());
        for(int i = 0; i < len; i++){
            minQueue.offer(golden[i]);
        }
        int cost = 0;
        while(!minQueue.isEmpty()){
            while (maxQueue.isEmpty()){
                maxQueue.offer(minQueue.poll());
            }
            int minTemp = minQueue.poll();
            int maxTemp = maxQueue.poll();
            maxQueue.offer(maxTemp + minTemp);
            cost += (maxTemp + minTemp);
        }
        return cost;
    }
```

## 25）++最大利润

输入： 参数1， 正数数组costs 参数2， 正数数组profits 参数3，正数k 参数4， 正数m。costs[i]表示i号项目的花费 profits[i]表示i号项目在扣除花费之后还能挣到的钱(利润) k表示你不能并行、 只能串行的最多做k个项目 m表示你初始的资金
说明： 你每做完一个项目， 马上获得的收益， 可以支持你去做下一个项目。
输出： 你最后获得的最大钱数 

![1545699618023](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1545699618023.png)



```java
 public static class Goods{
        int cost;
        int profit;

        public Goods(int cost, int profit){
            this.cost = cost;
            this.profit = profit;
        }
    }

    public static class MaxComparator implements Comparator<Goods> {
        @Override
        public int compare(Goods o1, Goods o2) {
            return o2.profit- o1.profit;
        }
    }

    public static class MinComparator implements Comparator<Goods>{
        @Override
        public int compare(Goods o1, Goods o2) {
            return o1.cost - o2.cost;
        }
    }

    public static int getMaxProfit(Goods[] goods, int k, int m){
        if(goods == null || goods.length == 0)
            return 0;
        int len = goods.length;
        PriorityQueue<Goods> minQueue = new PriorityQueue<>(new MinComparator());
        PriorityQueue<Goods> maxQueue = new PriorityQueue<>(new MaxComparator());
        for(int i = 0; i < len; i++)
            minQueue.offer(goods[i]);
        for(int i = 0; i < k; i++){
            if(minQueue.isEmpty() && maxQueue.isEmpty())
                return m;
            while(!minQueue.isEmpty() && minQueue.peek().cost <= m)
                maxQueue.offer(minQueue.poll());
            m += maxQueue.poll().profit;
        }
        return m;
    }
```

## 26）最多宣讲场次

一些项目要占用一个会议室宣讲， 会议室不能同时容纳两个项目的宣讲。 给你每一个项目开始的时间和结束的时间(给你一个数组， 里面 是一个个具体的项目)， 你来安排宣讲的日程， 要求会议室进行 的宣讲的场次最多。 返回这个最多的宣讲场次 

```java
public static class Program {
		public int start;
		public int end;

		public Program(int start, int end) {
			this.start = start;
			this.end = end;
		}
	}

	public static class ProgramComparator implements Comparator<Program> {

		@Override
		public int compare(Program o1, Program o2) {
			return o1.end - o2.end;
		}

	}

	public static int bestArrange(Program[] programs, int start) {
		Arrays.sort(programs, new ProgramComparator());
		int result = 0;
		for (int i = 0; i < programs.length; i++) {
			if (start <= programs[i].start) {
				result++;
				start = programs[i].end;
			}
		}
		return result;
	}
```

## 27）打印一个字符串的全部子序列，包括空字符串 

```java
static List list = new ArrayList();

    public static void getSubquences(char[] tempS, int index){
        if(index == tempS.length){
            list.add(String.valueOf(tempS));
            return;
        }
        char temp = tempS[index];
        getSubquences(tempS,index + 1);
        tempS[index] = ' ';
        getSubquences(tempS,index+1);
        tempS[index] = temp;
    }

    public static void main(String[] args) {
        String s = "abs";
        char[] arry = s.toCharArray();
        getSubquences(arry,0);
        for(Object l : list)
            System.out.println(l);
    }
```

## 28）二维数组最小路径和

给你一个二维数组， 二维数组中的每个数都是正数， 要求从左上角走到右下角， 每一步只能向右或者向下。 沿途经过的数字要累加起来。 返回最小的路径和 

```java
    // 开辟dp缓存 避免重复计算
	public static int minPath(int[][] m){
        if(m == null)
            return 0;
        int R = m.length;
        int C = m[0].length;
        int[][] dp = new int[R][C];
        dp[R - 1][C - 1] = m[R -1][C - 1];
        for(int i = R - 2; i >= 0; i--){
            dp[i][C - 1] = m[i][C - 1] + dp[i + 1][C - 1];
        }
        for(int i = C - 2; i >= 0; i--){
            dp[R - 1][i] = m[R - 1][i] + dp[R - 1][i + 1];
        }
        for(int i = R - 2; i >= 0 ; i--)
            for(int j = C - 2; j >= 0;j--){
                dp[i][j] = Math.min(m[i][j] + dp[i+1][j],m[i][j]+dp[i][j+1]);
            }
        return dp[0][0];
    }
```

## 29）滑动窗口

![1546441685841](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1546441685841.png)

如上应返回 555467

```java
private static int[] getMaxW(int[] arr, int window) {
        if(arr == null || window > arr.length || window < 1)
            return null;
        LinkedList<Integer> max = new LinkedList<>();
        int[] ans = new int[arr.length - window + 1];
        int index = 0;
        for(int i = 0; i < arr.length;i++){
            while (!max.isEmpty() && arr[max.peekLast()] < arr[i]){
                max.pollFirst();
            }
            max.addLast(i);
            if(max.peekFirst() == i - window){
                max.pollFirst();
            }
            if(i >= window - 1)
                ans[index++] = arr[max.peekFirst()];
        }
        return ans;
    }
```

## 30）子数组最大差值小于某阈值，求满足条件的子数组个数

![img](https://ask.qcloudimg.com/http-save/yehe-3004461/y8mmla1xyg.png?imageView2/2/w/1620)

解法思路：

 　　本题其实是滑动窗口的变形。主体思路为：

　　１．从第一个元素开始依次向后遍历，同时维护两个窗口（由于要同时操作窗口的头部和尾部，故采用双端队列）：

　　　　　　最大值窗口（递减），头部永远存最大值

　　　　　　最小值窗口（递增），头部永远存最小值

　　２．比较两个窗口的头部元素差值，若差值大于阈值，即可跳出内循环。

　　３．跳出内循环后，检查头部元素是否过期，若过期，则清除。

复杂度：

　　时间复杂度：O(n)，注意虽然是两层循环，但元素只从滑动窗口尾部进，从头部清除，只是顺序扫描了一遍。

　　空间复杂度：O(n),这里利用两个滑动窗口分别保存最大值和最小值。

```java
    private static int getNum(int[] arr, int num) {
        if(arr == null || arr.length < 1)
            return 0;
        int res = 0;
        int i = 0;
        int j = 0;
        LinkedList<Integer> max = new LinkedList<>();
        LinkedList<Integer> min = new LinkedList<>();
        while(i < arr.length) {
            while (j < arr.length) {
                while (!min.isEmpty() && arr[min.peekLast()] >= arr[j]) {
                    min.pollLast();
                }
                min.addLast(j);
                while (!max.isEmpty() && arr[max.peekLast()] <= arr[j]) {
                    max.pollLast();
                }
                max.addLast(j);
                if (arr[max.peekFirst()] - arr[min.peekFirst()] > num)
                    break;
                j++;
            }
            if(min.peekFirst() == i)
                min.pollFirst();
            if(max.peekFirst() == i)
                max.pollFirst();
            res += j - i;
            i++;
        }
        return res;
    }
```

## 31）最大子矩阵大小

![1546488750354](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1546488750354.png)

```java
public static int maxRecSize(int[][] map){
        if(map == null || map.length == 0 || map[0].length == 0)
            return 0;
        int maxArea = 0;
        int rowL = map.length;
        int colL = map[0].length;
        int[] height = new int[colL];
        for(int i = 0; i < rowL; i++){
            for(int j = 0; j < colL; j++){
                height[j] += map[i][j] == 0 ? 0 : height[j] + 1;
            }
            maxArea = Math.max(maxArea,getArea(height));
        }
        return  maxArea;
    }

    public static int getArea(int[] height){
        if(height == null || height.length == 0)
            return 0;
        Stack<Integer> stack = new Stack<>();
        int maxArea = 0;
        int len = height.length;
        for(int i = 0; i < len; i++){
            while (!stack.isEmpty() && height[stack.peek()] >= height[i]){
                int j = stack.pop();
                int k = stack.isEmpty() ? -1 : stack.peek();
                int areaTemp = (i - k -1)*height[j];
                maxArea = Math.max(maxArea,areaTemp);
            }
            stack.add(i);
        }
        while (!stack.isEmpty()){
            int j = stack.pop();
            int k = stack.isEmpty() ? -1 : stack.peek();
            int areaTemp = (len - k -1)*height[j];
            maxArea = Math.max(maxArea,areaTemp);
        }
        return  maxArea;
    }
```

## 32）×××轮 廓 线

给定一个N行3列二维数组， 每一行表示有一座大楼， 一共有N座大楼。 所有大楼的底部都坐落在X轴上， 每一行的三个值(a,b,c)代表每座大楼的从(a,0)点开始， 到 (b,0)点结束， 高度为c。 输入的数据可以保证a<b,且a， b， c均为正数。 大楼之间可以有重合。 请输出整体的轮廓线。
例子： 给定一个二维数组 [ [1, 3, 3], [2, 4, 4], [5, 6,1] ]
输出为轮廓线 [ [1, 2, 3], [2, 4, 4], [5, 6, 1] ] 

```java
public static class Node {
		public boolean isUp;
		public int posi;
		public int h;

		public Node(boolean bORe, int position, int height) {
			isUp = bORe;
			posi = position;
			h = height;
		}
	}

	public static class NodeComparator implements Comparator<Node> {
		@Override
		public int compare(Node o1, Node o2) {
			if (o1.posi != o2.posi) {				
				return o1.posi - o2.posi;
			}
			if (o1.isUp != o2.isUp) {
				return o1.isUp ? -1 : 1;
			}
			return 0;
		}
	}

	public static List<List<Integer>> buildingOutline(int[][] buildings) {
		Node[] nodes = new Node[buildings.length * 2];
		for (int i = 0; i < buildings.length; i++) {
			nodes[i * 2] = new Node(true, buildings[i][0], buildings[i][2]);
			nodes[i * 2 + 1] = new Node(false, buildings[i][1], buildings[i][2]);
		}
		Arrays.sort(nodes, new NodeComparator());
		TreeMap<Integer, Integer> htMap = new TreeMap<>();
		TreeMap<Integer, Integer> pmMap = new TreeMap<>();
		for (int i = 0; i < nodes.length; i++) {
			if (nodes[i].isUp) {
				if (!htMap.containsKey(nodes[i].h)) {
					htMap.put(nodes[i].h, 1);
				} else {
					htMap.put(nodes[i].h, htMap.get(nodes[i].h) + 1);
				}
			} else {
				if (htMap.containsKey(nodes[i].h)) {
					if (htMap.get(nodes[i].h) == 1) {
						htMap.remove(nodes[i].h);
					} else {
						htMap.put(nodes[i].h, htMap.get(nodes[i].h) - 1);
					}
				}
			}
			if (htMap.isEmpty()) {
				pmMap.put(nodes[i].posi, 0);
			} else {
				pmMap.put(nodes[i].posi, htMap.lastKey());
			}
		}
		List<List<Integer>> res = new ArrayList<>();
		int start = 0;
		int height = 0;
		for (Entry<Integer, Integer> entry : pmMap.entrySet()) {
			int curPosition = entry.getKey();
			int curMaxHeight = entry.getValue();
			if (height != curMaxHeight) {
				if (height != 0) {
					List<Integer> newRecord = new ArrayList<Integer>();
					newRecord.add(start);
					newRecord.add(curPosition);
					newRecord.add(height);
					res.add(newRecord);
				}
				start = curPosition;
				height = curMaxHeight;
			}
		}
		return res;
	}
```

## 33）累加和为aim的最长子数组

如7,3,2,1,1,7,7,7  求aim的最长子数组的长度为4

```java
    public static int maxLength(int[] arr, int aim){
        if(arr == null || arr.length == 0)
            return 0;
        int sum = 0;
        int maxLen = 0;
        HashMap<Integer,Integer> map = new HashMap<>();
        map.put(0,-1);
        for(int i = 0,len = arr.length; i < len; i++){
            sum += arr[i];
            if(map.containsKey(sum - aim)){
                int last = map.get(sum - aim);
                maxLen = Math.max(i - last,maxLen);
            }
            if(!map.containsKey(sum))
                map.put(sum,i);
        }
        return  maxLen;
    }
```

## 34）奇偶数（某两数）数量相同

为33变形

## 35）最大子搜索二叉树

![1546565557938](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1546565557938.png)

```java
    public static class ReturnData{
        int size;
        Node node;
        int max;
        int min;

        public ReturnData(int size, Node node, int max, int min){
            this.size = size;
            this.max = max;
            this.min = min;
            this.node = node;
        }
    }

    public static Node biggestSubBST(Node head){
        ReturnData ans = process(head);
        return ans.node;
    }

    public static ReturnData process(Node head){
        if(head == null)
            return new ReturnData(0,null,Integer.MIN_VALUE,Integer.MAX_VALUE);
        ReturnData left = process(head.left);
        ReturnData right = process(head.right);
        if(left.node == head.left && right.node == head.right
        && head.value > left.max && head.value < right.min){
                return new ReturnData(left.size + 1 + right.size,head,
                        right.max > head.value ? right.max : head.value,
                        left.min < head.value ? left.min :head.value);
        }
        if(left.size > right.size)
            return new ReturnData(left.size,left.node,left.max,left.min);
        return new ReturnData(right.size,right.node,right.max,right.min);
    }
```



## 36）二叉树的最远距离

二叉树中， 一个节点可以往上走和往下走， 那么从节点A总能走到节点B。
节点A走到节点B的距离为： A走到B最短路径上的节点个数。
求一棵二叉树上的最远距离 

```java
    public static class ReturnData{
        public int h;
        public int maxD;

        public ReturnData(int h,int maxD){
            this.h = h;
            this.maxD = maxD;
        }
    }

    public static int maxDistance(Node head){
        if(head == null)
            return 0;
        ReturnData ans = process(head);
        return ans.maxD;
    }

    private static ReturnData process(Node head) {
        if(head == null)
            return new ReturnData(0,0);
        ReturnData ldata = process(head.left);
        ReturnData rdata = process(head.right);
        int curH = ldata.h + rdata.h + 1;
        int maxD = Math.max(curH,Math.max(ldata.maxD, rdata.maxD));
        int maxH = Math.max(rdata.h,ldata.h) + 1;
        return new ReturnData(maxH,maxD);
    }
```



## 37）最大活跃值

一个公司的上下节关系是一棵多叉树， 这个公司要举办晚会， 你作为组织者已经摸清了大家的心理： 一个员工的直
接上级如果到场， 这个员工肯定不会来。 每个员工都有一个活跃度的值， 决定谁来你会给这个员工发邀请函， 怎么
让舞会的气氛最活跃？ 返回最大的活跃值。
举例：
给定一个矩阵来表述这种关系
matrix =
{ 1,6
1,5
1,4
} 这个矩阵的含义是：
matrix[0] = {1 , 6}， 表示0这个员工的直接上级为1,0这个员工自己的活跃度为6
matrix[1] = {1 , 5}， 表示1这个员工的直接上级为1（他自己是这个公司的最大boss） ,1这个员工自己的活跃度为5
matrix[2] = {1 , 4}， 表示2这个员工的直接上级为1,2这个员工自己的活跃度为4
为了让晚会活跃度最大， 应该让1不来， 0和2来。 最后返回活跃度为10 

```java
public static class Node{
        int happy;
        List<Node> nexts;

        public Node(int happy){
            this.happy = happy;
            nexts = new ArrayList<>();
        }
    }

    public static class ReturnData{
        public int accept;
        public int refuse;

        public ReturnData(int a, int r){
            this.accept = a;
            this.refuse = r;
        }
    }

    public static ReturnData process(Node head){
        if(head == null)
            return new ReturnData(0,0);
        int accept = head.happy;
        int refuse = 0;
        for(int i = 0; i < head.nexts.size(); i++){
            Node temp = head.nexts.get(i);
            ReturnData rd = process(temp);
            accept += rd.refuse;
            refuse += Math.max(rd.accept, rd.refuse);
        }
        return new ReturnData(accept,refuse);
    }

    public static int maxHappy(Node head){
        ReturnData ans = process(head);
        return Math.max(ans.accept,ans.refuse);
    }
```

## 38）判断是否为完全二叉树

```java
    public static boolean isCBT(Node head){
        if(head == null)
            return false;
        boolean leaf = false;
        Queue<Node> queue = new LinkedList<>();
        queue.offer(head);
        Node left;
        Node right;
        Node node;
        while (!queue.isEmpty()){
            node = queue.poll();
            left = node.left;
            right = node.right;
            if(leaf && (left != null || right != null) || (left == null && right != null)){
                return false;
            }
            if(left != null)
                queue.offer(left);
            if(right != null)
                queue.offer(right);
            else
                leaf = true;
        }
        return true;
    }
```

## 39）×××字 符 串 计 算

```java
	public static int getValue(String str) {
		return value(str.toCharArray(), 0)[0];
	}

	public static int[] value(char[] str, int i) {
		LinkedList<String> que = new LinkedList<String>();
		int pre = 0;
		int[] bra = null;
		while (i < str.length && str[i] != ')') {
			if (str[i] >= '0' && str[i] <= '9') {
				pre = pre * 10 + str[i++] - '0';
			} else if (str[i] != '(') {
				addNum(que, pre);
				que.addLast(String.valueOf(str[i++]));
				pre = 0;
			} else {
				bra = value(str, i + 1);
				pre = bra[0];
				i = bra[1] + 1;
			}
		}
		addNum(que, pre);
		return new int[] { getNum(que), i };
	}

	public static void addNum(LinkedList<String> que, int num) {
		if (!que.isEmpty()) {
			int cur = 0;
			String top = que.pollLast();
			if (top.equals("+") || top.equals("-")) {
				que.addLast(top);
			} else {
				cur = Integer.valueOf(que.pollLast());
				num = top.equals("*") ? (cur * num) : (cur / num);
			}
		}
		que.addLast(String.valueOf(num));
	}

	public static int getNum(LinkedList<String> que) {
		int res = 0;
		boolean add = true;
		String cur = null;
		int num = 0;
		while (!que.isEmpty()) {
			cur = que.pollFirst();
			if (cur.equals("+")) {
				add = true;
			} else if (cur.equals("-")) {
				add = false;
			} else {
				num = Integer.valueOf(cur);
				res += add ? num : (-num);
			}
		}
		return res;
	}

	public static void main(String[] args) {
		String exp = "48*((70-65)-43)+8*1";
		System.out.println(getValue(exp));

		exp = "4*(6+78)+53-9/2+45*8";
		System.out.println(getValue(exp));

		exp = "10-5*3";
		System.out.println(getValue(exp));

		exp = "-3*4";
		System.out.println(getValue(exp));

		exp = "3+1*4";
		System.out.println(getValue(exp));

	}
```

## 40）换钱数

【题目】
给定数组arr， arr中所有的值都为正数且不重复。 每个值代表一种面值的货币， 每种面值的货币可以使用任意张， 再给定一个整数aim代表要找的钱数， 求换钱有多少种方法。
【举例】
arr=[5,10,25,1]， aim=0。组成0元的方法有1种， 就是所有面值的货币都不用。 所以返回1。
arr=[5,10,25,1]， aim=15。组成15元的方法有6种， 分别为3张5元、 1张10元+1张5元、 1张
10元+5张1元、 10张1元+1张5元、 2张5元+5张1元和15张1元。 所以返回6。
arr=[3,5]， aim=2。
任何方法都无法组成2元。 所以返回0。 

![1546764191266](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1546764191266.png)

```java
private static int coinsW(int[] coins, int aim) {
        int len = coins.length;
        if(coins == null || aim < 0 || len == 0)
            return 0;
        int[][] dp = new int[len][aim + 1];
        for(int i = 0; i < len; i++)
            dp[i][0] = 1;
        for(int j = 0,l = dp[0].length; j <= l && j*coins[len - 1] <= aim; j++){
            dp[len - 1][j*coins[len - 1]] = 1;
        }
        for(int i = len - 2; i >= 0; i--){
            for(int j = 0; j <= aim; j++){
                dp[i][j] = dp[i + 1][j];
                dp[i][j] += (j - coins[i]) >= 0 ? dp[i][j - coins[i]] : 0;
            }
        }
        for(int i = 0;i < dp.length;i++){
            for(int j = 0;j < dp[0].length;j++)
                System.out.print(dp[i][j] + "  ");
            System.out.println();
        }
        return dp[0][aim];
    }
```

## 41）博弈取数

有一排正数，玩家A和玩家B都可以看到。每位玩家在拿走数字的时候，都只能从最左和最右的数中选择一个。玩家A先拿，玩家B再拿，两人交替拿走所有的数字，两人都力争自己拿到的数的总和比对方多。请返回最后获胜者的分数。
例如：
5,2,3,4
玩家A先拿，当前他只能拿走5或者4。
如果玩家A拿走5，那么剩下2，3，4。轮到玩家B，此时玩家B可以选择2或4中的一个，…
如果玩家A拿走4，那么剩下5，2，3。轮到玩家B，此时玩家B可以选择5或3中的一个，…

```java
public static void main(String[] args) {
        int[] test = {5,2,3,4,51,23,5,6};
        System.out.println(getWinOne(test));
        System.out.println(getWinTwo(test));
    }

    private static int getWinTwo(int[] arr) {
        int len = arr.length;
        if(arr == null || len == 0)
            return 0;
        int sum = 0;
        for(int i = 0; i < len; i++)
            sum += arr[i];
        int[][] dp = new int[len][len];
        for(int i = 0; i < len; i++)
            dp[i][i] = arr[i];
        for (int i = 0; i < len - 1; i++)
            dp[i][i+1] = Math.max(arr[i],arr[i + 1]);
        for (int k = 2; k < arr.length; k++) {
            for (int j = k; j < arr.length; j++) {
                int i = j - k;
                dp[i][j] = Math.max(arr[i] + Math.min(dp[i + 2][j], dp[i + 1][j - 1]),
                        arr[j] + Math.min(dp[i + 1][j - 1], dp[i][j - 2]));
            }
        }
        return Math.max(dp[0][len - 1], sum - dp[0][len - 1]);
    }

    private static int getWinOne(int[] test) {
        int len = test.length;
        if(test == null || len == 0)
            return 0;
        int sum = 0;
        for(int i = 0; i < len; i++)
            sum += test[i];
        int maxA = firstP(test, 0, len - 1);
        return Math.max(maxA, sum - maxA);
    }

    public static int firstP(int[] arr, int from, int to){
        if(from == to)
            return arr[from];
        if(to - from == 1)
            return Math.max(arr[from],arr[to]);
        int max = 0;
        max = Math.max(arr[from] + Math.min(firstP(arr,from + 1, to - 1),firstP(arr, from + 2, to)),
                    arr[to] + Math.min(firstP(arr, from + 1, to - 1),firstP(arr, from, to - 2)));
        return max;
    }
```

## 42）含负数的小于k的最长子数组

给定一个数组，值可以为正、负和0，请返回累加和小于等于k的最长子数组长度

```java
public static int maxLengthAwesome(int[] arr, int k) {
		if (arr == null || arr.length == 0) {
			return 0;
		}
		int[] sums = new int[arr.length];
		HashMap<Integer, Integer> ends = new HashMap<Integer, Integer>();
		sums[arr.length - 1] = arr[arr.length - 1];
		ends.put(arr.length - 1, arr.length - 1);
		for (int i = arr.length - 2; i >= 0; i--) {
			if (sums[i + 1] < 0) {
				sums[i] = arr[i] + sums[i + 1];
				ends.put(i, ends.get(i + 1));
			} else {
				sums[i] = arr[i];
				ends.put(i, i);
			}
		}
		int end = 0;
		int sum = 0;
		int res = 0;
		for (int i = 0; i < arr.length; i++) {
			while (end < arr.length && sum + sums[end] <= k) {
				sum += sums[end];
				end = ends.get(end) + 1;
			}
			sum -= end > i ? arr[i] : 0;
			res = Math.max(res, end - i);
			end = Math.max(end, i + 1);
		}
		return res;
	}

	public static int maxLength(int[] arr, int k) {
		int[] h = new int[arr.length + 1];
		int sum = 0;
		h[0] = sum;
		for (int i = 0; i != arr.length; i++) {
			sum += arr[i];
			h[i + 1] = Math.max(sum, h[i]);
		}
		sum = 0;
		int res = 0;
		int pre = 0;
		int len = 0;
		for (int i = 0; i != arr.length; i++) {
			sum += arr[i];
			pre = getLessIndex(h, sum - k);
			len = pre == -1 ? 0 : i - pre + 1;
			res = Math.max(res, len);
		}
		return res;
	}

	public static int getLessIndex(int[] arr, int num) {
		int low = 0;
		int high = arr.length - 1;
		int mid = 0;
		int res = -1;
		while (low <= high) {
			mid = (low + high) / 2;
			if (arr[mid] >= num) {
				res = mid;
				high = mid - 1;
			} else {
				low = mid + 1;
			}
		}
		return res;
	}

	// for test
	public static int[] generateRandomArray(int len, int maxValue) {
		int[] res = new int[len];
		for (int i = 0; i != res.length; i++) {
			res[i] = (int) (Math.random() * maxValue) - (maxValue / 3);
		}
		return res;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 1000000; i++) {
			int[] arr = generateRandomArray(10, 20);
			int k = (int) (Math.random() * 20) - 5;
			if (maxLengthAwesome(arr, k) != maxLength(arr, k)) {
				System.out.println("oops!");
			}
		}
	}
```

43）倒叙栈

比如栈顶到栈底为1,2,3，操作后栈顶到栈底为3,2,1

```java
private static void reverseIt(Stack<Integer> stack) {
        if (stack.isEmpty())
            return;
        else{
            int t = getLastNum(stack);
            reverseIt(stack);
            stack.push(t);
        }
    }

    public static int getLastNum(Stack<Integer> stack){
        int reslut = stack.pop();
        if(stack.isEmpty())
            return reslut;
        int temp = getLastNum(stack);
        stack.push(reslut);
        return temp;
    }
```

## 43）漂亮砖块

![1547379994313](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547379994313.png)

```java
public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		String s = sc.nextLine();
		Set<Character> set = new HashSet<Character>();
		int count = 0;
		for (char c : s.toCharArray()) {
			if (!set.contains(c)) {
				set.add(c);
				count++;
			}
		}
		if (count > 2)
			System.out.println(0);
		else if (count == 2)
			System.out.println(2);
		else
			System.out.println(count);
		sc.close();
	}
```

44）等差数列

![1547380301103](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547380301103.png)

```java
private static boolean isAP(int[] num) {
        int len = num.length;
        if(num == null || len == 0)
            return false;
        int min = Integer.MAX_VALUE;
        int sum = 0;
        for(int i = 0; i < len; i++){
            sum += num[i];
            min = Math.min(min,num[i]);
        }
        if((sum - min*len) % (len*(len - 1)) == 0)
            return true;
        else
            return false;
    }
```

## 44）01最长连续交错串

![1547380867542](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547380867542.png)

```java
    public static int maxLength(int[] arr, int aim){
        int len = arr.length;
        if(arr == null || len == 0)
            return 0;
        for(int i = 0; i < len; i++)
            if(arr[i] == 0)
                arr[i] = -1;
        int sum = 0;
        HashMap<Integer,Integer> map = new HashMap<>();
        map.put(0,-1);
        int maxL = 0;
        for(int i = 0;i < len; i++){
            sum += arr[i];
            if(!map.containsKey(sum))
                map.put(sum,i);
            if(map.containsKey(sum - aim))
                maxL = Math.max(i - map.get(sum - aim),maxL);
        }
        return maxL;
    }
```

## 45）xxx 逆 置 B 序 列

![1547382583754](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547382583754.png)

```java
public static void main(String[] args) {
		Scanner in = new Scanner(System.in);
		while (in.hasNextInt()) {
			int n = in.nextInt();
			Deque<Integer> deque = new LinkedList<Integer>();
			boolean convert = false;
			for (int i = 0; i < n; i++) {
				if (convert) {
					deque.addLast(in.nextInt());
				} else {
					deque.addFirst(in.nextInt());
				}
				convert = !convert;
			}
			if (convert) {
				while (deque.size() != 1) {
					System.out.print(deque.pollFirst() + " ");
				}
				System.out.println(deque.pollFirst());
			} else {
				while (deque.size() != 1) {
					System.out.print(deque.pollLast() + " ");
				}
				System.out.println(deque.pollLast());
			}
		}
		in.close();
	}
```

## 46）机器人行走

![1547382669245](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547382669245.png)

```java
public static int f1(int N, int P, int K, int T) {
		if (N < 2 || P < 0 || K < 1 || T < 0 || P >= N || T >= N) {
			return 0;
		}
		if (K == 1) {
			return T == P ? 1 : 0;
		}
		if (T == 0) {
			return f1(N, P, K - 1, 1);
		}
		if (T == N - 1) {
			return f1(N, P, K - 1, T - 1);
		}
		return f1(N, P, K - 1, T - 1) + f1(N, P, K - 1, T + 1);
	}

	public static int f2(int N, int P, int K, int T) {
		if (N < 2 || P < 0 || K < 1 || T < 0 || P >= N || T >= N) {
			return 0;
		}
		int[][] dp = new int[K][N];
		dp[0][P] = 1;
		for (int i = 1; i < K; i++) {
			dp[i][0] = dp[i - 1][1];
			dp[i][N - 1] = dp[i - 1][N - 2];
			for (int j = 1; j < N - 1; j++) {
				dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j + 1];
			}
		}
		return dp[K - 1][T];
	}
```

## 47）xxx 二维平面整数点集

![1547794015476](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547794015476.png)

![1547794033276](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547794033276.png)

![1547794042818](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547794042818.png)

```java
public static class Node {
		public int x;
		public int y;

		public Node(int x, int y) {
			this.x = x;
			this.y = y;
		}
	}

	public static class MyComparator implements Comparator<Node> {

		@Override
		public int compare(Node o1, Node o2) {
			if (o1.x != o2.x) {
				return o1.x - o2.x;
			} else {
				return o2.y - o1.y;
			}
		}

	}

	public static LinkedList<Node> getRightCornerNodes(int[] x, int[] y) {
		int size = x.length;
		LinkedList<Node> res = new LinkedList<Node>();
		Node[] nodes = new Node[size];
		for (int i = 0; i < size; i++) {
			nodes[i] = new Node(x[i], y[i]);
		}
		Arrays.sort(nodes, new MyComparator());
		res.add(nodes[size - 1]);
		int rightMaxY = nodes[size - 1].y;
		for (int i = size - 2; i >= 0; i--) {
			if (nodes[i].y >= rightMaxY) {
				res.addFirst(nodes[i]);
			}
			rightMaxY = Math.max(rightMaxY, nodes[i].y);
		}
		return res;
	}
```

## 48）xxx 区间最小数*区间和

![1547820248803](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547820248803.png)

```java
public static int max(int[] arr) {
		int size = arr.length;
		int[] sums = new int[size];
		sums[0] = arr[0];
		for (int i = 1; i < size; i++) {
			sums[i] = sums[i - 1] + arr[i];
		}
		int max = Integer.MIN_VALUE;
		Stack<Integer> stack = new Stack<Integer>();
		for (int i = 0; i < size; i++) {
			while (!stack.isEmpty() && arr[stack.peek()] >= arr[i]) {
				int j = stack.pop();
				max = Math.max(max, (stack.isEmpty() ? sums[i - 1] : (sums[i - 1] - sums[stack.peek()])) * arr[j]);
			}
			stack.push(i);
		}
		while (!stack.isEmpty()) {
			int j = stack.pop();
			max = Math.max(max, (stack.isEmpty() ? sums[size - 1] : (sums[size - 1] - sums[stack.peek()])) * arr[j]);
		}
		return max;
	}
```

## 49）xxx  产品经理 程序员 IDEA

![1547820347667](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547820347667.png)

![1547820360017](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547820360017.png)

![1547820367927](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547820367927.png)

```java
	public static class Program {
		public int index;
		public int pm;
		public int start;
		public int rank;
		public int cost;

		public Program(int index, int pmNum, int begin, int rank, int cost) {
			this.index = index;
			this.pm = pmNum;
			this.start = begin;
			this.rank = rank;
			this.cost = cost;
		}
	}

	public static class PmLoveRule implements Comparator<Program> {

		@Override
		public int compare(Program o1, Program o2) {
			if (o1.rank != o2.rank) {
				return o1.rank - o2.rank;
			} else if (o1.cost != o2.cost) {
				return o1.cost - o2.cost;
			} else {
				return o1.start - o2.start;
			}
		}

	}

	public static class BigQueues {
		private List<PriorityQueue<Program>> pmQueues;
		private Program[] heap;
		private int[] indexes;
		private int heapsize;

		public BigQueues(int size) {
			this.heapsize = 0;
			heap = new Program[size];
			indexes = new int[size + 1];
			for (int i = 0; i <= size; i++) {
				indexes[i] = -1;
			}
			pmQueues = new ArrayList<>();
			for (int i = 0; i <= size; i++) {
				pmQueues.add(new PriorityQueue<Program>(new PmLoveRule()));
			}
		}

		public boolean isEmpty() {
			return heapsize == 0;
		}

		public void add(Program program) {
			PriorityQueue<Program> queue = pmQueues.get(program.pm);
			queue.add(program);
			Program head = queue.peek();
			int heapindex = indexes[head.pm];
			if (heapindex == -1) {
				heap[heapsize] = head;
				indexes[head.pm] = heapsize;
				heapInsert(heapsize++);
			} else {
				heap[heapindex] = head;
				heapInsert(heapindex);
			}
		}

		public Program pop() {
			Program head = heap[0];
			PriorityQueue<Program> queue = pmQueues.get(head.pm);
			queue.poll();
			if (queue.isEmpty()) {
				swap(0, heapsize - 1);
				heap[--heapsize] = null;
				indexes[head.pm] = -1;
			} else {
				heap[0] = queue.peek();
			}
			heapify(0);
			return head;
		}

		private void heapInsert(int index) {
			while (index != 0) {
				int parent = (index - 1) / 2;
				if (sdeLoveRule(heap[parent], heap[index]) > 0) {
					swap(parent, index);
					index = parent;
				} else {
					break;
				}
			}
		}

		private void heapify(int index) {
			int left = index * 2 + 1;
			int right = index * 2 + 2;
			int best = index;
			while (left < heapsize) {
				if (sdeLoveRule(heap[left], heap[index]) < 0) {
					best = left;
				}
				if (right < heapsize && sdeLoveRule(heap[right], heap[best]) < 0) {
					best = right;
				}
				if (best == index) {
					break;
				}
				swap(best, index);
				index = best;
				left = index * 2 + 1;
				right = index * 2 + 2;
			}
		}

		private void swap(int index1, int index2) {
			Program p1 = heap[index1];
			Program p2 = heap[index2];
			heap[index1] = p2;
			heap[index2] = p1;
			indexes[p1.pm] = index2;
			indexes[p2.pm] = index1;
		}

		private int sdeLoveRule(Program p1, Program p2) {
			if (p1.cost != p2.cost) {
				return p1.cost - p2.cost;
			} else {
				return p1.pm - p2.pm;
			}
		}

	}

	public static class StartRule implements Comparator<Program> {

		@Override
		public int compare(Program o1, Program o2) {
			return o1.start - o2.start;
		}

	}

	public static int[] workFinish(int pms, int sdes, int[][] programs) {
		PriorityQueue<Program> programsQueue = new PriorityQueue<Program>(new StartRule());
		for (int i = 0; i < programs.length; i++) {
			Program program = new Program(i, programs[i][0], programs[i][1], programs[i][2], programs[i][3]);
			programsQueue.add(program);
		}
		PriorityQueue<Integer> sdeWakeQueue = new PriorityQueue<Integer>();
		for (int i = 0; i < sdes; i++) {
			sdeWakeQueue.add(1);
		}
		BigQueues bigQueues = new BigQueues(pms);
		int finish = 0;
		int[] ans = new int[programs.length];
		while (finish != ans.length) {
			int sdeWakeTime = sdeWakeQueue.poll();
			while (!programsQueue.isEmpty()) {
				if (programsQueue.peek().start > sdeWakeTime) {
					break;
				}
				bigQueues.add(programsQueue.poll());
			}
			if (bigQueues.isEmpty()) {
				sdeWakeQueue.add(programsQueue.peek().start);
			} else {
				Program program = bigQueues.pop();
				ans[program.index] = sdeWakeTime + program.cost;
				sdeWakeQueue.add(ans[program.index]);
				finish++;
			}
		}
		return ans;
	}

	public static void printArray(int[] arr) {
		for (int i = 0; i < arr.length; i++) {
			System.out.println(arr[i]);
		}
	}

	public static void main(String[] args) {
		int pms = 2;
		int sde = 2;
		int[][] programs = { { 1, 1, 1, 2 }, { 1, 2, 1, 1 }, { 1, 3, 2, 2 }, { 2, 1, 1, 2 }, { 2, 3, 5, 5 } };
		int[] ans = workFinish(pms, sde, programs);
		printArray(ans);
	}
```

## 50）网易2018集

第七章

## 51）最多信封问题

![1547967927113](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547967927113.png)

```java
public static class Dot{
        int w;
        int h;
        
        public Dot(int w, int h){
            this.w = w;
            this.h = h;
        }
    }
    
    public static class DotComparator implements Comparator<Dot>{
        @Override
        public int compare(Dot o1, Dot o2) {
            if(o1.w != o2.w)
                return o1.w - o2.w;
            else
                return o2.h - o1.h;
        }
    }

    public static int maxEnvelopes(int[][] arr) {
        if(arr == null || arr.length == 0 || arr[0].length != 2)
            return 0;
        int len = arr.length;
        Dot[] dots = new Dot[len];
        for(int i = 0; i < len; i++)
            dots[i] = new Dot(arr[i][0],arr[i][1]);
        Arrays.sort(dots,new DotComparator());
        for(int i = 0; i < len; i++){
            System.out.println(dots[i].h + "...");
        }
        int[] ends = new int[len];
        int right = 0;
        int r = 0;
        int l = 0;
        int m = 0;
        ends[0] = dots[0].h;
        for(int i = 1; i < len; i++){
            l = 0;
            r = right;
            while(l <= r){
                m = l + (r - l)/2;
                if(dots[i].h > ends[m])
                    l = m + 1;
                else
                    r = m - 1;
            }
            right = Math.max(right, l);
            ends[l] = dots[i].h;
        }
        return right + 1;
    }

    public static void main(String[] args) {
        int[][] test = { { 4, 3 }, { 1, 2 }, { 5, 7 }, { 5, 3 }, { 1, 1 }, { 4, 9 } };
        System.out.println(maxEnvelopes(test));
    }
```

## 52）Top K问题

![1547971546327](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547971546327.png)

```java
    public static class Node{
        String str;
        int times;

        public Node(String s, int times){
            this.str = s;
            this.times = times;
        }
    }

    public static class TopKRecord{
        Node[] heap;
        int index;
        HashMap<String,Node> strNodeMap;
        HashMap<Node,Integer> indexNodeMap;

        public TopKRecord(int size){
            heap = new Node[size];
            index = 0;
            strNodeMap = new HashMap<>();
            indexNodeMap = new HashMap<>();
        }

        public void add(String str){
            if(str == null)
                return;
            int preIndex = -1;
            Node currNode = null;
            if(!strNodeMap.containsKey(str)){
                currNode = new Node(str,1);
                strNodeMap.put(str,currNode);
                indexNodeMap.put(currNode,-1);
            }else{
                currNode = strNodeMap.get(str);
                preIndex = indexNodeMap.get(currNode);
                currNode.times++;
            }
            if(preIndex == -1){
                if(index == heap.length){
                    if(currNode.times > heap[0].times){
                        indexNodeMap.put(currNode,0);
                        indexNodeMap.put(heap[0],-1);
                        heap[0] = currNode;
                        heapify(0,index);
                    }
                }else{
                    indexNodeMap.put(currNode,index);
                    heap[index] = currNode;
                    heapInsert(index++);
                }
            }else{
                heapify(preIndex,index);
            }
        }

        public void heapInsert(int index){
            while(index > 0){
                int parent = (index - 1)/2;
                if(heap[parent].times > heap[index].times){
                    swap(parent,index);
                    index = parent;
                }else
                    break;
            }
        }

        public void heapify(int index,int size){
            int l = index*2 + 1;
            int r = index*2 + 1;
            int smallest = index;
            while (l < size){
                if(heap[smallest].times > heap[l].times)
                   smallest = l;
                if(r < size && heap[smallest].times > heap[r].times)
                    smallest = r;
                if(smallest != index)
                    swap(smallest, index);
                else
                    break;
                index = smallest;
                l = index*2 + 1;
                r = index*2 + 2;
            }

        }

        public void printTopK(){
            for(int i = 0; i != heap.length; i++){
                if(heap[i] == null)
                    break;
                System.out.println(heap[i].str + "..."+ heap[i].times);
            }
        }

        public void swap(int index1, int index2) {
            indexNodeMap.put(heap[index1], index2);
            indexNodeMap.put(heap[index2], index1);
            Node tmp = heap[index1];
            heap[index1] = heap[index2];
            heap[index2] = tmp;
        }
    }
```

## 53）有序数组相加和Top K问题

![1547975284929](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547975284929.png)

```java
    public static class Node{
        int x;
        int y;
        int value;

        public Node(int x, int y, int v){
            this.x = x;
            this.y = y;
            this.value = v;
        }
    }

    public static List<Integer> topKSum(int[] arr1, int[] arr2, int k){
        if(arr1 == null || arr2 == null || k < 1)
            return null;
        int l1 = arr1.length;
        int l2 = arr2.length;
        List<Integer> result = new ArrayList<>();
        int topK = Math.min(l1*l2,k);
        Node[] heap = new Node[k+1];
        Node currNode = new Node(l1 - 1,l2 - 1,arr1[l1 - 1]+arr2[l2 - 1]);
        HashSet<String> set = new HashSet<>();
        heapInsert(heap,currNode,0);
        set.add((l1 - 1) + "_" + (l2 - 1));
        int heapSize = 1;
        while(topK > 0 && heap[0] != null){
            result.add(heap[0].value);
            topK--;
            heapSize--;
            int tempX = heap[0].x;
            int tempY = heap[0].y;
            if(heapSize != 0){
                swap(heap,0,heapSize);
                heapify(heap,0,heapSize);
            }
            if(!isContains(tempX,(tempY - 1),set)){
                currNode =  new Node(tempX,tempY - 1,arr1[tempX] + arr2[tempY - 1]);
                heapInsert(heap,currNode,heapSize++);
                set.add(tempX + "_" + (tempY - 1));
            }
            if(!isContains((tempX - 1),tempY,set)){
                currNode =  new Node(tempX - 1,tempY,arr1[tempX - 1] + arr2[tempY]);
                heapInsert(heap,currNode,heapSize++);
                set.add((tempX - 1) + "_" + tempY);
            }
        }
        return result;
    }

    public static boolean isContains(int x, int y, HashSet<String> set) {
        return set.contains(String.valueOf(x + "_" + y));
    }

    public static void heapify(Node[] heap, int index, int heapSize) {
        int left = index * 2 + 1;
        int right = index * 2 + 2;
        int largest = index;
        while (left < heapSize) {
            if (heap[left].value > heap[index].value) {
                largest = left;
            }
            if (right < heapSize && heap[right].value > heap[largest].value) {
                largest = right;
            }
            if (largest != index) {
                swap(heap, largest, index);
            } else {
                break;
            }
            index = largest;
            left = index * 2 + 1;
            right = index * 2 + 2;
        }
    }

    public static void heapInsert(Node[] heap, Node currNode, int index){
        heap[index] = currNode;
        int parent = (index - 1)/2;
        while(index != 0){
            if(heap[parent].value < heap[index].value){
                swap(heap,parent,index);
                index = parent;
                parent = (index - 1)/2;
            }else
                break;
        }
    }

    public static void swap(Node[] heap, int index1, int index2) {
        Node tmp = heap[index1];
        heap[index1] = heap[index2];
        heap[index2] = tmp;
    }
```

## 54）正数数组最小不可组成和

![1547987410980](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547987410980.png)

![1547987420037](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1547987420037.png)

```java
    public static int unformedSum1(int[] arr) {
        if (arr == null || arr.length == 0) {
            return 1;
        }
        HashSet<Integer> set = new HashSet<Integer>();
        process(arr, 0, 0, set);
        int min = Integer.MAX_VALUE;
        for (int i = 0; i != arr.length; i++) {
            min = Math.min(min, arr[i]);
        }
        for (int i = min + 1; i != Integer.MIN_VALUE; i++) {
            if (!set.contains(i)) {
                return i;
            }
        }
        return 0;
    }

    public static void process(int[] arr, int i, int sum, HashSet<Integer> set) {
        if (i == arr.length) {
            set.add(sum);
            return;
        }
        process(arr, i + 1, sum, set);
        process(arr, i + 1, sum + arr[i], set);
    }

    public static int unformedSum2(int[] arr) {
        if (arr == null || arr.length == 0) {
            return 1;
        }
        int sum = 0;
        int min = Integer.MAX_VALUE;
        for (int i = 0; i != arr.length; i++) {
            sum += arr[i];
            min = Math.min(min, arr[i]);
        }
        boolean[] dp = new boolean[sum + 1];
        dp[0] = true;
        for (int i = 0; i != arr.length; i++) {
            for (int j = sum; j >= arr[i]; j--) {
                dp[j] = dp[j - arr[i]] ? true : dp[j];
            }
        }
        for (int i = min; i != dp.length; i++) {
            if (!dp[i]) {
                return i;
            }
        }
        return sum + 1;
    }

    public static int unformedSum3(int[] arr) {
        if (arr == null || arr.length == 0) {
            return 0;
        }
        Arrays.sort(arr);
        int range = 0;
        for (int i = 0; i != arr.length; i++) {
            if (arr[i] > range + 1) {
                return range + 1;
            } else {
                range += arr[i];
            }
        }
        return range + 1;
    }
```

## 55）最小包含子串长度

![1548155775243](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1548155775243.png)

```java
    public static int minLength(String s1, String s2){
        if(s1 == null || s2 == null || s1.length() < s2.length())
            return 0;
        char[] chars1 = s1.toCharArray();
        char[] chars2 = s2.toCharArray();
        int[] map = new int[256];
        int sumMatch = chars2.length;
        for(int i = 0; i < sumMatch; i++){
            map[chars2[i]]++;
        }
        int left = 0;
        int right = 0;
        int lenC = chars1.length;
        int minLen = Integer.MAX_VALUE;
        while(right != lenC){
            if(map[chars1[right]] > 0){
                sumMatch--;
            }
            map[chars1[right]]--;
            if(sumMatch == 0){
                while(map[chars1[left]] < 0){
                    map[chars1[left]]++;
                    left++;
                }
                minLen = Math.min(minLen,right - left + 1);
                map[chars1[left]]++;
                left++;
                sumMatch++;
            }
            right++;
        }
        return minLen;
    }
```

## 56）xxx  分糖果问题

![1548157035456](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1548157035456.png)

其实是一个  **上下坡问题**

```java
	public static int candy1(int[] arr) {
		if (arr == null || arr.length == 0) {
			return 0;
		}
		int index = nextMinIndex1(arr, 0);
		int res = rightCands(arr, 0, index++);
		int lbase = 1;
		int next = 0;
		int rcands = 0;
		int rbase = 0;
		while (index != arr.length) {
			if (arr[index] > arr[index - 1]) {
				res += ++lbase;
				index++;
			} else if (arr[index] < arr[index - 1]) {
				next = nextMinIndex1(arr, index - 1);
				rcands = rightCands(arr, index - 1, next++);
				rbase = next - index + 1;
				res += rcands + (rbase > lbase ? -lbase : -rbase);
				lbase = 1;
				index = next;
			} else {
				res += 1;
				lbase = 1;
				index++;
			}
		}
		return res;
	}

	public static int nextMinIndex1(int[] arr, int start) {
		for (int i = start; i != arr.length - 1; i++) {
			if (arr[i] <= arr[i + 1]) {
				return i;
			}
		}
		return arr.length - 1;
	}

	public static int rightCands(int[] arr, int left, int right) {
		int n = right - left + 1;
		return n + n * (n - 1) / 2;
	}

	public static int candy2(int[] arr) {
		if (arr == null || arr.length == 0) {
			return 0;
		}
		int index = nextMinIndex2(arr, 0);
		int[] data = rightCandsAndBase(arr, 0, index++);
		int res = data[0];
		int lbase = 1;
		int same = 1;
		int next = 0;
		while (index != arr.length) {
			if (arr[index] > arr[index - 1]) {
				res += ++lbase;
				same = 1;
				index++;
			} else if (arr[index] < arr[index - 1]) {
				next = nextMinIndex2(arr, index - 1);
				data = rightCandsAndBase(arr, index - 1, next++);
				if (data[1] <= lbase) {
					res += data[0] - data[1];
				} else {
					res += -lbase * same + data[0] - data[1] + data[1] * same;
				}
				index = next;
				lbase = 1;
				same = 1;
			} else {
				res += lbase;
				same++;
				index++;
			}
		}
		return res;
	}

	public static int nextMinIndex2(int[] arr, int start) {
		for (int i = start; i != arr.length - 1; i++) {
			if (arr[i] < arr[i + 1]) {
				return i;
			}
		}
		return arr.length - 1;
	}

	public static int[] rightCandsAndBase(int[] arr, int left, int right) {
		int base = 1;
		int cands = 1;
		for (int i = right - 1; i >= left; i--) {
			if (arr[i] == arr[i + 1]) {
				cands += base;
			} else {
				cands += ++base;
			}
		}
		return new int[] { cands, base };
	}

	public static void main(String[] args) {
		int[] test1 = { 3, 0, 5, 5, 4, 4, 0 };
		System.out.println(candy1(test1));

		int[] test2 = { 3, 0, 5, 5, 4, 4, 0 };
		System.out.println(candy2(test2));
	}
```

## 57）完美洗牌

```java
    public static int[] perfectS(int[] arr){
        if(arr == null || arr.length == 0)
            return null;
        int n = arr.length / 2;
        int start = 0;
        while (n >= 1){
            int r = 3;
            int k = 0;
            while( r <= 2*n + 1){
                r *= 3;
                k++;
            }
            int m = (r/3 - 1)/2;
            if(n != m){
                Rotate(arr,start + m,start + n - 1);
                Rotate(arr,start + n,start + n + m - 1);
                Rotate(arr,start + m,start + n + m - 1);
            }
            for(int i = 0; i < k; i++){
                Cycle(arr, (int)Math.pow(3,i) - 1,m,start);
            }
            start = start + 2 * m;
            n -= m;
        }
        return arr;
    }

    public static void Cycle(int[] arr, int i, int n,int start){
        int mod = 2 * n + 1;
        int next = ((i + 1) * 2) % mod;
        while((next - 1) != i){
            int temp = arr[start + i];
            arr[start + i] = arr[start + next - 1];
            arr[start + next - 1] = temp;
            next = 2 * next % mod;
        }
    }

    public static void Rotate(int[] arr, int a, int b){
        while(b > a){
            int temp = arr[b];
            arr[b] = arr[a];
            arr[a] = temp;
            a++;
            b--;
        }
    }

    public static void printArray(int[] arr) {
        for (int i = 0; i != arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int len = 10;
        int[] arr = {1,2,3,4,5,6,7,8,9,10,11,12};
        printArray(perfectS(arr));
    }
```

## 58）两个长度相同的排序数组找上中位数

![1548168263571](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1548168263571.png)

```java
public static int getMid(int[] arr1, int[] arr2){
        if(arr1 == null || arr2 == null || arr1.length != arr2.length)
            return -1;
        int start1 = 0;
        int end1 = arr1.length - 1;
        int middle1 = 0;
        int start2 = 0;
        int end2 = arr2.length - 1;
        int middle2 = 0;
        int offset = 0;
        while(start1 < end1){
            middle1 = (end1 + start1) / 2;
            middle2 = (end2 + start2) / 2;
            offset = (end1 - start1 + 1) % 2;
            if(arr1[middle1] < arr2[middle2]){
                start1 = middle1 + offset;
                end2 = middle2;
            }else if(arr1[middle1] > arr2[middle2]){
                start2 = middle2 + offset;
                end1 = middle1;
            }else
                return arr1[middle1];
        }
        return Math.min(arr1[start1],arr2[start2]);
    }

    public static void main(String[] args) {
        int[] test1 = {1,2,3,4};
        int[] test2 = {3,4,5,6};
        System.out.println(getMid(test1,test2));
    }
```

## 59）两个排序数组找第k小的数

![1548223516367](C:\Users\xuan\AppData\Roaming\Typora\typora-user-images\1548223516367.png)

```java
public static int findKthNum(int[] arr1, int[] arr2, int kth) {
		if (arr1 == null || arr2 == null) {
			throw new RuntimeException("Your arr is invalid!");
		}
		if (kth < 1 || kth > arr1.length + arr2.length) {
			throw new RuntimeException("K is invalid!");
		}
		int[] longs = arr1.length >= arr2.length ? arr1 : arr2;
		int[] shorts = arr1.length < arr2.length ? arr1 : arr2;
		int l = longs.length;
		int s = shorts.length;
		if (kth <= s) {
			return getUpMedian(shorts, 0, kth - 1, longs, 0, kth - 1);
		}
		if (kth > l) {
			if (shorts[kth - l - 1] >= longs[l - 1]) {
				return shorts[kth - l - 1];
			}
			if (longs[kth - s - 1] >= shorts[s - 1]) {
				return longs[kth - s - 1];
			}
			return getUpMedian(shorts, kth - l, s - 1, longs, kth - s, l - 1);
		}
		if (longs[kth - s - 1] >= shorts[s - 1]) {
			return longs[kth - s - 1];
		}
		return getUpMedian(shorts, 0, s - 1, longs, kth - s, kth - 1);
	}

	public static int getUpMedian(int[] a1, int s1, int e1, int[] a2, int s2, int e2) {
		int mid1 = 0;
		int mid2 = 0;
		int offset = 0;
		while (s1 < e1) {
			mid1 = (s1 + e1) / 2;
			mid2 = (s2 + e2) / 2;
			offset = ((e1 - s1 + 1) & 1) ^ 1;
			if (a1[mid1] > a2[mid2]) {
				e1 = mid1;
				s2 = mid2 + offset;
			} else if (a1[mid1] < a2[mid2]) {
				s1 = mid1 + offset;
				e2 = mid2;
			} else {
				return a1[mid1];
			}
		}
		return Math.min(a1[s1], a2[s2]);
	}
```

