[TOC]

# Stream流

`Stream` 流分为顺序流和并行流，所谓顺序流就是按照顺序对集合中的元素进行处理，而并行流则是使用多线程同时对集合中多个元素进行处理，所以在使用并行流的时候就要注意线程安全的问题了。

## 创建流

- 调用集合的 `stream()` 方法或者 `parallelStream()` 方法创建流。
- Stream 类的静态 `of()` 方法创建流。

```
List< String> createStream = new ArrayList< String>();
// 顺序流
Stream< String> stream = createStream.stream();
// 并行流
Stream< String> parallelStream = createStream.parallelStream();
// of()方法创建
Stream< String> stringStream = Stream.of(
createStream.toArray(new String[createStream.size()]));
```

## 使用流

```java
public static void streamImpl(List< Student> students) {
  List< Student> filterStudent = students.stream()
       .filter(one -> one.getScore() <  60).collect(Collectors.toList()); // 使用流筛选后转换为集合
  System.out.println(filterStudent);
}
```

## 终端操作和中间操作

终端操作会消费 Stream 流，并且会产生一个结果，比如 `iterator()` 和 `spliterator()`。如果一个 Stream 流被消费过了，那它就不能被重用的。

中间操作会产生另一个流。需要注意的是中间操作不是立即发生的。而是当在中间操作创建的新流上执行完终端操作后，中间操作指定的操作才会发生。流的中间操作还分无状态操作和有状态操作两种。

- 在无状态操作中，在处理流中的元素时，会对当前的元素进行单独处理。比如，过滤操作，因为每个元素都是被单独进行处理的，所有它和流中的其它元素无关。
- 在有状态操作中，某个元素的处理可能依赖于其他元素。比如查找最小值，最大值，和排序，因为他们都依赖于其他的元素。

## 流接口

### BaseStream 

BaseStream是Stream接口的父接口，是 Stream 流最基础的接口，它提供了所有流都可以使用的基本功能。 `BaseStream` 是一个泛型接口,它有两个类型参数 `T` 和 `S` ， 其中 `T` 指定了流中的元素的类型， `S` 指定了具体流的类型，由 `<S extends BaseStream<T,S>>` 可以知道 `S` 必须为 `BaseStream` 或 `BaseStream` 子类，换句话说,就是 `S` 必须是扩展自 `BaseStream` 的。 `BaseStream` 继承了 `AutoCloseable` 接口，简化了关闭资源的操作，但是像平时我们操作的集合或数组，基本上都不会出现关闭流的情况。下面是 `BaseStream` 接口下定义的方法的相关解释：

- `Iterator<T> iterator()` ：获取流的迭代器。
- `Spliterator spliterator()` ：获取流的 `spliterator` 。
- `boolean isParallel()` ：判断一个流是否是并行流，如果是则返回 `true` ，否则返回 `false` 。
- `S sequential()` ：基于调用流返回一个顺序流，如果调用流本身就是顺序流，则返回其本身。
- `S parallel()` ：基于调用流，返回一个并行流，如果调用流本身就是并行流，则返回其本身。
- `S unordered()` ：基于调用流，返回一个无序流。
- `S onClose(Runnable closeHandler)` ：返回一个新流， `closeHandler` 指定了该流的关闭处理程序，当关闭该流时，将调用这个处理程序。
- `void close()` ：从 `AutoCloseable` 继承来的，调用注册关闭处理程序，关闭调用流(很少会被使用到)。

### stream().forEach、stream().map、stream().filter、stream().sorted的区别和用法

```
List<Teacher> list = new ArrayList<>();
        list.add(Teacher.builder().age(28).name("李四").build());
        list.add(Teacher.builder().age(27).name("张三").build());
        list.add(Teacher.builder().age(29).name("王五").build());
 
        //适合只对原数据读操作
        list.stream().forEach(teacher -> {
            if ("王五".equals(teacher.getName())) {
                Address address = Address.builder().address("南京").build();
                teacher.setAddress(address);
            }
        });
        System.out.println("还是老的List：" + list);
 
        //不影响原数据，生成新数据
        List<Teacher> listCopy = list.stream().map(teacher -> {
            if ("王五".equals(teacher.getName())) {
                Address address = Address.builder().address("南京").build();
                teacher.setAddress(address);
            }
            return teacher;
        }).collect(Collectors.toList());
        System.out.println("产生新的List：" + listCopy);
 
        //过滤
        List<Teacher> listFilter = list.stream().filter(
                teacher -> AllTypeUtils.isNotEmptyAndNotNull(teacher.getAddress()
                )).collect(Collectors.toList());
        System.out.println("产生新的List：" + listFilter);
 
        //排序：升序（第一种写法）
        List<Teacher> listSortedAsc1 = list.stream().sorted(Comparator.comparing(Teacher::getAge)
        ).collect(Collectors.toList());
        System.out.println("产生新的List：" + listSortedAsc1);
 
        //排序：升序（第二种写法）
        List<Teacher> listSortedAsc2 = list.stream().sorted((teacher1, teacher2) -> {
                    return teacher1.getAge().compareTo(teacher2.getAge());
                    //这种升序排列被编译器建议写成：Comparator.comparing(Teacher::getAge)
                }
        ).collect(Collectors.toList());
        System.out.println("产生新的List：" + listSortedAsc2);
 
        //排序：降序（第一种写法）
        List<Teacher> listSortedDesc = list.stream().sorted(Comparator.comparing(Teacher::getAge).reversed()
        ).collect(Collectors.toList());
        System.out.println("产生新的List：" + listSortedDesc);
 
        //排序：降序（第二种写法）
        List<Teacher> listSortedDesc2 = list.stream().sorted((teacher1, teacher2) -> {
                    return teacher2.getAge().compareTo(teacher1.getAge());
                }
        ).collect(Collectors.toList());
        System.out.println("产生新的List：" + listSortedDesc2);
```

### Stream 接口

- `Stream filter(Predicate predicate)` ：产生一个新流，其中包含调用流中满足 `predicate` 指定的谓词元素，即筛选符合条件的元素后重新生成一个新的流。(中间操作)

  ```java
  List<User> newlist = list.stream().filter(user -> 
                                            user.getAge() > 20
                                           ).collect(Collectors.toList());
  ```

  

- `Stream map(Function mapper)` ，产生一个新流，对调用流中的元素应用 `mapper` ，新 `Stream` 流中包含这些元素。(中间操作)

- `IntStream mapToInt(ToIntFunction mapper)` ：对调用流中元素应用 `mapper` ，产生包含这些元素的一个新 `IntStream` 流。(中间操作)

- `Stream sorted(Comparator comparator)` ：产生一个自然顺序排序或者指定排序条件的新流。(中间操作)

  ```java
  List<Transaction> tr2011 = transactions.stream()
                  .filter(transaction -> transaction.getYear() == 2011)
                  .sorted(comparing(Transaction::getValue))
                  .collect(toList());
          System.out.println(tr2011);
  ```

  

- `void forEach(Consumer action)` ：遍历了流中的元素。(终端操作)

  ```
  transactions.stream()
                  .map(Transaction::getTrader) // Java 8 中可以通过 `::` 关键字来访问类的构造方法，对象方法，静态方法。
                  .filter(trader -> trader.getCity().equals("Milan"))
                  .forEach(trader -> trader.setCity("Cambridge"));
          System.out.println(transactions);
  ```

  

- `Optional min(Comparator comparator)` 和 `Optional max(Comparator comparator)` ：获得流中最大最小值，比较器可以由自己定义。(终端操作)

- `boolean anyMatch(Predicate<? super T> predicate)` ：判断 `Stream` 流中是否有任何符合要求的元素，如果有则返回 `ture`,没有返回 `false` 。（终端操作）

  ```
  boolean milanBased =
                  transactions.stream()
                          .anyMatch(transaction -> transaction.getTrader()
                                  .getCity()
                                  .equals("Milan")
                          );
          System.out.println(milanBased);
  ```

  

- `Stream<T> distinct()` ，去重操作，将 `Stream` 流中的元素去重后，返回一个新的流。（中间操作）

  ```
  List<String> cities =
                  transactions.stream()
                          .map(transaction -> transaction.getTrader().getCity())
                          .distinct()
                          .collect(toList());
          System.out.println(cities);
  ```

  

## stream流的API操作

### 缩减操作reduce()

最终将流缩减为一个值的终端操作，我们称之为缩减操作。在上一节中提到的 `min()，max()` 方法返回的是流中的最小或者最大值，这两个方法属于特例缩减操作。而通用的缩减操作就是指的我们的 `reduce()` 方法了，在 Stream 类中 `reduce` 方法有三种签名方法

reduce 方法的三种实现

```java
Optional<T> reduce(BinaryOperator<T> accumulator);
    
T reduce(T identity, BinaryOperator<T> accumulator);
    
<U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
```

#### reduce的应用

* Optional<T> reduce(BinaryOperator<T> accumulator);

  ```
  public static void reduceFirstSign() {
    List<Integer> list = Arrays.asList(1,2,3,4,5,6);
    ptional<Integer> count = list.stream().reduce((a, b) -> (a + b));  // 所有元素的和
    System.out.println(count.get()); // 21
  }
  ```

  

* T reduce(T identity, BinaryOperator<T> accumulator);

  ```
  public static void reduceSecondSign() {
    List<Integer> list = Arrays.asList(1,2,3,4,5,6);
    Integer count = list.stream().reduce(2, (a, b) -> (a * b));  // 所有元素乘积的两倍
    System.out.println(count);  // 1440
  }
  ```



* 前面两种前面的一个缺点在于返回的数据都只能和 Stream 流中元素类型一致，但这在某些情况下是无法满足我们的需求的，比如 Stream 流中元素都是 `Integer` 类型，但是求和之后数值超过了 `Integer` 能够表示的范围，需要使用 `Long` 类型接收，这就用到了我们第三种签名的 `reduce()` 方法。

  <U> U reduce(U identity,
                   BiFunction<U, ? super T, U> accumulator,
                   BinaryOperator<U> combiner);    // 第三个参数表示接收的类型

```java
    public static void reduceThirdSign() {
      List<Integer> list = Arrays.asList(Integer.MAX_VALUE, Integer.MAX_VALUE);
      long count = list.stream().reduce(0L, (a, b) -> (a + b), (a,b) -> 0L); 
      System.out.println(count);
    }
```

##### 总的来说缩减操作有两个特点，一是他只返回一个值，二是它是一个终端操作

## 映射

可能在我们的日常开发过程中经常会遇到将一个集合转换成另外一个对象的集合，那么这种操作放到 Stream 流中就是映射操作。映射操作主要就是将一个 Stream 流转换成另外一个对象的 Stream 流或者将一个 Stream 流中符合条件的元素放到一个新的 Stream 流里面。

### map()

```
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

`map()` 方法可以将一个流转换成另外一种对象的流，其中的 `T` 是原始流中元素的类型，而 `R` 则是转换之后的流中元素的类型

示例

```java
public static void useMap() {
  List<Student> students = initData();
  double scoreCount = students.stream()
            .map(Student::getScore)
            .reduce(0.0, (a,b) -> (a + b));  // stream对象的元素转换为Double类型并求和后输出
  System.out.println(scoreCount);
}
```

####  mapToDouble()

mapToDouble和map功能相似，前者相对方便些

```
double scoreCount = students.stream()
                .mapToDouble(Student::getScore)
                .sum();
```

#### flatMap()

`flatMap()` 操作能把原始流中的元素进行一对多的转换，并且将新生成的元素全都合并到它返回的流里面。

代码中 `flatMap()` 中返回的是一个一个的 `String` 类型的 Stream 流，它们会被合并到最终返回的 Stream 流（String 类型）中。而后面的 `distinct()` 则是一个去重的操作， `collect()` 是收集操作。

```
public static void useFlatMap() {
  List<Student> students = initData();
  List<String> course = students.stream().flatMap(one -> one.getCourse().stream()).distinct()
                .collect(Collectors.toList());
  System.out.println(course);
}
```

## 收集操作

从流中收集一些元素，并以集合的方式返回，我们把这种反向操作称为收集操作。

```
<R, A> R collect(Collector<? super T, A, R> collector);
```

其中 `R` 指定结果的类型， `T` 指定了调用流的元素类型。内部积累的类型由 `A` 指定。 `collector` 是一个收集器，指定收集过程如何执行， `collect()` 方法是一个终端方法。一般情况我们只需要借助 `Collectors` 中的方法就可以完成收集操作。

`Collectors` 类是一个最终类，里面提供了大量的静态的收集器方法，借助他，我们基本可以实现各种复杂的功能了。

#### Collectors

`Collectors` 给我们提供了非常丰富的收集器，这里只列出来了 `toList` 和 `toMap` 两种，其他的可以参考 `Collectors` 类的源码。 `toList()` 相信您在清单 14 中已经见到了，那么下面将展示如何将一个使用收集操作将一个 `List` 集合转为 `Map` 。

```
public final class Collectors {
...
public static <T> Collector<T, ?, List<T>> toList() {
...
}
public static <T, K, U> Collector<T, ?, Map<K,U>> toMap(
Function<? super T, ? extends K> keyMapper,
Function<? super T, ? extends U> valueMapper) {
  ...
}
...
}
```

### **收集操作将 List 转 Map**

```
public static void list2Map() {
  List<Student> students = initData();
  Map<String, Double> collect = students.stream()
         .collect(Collectors.toMap(one -> one.getName(),
                                   one -> one.getScore()));
  System.out.println(collect);
}
```

可以看到通过 Stream API 可以很方便地将一个 `List` 转成了 `Map` ，但是这里有一个地方需要注意。那就是在通过 Stream API 将 `List` 转成 `Map` 的时候我们需要确保 `key` 不会重复，否则转换的过程将会直接抛出异常。

## 并行流

Stream API 提供了相应的并行流来支持我们并行地操作数组和集合框架，从而高速地执行我们对数组或者集合的一些操作。

其实创建一个并行流非常简单，在[创建流](https://developer.ibm.com/zh/articles/j-experience-stream/#创建流) 部分已经提到过如何创建一个并行流，我们只需要调用集合的 `parallelStream()` 方法就可以轻松的得到一个并行流。相信大家也知道多线程编程非常容易出错，所以使用并行流也有一些限制，一般来说，应用到并行流的任何操作都必须符合三个约束条件：无状态、不干预、关联性。因为这三大约束确保在并行流上执行操作的结果和在顺序流上执行的结果是相同的。

在[缩减操作](https://developer.ibm.com/zh/articles/j-experience-stream/#缩减操作)部分我们一共提到了三种签名的 `reduce()` 方法，其中第三种签名的 `reduce()` 方法最适合与并行流结合使用。

**清单 16. 第三种签名方式的 `reduce()` 方法与并行流结合使用**

```
public interface Stream<T> extends BaseStream<T, Stream<T>> {
 ...
<U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
 ...
}
```

其中 `accumulator` 被为累加器， `combiner` 为合成器。 `combiner` 定义的函数将 `accumulator` 提到的两个值合并起来，在之前的例子中我们没有为合并器设置具体的表达式，因为在那个场景下我们不会使用到合并器。下面我们来看一个例子，并且分析其执行的步骤：

**清单 17. 并行流使用场景**

```java
public static void main(String[] args) {
  List<Integer> list = Arrays.asList(2,2);
  Integer result = list.stream().parallel().reduce(2, (a, b) -> (a + b), (a, b) -> (a + b));
  System.out.println(result);
}
```

先使用累加器把 Stream 流中的两个元素都加 `2` 后，然后再使用合并器将两部分的结果相加。最终得到的结果也就是 `8` 。并行流的使用场景也不光是在这中缩减操作上，比如我会经常使用并行流处理一些复杂的对象集合转换，或者是一些必须循环调用的网络请求等等，当然在使用的过程中最需要注意的还是线程安全问题。























