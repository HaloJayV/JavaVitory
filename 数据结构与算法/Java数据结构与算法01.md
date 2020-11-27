[TOC]

### java中的ArrayList 、List、LinkedList、Collection

代码：D:\JAVA\Java_Learning\Elipse_Project\workspace200301EE\DataStructures

* 除了vector，statck、hashtable、enumeration、StringBuffer是线程安全，其他的集合类都是线程不安全的

- Set（集）：集合中的元素不按特定方式排序，并且没有重复对象。他的有些实现类能对集合中的对象按特定方式排序。
  - HashSet类按照哈希算法来存取集合中的对象，具有很好的存取性能。当HashSet向集合中加入一个对象时，会调用对象的hashCode()方法获取哈希码，然后根据这个哈希码进一步计算出对象在集合中的存放位置。
  - TreeSet实现了SortedSet接口，可以对集合中的元素排序
- List（列表）：集合中的元素按索引位置排序，可以有重复对象，允许按照对象在集合中的索引位置检索对象。
- Vector：线程安全，是ArrayList的前身
- Map（映射）：集合中的每一个元素包含一对键对象和值对象，集合中没有重复的键对象，值对象可以重复。他的有些实现类能对集合中的键对象进行排序。

<img src="../../../../Software/Typora/Picture/06144256-88e64aab8a3641eabbc144bb260e2364.png" alt="img"  />

![image-20200705185401616](../../../../Software/Typora/Picture/image-20200705185401616.png)



![image-20200705185416229](../../../../Software/Typora/Picture/image-20200705185416229.png)



* Iterator接口

  * public boolean hasNext()：判断是否还有下一个元素。
  * public Object next()：取得下一个元素，注意返回值为 Object，可能需要类型转换。如果不再有可取元素，则抛出NoSuchElementException异常。在使用该方法之前，必须先使用hasNext()方法判断。
  * public void remove()：删除当前元素，很少用。

* Collection接口

  * public boolean add(Object?o)：往集合中添加新元素。添加成功，返回true，否则返回false。
  * public Iterator iterator()：返回Iterator对象，这样就可以遍历集合中的所有元素了。
  * public boolean contains(Object?o)：判断集合中是否包含指定的元素。
  * public int size()：取得集合中元素的个数。
  * public void clear()：删除集合中的所有元素。 

* Java不提供直接继承自Collection的类，只提供继承自Collection的“子接口”如List和Set。

* HashSet：线程不安全

  *   HashSet类按照哈希算法来存取集合中的对象，具有很好的存取性能。当HashSet向集合中加入一个对象时，会调用对象的hashCode()方法获取哈希码，然后根据这个哈希码进一步计算出对象在集合中的存放位置。

* Hashtable：就比Hashmap多了个线程安全。

* ConcurrentHashMap:是一种高效但是线程安全的集合。

* Stack：栈，也是线程安全的，继承于Vector。

* List：

  * List是一种有序集合，List中的元素可以根据索引进行取得/删除/插入操作，提供listIterator()方法，返回一个ListIterator接口对象，能向前或向后遍历。List接口的实现类主要有ArrayList，LinkedList，Vector，Stack等。

* ArrayList：线程不安全

  * 实现了可变大小的数组，容量可随着不断添加新元素而自动增加，插入前调用ensureCapacity方法来增加ArrayList的容量
  * public boolean add(Object?o)：添加元素，添加n个元素需要O(n)的时间。其他的方法运行时间为线性
  * public void add(int index, Object element)：在指定位置添加元素
  * public Iterator iterator()：取得Iterator对象便于遍历所有元素
  * public Object get(int?index)：根据索引获取指定位置的元素
  * public Object set(int index,Object element)：替换掉指定位置的元素
  * Collections.sort(List list)：对List的元素进行自然排序
  * Collections.sort(List list, Comparator comparator)：对List中的元素进行客户化排序 

* LinkedList：线程不安全

  * 提供额外的get，remove，insert方法在LinkedList的首部或尾部
  * LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）
  * LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List： `List list = Collections.synchronizedList(new LinkedList(...));`

* ### Map

  * HashMap按照哈希算法来存取键对象，有很好的存取性能。和HashSet一样，要求当两个键对象通过equals()方法比较为true时，这两个键对象的hashCode()方法返回的哈希码也一样。
  * TreeMap实现了SortedMap接口，能对键对象进行排序。同TreeSet一样，TreeMap也支持自然排序和客户化排序两种方式。
  * public Object put(Object key, Object value)：插入元素
  * public Object get(Object?key)：根据键对象获取值对象
  * public Set keySet()：取得所有键对象集合
  * public Collection values()：取得所有值对象集合
  * public Set entrySet()：取得Map.Entry对象集合，一个Map.Entry代表一个Map中的元素 

  ![img](../../../../Software/Typora/Picture/06151803-d9a5bb33dcec4f62a2abaf882d42056d.png)

* 应用场景选择

  * 如果涉及到堆栈，队列等操作，应该考虑用List；对于需要快速插入，删除元素，应该使用LinkedList；如果需要快速随机访问元素，应该使用ArrayList。  
  * 尽量返回接口而非实际的类型，如返回List而非ArrayList，这样如果以后需要将ArrayList换成LinkedList时，客户端代码不用改变。这就是针对抽象编程。

### 数据结构

* 数据结构包括线性结构和非线性结构

* 线性结构：一对一的线性关系，包括顺序存储（顺序表）和链式存储结构（链表)，常见：数组、队列、链表和栈 
  
  * 顺序表的存储元素是连续的，链表的存储元素不一定是连续的，元素节点保存数据元素以及相邻元素的地址信息
  
* 非线性结构：二维数组，多维数组，广义表，树结构、图结构

* 当数组中大部分元素为同一个值时，可使用洗漱数组来保存该数组，

* 稀疏数组：第一行记录共有几行几列，有多少个不同值，其他行记录这些不同值在原数组的坐标

* ```
  	public static void main(String[] args) throws FileNotFoundException, IOException {
    		int chessArr1[][] = new int[11][11];  
    		chessArr1[1][2] = 1;
    		chessArr1[2][3] = 2;
    		System.out.println("原始的二维数组");
    		for(int[] row : chessArr1){ // 遍历每一行元素
    			for(int data : row){
    				System.out.printf("%d\t", data);
    			}
    			System.out.println();
    		}
    		// 将二位数组转换为稀疏数组
    		int sum = 0;
    		for (int i = 0; i < 11; i++) {
    			for (int j = 0; j < 11; j++) {
    				if(chessArr1[i][j] != 0){
    					sum++;
    				}
    			}
    		}
    		// 创建对应稀疏数组
    		Integer sparseArr[][] = new Integer[sum+1][3];
    		sparseArr[0][0] = 11;
    		sparseArr[0][1] = 11;
    		sparseArr[0][2] = sum;
    		
    		int count = 0;
    		for (int i = 0; i < 11; i++) {
    			for (int j = 0; j < 11; j++) {
    				if(chessArr1[i][j] != 0){
    					count++;
    					sparseArr[count][0] = i;
    					sparseArr[count][1] = j;
    					sparseArr[count][2] = chessArr1[i][j];
    				}
    			}
    		}
    		System.out.println();
    		for (int i = 0; i < sparseArr.length; i++) {
    			System.out.printf("%d\t%d\t%d\t\n", sparseArr[i][0], sparseArr[i][1], sparseArr[0][2]);
    			
    		}
    		ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("D:/os.txt"));
    		os.writeObject(sparseArr);
    		os.close();
    		// 稀疏数组恢复成二维数组
    		int chessArr2[][] = new int[sparseArr[0][0]][sparseArr[0][1]];
    		for (int i = 0; i < sparseArr.length; i++) {
    			chessArr2[sparseArr[i][0]][sparseArr[i][1]] = sparseArr[i][2];
    		}
    		System.out.println("稀疏数组恢复成二维数组");
    		for(int[] row : chessArr2){ // 遍历二维数组的每一行元素
    			for(int data : row){
    				System.out.printf("%d\t", data);
    			}
    			System.out.println();
    		}
    	}
  ```

  

### 队列queue

* 队列Queue：有序列表，可以用数组或链表实现，遵循先入选出的原则
  * 存入数据时，rear+1，取出数据时，front+1； 当front==rear时队列为空x
  *  若rear < maxSize-1，可以存入数据； 否则当rear == maxSize-1 表示队列已满，无法存入数据
* **使用数组模拟环形队列：front指向队列第一个元素，rear指向队列中最后一个元素的后一个位置，两者初始值都为0**
* **当队列满时，条件是(rear+1) % maxSize == front      队列空：rear == front**     注意：(rear+1)是防止数组下标越界
  * **maxSize-1 这一位空出来作为一个约定** ,  取出数据后，front++， 0~front之间的空间还可以使用
  * **队列中有效的数据个数：(rear + maxSize - front) % maxSize**
* ![ ](../../../../Software/Typora/Picture/image-20200623193109398.png)

### 单链表SingleLinkedList

* 单链表是以节点的方式存储，每个节点包括data域，next域，指向下一个节点，注意:各个节点不一定是连续存储的，链表分带头节点的链表和无头节点链表
  * 添加：先常见一个head头节点，表示单链表的头，指向第一个节点，每添加一个节点直接加入到链表的最后遍历
    * 在添加和显示时需要遍历，此时需要新建临时变量temp，当`temp = temp.next`; 表示temp指向下个节点，  `temp.next==null`表示最后一个节点
  * 插入节点：根据排名将英雄插入到指定位置
    * 通过辅助变量指针先找到新添加节点的位置，并且该temp要位于添加位置的前一个节点，再 `newNode.next = temp.next`,  然后`temp.next=newNode`
  * 删除节点：`temp.next = temp.next.next`

* 获取到单链表的有效节点个数: 创建临时指针cur，通过遍历`while(cur!=null){length++};`  可以获取节点个数

  * ```
    	public static int getLength(HeroNode head){
      		if(head.next == null){ // 空链表 
      			return 0;
      		}
      		int length = 0;
      		HeroNode cur = head.next;
      		while(cur != null){
      			length++;
      			cur = cur.next;
      		}
      		return length;
      	}
    ```

* 新浪：查找链表倒数第K个节点: 只需要从0遍历到 (size - k) , cur = cur.next,  便可查找得到结果

  * ```
    	// 查找链表倒数第K个节点
      	public static HeroNode findLastIndexNode(HeroNode head, int index){
      		if(head.next == null){
      			return null;
      		}
      		int size = getLength(head);
      		if(index <= 0 || size < index){
      			return null;
      		}
      		HeroNode cur = head.next;
      		for(int i = 0; i < size - index; i++){
      			cur = cur.next;
      		}
      		return cur;
      	}
    ```

* 腾讯：将单链表进行反转: new一个新的链表，遍历原链表然后逐个以头插法插入新链表，最后将head头节点指针指向新链表的首节点

  * ```
    	// 将单链表进行反转
      	public static void reverseList(HeroNode head){
      		if(head.next == null || head.next.next == null){
      			return;
      		} 
      		HeroNode cur = head.next;
      		HeroNode next = null; // 指向当前节点的下一个节点
      		HeroNode reverseHead = new HeroNode(0, "", "");
      		
      		// 遍历原来的链表
      		while(cur != null){
      			next = cur.next;  // 保存cur的下一个节点
      			
      			// 节点插入
      			cur.next = reverseHead.next;  // cur指针指向新链表的最前端, 建立连接
      			reverseHead.next = cur;  // 将源链表的节点连接到反转链表的头节点，属于头插法
      			
      			cur = next; // cur重新回到原链表，并且作为原先节点的下一个节点，依次往复
      		}
      		head.next = reverseHead.next;
      	}
    ```

* 百度：逆序打印链表要求不破坏结构：使用栈结构的先进后出即可

  * ```
    	// 百度：逆序打印链表要求不破坏结构
      	public static void reversePrint(HeroNode head){
      		if(head.next == null ){
      			System.out.println("空链表无法打印");
      			return;
      		}
      		Stack<HeroNode> stack = new Stack<HeroNode>();   
      		HeroNode cur = head.next;
      		while(cur != null){
      			stack.push(cur);
      			cur = cur.next;
      		}
      		while(stack.size() > 0){
      			System.out.println(stack.pop());
      		}
      	}
    ```

### 双向链表

* 遍历：和单链表一样，可以向前可以向后
* 添加(默认在尾部添加)：先找到最后节点， 再 `temp.next = newNode`   最后  `newNode.pre = temp;`
* 修改：同单链表
* 删除：直接找到要删除的节点如temp， `temp.pre.next = temp.next`  然后  `temp.next.pre = temp.pre // 注意：如果是删除最后节点，不需这句代码`.



###  环形链表 和 JosePhu 约瑟夫问题 

环形链表：

* ![image-20200627145738762](../../../../Software/Typora/Picture/image-20200627145738762.png)

  * 在构建环形链表，添加新节点时，让最后的节点先指向新节点，然后新节点再指向first节点，最后curNode=newNode，以便下次辅助构建环形链表 

  * 遍历环形链表：`curBoy.getNext() == first  //表示已经遍历完成`

* 约瑟夫问题：![image-20200627152942819](../../../../Software/Typora/Picture/image-20200627152942819.png)
  * 思路：
    * 1. 先创建一个辅助指针变量helper，事先指向环形链表的最后节点，first一直在helper前面
      2. 报数前让first和helper移动 k - 1 次， 然后开始报数时，让first和helper指针同时移动 m - 1 次
      3. 虽然将first指向的小孩节点出圈， `first = first.next;     helper.next = first` ，而原来first指向的节点没有引用会被GC

* ```
  	// 约瑟夫问题：根据输入计算出圈的顺序， counNum:数几次 
    	public void countBoy(int startNo, int countNum, int nums) {
    		if (first == null && startNo < 1 && startNo > nums) {
    			System.out.println("参数有误");
    			return;
    		}
    		Boy helper = first;
    		// 将helper设为最后节点, helper 总是在最后节点
    		while (true) {
    			if (helper.getNext() == first) {
    				break;
    			}
    			helper = helper.getNext();
    		}
    		
    		// 小孩报数前，先将first移动到startNo，移动startNo-1次， helper一直跟在后面
    		for(int i = 0; i < startNo - 1; i++){
    			first = first.getNext();
    			helper = helper.getNext();
    		}
    		
    		// 小孩报数时，first和 helper开始移动
    		while(true){
    			if(first == helper){
    				break;
    			}
    			for(int i = 0; i < countNum - 1; i++){
    				first = first.getNext();
    				helper = helper.getNext();
    			}
    			System.out.printf("小孩%d出圈", first.getNo());
    			System.out.println();
    			first = first.getNext();
    			helper.setNext(first);
    		}
    		System.out.printf("最后留在圈中的小孩编号:%d", first.getNo());
    	}
  }
  ```

  

##  栈

* 栈的应用场景：
  * 子程序的调用：在跳转子程序之前，会先将下个指令的地址存到栈中，直到子程序执行完后再将地址取出，才能返回到原来的程序中
  * 处理递归调用：与子程序调用类似，只是除了将下个指令的地址外，也将参数、区域变量等数据存入堆栈中 
  * 表达式的转换(中缀表达式转后缀表达式)与求值，二叉树的遍历，图的深度优先搜索法等

* 使用数组模拟栈：定义top表示栈顶并初始化为-1，入栈时top++，出栈时top--

* 使用栈完成计算表达式功能：

  * ###### 创建2个栈，分别用来存放数和表达式，接着开始扫描表达式

  * 当扫描到的是一个符号时，如果符号栈为空就直接入栈，不为空则判断：如果当前操作符优先级小于等于栈中的操作符，就需要从数栈中pop出两个数，栈中pop出1个符号进行运算，得到结果入栈；繁殖直接入符号栈

  * 当扫描完毕，就按顺序从数栈和符号栈中pop出相应的数和符号并运行，最后再数栈中只有一个数字便是最终结果

---

### 前缀、中缀、后缀（逆波兰表达式）：

* 前缀表达式：从左至右扫描前缀表达式，先将所有数字压栈，然后从左开始扫描符号，扫描到符号时弹出2个数字进行计算后再压栈，依次反复得出最终结果
* 中缀表达式： 需要判断符号优先级，一般都是转换为计算机比较容易计算的后缀表达式
* 后缀表达式：从左至右扫描后缀表达式，遇到数字将数字压栈，遇到运算发时弹出栈顶两个数，进行计算后将结果入栈，重复直到表达式最右端，得出结果
  
* 逆波兰计算器：输入后缀表达式，使用栈计算结果。思路：创建List用来将表达式用分开并存放进该集合，随后创建一个栈并开始在数组内扫描每个元素，当扫描到数字时存放到栈中，当扫描到运算符时，从栈中弹出2个元素并与这个运算符进行计算后再压入栈中，以此往复，最后栈的最后一个元素便是结果
  
  * ```
    	// 将逆波兰表达式放入list
      	public static List<String> getListString(String suffixExpression){
      		String[] split = suffixExpression.split(" ");
      		List<String> list = new ArrayList<String>();
      		for (String ele : split) {
      			list.add(ele);
      		}
      		return list; 
      	}
    ```
  
    
  
* 中缀表达式转为后缀表达式：
  
  * 思路：初始化运算符栈s1和存储中间结果的集合s2，从左到右扫描中缀表达式。开始扫描，如果是运算符则需要比较s1栈顶运算符的优先级，若优先级比s1栈顶运算符的高，则直接压入s1，否则将栈顶运算符弹出并存入s2，才能压栈s1。若是'('则直接压入s1，若是‘）’则需要将s1逐次弹出运算符直到弹出‘（’，此时弹出的除括号被丢弃外，其他运算符存入s2。数字则直接存入s2。当表达式遍历完成后，将s1的运算符依次存入s2 
  
  * ```
    	// 中缀转为后缀表达式
      	public static List<String> parseSuffixExpressionList(List<String> ls){
      		Stack<String> s1 = new Stack<String>(); // 符号栈
      		List<String> s2 = new ArrayList<String>(); // 存放操作数和中间结果
      		for (String item : ls) {
      			if(item.matches("\\d+")){
      				s2.add(item);
      			}else if(item.equals("(")){
      				s1.push(item);
      			}else if(item.equals(")")){
      				while(!s1.peek().equals("(")){
      					s2.add(s1.pop());
      				}
      				s1.pop(); // 将 '(' 弹出，消除括号
      			}else {  // 遍历到运算符
      				while(s1.size() != 0 && Operation.getValue(s1.peek()) >= Operation.getValue(item)){
      					s2.add(s1.pop());  // 该运算符优先级小于等于s1的栈顶运算符优先级
      				}
      				s1.push(item);
      			}
      		}
      		// 将s1剩余的运算符压入s2	
      		while(s1.size() != 0){
      			s2.add(s1.pop());
      		}
      		return s2;
      	}
    ```
  
    

---

## 递归（基于栈实现）

* 递归规则：每次递归调用一个方法时，都会在栈中开辟一片独立的内存空间(栈)，每个空间的数据（局部变量）都是独立的。如果方法中使用的变量是引用类型变量，就会共享该引用类型的数据。递归必须向退出递归的条件逼近，否则就无限递归，出现StackOverflowError。谁调用就返回给谁。

* 1.迷宫回溯问题思路：
  
  * 约定：当map[i][j]为 0表示没有走过，1表示墙，2表示通路可以走，3表示该点已走过但走不通。

  * 策略：下-》右-》上=》左 ， `setWay(int[][] map, int i, int j) //map是引用类型，在堆中共享map 。按策略走都走不通，置为3`
  
  * ```
    		
    		// 使用递归回溯来给小球线路
      		/**
      		 * i，j表示从哪个位置开始出发（1，1），map[6][5]为通路终点
      		 * 约定：当map[i][j]为 0表示没有走过，1表示墙，2表示通路可以走，3表示该点已走过但走不通
      		 * 策略：下-》右-》上=》左 
      		 */
      		public static boolean setWay(int[][] map, int i, int j){
      			if(map[6][5] == 2){
      				return true;
      			}else{
      				if(map[i][j] == 0){
      					map[i][j] = 2; // 假定该点可以走通
      					if(setWay(map, i+1, j)){ // 下
      						return true;
      					}else if(setWay(map, i, j+1)){ // 右
      						return true;
      					}else if(setWay(map, i-1, j)){ // 上
      						return true; 
      					}else if(setWay(map, i, j-1)){ // 左
      						return true;
      					}else{ 
      						map[i][j] = 3;
      						return false;
      					}
      				}else { 
      					return false;
      				}
      			}
      		}
    ```
  
    
  
* 2.递归-八皇后问题（回溯算法）
  
  * 概述：在8X8格的国际象棋上拜访8个皇后棋子，任意两个皇后都不能处于同一行、同一列或同一斜线上，有多少种摆法。
  
* 思路分析：把第一个皇后放在第一行第一列，之后把第二个皇后放在第二行第一列，判断是否OK，不OK则放第三列、第四列。。。直到OK。继续之后的其他皇后。当得到第一个正确解时，栈回退到上一个栈，开始回溯，将第一个皇后在第一行第一列全部解得到，之后计算其他列的正确解法
  
  * 用一个一维数组表示皇后的位置，下标表示第几个皇后 和第几行，值表示第几列
  
  * ```
    	private void check(int n){
      		if(n == max){  // n == 8, 表示8个皇后已经放好
      			print();
      			return;
      		}
      		for (int i = 0; i < max; i++) {
      			array[n] = i; // 从第一列开始尝试
      			if(judge(n)){  // 该皇后在 i 列不冲突
      				check(n + 1); // 开始递归，判断下个皇后不冲突的情况
      			}
      			// 如果冲突则继续判断下一列
      		}
      	}
      	private boolean judge(int n){
      		judgeCount++;
      		for (int i = 0; i < n; i++) {
      			// 分别表示是否是同一列和是否在同意斜线（行差绝对值==列差绝对值）
      			if(array[i] == array[n] || Math.abs(n - i) == Math.abs(array[n] - array[i])){
      				return false;
      			}
      		}
      		return true;
      	}
    ```
  
    

---

## 排序算法（八大算法）

* 时间复杂度：

  * T(n) = O(f(n)) ，规定常数项和最低次项忽略，高次项系数忽略，例如： T(n) = 3n²+2n+2 = O(n²)

  ![image-20200701122706621](../../../../Software/Typora/Picture/image-20200701122706621.png)

  ![image-20200706002731606](../../../../Software/Typora/Picture/image-20200706002731606.png)

* 冒泡：

  * 两两比较，第一趟全部比较过后，数组中下标最大的元素确定为最大值，下一趟比较不参与比较，依次反复，需进行n-1趟(n为数组长度)

  * 优化：定义一个flag标志位，当一趟排序中有其中一对前大后小时flag=true，否则为false，若有一趟排序判断flag为false，则表示排序完成，退出循环

  * ```
    public static void BubbleSort(int[] arr) {
    		boolean flag = false; // 表示是否进行过交换
    		for (int j = 0; j < arr.length - 1; j++) { // 第j趟
    			for (int i = 0; i < arr.length - 1; i++) { // 索引为i的数
    				if (arr[i] > arr[i + 1]) {
    					flag = true;
    					arr[i] ^= arr[i+1];
    					arr[i+1] ^= arr[i];
    					arr[i] ^= arr[i+1]S;
    				}
    			}
    			if (!flag) { // 一次都没有发生交换
    				break;
    			} else { 
    				flag = false;
    			}
    		}
    	}
    ```

    

* 选择排序：

  * 每次遍历所有元素，从中找出最小的元素放在arr[0], 接着从arr[1:]找出最小的值放在arr[1]，依次反复执行，需进行n-1次轮比较

  * 在比较时，先假设第一个数为最小值，当与其他元素比较时，若小于当前数，则重新选择最小元素

  * 做法：minIndex和min分别用来保存一趟排序中的最小值的索引和最小值，`minIndex != i` 用来判断这一轮排序不需要进行交换和排序

  * ```
    public static void SelectSort(int[] arr){
    		int minIndex, min;
    		for(int i = 0; i < arr.length;i++){  // 寻找第i个最小元素
    			minIndex = i;
    			min = arr[i];
    			for(int j = i + 1; j < arr.length; j++){
    				if(min > arr[j]){
    					min = arr[j];
    					minIndex = j;
    				}
    			}
    			if(minIndex != i){  // 最小值索引不是在i，需要将最小值交换位置
    				arr[minIndex] = arr[i];  // 交换最小元素的位置
    				arr[i] = min;
    			}
    		}
    ```
  
* 插入排序：

  * 思想：把n个待排序的元素看成有序和无序表，开始时有序表有1个元素，无序表有n-1个元素，每次从无序表取出元素与有序表元素比较后插入相应位置

  * 思路：从数组的第2个元素遍历，每个元素与前一个元素比较，如果小于则需要将前一个元素后移，再与前一个元素比较，直到大于前一个元素才插入

  * 代码：定义insertVal和insertIndex分别表示要插入的元素，要插入的位置，初始为要插入元素的前一个位置索引

  * ```
    public static void InsertSort(int[] arr){
    		int insertVal, insertIndex;
    		for (int i = 1; i < arr.length; i++) {
    			insertVal = arr[i]; // 要插入的数
    			insertIndex = i - 1; // 有序表
    			//当插入元素小于前一个元素时需要更换位置，此时大于inserVal的元素后移知道出现小于insertVal的元素
    			while(insertIndex >= 0 && insertVal < arr[insertIndex]){
    				arr[insertIndex + 1] = arr[insertIndex];  // 后移，为了空出位置插入
    				insertIndex--; // 指针左移
    			}
    			if(insertIndex + 1 != i){ // 表示当要插入的元素比前一个元素大时，自动加入有序表
    				arr[insertIndex + 1] = insertVal;
    			}
    		}
    	}
    ```

* 希尔排序：

  * 思想：对于arr[n], 将其分割为g=n/2组，然后每一组之间的两个元素相距n/2，每组的两个元素比较后进行交换或移动排序，再不断进行分割为g=g/2组 

  * <img src="../../../../Software/Typora/Picture/image-20200702223907290.png" alt="image-20200702223907290"  />
  
  * ```
    // 使用移动法进行元素的交换,
    public static void ShellSort(int[] arr){
    		for(int gap = arr.length / 2; gap > 0; gap /= 2){  // 每一轮分组，gap为组内元素间的距离，最小为1
    			for(int i = gap; i < arr.length; i++){ // i从第一个分组的终点开始遍历
    				int j = i; // i为要插入的数的索引
    				int temp = arr[j]; // 存储可能要交换的数
    				if(arr[j] < arr[j - gap]){  // 当前组的后面的数小于当前组前一个数，需要移动交换
    					while(j - gap >= 0 && temp < arr[j - gap]){
    						arr[j] = arr[j - gap]; // 移动， 采用了插入排序
    						j -= gap; // 下一组，gap为组数
    					}
    					arr[j] = temp;
    				}
    			}
    		}
    	}
    ```
  
* 快速排序（冒泡排序的改进）

  * 思想：通过一趟排序将要排序的数据分割为独立的两部分，其中一部分的数据都比另一部分的所有数据都要小，再按此方法对这两部分数据分别进行快速排序，整个排序过程是递归进行。
  * <img src="../../../../Software/Typora/Picture/image-20200703134913906.png" alt="image-20200703134913906"  />

```
public static void quickSort(int[] arr, int left, int right){
		int l = left;
		int r = right;
		int pivot = arr[(left  + right) / 2];
		while(l < r){ // 比pivot小的放左边，大 的放右边
			while(arr[l] < pivot){ // 在pivot左边一直找，直到找到比pivot还大的 
				l++; // 从最左边开始找
			} 
			while(arr[r] > pivot){ // 在pivot右边一直找，直到找到比pivot还小的
				r--;
			}
			if(l >= r){ // pivot左边的值已经全部小于等于pivot，右边全部大于等于pivot
				break;  
			}
			arr[l] ^= arr[r];
			arr[r] ^= arr[l];
			arr[l] ^= arr[r];
			// 下面两个if是为了避免当前的l和r两个位置的元素刚好等于pivot，那将会导致无限交换循环，如   4(l) 3 4 6 4(r)    
			if(arr[l] == pivot){
				r--;
			}
			if(arr[r] == pivot){
				l++;
			}
		}
		if(l == r){  // 左右指针相遇，即l和r在privot位置上
			l++;
			r--;
		}
		if(left < r){  // r为每一轮最右边的一个小于privot的索引，直到r==0
			quickSort(arr, left, r);
		}
		if(right > l){
			quickSort(arr, l, right);
		}
	}
```

* 归并排序（时间复杂度O(n) = n-1）(速度和快排差不多)

  * 思想：采用分治策略，将问题分成一些小问题然后递归求解，然后将分的阶段得到的答案修补在一起，得出答案，即分而治之
  
* ![image-20200704151344510](../../../../Software/Typora/Picture/image-20200704151344510.png)
  
  * ![image-20200704165449266](../../../../Software/Typora/Picture/image-20200704165449266.png)
  
  * ```java
    	// 分 + 合
      	public static void mergeSort(int[] arr, int left, int right, int[] temp){
      		if(left < right){
      			int mid = (left + right) / 2;   
      			// 向左递归
      			mergeSort(arr, left, mid, temp);
      			// 向右递归
      			mergeSort(arr, mid + 1, right, temp);
      			// 合并 
      			merge(arr, left, mid, right, temp);
      		}
      	}
      	// 合并, 需要递归到的次数为： arr.length - 1
      	private static void merge(int[] arr, int left, int mid, int right, int[] temp) {
      		int i = left;   // 左边的起始索引
      		int j = mid + 1; // 右边的起始索引
      		int t = 0; // 指向temp数组的当前索引
      		
      		// 1.先把左右两边有序的数据按照规则填充到temp数组
      		// 直到左右两边的有序序列，有一边处理完毕为止
      		while(i <= mid && j <= right){ // 继续
      			if(arr[i] <= arr[j]){
      				temp[t] = arr[i];
      				t++;
      				i++;
      			}else{
      				temp[t] = arr[j];
      				t++;
      				j++;
      			}
      		}
      		// 2. 把有剩余数据的一边的数据依次全部填充到temp
      		while(i <= mid){ // 左边的有序序列还有剩余元素未填充到temp
      			temp[t] = arr[i];
      			t++;
      			i++;
      		}
      		while(j <= right){ // 左边的有序序列还有剩余元素未填充到temp
      			temp[t] = arr[j];
      			t++;
      			j++;
      		}
      		// 3. 将temp数组的元素拷贝到arr
      		t = 0;
      		int tempLeft = left; 
      		while(tempLeft <= right){ // 第一次合并时，tempLeft=0, right=1
      			arr[tempLeft] = temp[t];
      			t++;
      			tempLeft++;
      		}
      	}
    ```
  
    

* 基数排序（桶排序）：（速度最快，空间换时间，稳定的排序方法）

  * 属于“分配式排序”，又称桶子法，通过键值的各个位的值，将要排序的元素分配至某些桶(数组)中，达到排序的作用

  * 基数排序的桶排序的扩展，属于效率高的稳定性排序法。（稳定性：如33211，排序后为11233，且第一个“1”仍在第二个“1”前面）

  * 思想：将所有待比较数值统一为同样的数位长度，数位较短的数前面补零，然后从最低位开始依次进行排序后，就变成一个有序序列
  * 过程：
    * 需定义10个桶代表0~9位数，每个桶长度由数组中长度最长的元素决定，即需要定义一个二维数组 `int[10][arr.length]`
    * 第一轮排序：根据每个元素的个位数，分别放在对应位数的桶(0~9的一维数组)中，最后按照顺序从0~9回收元素到原数组
    * 接下来一次进行根据十位、百位....的排序，如果没有该位数，则补0。

  * ![image-20200706002826574](../../../../Software/Typora/Picture/image-20200706002826574.png)
  
  * ```java
  		public static void redixSort(int arr[]) {
      		int max = arr[0];  // 得到数组中最大的数的位数
  			for(int i = 0; i < arr.length; i++) {
      			if(arr[i] > max) {
      				max = arr[i];   // 每个桶的长度由最大元素的位长度决定
      			}
      		}
      		int maxLength = (max + "").length();
      		int[][] bucket = new int[10][maxLength]; // 创建10个桶
      		int[] bucketElementCounts = new int[10]; // 额外定义一个一维数组来记录每个桶中存放了多少数据，用来计数的桶
      		// 遍历次数取决于最长元素的长度
      		for(int i = 0, n = 1; i < maxLength; i++, n *= 10) {
      			for(int j = 0; j < arr.length; j++) {   // 遍历数组每个元素并根据位数排序
      				int digitOfElement = arr[j] / n % 10; // 取出每个元素的位数,根据位数放在0-9号桶里
      				bucket[digitOfElement][bucketElementCounts[digitOfElement]] = arr[j];
      				bucketElementCounts[digitOfElement]++;
      			}
      			int index = 0; // 存放回数组时用到的索引
      			// 按照桶的元素顺序将元素重新放回数组中
      			// 遍历每个桶，总共有10个桶
      			for(int k = 0; k < bucketElementCounts.length; k++) {
      				if(bucketElementCounts[k] != 0) { // 如果当前遍历的桶为0表示当前位的桶没有元素了，则退出当前位的桶
      					// bucketElementCounts[k] 表示10个桶中分别存放了多少元素
      					for(int l = 0; l < bucketElementCounts[k]; l++) {
      						arr[index++] = bucket[k][l];// 将桶中已经排好序的元素放回数组中
      					}
      				}
      				bucketElementCounts[k] = 0;  // 清空用来计数的桶
      			}
      		}
      	}
    ```
    
    



















 


