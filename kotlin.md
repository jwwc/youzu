# kotlin的学习笔记

## kotlin的基础语法

### 不一样的声明

	- java语法 
		String a = " I am Java" ;
	- Kotlin语法
		val a: String = "I am Kotlin" // 类型名在变量名后面
### 增强的类型推导
	
	案例

	val string = "I am Kotlin"
	val int = 1214
	val long = 1314L
	val float = 13.14f
	val double = 13.34
	val double2 = 10.1e6

	println(string.javaClass.name)可以输出变量类型

	java.lang.String
	int
	long
	float
	double
	double

类型推导在很大程度上提高了Kotlin这种静态语言的开发效率，虽然静态类型的语言的优点有很多，但是在编码过程中却需要书写大量的类型，类型推导可以帮助Kotlin改善这一情况

### 声明函数返回值类型

Kotlin定义一个函数： fun sum( x: Int, y :Int): Int { return x+ y}

中间的:Int表示返回值类型，否则就会报错

Kotlin的表达式函数体 fun sum(x: Int,y :Int) = x+y
sum(1,2) >> 3

使用等号时，不需要定义返回值类型，不需要使用return进行返回，可以省略{}

普通的函数声明称为代码块函数体

kotlin不能对递归函数进行自动推导

- 错误代码  
	fun foo(n:Int) = if( n== 0) 1 else n * foo(n-1)
- 正确写法
	fun foo(n:Int): Int = if(n==0) 1 else n * foo(n-1) // 显示声明返回类型

**需要显示声明的类型**

- 函数的参数

- 非表达式定义的函数，除了返回Unit类型

- 递归函数

- 公有方法的返回值

### val 和 var的使用规则

var是用来定义变量的

	var temp :String = " this is a variable"
	
	temp = "exchange value" // var定义的变量是可以更改的

val修饰的变量的引用不可变，相当于java中的final修饰的变量

	val temp: String = "this is a variable"

	temp = "exchange value" // val定义的变量引用不可更改，会报错

使用IntelliJ IDEA或者 Android Studio中查看val语法反编译后的java代码，可以看出它使用final实现这一特性

val修饰的引用不可变，但是其用于的地址的值是可以变的

	val x = intArrayOf(1,2,3)
	x[0] = 2
	println(x[0])
	>> 2

val修饰的变量的引用对应是可以更改的

代码举例

	class Book(var name: String){
		fun printName(){
			println(this.name)
		}
	}

	fun main(args: Array<String>){
		val book = Book("Thinking in Java")
		book.name = "Diving into Kotlin"
		book.printName()
	}

### 优先使用val来避免副作用

代码举例

	var a = 1

	fun count(x :Int){
		a = a+1
		println(x+a)
	}
	>>> count(1)
	3
	>>> count(1)
	4

以上代码我们可以看出两次调用count(1)，得到的结果不同，其结果受到外部变量a的影响，是典型的副作用

### var的适用场景

代码举例
	
	fun cal(list:List<Int>) : Int{
		var res = 0
		for(el in list){
			res *= el
			res += el
		}
		return res
	}

res是一个局部的可变变量，与外界没有联系，安全可控

val的实现：

	fun cal(list:List<Int>): Int{
		fun recurse(listr: List<Int>, res: Int) :Int{
			if(listr.size > 0){
				val el = listr.first()
				return recurse(listr.drop(1),res*el+el)
			} else {
				return res
			}
		}
		return recurse(list,0)
	}

由于val修饰的变量的引用不可变，必须借助递归才可以实现

使用fold方法进行精简代码

	fun cal(list :List<Int>) : Int{
		return list.fold(0) {res,el->res*el+el}
	}


[高阶函数的学习链接](https://blog.csdn.net/weixin_43841938/article/details/108713054)

### 高阶函数和Lambda

Kotlin中函数是头等公民，不仅可以像类一样在顶层直接定义一个函数，也可以在一个函数内部定义一个局部函数

示例代码

	fun foo(x: Int){
		fun double(y : Int): Int{
			return y * 2
		}
		println(double(x))
	
	}

### 抽象和高阶函数

高阶函数：可以简单的理解为我们的函数能够接收一个或者多个过程位参数；或者以一个过程作为返回结果，是一个更深层次的抽象

#### 函数作为参数的需求举例

设计一个CountryApp类对国家数据操作，Shaw偏爱欧洲国家，于是设计了一个程序来获取欧洲所有国家

	data class Country (
		val name: String
		val continient: String
		val population: Int)

	class CountryApp{
		fun filterCountries(contries: List<Country>) :List<Country>{
			val res = mutableListOf<Country>()
			for(c in countries){
				if(c.continient == "EU"){
					res.add(c)
				}
			}
			return res
		}
	}
	// data class是什么？
	// mutableListOf<Country>()是什么

shaw对非洲也产生了兴趣，于是改进了上述方法的实现，支持具体洲来筛选国家。

	fun filterCountries(countries: List<Country>, continient: String): List<Country>{
		val res = mutableListOf<Country>()
		for( c in countries){
			if (c.continient == continient){
				res.add(c)
			}
		}
		return res
	}

随着业务的更换，更多的筛选条件会作为参数而不断累加，而且业务逻辑也高度耦合。解决问题的核心在于对filterCountries方法进行解耦，我们能否将所有的筛选条件逻辑行为抽象成一个参数。

代码示例
	class CountryTest {
		fun isBigEuropeanCountry(country: Country): Boolean {
			return Country.continient == "EU" && country.population > 10000
			}
	}
调用isBigEuropeanCountry方法就能够判断一个国家是否是一个人口超过一亿的欧洲国家，怎么样才能够把这个方法变成filterCountries方法的一个参数呢，要实现这一点要解决下面两个问题

- 方法作为参数传入，必须像其他参数一样具备具体的类型信息
- 需要把isBigEuropeanCountry的方法引用当作参数传递给filterCountries

#### 函数的类型

Kotlin中函数类型声明遵行以下几点：

- 通过->符号来组织参数类型和返回值类型，左边是参数类型，右边是返回值类型
- 必须使用括号来包裹参数类型
- 返回值类型即使是Unit,也必须显示声明

如果一个没有参数的函数类型，参数类型部分就用()来表示。

() -> Uint

kotlin还支持为声明参数指定名称，如下所示：

(errCode : Int, errMsg: String) ->Uint

如果该函数的变量也是可选的，我们可以将整个函数类型变成可选：

((errCode: Int, errMsg: String?) -> Unit)?

// 这里的？号用法不清楚

高阶函数还支持返回另一个函数

（Int) ->((Int) -> Unit)

这表示传入一个类型为Int的参数，然后返回另一个类型为(Int)->Unit的函数，简化它的表达可以简化为

 （Int) -> Int -> Uint

需要注意的是，以下的函数类型则不同, 它表示传入一个函数类型的参数，再返回一个Unit

((Int) -> Int) -> Unit

接下来可以重新定义filterCountries方法的参数声明

	fun filterCountires(countries: List<Country>, test: (Country)-> Boolean) : List<Country>{
		val res = mutableListOf<Country>()
		for( c in countries){
			if(test(c)){
				res.add(c)
			}
		}
		return res
	}

怎么将isBigEuropeanCountry方法传递给filterCountries呢？

// 直接将isBigEuropeanCountry当参数肯定不行，因为函数名并不是一个表达式，不具有类型信息，它在带上括号，被执行后才存在值，需要用一个单纯的方法引用表达式。

### 方法和成员方法

Kotlin存在一种特殊的语法，通过两个冒号来实现对于某个类的方法进行引用。假如我们有一个CounryTest类的对象实例countryTest,如果要引用它的isBigEuropeanCountry方法，就可以这么写：
	countryTest::isBigEuropeanCountry

示例

	class Book(val name: String)
	fun main(args: Array<String>){
		val getBook = ::Book
		println(getBook("Dive int Kotlin").name)
	}

可以发现，getBook的类型为（name:String)->Book。


上面示例代码最终的版本

	val countryApp = CountryApp()
	val countryTest = CountryApp()

	val countries = ......
	countryApp.filterCountries(countries, countryTest::isBigEuropeanCountry)

### 匿名函数

	上面的方案依旧不是一个很好的方案，因为每增加一个需求，我们就需要专门写一个新增的赛选方法，可以使用匿名函数进行优化。
	
	Kotlin支持缺省的函数名

	fun(country: Country): Boolean{ // 没有函数名字
		return country.continient == "EU" && country.population > 10000
	}

	改写上述代码

	countryApp.filterCountries(countries, fun(country: Country): Boolean{
		return country.continient == "EU" && country.population > 1000
	})

### Lambda语法糖

	使用Lambda表达式改写上述代码

	countryApp.filterCountries(countries, { country ->country.continient == "EU" && country.population > 1000})

Lambda 表达式简化举例

	val sum:(Int,Int) ->Int = {x: Int,y: Int->x+y }
	
	// 由于支持里类型推导
	- val sum = {x:Int, y:Int-> x+y}
	
	- val sum: (Int,Int) -> Int = {x,y -> x+y}

Lambda表达式的语法总结
	
	- Lambda表达式必须通过{}来包裹

	- 如果Lambda声明了参数部分类型，Lambda变量就可以省略函数类型声明

	- 如果Lambda变量声明了函数类型， 那么Lambda的参数部分类型就可以省略

用fun关键字来声明Lambda表达式

	fun foo(int :Int) = {
		print(int)
	}

	>>> listOf(1,2,3).forEach{ foo(it) } //对一个整数列表遍历调用foo

	it是单个参数的隐式表达，相当于{ item -> foo(item) }

但是上述代码运行之后没有任何输出，foo函数反编译出的java代码为：

	public static final Function0 foo(final int var0){
		return (Function0)(new Funtion0 (){
			public Object invoke() {
				this.invoke();
				return Unit.INSTANCE;
			}

			public final void invoke(){

				int varl = var0;
				System.out.print(varl);
			}
		});
	}

上面的代码仅仅构造了以一个Function0对象，并没有执行相应的invoke方法，因此需要修改上述代码为foo(it).invoke()才能够输出结果。

### 函数，Lambda和闭包

fun声明函数，Lambda表达式的语法区分

	- fun在没有等号，只有花括号的情况下，是我们最常见代码块函数体，如果返回非Unit值，必须带return

		fun foo(x: Int){ print(x) }

		fun foo(x: Int, y: Int): Int{ return x*y}

	- fun带有等号，是单表达式函数体，该情况下可以省略return

		fun foo(x: Int, y: Int) = x+y

	- 不管是val还是fun，如果是等号加花括号的语法，那么构架的就是一个Lambda表达式

		val foo = {x: Int, y: Int -> x+y} // foo.invoke(1,2)或者foo(1,2)

		fun foo(x:int) = {y: Int-> x+y} // foo(1).invoke(2)或者foo(1)(2)

### 面向表达式编程

- if表达式

- 函数体表达式

- Lambda表达式

- 函数引用表达式

#### 表达式比语句更安全

示例代码
	
	void ifStatement(Boolean flag){
		String a = null;
		if (flag){
			a = "dive into kotlin";
		}
		System.out.println(a.toUpperCase())
	}

由于if语句不是一个表达式，所以只能在外部对变量进行声明吗，会存在潜在问题

- 如果flag不为true, a没有被赋值，就会造成空指针异常

- 如果a来自上下文很远，可能会被忽略，会造成不安全的异常

使用kotlin语言的修正版本

	fun ifExpression(flag: Boolean){
		val a = if (flag) "dive into Kotlin" else ""
		println(a.toUpperCase)
	}

1 -代码更加紧凑，不存在a没有初始值的情况

### Unit类型： 让函数调用皆为表达式

Unit和int一样，都是一种类型，然而它不代表任何信息，用面向对象的术语来描述就是一个单例，它的实例只有一个，可以写为(). Kotlin之所以要引入Unit的一个很重要的原因是函数式编程侧重于组合，尤其是很多高阶函数，在源码实现的时候都是采用泛型来实现的，然而void在涉及泛型的情况下会存在问题

代码举例
	
	interface Function<Arg, Return>{
		return apply(Arg arg);	
	}
	Function<String,Integer> stringLength = new Function<String, Integer>(){
		public Integer apply(String arg）{
			return arg.length();
		}
	};
	int result = stringLength.apply("hello");
	// 运行结果
	5

看上去没有什么问题，我们希望重新实现一个print方法，但是问题来了，Return的类型用什么来表示呢，可能有人会使用void,但是在java中不能这么干，我们只能把Return换成Void,即Function<String,Void>,由于Void没有实例，则返回null,这种做法相当丑陋。

### 复合表达式：更好的表达力

代码举例

	val res: Int? = try{
		if(result.succcess){
			jsonDecode(result.response)
		}else null
	}catch( e: JsonDecodeException){
		null
	}


这个程序描述了获取一个HTTP响应结果，然后进行json解码，最终赋值给res变量的过程，它向我们展示了Kotlin如何利用多个表达组合表达的能力


?: 和 ? 的作用和区别

？是来表示一种类型的可空性

？: 来给一种可空类型的变量指定为空情况下的值

	val maybeInt : Int? = null
	>>> maybeInt ?: 1
	1

### 枚举类和when表达式


	enum class Day{
		MON,
		TUE,
		WEN,
		THU,
		FRI,
		SAI,
	}
	
	// Kotlin中枚举类。

	enum class DayOfWeek( val day: Int){
		MON(1),
		TUE(2),
		WEN(3),
		THU(4),
		FRI(5),
		SAI(6),
		SUN(7)
	
	;
	fun getDayNumber():Int{
		return day
		}
	}

##### 用when来代替if-else


代码示例

	fun schedule(day: Day, sunny: Booolean) = {
		if( day == Day.SAT){
			basketball()
		}else if(day == Day.SUN){
			fishing()
		}else if(day == Day.FRI){
			appointment()
		}else {
			if(sunny){
				library()
			}else {
				study()
			}
		}
	}

使用when进行代码优化

	fun schedule(sunny: Boolean, day: Day) = when (day){
		Day.SAT -> basketaball()
		Day.SUN -> fishing()
		Day.FRI -> appointment()
		else -> when{
			sunny -> library()
			else -> study()
		}
	}

#### for循环和范围表达式

	- 1

	for(i in 1...10) println(i)

	- 2

	for(i:Int in 1...10){
		println(i)
	}

#### 范围表达式

	for( i in 1..10 step 2) print(i) //step定义迭代的步长
	>>> 13579

	for(i in 10 downTo 1 step 2) print(i) 
	>>> 108642

	for（i in 1 until 10){ print(i) }
	123456789 // 并不包含10 ，半开区间

#### 用in来检查成员关系

	>>>”a" in listOf("b","c")
	false
	>>> "a" !in listOf("b","c")
	true

	>>> "kot" in "abc".."xyz"
	true
	// 等价于
	"kot" >= "abc" && "abc" <= "xyz"

	使用withIndex方法，提供一个键值元组

	for((index, value) in array.withIndex()){
		println("the element at $index is $value")
	}

#### 中缀表达式

在Kotlin中，to这种形式定义的函数称为中缀函数，表现形式 A 中缀方法 B

定义中缀表达式需要满足的条件：

- 中缀函数必须是某一个类型的扩展函数或者成员

- 中缀函数只能有一个参数

- 中缀函数不能有默认值，否则以上形式的B会缺失，对中缀表达式的语义造成破环

kotlin的可变参数是通过vararg来表示的

	fun printLetters(vararg letters: String, count: Int):Unit{
		print("${count} letters are")
		for (letter in letters) print(letter)
	}

	>>> printLetters("a","b","c",count = 3)
	3 letters are abc

由于to会返回Pair这种键值对的数据结构，因此我们经常会把它与map结合在一起使用

	mapOf(
		1 to "one",
		2 to "two",
		3 to "three"
	)

中缀表达式调用举例

	class Person {
		infix fun called(name: String){
			println("My name is $(name).")
		}	
	}

	fun main(args: Array<String>){
		val p = Person()
		p called "Shaw" // 使用infix对call方法进行调用，可以中缀函数的调用形式
	// 运行结果
	My name is Shaw

### 字符串的定义和操作

代码举例

	val str = "hello world!"

	str.length // 12
	str.substring(0,5) // hello
	str + " hello Kotlin!" // hello word! hello Kotlin!
	str.replace("world","Kotlin") // hello Kotlin!

	for(i in str.toUpperCase()) {print(i)}
	HELLO WORLD!

	str[0] //h
	str.first() // h
	str.last() //!
	str[str.length- 1] // !

	"".isEmpty() // true
	" ".isEmpty() // false
	" ".isBlank() // true
	"abcdefg".filter { c ->c in 'a'..'d' } // abcd

#### 定义原生字符串

代码举例

	val rawString  = """ \n Kotlin is awesome. \n Kotlin is a better Java. """
	println(rawString) // \n Kotlin is awesome . \n Kotlin is a better Java

#### 字符串模板

	fun message(name: String, lang: String) = "Hi" + name + ", welcome to " + lang + "!"
	>>> message("Shaw", "Java")
	Hi Shaw, welcome to Java!

	//表达式插入字符串
	>>> "Kotlin has ${if ("Kotlin".length > 0) "Kotlin".length else "no"} letters."
	Kotlin has 6 letters

#### 字符串判等

	- 结构相等 ==
	- 引用相等 ===

	
