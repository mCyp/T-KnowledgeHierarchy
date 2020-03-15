# Kotlin知识回顾

## 一、Kotlin基础

### 1. 基本要素

#### # 函数

格式：

```kotlin
// 正常
fun max(a:Int,a:Int):Int{
	return if(a>b) a else b
}
// 表达式
fun max(a:Int,a:Int):Int = if(a>b) a else b
```

#### # 变量

特征：

- 编译器可以分析表达式的值退出类型

关于`val`和`var`：

- val：不可变引用，尽管引用自身不可变，但是引用的对象是可变的
- var：引用可变，类型不可变

#### # 字符串模板

```kotlin
print("a:${a}")
```

### 2. 类和属性

```kotlin
class Person(val name:String)
```

#### # 属性

在Kotlin中，属性是头等的语言特性。

```kotlin
class Person(
	val name:String // 一个val = 一个属性和 + get方法
	var age:Int // 一个var = 一个属性 + get + set
)
```

对于`ge`t和`set`方法，开发者是可以自己复写的。

### 3. 枚举和when

#### # 枚举

枚举是软关键字，后面还需要加一个`class`：

```kotlin
enum class Color{
  RED，BLUE
}
```

#### # when

`when`的使用跟Java中使用`switch`类似，它的特点是：

- 具有返回值，可以作为表达式直接返回(`if`也可以)
- `switch`只支持基本类型、枚举以及数字字面值，`when`支持任意对象
- 支持不使用参数，功能跟if类似

#### # is

`is`跟Java中的`instance of`类似，`is`支持智能转换

### 4. 迭代

#### # while

同Java

#### # 区间和数列

- `..`：闭区间，`1..10` -> [1,10]
- `until`：开区间，`1 until 10` -> [1,10)
- `down to`：开区间，到下届
- `step`：步长

#### # 迭代map

map使用的一些技巧：

- 使用key当做下标，跟java数组的访问方式一样
- 解构声明方法访问

#### # in

```kotlin
val list = listof(1,2,3)
val isContain = 2 in list
// 结合区间
val c = 1 in 1..2
// 结合when 省略
// ...
```

使用`in`可以检测集合是否包含该成员，



## 二、函数

### 1. 使用集合

Kotlin没有没用自己的集合类，而是采用Java中的集合类。

虽然采用的Java中的集合类，但是Kotlin为其新增了许多扩展函数，因而在使用上更加方便。

### 2. 更方便的函数

Kotlin为了让函数有更好的易用性，增加了许多方便的功能。

#### # 打印数组或者集合更加方便

```kotlin
>>> val list = listof(1,2,3)
>>> print(list)
[1,2,3]
```

#### # 命名参数

```kotlin
// 比如调用一个方法
joinToString(collection,seperator = "",prefix = "[]",postfix = "]")
```

#### # 默认参数

默认参数可以用来解决Java中重载函数过多的问题。

```kotlin
fun <T> joinToString(
	collection:Collection<T>,
  separator:String = ",",
  prefix:String="",
  postfix:String=""
)
```

可以这么调用：

```kotlin
>>> joinToString(list,",","[","]")
[1,2,3]
>>> jointToString(list)
1,2,3
>>> jointToString(list,";")
1;2;3
```

如果想对Java调用者同样方便，可以使用`@JymOverloads`关键字。

#### # Kotlin中的静态函数和属性

顶层方法和函数

如果想修改顶层函数的生成的类的名称，可以使用`@JymName`注解。

#### # 扩展函数和属性

扩展函数和属性的实质是Java中的静态方法。

扩展函数：

```kotlin
fun String.lastChar():Char = this.get(this.length-1)
```

扩展属性：

```kotlin
fun String.lastChar : Char
	get() = get(length-1)
```

注意：

- 扩展函数不能访问私有的成员函数和属性
- 扩展函数实质是静态函数，所以它不能被子类复写，所以不存在重载

#### # 可变参数

可以参数使用关键字`vararg`，实质上是一个`ArrayList`

#### # 中缀调用

如`until`：

```kotlin
for(i in 1 until 100){
  // 1 until 100 就是一个中缀调用
  ...
}
```

中缀的声明，使用`infix`关键字：

```kotlin
public infix fun Int.until(to: Int): IntRange {
    if (to <= Int.MIN_VALUE) return IntRange.EMPTY
    return this .. (to - 1).toInt()
}
```

声明条件：

- 函数需要使用`infix`关键字
- 函数只能有一个参数

#### # 解构声明

- `data`类默认支持解构声明的
- 最多可以支持5个参数

### 3. 字符串

#### # 三引号

三引号可以包含任意字符，包括转义字符和换行字符



## 三、类、对象和接口

### 1. 接口

#### # 定义

跟Java中定义方式相同，不过Kotlin中可带有默认实现方法：

```kotlin
interface{
  fun onclick()
  fun showOff() = println("i'm off")
}
```

#### # 实现

使用符号`:`

#### # 注意

- 多接口具有相同方法，且同一个类实现了多接口，需要在实现的同名方法中显示声明继承了哪个方法

- 接口中可以定义属性，不过每次该属性的获取都需要计算

### 2. 访问权限

Java和Kotlin对于访问权限的控制不同。

#### # 类和方法

对于类和方法的修饰符有：`open`、`final`和`abstract`

Java里面默认`open`，kotlin默认`final`，如果想要该类能够被当做基类，需要使用`open`修饰`class`

#### # 属性

关于属性：

- Java默认包访问权限，Kotlin没有，Kotlin提供一种新的修饰符`internal`，表示只在模块内部可见
- kotlin支持的可见性修饰符有`private`、`internal`、`protected`和`public`
- Kotlin默认属性的访问权限是`public`的

#### # 内部类

Java和Kotlin中的内部类也有很大的区别：

- Java：默认持有外部类的引用(这也是`Handler`容易发生泄漏的原因)，使`static`就不会持有外部类的引用
- Kotlin：默认不会持有外部类的引用，相当于静态类，如果想持有外部类的引用，需要使用`inner`修饰类

#### # 密封类

`sealed`：

```kotlin
sealed class Expr{
	Class Num(val v:Int):Expr()
  class Sum(val a:Int,val b:Int):Expr()
}
```

如上所见，所有子类都要实现在`Expr`类的内部。

### 3. 编译器生成的方法

#### # 通用对象方法

- `toString()`
- `equals()`：Kotlin中`==`等同于`equals`，`==`会检查是否为空，`===`跟Java中的`==`一样，比较引用，基本类型会比较值
- `hashcode()`

#### # 数据类

`data`默认实现了通用对象方法。

```kotlin
data class person(val name:String,val age:Int)
```

#### # 委托类

`by`：将实现委托给包装的类对象

### 4. object

关于`object`：

- object是声明单例的一种方式
- 可以当做工厂方法使用
- 匿名对象使用，对象表达式

`companion object`：

- 不同于`object`，`companion object`可以访问私有的构造函数



## 五、lambda

lambda表达式，简称lambda，本质上是传给其他函数的一小段代码。

### 1. lambda表达式和成员引用

#### # 表达式

```kotlin
{x:Int,y:Int -> x+y}
```

kotlin中的lambda有如下特点：

- 表达式总是用花括号包围
- lambda表达式作为最后一个实参，可以放在圆括号外边
- lambda作为唯一参数，圆括号括号可以省略
- lambda参数的类型可以被显示的推倒，就不需要声明
- 可以使用默认参数it代替参数声明

除此以外，lambda表达式还可以存储在一个变量中：

```kotlin
val s = {x:Int,y:Int->x+y}
val res = s(2,3)
```

#### # 在作用域访问局部变量

在Java中，你仅仅可以访问带有final修饰符修饰的局部变量，如果你想捕捉布局变量，可以使用两种方法：

1. 使用数组存储可变值
2. 使用包装类

在Kotlin中，你不仅可以反问局部不可变变量，还可以访问可变变量。那么访问可变变量的原理是什么，就是上述的第二种方法

```kotlin
class Ref<T>(var value:T)
>>> val counter = Ref(0)
>>> val n = {counter.value++}
```

#### # 成员引用

如果你想要使用的传递参数代码块已经被封装成函数，你可以直接引用它。

```kotlin
Person::age
```

你可以将成员引用用于：

- 成员方法
- 构造方法
- 扩展方法

### 2. 集合的函数Api

#### # filter和map

`filter`：用来过滤

```kotlin
>>> val list = listof(1,2,3,4)
>>> print(list.filter{it % 2 == 0})
[2,4]
```

`map`：用来转换

```kotlin
>>> val persons = listof(Person("wang",24),Person("chen",24))
>>> print(list.map{it.name})
["wang","chen"]
```

#### # all\any\count\find

`all`：所有元素满足

`any`：至少一个元素满足

`count`：满足的数量

`find`：返回满足的第一个元素

#### # flatmap

#### # 集合类lambda函数的特点

- `map`和`filter`操作每次都会生成一个新的集合，对内存的消耗比较大
- 但因为是内联函数不会为lambda函数生成新的对象

### 3. 惰性集合：序列

```kotlin
// 一个集合
people.asSequence()
	.map(Person::name) // 中间操作
	.filter(it.startWith('A')) // 中间操作
	.toList() // 末端操作
```

Kotlin操作的惰性集合的操作入口是Sequence接口，这个接口表示的是一个可以逐个列举元素的序列。Sequence只提供一个方法，iterator。

序列的特点：

- 惰性集合：中间操作都会被延期，末端操作会执行所有的延期计算
- 按顺序挨个执行：不同于集合，序列是单个元素执行完所有的操作再轮到下个元素执行
- 中间操作非内联，也就是lambda会生成对象
- 惰性方法有利于我们跳过处理部分元素，从而提升性能

使用序列或者集合：

- 如果使用的数据量比较大，优先选择序列

### 4. 使用Java函数式接口

#### # SAM接口

单函数接口，比如`OnClickListener`。

导致的问题：

- 不能使用引用lambda代码的实例，进行删除，因为lambda始终是一个代码块，不能当做对象

#### # lamda当做参数传递

- 使用`object`显示声明匿名类每次都会生成一个新的对象
- 使用lambda会导致响应的匿名类实例重用，但是如果捕捉了方法中局部变量，会导致每次重新生成新的对象

### 5. 带接受者的lambda

#### # with和apply

`with`和`apply`类似，`apply`会返回自身



