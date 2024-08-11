---
title: ava8的Stream流太难用了？看看JDFrame
date: 2024-08-11 12:06:50
tags: JDFrame
categories: 教程
---



由于经常记不住stream的一些API每次要复制来复制去并且又长又臭，想要更加语意化的api，于是想到了以前写大数据Spark pandnas 等DataFrame模型时的API， 然后发现其实也存在java的JVM层的DataFrame模型比如 tablesaw，joinery

但是他们得硬编码去指定字段名，这对于有代码洁癖的人实在难以忍受，而且我只是简单统计下数据，我想在一些场景下能不能使用匿名函数去指定的字段处理去处理，于是便有了这个

一个jvm层级的仿DataFrame工具，语意化和简化java8的stream流式处理工具

1、快速开始
======

1.1、引入依赖
--------

```
<dependency>  
    <groupId>io.github.burukeyou</groupId>  
    <artifactId>jdframe</artifactId>  
    <version>0.0.6</version>  
</dependency>  

```

1.2、案例
------

统计每个学校的里学生年龄不为空并且年龄在9到16岁间的合计分数，并且获取合计分前2名的学校

```
static List<Student> studentList = new ArrayList<>();  
  
static {  
    studentList.add(new Student(1,"a","一中","一年级",11, new BigDecimal(1)));  
    studentList.add(new Student(2,"a","一中","一年级",11, new BigDecimal(1)));  
    studentList.add(new Student(3,"b","一中","三年级",12, new BigDecimal(2)));  
    studentList.add(new Student(4,"c","二中","一年级",13, new BigDecimal(3)));  
    studentList.add(new Student(5,"d","二中","一年级",14, new BigDecimal(4)));  
    studentList.add(new Student(6,"e","三中","二年级",14, new BigDecimal(5)));  
    studentList.add(new Student(7,"e","三中","二年级",15, new BigDecimal(5)));  
}  
  
// 等价于SQL:  
//       select school,sum(score)    
//       from students  
//       where age is not null and age >=9 and age <= 16  
//       group by school  
//       order by sum(score) desc  
//       limit 2  
SDFrame<FI2<String, BigDecimal>> sdf2 = SDFrame.read(studentList)  
    .whereNotNull(Student::getAge)  
    .whereBetween(Student::getAge,9,16)  
    .groupBySum(Student::getSchool, Student::getScore)  
    .sortDesc(FI2::getC2)  
    .cutFirst(2);  
  
sdf2.show();  

```

输出信息;

```
c1  c2   
三中 10   
二中 7  

```
```
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
public class Student {  
  
  private int id;  
  private String name;  
  private String school;  
  private String level;  
  private Integer age;  
  private BigDecimal score;  
  
  private Integer rank;  
  
  public Student(String level, BigDecimal score) {  
      this.level = level;  
      this.score = score;  
  }  
  
  public Student(int id, String name, String school, String level, Integer age, BigDecimal score) {  
      this.id = id;  
      this.name = name;  
      this.school = school;  
      this.level = level;  
      this.age = age;  
      this.score = score;  
  }  
}  

```

2、API案例
=======

2.1、矩阵查看相关
----------

```
void show(int n); // 打印矩阵信息到控制台  
List<String> columns();   // 获取矩阵的表头字段名  
List<R> col(Function<T, R> function);   // 获取矩阵某一列值  
T head();                   // 获取第一个元素  
List<T> head(int n);          // 获取前n个元素  
T tail();                       // 获取最后一个元素  
List<T> tail(int n);            // 获取后n个元素  
List<T> page(int page,int pageSize) // 获取分页数据  

```

2.2、筛选相关
--------

```
SDFrame.read(studentList)  
.whereBetween(Student::getAge,3,6) // 过滤年龄在[3，6]岁的  
.whereBetweenR(Student::getAge,3,6) // 过滤年龄在(3，6]岁的, 不含3岁  
.whereBetweenL(Student::getAge,3,6)      // 过滤年龄在[3，6)岁的, 不含6岁  
.whereNotNull(Student::getName) // 过滤名字不为空的数据， 兼容了空字符串''的判断  
.whereGt(Student::getAge,3)    // 过滤年龄大于3岁  
.whereGe(Student::getAge,3)   // 过滤年龄大于等于3岁  
.whereLt(Student::getAge,3)  // 过滤年龄小于3岁的  
.whereIn(Student::getAge, Arrays.asList(3,7,8)) // 过滤年龄为3岁 或者7岁 或者 8岁的数据  
.whereNotIn(Student::getAge, Arrays.asList(3,7,8)) // 过滤年龄不为为3岁 或者7岁 或者 8岁的数据  
.whereEq(Student::getAge,3) // 过滤年龄等于3岁的数据  
.whereNotEq(Student::getAge,3) // 过滤年龄不等于3岁的数据  
.whereLike(Student::getName,"jay") // 模糊查询，等价于 like "%jay%"  
.whereLikeLeft(Student::getName,"jay") // 模糊查询，等价于 like "jay%"  
.whereLikeRight(Student::getName,"jay"); // 模糊查询，等价于 like "%jay"  

```

2.3、汇总相关
--------

```
JDFrame<Student> frame = JDFrame.read(studentList);  
Student s1 = frame.max(Student::getAge);// 获取年龄最大的学生  
Integer s2  = frame.maxValue(Student::getAge);      // 获取学生里最大的年龄  
Student s3 = frame.min(Student::getAge);// 获取年龄最小的学生  
Integer s4  = frame.minValue(Student::getAge);      // 获取学生里最小的年龄  
BigDecimal s5 = frame.avg(Student::getAge); // 获取所有学生的年龄的平均值  
BigDecimal s6 = frame.sum(Student::getAge); // 获取所有学生的年龄合计  
MaxMin<Student> s7 = frame.maxMin(Student::getAge); // 同时获取年龄最大和最小的学生  
MaxMin<Integer> s8 = frame.maxMinValue(Student::getAge); // 同时获取学生里最大和最小的年龄  

```

2.4、去重相关
--------

原生steam只支持对象去重，不支持按特定字段去重

```
java  
  
 代码解读  
复制代码List<Student> std = null;  
std = SDFrame.read(studentList).distinct().toLists(); // 根据对象hashCode去重  
std = SDFrame.read(studentList).distinct(Student::getSchool).toLists(); // 根据学校名去重  
std = SDFrame.read(studentList).distinct(e -> e.getSchool() + e.getLevel()).toLists(); // 根据学校名拼接级别去重复  
std =SDFrame.read(studentList).distinct(Student::getSchool).distinct(Student::getLevel).toLists(); // 先根据学校名去除重复再根据级别去除重复  

```

2.5、分组聚合相关
----------

类似sql的 group by语义 简化处理分组和聚合的逻辑， 如果用原生stream需要写可能一大串逻辑.

```
JDFrame<Student> frame = JDFrame.from(studentList);  
// 等价于 select school,sum(age) ... group by school  
List<FI2<String, BigDecimal>> a = frame.groupBySum(Student::getSchool, Student::getAge).toLists();  
// 等价于 select school,max(age) ... group by school  
List<FI2<String, Integer>> a2 = frame.groupByMaxValue(Student::getSchool, Student::getAge).toLists();  
//  与 groupByMaxValue 含义一致，只是返回的是最大的值对象  
List<FI2<String, Student>> a3 = frame.groupByMax(Student::getSchool, Student::getAge).toLists();  
// 等价于 select school,min(age) ... group by school  
List<FI2<String, Integer>> a4 = frame.groupByMinValue(Student::getSchool, Student::getAge).toLists();  
// 等价于 select school,count(*) ... group by school  
List<FI2<String, Long>> a5 = frame.groupByCount(Student::getSchool).toLists();  
// 等价于 select school,avg(age) ... group by school  
List<FI2<String, BigDecimal>> a6 = frame.groupByAvg(Student::getSchool, Student::getAge).toLists();  
  
// 等价于 select school,sum(age),count(age) group by school  
List<FI3<String, BigDecimal, Long>> a7 = frame.groupBySumCount(Student::getSchool, Student::getAge).toLists();  
  
// (二级分组)等价于 select school,level,sum(age),count(age) group by school,level  
List<FI3<String, String, BigDecimal>> a8 = frame.groupBySum(Student::getSchool, Student::getLevel, Student::getAge).toLists();  
  
// （三级分组）等价于 select school,level,name,sum(age),count(age) group by school,level,name  
List<FI4<String, String, String, BigDecimal>> a9 = frame.groupBySum(Student::getSchool, Student::getLevel, Student::getName, Student::getAge).toLists();  

```

2.6、排序相关
--------

简化原生stream的排序方式，直接指定字段即可，不用使用Comparator还要去关注升序还是降序. 如果是多级排序使用Compartor或者Sorter去指定多级排序的逻辑。Sorter也是Compartor的一种实现，只是提供了更加语义化的多级排序指定逻辑， 相当于内置了Compartor的thenComparing

如果你近期准备面试跳槽，建议在ddkk.com在线刷题，涵盖 一万+ 道 Java 面试题，几乎覆盖了所有主流技术面试题，还有市面上最全的技术五百套，精品系列教程，免费提供。

```
// 等价于 order by age desc  
SDFrame.read(studentList).sortDesc(Student::getAge);  
//  (多级排序) 等价于 order by age desc, level asc.   
SDFrame.read(studentList).sortAsc(Sorter.sortDescBy(Student::getAge).sortAsc(Student::getLevel));  
// 等价于 order by age asc  
SDFrame.read(studentList).sortAsc(Student::getAge);  
// 使用Comparator 排序  
SDFrame.read(studentList).sortAsc(Comparator.comparing(e -> e.getLevel() + e.getId()));  

```

2.7、连接矩阵相关
----------

API列表

```
append(T t);                    // 等价于集合 add  
union(IFrame<T> other);         //  等价于集合 addAll  
join(IFrame<K> other, JoinOn<T,K> on, Join<T,K,R> join);   // 等价于 sql内连接  
leftJoin(IFrame<K> other, JoinOn<T,K> on, Join<T,K,R> join);   // 等价于sql左连接，如果左连接失败，K值为null，需手动判断  
rightJoin(IFrame<K> other, JoinOn<T,K> on, Join<T,K,R> join);    // 等价于sql右连接，如果右连接失败，T值为null，需手动判断  

```

内连接例子：

```
System.out.println("======== 矩阵1 =======");  
  
SDFrame<Student> sdf = SDFrame.read(studentList);  
  
sdf.show(20);  
  
// 获取学生年龄在9到16岁的学学校合计分数最高的前10名  
SDFrame<FI2<String, BigDecimal>> sdf2 = SDFrame.read(studentList)  
      .whereNotNull(Student::getAge)  
      .whereBetween(Student::getAge,9,16)  
      .groupBySum(Student::getSchool, Student::getScore)  
      .sortDesc(FI2::getC2)  
      .cutFirst(10);  
  
System.out.println("======== 矩阵2 =======");  
sdf2.show();  
  
SDFrame<UserInfo> frame = sdf.join(sdf2, (a, b) -> a.getSchool().equals(b.getC1()), (a, b) -> {  
  UserInfo userInfo = new UserInfo();  
  userInfo.setKey1(a.getSchool());  
  userInfo.setKey2(b.getC2().intValue());  
  userInfo.setKey3(String.valueOf(a.getId()));  
  return userInfo;  
});  
  
System.out.println("======== 连接后结果 =======");  
frame.show(5);  

```

打印信息：

```
======== 矩阵1 =======  
id name school level age score rank   
1  a    一中     一年级   11  1            
2  a    一中     一年级   11  1            
3  b    一中     一年级   12  2            
4  c    二中     一年级   13  3            
5  d    二中     一年级   14  4            
6  e    三中     二年级   14  5            
7  e    三中     二年级   15  5            
  
======== 矩阵2 =======  
c1 c2   
三中 10   
二中 7    
一中 4    
  
======== 连接后结果 =======  
key1 key2 key3 key4   
一中   4    1           
一中   4    2           
一中   4    3           
二中   7    4           
二中   7    5  

```

类似于

```
select a.*,b.* from sdf a inner join sdf2 b on  a.school = b.c1  

```

2.8、截取相关
--------

```
cutFirst(int n); // 截取前N个  
cutLast(int n); // 截取后N个  
cut(Integer startIndex,Integer endIndex) // 按照索引范围截取 [startIndex,endIndex). 等价于 List.subList  
cutPage(int page,int pageSize)      // 按分页截取  
cutFirstRank(Sorter<T> sorter, int n);    // 截取前N排名的数据  

```

2.9、Frame参数设置相关
---------------

```
defaultScale(int scale, RoundingMode roundingMode); // 设置计算结果的默认小数精度  

```

2.10、其他
-------

### 百分数转换

```
// 等价于 select round(score*100,2) from student  
SDFrame<Student> map2 = SDFrame.read(studentList).mapPercent(Student::getScore, Student::setScore,2);  

```

### 分区

将每个5个元素分成一个小集合，用于将大任务拆成小任务

```
List<List<Student>> t = SDFrame.read(studentList).partition(5).toLists();  

```

### 生成序号列

按照age排序，然后根据当前顺序生成排序号到rank字段 （序号从1开始）

```
SDFrame.read(studentList)  
    .sortDesc(Student::getAge)  
    .addRowNumberCol(Student::setRank)  
    .show(30);  

```

输出信息:

```
id name school level age score rank   
7  e    三中     二年级   15  5     1      
5  d    二中     一年级   14  4     2      
6  e    三中     二年级   14  5     3      
4  c    二中     一年级   13  3     4      
3  b    一中     三年级   12  2     5      
1  a    一中     一年级   11  1     6      
2  a    一中     一年级   11  1     7  

```

### 补充条目

1、补充缺失的学校条目

```
// 所有需要的学校条目  
List<String> allDim = Arrays.asList("一中","二中","三中","四中");  
// 根据学校字段和allDim比较去补充缺失的条目， 缺失的学校按照ReplenishFunction生成补充条目作为结果一起返回  
SDFrame.read(studentList).replenish(Student::getSchool,allDim,(school) -> new Student(school)).show();  

```

输出

```
id name school level age score rank   
1  a    一中     一年级   11  1            
2  a    一中     一年级   11  1            
3  b    一中     一年级   12  2            
4  c    二中     一年级   13  3            
5  d    二中     一年级   14  4            
6  e    三中     二年级   14  5            
7  e    三中     二年级   15  5            
0       四中  

```

2、分组补充组内缺失的条目

按照学校进行分组， 汇总所有年级allDim. 然后与allDim比较补充每个分组内缺失的年级，缺失的年级按照ReplenishFunction生成补充条目

```
SDFrame.read(studentList).replenish(Student::getSchool,Student::getLevel,(school,level) -> new Student(school,level)).show(30);  

```

输出

```
id name school level age score rank   
1  a    一中     一年级   11  1            
2  a    一中     一年级   11  1            
3  b    一中     三年级   12  2            
0       一中     二年级                    
4  c    二中     一年级   13  3            
5  d    二中     一年级   14  4            
0       二中     三年级                    
0       二中     二年级                    
6  e    三中     二年级   14  5            
7  e    三中     二年级   15  5            
0       三中     一年级                    
0       三中     三年级  

```

应用场景举例：要求计算近两年每个月的数据，但是数据的年月可能不全，这时就补充缺失的年月数据作为结果一起返回

最后
==

提供了两种Frame，SDFrame和JDFrame 在API层面一模一样， 区别是JDFrame的所有操作实时生效， 无需要重新read生成，而SDFrame与stream流一致，只有执行终止操作才会生效，并且需要重新read生成流， 而且在同一个流之间的操作是互相影响的。如果只是需要流式操作一条流执行完就用SDFrame， 如果需要“中间站点”数据，然后从“中间站点数据“开始计算就用JDFrame， 这个在含义层面与DataFrame模型类似。

这个在语法层面能实现的矩阵还是比较有限的因为行列是通过枚举的几个FI去描述，但是不同的逻辑导致的矩阵变换的变化可能是非常大的，除非JDK能语法层面支持到吧或者放弃强类型全部硬编码才能实现各种矩阵的表示和变换。期待JDK一个JVM层面的“pandans” 出现。

还有一些api没有列举出来使用的比较少 主要是对逻辑的封装和语意化，如果还有哪些逻辑和api可以扩展可以在评论区留下你的想法。
