[TOC]

# 查找算法

## 线性查找

```java
	public static int seqSearch(int[] arr, int value){
		for(int i = 0; i < arr.length; i++){
			if(arr[i] == value){
				return i;
			}
		}
		return -1;
	}
```

## 二分查找：（要求数组有序）

```JAVA
	public static List<Integer> binarySearch2(int[] arr, int left, int right, int findVal){
		if(left > right){
			return new ArrayList<Integer>();
		}
		int mid = (left + right) / 2;
		int midVal = arr[mid];
		
		if(findVal > midVal) {
			return binarySearch2(arr, mid+1, right, findVal);
		}else if(findVal < midVal){
			return binarySearch2(arr, left, mid-1, findVal);
		} else{
			List<Integer> resIndexList = new ArrayList<Integer>();
			int temp = mid - 1;
			while(true){
				if(temp < 0 || arr[temp] != findVal){
					break;
				}
				resIndexList.add(temp);
				temp--;
			}
			resIndexList.add(mid);
			temp = mid + 1;
			while(true){
				if(temp > arr.length-1 || arr[temp] != findVal){
					break;
				}
				resIndexList.add(temp);
				temp++;
			}
			return resIndexList;
		}
	}
```

 ## 插值查找算法 （适合有序、数量大的数组）

![image-20200830221458237](../../../../Software/Typora/Picture/image-20200830221458237.png)

```java
public static int InsertValueSearch(int[] arr, int left, int right, int findVal){
		if(left > right || findVal < arr[0] || findVal > arr[arr.length - 1]){
			return -1;
		}
		int mid = left + (right - left) * (findVal - arr[left]) / (arr[right] - arr[left]);
		int midVal = arr[mid];
		if(findVal > midVal){
			return InsertValueSearch(arr, mid+1, right, findVal);
		}else if(findVal < midVal){
			return InsertValueSearch(arr, left, midVal - 1, findVal);
		}else {
			return mid;
		}
	}
```

**插**值查找注意事项：



1)对于数据量较大，**关键字分布比较均匀**的查找表来说，采用**插值查找****,** **速度较快****.**

2)关键字分布不均匀的情况下，该方法不一定比折半查找要好



## 斐波那契（黄金分割法）查找算法（有序数组）

 斐波那契查找原理与前两种相似，仅仅改变了中间结点（mid）的位置，mid 不再是中间或插值得到，而是位
于黄金分割点附近，即 mid=low+F(k-1)-1（F 代表斐波那契数列），如下图所示

![image-20200831211427453](../../../../Software/Typora/Picture/image-20200831211427453.png)

1) 由斐波那契数列 F[k]=F[k-1]+F[k-2] 的性质，可以得到 （F[k]-1）=（F[k-1]-1）+（F[k-2]-1）+1 。该式说明：
只要顺序表的长度为 F[k]-1，则可以将该表分成长度为 F[k-1]-1 和 F[k-2]-1 的两段，即如上图所示。从而中间位置为 mid=low+F(k-1)-1
2) 类似的，每一子段也可以用相同的方式分割
3) 但顺序表长度 n 不一定刚好等于 F[k]-1，所以需要将原来的顺序表长度 n 增加至 F[k]-1。这里的 k 值只要能使
得 F[k]-1 恰好大于或等于 n 即可，由以下代码得到,顺序表长度增加后，新增的位置（从 n+1 到 F[k]-1 位置），
都赋为 n 位置的值即可。

```
while(n>fib(k)-1)
k++;
```

* 代码

```JAVA
// 非递归方式得到一个斐波那契数列
	public static int[] fib(){
		int[] f = new int[maxSize];
		f[0] = 1;
		f[1] = 1;
		for(int i = 2; i < maxSize; i++){
			f[i] = f[i-1] + f[i-2];
		}
		return f;
	}
	
	// key:需要查找的数
	// 如果找到就返回对应下标
	public static int fibSearch(int[] a, int key){
		int low = 0;
		int high = a.length - 1;
		int k = 0; // 表示斐波那契分割数值的下标
		int mid = 0;
		int f[] = fib();
		// 获取斐波那契分割数值的下标
		while(high > f[k]-1){
			k++;
		}
// a = {1,8,10, 90, 1000, 1235} => a = {1,8,10, 90, 1000, 1235, 0, 0, 0}
		int[] temp = Arrays.copyOf(a, f[k]);
		for(int i = high; i < temp.length; i++){
			// a = {1,8,10, 90, 1000, 1235, 1235, 1235, 1235};
			temp[i] = a[high];
		}
		
		while(low < high) {
			mid = low + f[k-1] - 1;// 黄金分割点
			if(key < temp[mid]) { // 目标值比改黄金分割点小
				high = mid - 1; // 缩小寻找范围
                //为甚是 k--
                //说明
                //1. 全部元素 = 前面的元素 + 后边元素
                //2. f[k] = f[k-1] + f[k-2]
                //因为 前面有 f[k-1]个元素,所以可以继续拆分 f[k-1] = f[k-2] + f[k-3]
                //即 在 f[k-1] 的前面继续查找 k--
                //即下次循环 mid = low + f[k-1-1]-1
				k--; // 
			}else if(key > temp[mid]) {
				low = mid + 1;
                //为甚是 k-2
                //说明
                //1. 全部元素 = 前面的元素 + 后边元素
                //2. f[k] = f[k-1] + f[k-2]
                //3. 因为后面我们有 f[k-2] 所以可以继续拆分 f[k-1] = f[k-3] + f[k-4]
                //4. 即在 f[k-2] 的前面进行查找 k -=2
                //5. 即下次循环 mid = low + f[k - 1 - 2] - 1
				k -= 2; 
			}else { // 找到目标值
				if(mid <= high) {
					return mid;
				}else {
					return high;
				}
			}
		}
		return -1;
	}
```



# 哈希表

可做为缓存层

1）哈希表 = 数组 + 链表

2）哈希表 = 数组 + 二叉树 

![image-20200831212543627](../../../../Software/Typora/Picture/image-20200831212543627.png)

要求:
1) 不使用数据库,,速度越快越好=>哈希表(散列)
2) 添加时，保证按照 id 从低到高插入 [课后思考： 如果 id  不是从低到高插入，但要求各条链表仍是从低到
高，怎么解决?]
3) 使用链表来实现哈希表, 该链表不带表头[即: 链表的第一个结点就存放雇员信息]
4) 思路分析并画出示意图



```java
public class HashTabDemo {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		//创建哈希表
		HashTab hashTab = new HashTab(7);
		
		//写一个简单的菜单
		String key = "";
		Scanner scanner = new Scanner(System.in);
		while(true) {
			System.out.println("add:  添加雇员");
			System.out.println("list: 显示雇员");
			System.out.println("find: 查找雇员");
			System.out.println("exit: 退出系统");
			
			key = scanner.next();
			switch (key) {
			case "add":
				System.out.println("输入id");
				int id = scanner.nextInt();
				System.out.println("输入名字");
				String name = scanner.next();
				//创建 雇员
				Emp emp = new Emp(id, name);
				hashTab.add(emp);
				break;
			case "list":
				hashTab.list();
				break;
			case "find":
				System.out.println("请输入要查找的id");
				id = scanner.nextInt();
				hashTab.findEmpById(id);
				break;
			case "exit":
				scanner.close();
				System.exit(0);
			default:
				break;
			}
		}
	}
	
}

// 创建哈希表. 管理多条链表
class HashTab {
	private EmpLinkedList[] empLinkedListArray;
	int size; // 链表数
	// 构造器
	public HashTab(int size) {
		this.size = size;
		empLinkedListArray = new EmpLinkedList[size];
		// 分别初始化每个链表
		for(int i = 0; i < size; i++){
			empLinkedListArray[i] = new EmpLinkedList();
		}
	}
	
	// 添加雇员
	public void add(Emp emp){
		// 根据emp的id，取模后获得放在哪个链表的编号
		int empLinkedListNo = hashFun(emp.id);
		// 添加到对应的链表中， 此时哈希表构建完成
		empLinkedListArray[empLinkedListNo].add(emp);
	}
	
	// 遍历所有链表
	public void list(){
		for(int i = 0; i < size; i++){
			empLinkedListArray[i].list(i);
		}
	}
	
	// 根据id查找emp
	public void findEmpById(int id){
		int empLinkedListNo = hashFun(id);
		Emp emp = empLinkedListArray[empLinkedListNo].findEmpById(id);
		if(emp != null){
			System.out.printf("在第%d条链表中找到该雇员id = %d\n", (empLinkedListNo + 1), id);
		}else{
			// 没找到该emp
			System.out.println("没找到该emp");
		}
	}
	
	// 取模，为了找出Emp放在哪个链表的位置
	public int hashFun(int id) {
		return id % size;
	}
}

// 链表里的节点
class Emp {
	public int id;
	public String name;
	public Emp next;
	public Emp(int id, String name) {
		super();
		this.id = id;
		this.name = name;
	}
}

// 链表
class EmpLinkedList {
	private Emp head;
	
	public void add(Emp emp){
		if(head == null){
			head = emp;
			return;
		}
		// 不是添加第一个节点
		Emp curEmp = head;
		while(true){
			if(curEmp.next == null){
				break;
			}
			curEmp = curEmp.next; // 遍历直到该链表最后一个
		}
		curEmp.next = emp;
	}
	
	// 遍历链表的员工信息
	public void list(int no){
		if(head == null){
			System.out.println("第"+ (no+1) +"条链表位空");
			return;
		}
		System.out.println("第"+ (no+1) +"条链表的员工信息为：");
		Emp curEmp = head;
		while(true){
			System.out.printf(" => id=%d name=%s\t", curEmp.id, curEmp.name);
			if(curEmp.next == null){
				break;
			}
			curEmp = curEmp.next;
		}
		System.out.println();
	}
	
	// 根据id查找emp
	public Emp findEmpById(int id) {
		if(head == null) {
			System.out.println("链表为空");
			return null;
		}
		
		Emp curEmp = head;
		while(true){
			if(curEmp.id == id) {
				break; // 此时curEmp指向要查找的emp
			}
			if(curEmp.next == null){
				curEmp = null;
				break;
			}
			curEmp = curEmp.next; // 后移
		}
		return curEmp;
	}
}
```



# 二叉数

1) 数组存储方式的分析
优点：通过 下标方式访问元素，速度快。对于有序数组，还可使用 二分查找提高检索速度。
缺点：如果要检索具体某个值，或者 插入值( 按一定顺序) 会整体移动，效率较低 [示意图]

2) 链式存储方式的分析
优点：在一定程度上对数组存储方式有优化(比如： 插入一个数值节点，只需要将插入节点，链接到链表中即可，
删除效率也很好)。
缺点：在进行 检索时，效率仍然较低，比如(检索某个值，需要从头节点开始遍历) 【示意图】

3)  树存储方式的分析
能提高数据 存储 ， 读取的效率, 比如利用  二叉排序树(Binary Sort Tree)，既可以保证数据的检索速度，同时也
可以保证数据的 插入，删除，修改的速度。【示意图,后面详讲】

### 前中后序遍历

![image-20200904203728041](../../../../Software/Typora/Picture/image-20200904203728041.png)

```java
// 前序遍历
	public void preOrder(){
		System.out.println(this); // 输出父节点
		if(this.left != null) {
			this.left.preOrder();
		}
		if(this.right != null) {
			this.right.preOrder();
		}
	}
	
	// 中序遍历
	public void infixOrder() {
		// 递归向左子树中序遍历
		if(this.left != null){
			this.left.infixOrder();
		}
		// 输出父节点
		System.out.println(this);
		if(this.right != null) {
			this.right.infixOrder();
		}
	}
	
	// 后序遍历
	public void postOrder() {
		if(this.left != null) {
			this.left.postOrder();
		}
		if(this.right != null) {
			this.right.postOrder();
		}
		System.out.println(this);
	}
```

### 前中后序遍历查找

![image-20200904203709954](../../../../Software/Typora/Picture/image-20200904203709954.png)

```java
	// 前序遍历查找
	public HeroNode preOrderSearch(int no) {
		if(this.no == no) {
			return this;
		}
		HeroNode resNode = null;
		if(this.left != null) {
			resNode = this.left.preOrderSearch(no);
		}
		// 左节点的前序遍历找到了
		if(resNode != null) {
			return resNode;
		}
		// 左节点的前序遍历找不到，则判断右节点
		if(this.right != null) {
			resNode = this.right.preOrderSearch(no);
		}
		return resNode;
	}
	
	// 中序遍历查找
	public HeroNode infixOrderSearch(int no) {
		HeroNode resNode = null;
		if(this.left != null) {
			resNode = this.left.preOrderSearch(no);
		}
		// 左节点的中序遍历找到了
		if(resNode != null) {
			return resNode;
		}
		// 判断当前节点
		if(this.no == no) {
			return this;
		}
		// 左子树的中序遍历找不到，则判断右节点
		if(this.right != null) {
			resNode = this.right.preOrderSearch(no);
		}
		return resNode;
	}
	
	// 后序遍历查找
	public HeroNode postOrderSearch(int no) {
		HeroNode resNode = null;
		if(this.left != null) {
			resNode = this.left.preOrderSearch(no);
		}
		// 左节点的后序遍历找到了
		if(resNode != null) {
			return resNode;
		}
		// 左子树的后序遍历找不到，则判断右节点
		if(this.right != null) {
			resNode = this.right.preOrderSearch(no);
		}
		if(resNode != null) {
			return resNode;
		}
		// 最后判断当前节点
		if(this.no == no) {
			return this;
		}
		return resNode;
	}
```

### 二叉树删除节点

![image-20200904203626289](../../../../Software/Typora/Picture/image-20200904203626289.png)

```java
	// 递归删除节点
	public void delNode(int no) {
        if(this != null) {  // this表示递归调用该方法的节点
			if(this.getNo() == no){
				this = null; 
			}else{
				this.delNode(no);
			}
		} else {
			System.out.println("空数无法删除");
		}
		if(this.left != null && this.left.no == no) {
			this.left = null;  // 指向空，删除
			return;
		}
		if(this.right != null && this.right.no == no) {
			this.right = null;  // 指向空，删除
			return;
		}
		if(this.left != null) {
			this.left.delNode(no);
		}
		if(this.right != null) {
			this.right.delNode(no);
		}
	}

```



### 顺序存储二叉树(应用：堆排序)

![image-20200904212745092](../../../../Software/Typora/Picture/image-20200904212745092.png)

* 数组以二叉树前序遍历的方式进行遍历：

  ```java
  	// index为数组下标
  	public void preOrder(int index) {
  		if(arr.length == 0 || arr == null) {
  			System.out.println("数组为空");
  		}
  		System.out.println(arr[index]);
  		if(index * 2 + 1 < arr.length) {
  			preOrder(index * 2 + 1);
  		}
  		if(index * 2 + 2 < arr.length) {
  			preOrder(index * 2 + 2);
  		}
  	}
  ```

## 线索化二叉树

**线索二叉树基本介绍**

1)n个结点的二叉链表中含有n+1 【公式 2n-(n-1)=n+1】 个空指针域。利用二叉链表中的空指针域，存放指向**该**[结](https://baike.baidu.com/item/结点)[点](https://baike.baidu.com/item/结点)在**某种遍历次序**下的前驱和后继结点的指针（这种附加的指针称为"线索"）

2)这种加上了线索的二叉链表称为**线索链表**，相应的二叉树称为**线索二叉树****(Threaded** **BinaryTree****)**。根据线索性质的不同，线索二叉树可分为**前序线索二叉树、中序线索二叉树**和**后序线索二叉树**三种

3)一个结点的前一个结点，称为**前驱**结点  

4)一个结点的后一个结点，称为**后继**结点

即{1,3,6,8,10,14} => {8,3,10,1,14,6}

![image-20200911200025481](../../../../Software/Typora/Picture/image-20200911200025481.png)

* 中序线索化：

```java
// 中序线索化
	// node：当前需要 线索化的节点
	public void threadedNode(HeroNode node) {
		if(node == null) {
			return;
		}
		// 1.先线索化当前左子树
		threadedNode(node.getLeft());

		// 2.线索化当前节点,第一次执行到达该行时是最左子树的左叶子节点
		// 2.1先处理当前节点的前驱结点
		if(node.getLeft() == null) {
			// 让当前节点的左指针指向前驱结点
			node.setLeft(pre); // 如果是最左子树的左叶子节点，此时pre=null,因为在中序遍历中这个节点是第一个遍历的，没有前驱结点
			// 修改当前节点的左指针类型， 指向前驱结点
			node.setLeftType(1);
		}
		// 2.2再处理当前节点的后继节点
		if(pre != null && pre.getRight() == null) { // 左子树最左节点不进入判断
			// 让前驱结点的右指针指向当前节点
			pre.setRight(node);	  // 假如node是左子树的最左叶子点的父节点，此时pre为左子树的最左叶子点
			// 让前驱结点的右指针类型为指向后继节点
			pre.setRightType(1);  // pre指向后继节点
		}
		pre = node; // 每次处理完一个节点后，当前节点就为要处理的下个节点的前一个节点
		
		// 3.线索化右子树
		threadedNode(node.getRight());
	}
```

* HeroNode

  ```
  	// leftType=0时，表示指向左子树，1表示指向前驱结点或指向null
  	// rightType=0时，表示指向右子树，1表示指向后继结点或指向nul
  	private int leftType;
  	private int rightType;
  ```

  

* 测试：

```java
	public static void main(String[] args) {
		// TODO Auto-generated method stub
//		 测试中序线索化二叉树
		HeroNode root = new HeroNode(1, "宋江");
		HeroNode node2 = new HeroNode(3, "吴用");
		HeroNode node3 = new HeroNode(6, "卢俊义");
		HeroNode node4 = new HeroNode(8, "林冲");
		HeroNode node5 = new HeroNode(10, "关胜");
		HeroNode node6 = new HeroNode(14, "tom");
		
		root.setLeft(node2);
		root.setRight(node3);
		node2.setLeft(node4);
		node2.setRight(node5);
		node3.setLeft(node6);
		
		// 测试线索化
		ThreadedBinaryTree threadedBinaryTree = new ThreadedBinaryTree();
		threadedBinaryTree.setRoot(root);
		threadedBinaryTree.threadedNode();
		
		HeroNode left = node5.getLeft();
		System.out.println(left);  // 输出：HeroNode [no=3, name=吴用]
	}
```

* 遍历线索化二叉树

```java
// 遍历中序线索化二叉树
	public void threadedList() {
		// 定义变量存储当前遍历的节点
		HeroNode node = root;
		while(node != null) {
			// 循环找到leftType == 1的节点，第一个找到就是8节点
			// 后面随着遍历而变化，因为当leftTyoe == 1 时，说明节点是按照线索化 的
			while(node.getLeftType() == 0) { // 0表示左指针类型是指向左子树，第一次找到的是左子树最左节点8
				node = node.getLeft(); 
			}
			System.out.println(node);
			// 如果当前节点的右指针指向的是后继节点，就一直输出，第一次当前节点为8，右指针就指向了后继节点3
			while(node.getRightType() == 1) {
				node = node.getRight();
				System.out.println(node);
			}
			// 替换遍历的节点, 才能
			node = node.getRight(); 
		}
	}
```

  # 堆排序

![image-20200912163740133](../../../../Software/Typora/Picture/image-20200912163740133.png)

![image-20200912163810013](../../../../Software/Typora/Picture/image-20200912163810013.png)

### **堆排序基本思想**

堆排序的基本思想是：

1)将待排序序列构造成一个大顶堆

2)此时，整个序列的最大值就是堆顶的根节点。

3)将其与末尾元素进行**交换，此时末尾就为最大值。**

4)然后将剩余n-1个元素重新构造成一个堆，这样会得到n个元素的次小值。如此反复执行，便能得到一个有序序列了。

可以看到在构建大顶堆的过程中，每次遍历出最末尾元素，元素的个数逐渐减少，最后就得到一个有序序列了.

### 代码实现

* 调整为大顶堆

```java
/**
	 * 将一个数组（二叉树），调整成一个大顶堆
	 * 每次递归调整，i都是从下到上，从左到右，寻找到的非叶子节点
	 * @param arr  
	 * @param i  表示非叶子节点在数组中的索引：
	 * @param length  表示对多个元素继续调整，length在主键减少
	 */
	public static void adjustHeap(int arr[], int i, int length) {
		// 非叶子节点, temp保存的是
		int temp = arr[i];
		// 开始调整，找出最大值, 每次从当前非叶子节点的左子节点开始向下遍历直到最左叶子节点
		for(int k = i * 2 + 1; k < length; k = k * 2 + 1) {
			 if(k + 1 < length && arr[k] < arr[k + 1]) { // 比较左右子节点，找出大的那个节点
				 k++;  // 右子结点更大，所有将k指向子节点中更大的右子节点
			 }
			 if(arr[k] > temp) { // 如果相对大的那个子节点大于父节点
				 arr[i] = arr[k];  // 父节点的值改为值最大的那个子节点
				 i = k;  // i指针指向值最大的子节点的位置，然后继续向下遍历比较
			 } else {
				 // 因为遍历比较的顺序是从左至右，从下至上，所以
				 break; // 结束比较该节点，继续比较下一个节点
			 }
		}
		// 当for循环结束后，我们已经将以i为父节点的树最大值放在了该次调整最顶（局部）的i位置上
		// 若存在子节点大于父节点的情况，则交换值，若不存在则此操作不改变arr[i]
		arr[i] = temp;  
	}
```

* 堆排序

  ```java
  	public static void heapSort(int arr[]) {
  		System.out.println("arr调整为大顶堆");
  		for(int i = arr.length / 2 - 1; i >= 0; i--) {
  			adjustHeap(arr, i, arr.length);
  		}
  		// 将对顶元素与末尾元素交换，将最大元素放到数组末端，然后
  		System.out.println("堆排序");
  		for(int j = arr.length - 1; j > 0; j--) {
  			adjustHeap(arr, 0, j);  // 每次从顶端开始调整为大顶堆
  			arr[0] ^= arr[j];
  			arr[j] ^= arr[0];
  			arr[0] ^= arr[j];
  		}
  		System.out.println(Arrays.toString(arr));
  	}
  ```

# 赫夫曼树

**基本介绍**

1)给定n个权值作为n个[叶子结点](https://baike.baidu.com/item/叶子结点/3620239)，构造一棵二叉树，若该树的带权路径长度(wpl)达到最小，称这样的二叉树为**最优二叉树**，也称为**哈夫曼树(Huffman Tree)**, 还有的书翻译为霍夫曼树。

2)赫夫曼树是带权路径长度最短的树，权值较大的结点离根较近。

##### **几个重要概念**

1)路径和路径长度：在一棵树中，从一个结点往下可以达到的孩子或孙子结点之间的通路，称为路径。通路中分支的数目称为路径长度。若规定根结点的层数为1，则从根结点到第L层结点的路径长度为L-1

2)**结点的权及带权路径长****度：**若将树中结点赋给一个有着某种含义的数值，则这个数值称为该结点的权。**结点的带权路径长度**为：从根结点到该结点之间的路径长度与该结点的权的乘积

3)**树****的带权路径长****度：**树的带权路径长度规定为所有**叶子结点**的带权路径长度之和，记为WPL(weighted path length) ,权值越大的结点离根结点越近的二叉树才是最优二叉树。

4)**WPL****最小的就是赫夫曼树**

### 构成赫夫曼树的步骤：

{13, 7, 8, 3, 29, 6, 1} 

1)从小到大进行排序, 将每一个数据，每个数据都是一个节点 ， 每个节点可以看成是一颗最简单的二叉树

2)取出根节点权值最小的两颗二叉树

3)组成一颗新的二叉树, 该新的二叉树的根节点的权值是前面两颗二叉树根节点权值的和 

4)再将这颗新的二叉树，以根节点的权值大小 再次排序， 不断重复 1-2-3-4 的步骤，直到数列中，所有的数据都被处理，就得到一颗赫夫曼树





























