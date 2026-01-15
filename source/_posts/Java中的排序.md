title: Java中的排序
typora-root-url: Java中的排序
date: 2026-01-13 00:26:26
tags: [Java]
categories: [技术,Java]
description: Java中对数组的排序

-----------------------------------------------------------------------------------------------------------------------------------------

## 1.排序的对象

在Java中，排序的对象主要有两类：数组和集合。

- 数组
  - 基本类型数组
  - 对象数组
- 集合
  - List
  - Set
  - Map



## 2.数组排序

对数组排序，使用`Arrays.sort()`方法，默认升序，降序排序可考虑将数组反转。

### 2.1 基本类型数组

```
 public static void main(String[] args) {
        int[] array = new int[]{154,562,105,4,46,7};
        System.out.println("排序前:" + Arrays.toString(array));
        Arrays.sort(array);
        System.out.println("排序后:" + Arrays.toString(array));
    }
```



### 2.2对象数组

**排序规则**

compare(o1, o2): 

- 返回负数: o1 < o2 (o1 排在前面) -
- 返回 0:   o1 == o2 
- 返回正数: o1 > o2 (o2 排在前面)

在这里，以User对象来进行排序

```
public class User{
    String name;
    int age;

    public User(String name,int age){
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return name + ":" + age;
    }
}

```



#### 2.2.1 实现Comparable接口

需要排序的对象，需要实现Comparable接口

```
public class User implements Comparable<User> {
    String name;
    int age;

    public User(String name,int age){
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(User other) {
        return this.age - other.age;
    }

    @Override
    public String toString() {
        return name + ":" + age;
    }
}

```

```
public static void main(String[] args) {

        // 创建测试数据
        User[] users = {
            new User("张三", 25),
            new User("李四", 20),
            new User("王五", 30),
            new User("赵六", 22),
            new User("孙七", 28),
            new User("周八", 18),
            new User("吴九", 35),
            new User("郑十", 26)
        };

        System.out.println("排序前:" + Arrays.toString(users));

        // 使用 Comparable 排序（按年龄升序）
        Arrays.sort(users);

        System.out.println("\n排序后（按年龄升序）:" + Arrays.toString(users));
        
    }
```



#### 2.2.2 Comparator接口

与实现Comparable接口不同，Comparator排序，是自定义了排序规则

```
 public static void main(String[] args) {

        // 创建测试数据
        User[] users = {
            new User("张三", 25),
            new User("李四", 20),
            new User("王五", 30),
            new User("赵六", 22),
            new User("孙七", 28),
            new User("周八", 18),
            new User("吴九", 35),
            new User("郑十", 26)
        };

        System.out.println("排序前:" + Arrays.toString(users));

        // 使用 Comparable 排序（按年龄升序）
        Arrays.sort(users, new Comparator<User>() {
            @Override
            public int compare(User o1, User o2) {
                return o1.age - o2.age;
            }
        });

        System.out.println("\n排序后（按年龄升序）:" + Arrays.toString(users));

    }
```

可以简化一些，使用Lambda 表达式

```
 // 使用 Comparable 排序（按年龄升序）
        Arrays.sort(users, (p,q) -> p.age - q.age);
```





#### 2.2.3 pair排序

常见于数组排序，举个例子,使用大小为2的数组表示一个区间，对区间数组排序

```
 int[][] intervals = {
            {1, 5},
            {3, 7},
            {1, 3},   // 起点和第一个区间相同
            {3, 4},   // 起点和第二个区间相同
            {2, 6},
            {1, 8}    // 起点和第一、三个区间相同
        };
```

排序方法和对象类似，以起点为准排序

```
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
```

还有一种可能就是，起点相同的情况下，看终点的位置，也就是多字段排序

- 手动排序

```
 Arrays.sort(intervals, (a, b) -> {
            if (a[0] != b[0]) {
                return a[0] - b[0];  // 起点不同，按起点排序
            }
            return a[1] - b[1];      // 起点相同，按终点排序
        });
```

- 链式调用

```
Arrays.sort(intervals, Comparator
        .comparingInt((int[] a) -> a[0])  // 先按起点
        .thenComparingInt(a -> a[1]));    // 再按终点
```



## 3.集合排序

集合排序和数组排序类似，只不过集合排序使用的是`Collections.sort`

### 3.1 List

### 3.1.1 实现Comparable接口

```
public static void main(String[] args) {
        List<User> users = new ArrayList<>();
        users.add(new User("张三", 25));
        users.add(new User("李四", 30));
        users.add(new User("王五", 25));
        users.add(new User("赵六", 25));
        users.add(new User("孙七", 30));
        System.out.println(users);
        Collections.sort(users);
        System.out.println(users);

    }
```


#### 3.1.2 实现Comparator

```
  Collections.sort(users, new Comparator<User>() {
            @Override
            public int compare(User o1, User o2) {
                return o1.age - o2.age;
            }
        });
```

Lambda 表达式简化

```
Collections.sort(users, (u1,u2) -> u1.age - u2.age);
```



#### 3.1.3 多字段排序

先按年龄排序，如果年龄相同，按姓名排序

```
Collections.sort(users, (u1,u2) -> {
            if(u1.age != u2.age){
                return u1.age - u2.age;
            }else{
                return u1.name.compareTo(u2.name);
            }
        });
```

#### 3.1.4 通过List的sort方法

以上通过Collections.sort排序的集合，都可以通过List.sort来实现

```
users.sort((u1,u2) -> {
            if(u1.age != u2.age){
                return u1.age - u2.age;
            }else{
                return u1.name.compareTo(u2.name);
            }
        });
```



### 3.2 Map按key排序

```
// 按 Key 排序 - 用 TreeMap
Map<K, V> treeMap = new TreeMap<>(hashMap);
```



### 3.3 Map按value排序

```
// 按 Value 排序 - 用 Stream
Map<K, V> sorted = map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (e1, e2) -> e1,
        LinkedHashMap::new
    ));
```



### 3.4 优先队列

```
// 最小堆
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// 最大堆
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder())
```

