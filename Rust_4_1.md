# 4. 语法和语义
本章将Rust切分成几个小节, 每节介绍一个概念.

如果你想自底向上的学习Rust, 最好按顺序来阅读.

## 4.1 变量绑定
实际上每个非"Hello World"的Rust程序都会使用***变量绑定***. 它们会将某些值绑定到一个变量上, 便于今后使用. 我们用let来介绍绑定:

	fn main() {
		let x = 5;
	}
	
### 模式
在很多语言中, 变量绑定被称之为***变量***, 而Rust的变量绑定有些特殊. 例如, 等号左边的***let***声明是一种"模式", 而不是变量名. 这意味着我们可以这么做:

	let (x, y) = (1, 2);
	
在这个声明之后, x被赋值成1, 而y被赋值成2. 模式十分强大, 我们会用特定的章节来介绍. 我们暂时不需要这些特性, 在后面会特别介绍.

### 类型注解
Rust是一种静态类型语言, 这意味着我们需要标明类型信息, 这样可以在编译期间被检查. 那为什么有些没有指定类型的代码也可以编译呢? 其实是因为Rust有"类型推导". 如果Rust自己能够判断出类型是什么, 就不需要你显示的指明.

我们还是可以指明类型信息. 类型跟在一个冒号(:)后:

	let x: i32 = 5;

我们称之为"x是一个i32类型的绑定, 它的值为5".

在这个例子中我们选择将x表述为一个32位有符号整数. Rust有很多原始整数类型. 一般以i开头的是有符号整形, 以u开头的是无符号整形. 可能的整形大小为8, 16, 32或者64位.

在后续的例子中, 我们可能会在注释中声明类型. 类似于:

	fn main() {
		let x = 5; // x: i32
	}
	

### 可变性
默认情况下, 绑定是***不可变***的,下面的代码无法编译通过:

	let x = 5;
	x = 10;
	
你会得到下面的错误:

	error: re-assignment of immutable variable `x`
		x = 10;
		^~~~~~~
		
如果你想让绑定可变, 你可以使用***mut***:

	let mut x = 5; // mut x: i32
	x = 10;
	
默认绑定不可变的原因有很多, 但是Rust中主要原因是: 安全. 如果你忘记声明mut, 编译器会捕捉到并告诉你正试图改变某些你不想改变的东西. 如果绑定默认是可变的, 编译器无法告诉你这些信息. 如果真的想修改, 解决方案很简单: 加上mut.

还有些其他的原因来避免可变状态, 不过超出了本文讨论的范围. 总的来说, 通常你可以避免显示的修改, 这在Rust里是被推荐的. 也就是说如果你真的希望修改, 那么就显示的指出来.

### 初始化绑定
Rust变量绑定和其他语言有些不同: 绑定在使用之前必须先被初始化.

我们来试试. 将你的src/main.rs文件修改成这样:

	fn main() {
		let x: i32;
		println!("Hello World!")
	}

你可以用cargo build来构建. 你会得到一个warning, 但是它还是会打印出"Hello, world!":

		Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
	src/main.rs:2:9: 2:10 warning: unused variable: `x`, #[warn(unused_variable)]
   		on by default
	src/main.rs:2     let x: i32;
	
Rust会警告我们从来没有使用过这个变量绑定, 但是也因为我们没有使用它, 因此不会有什么影响. 然而一旦我们真的想使用这个x, 一切就不同了. 将你的程序修改成下面这样:

	fn main() {
		let x: i32;
		
		println!("The value of x is: {}", x);
	}
	
尝试编译, 你会得到下面的错误:

	$ cargo build
   		Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
	src/main.rs:4:39: 4:40 error: use of possibly uninitialized variable: `x`
	src/main.rs:4     println!("The value of x is: {}", x);
                                                    ^
	note: in expansion of format_args!
	<std macros>:2:23: 2:77 note: expansion site
	<std macros>:1:1: 3:2 note: in expansion of println!
	src/main.rs:4:5: 4:42 note: expansion site
	error: aborting due to previous error
	Could not compile `hello_world`.
	
Rust不允许我们使用未初始化的值. 接下来, 我们讨论在println!中做的工作.

如果你在你想输出的字符串中包含了一个大括号, Rust会将此理解成从中插入一些值. ***字符串插值***是一个计算机科学术语, 含义是"stick in the middle of a string". 我们再加上一个逗号, 然后是x, 来表述我们想用x来表示插入的值. 逗号是用来分割我们传入函数或者宏的参数的.

当你使用大括号时, Rust会检查需要输出的值类型, 以更好的方式来打印该值. 如果你想要指定更具体的格式, 有很多选择. 到目前为止, 我们都使用默认方式: 整形打印起来并不复杂.

### 作用域和shadowing
回到绑定上来. 变量绑定都是有作用域的 - 它们被限制存在于被定义的代码块中. 代码块是指被包含在{和}之间的一系列声明. 函数定义也是代码块! 在下面的例子中我们定义了两种变量绑定, x和y, 存在于不同的代码块中. x可以在fn main() {}代码块中获取, 而y只能在内部的代码块中获取:

	fn main() {
		let x: i32 = 17;
		{
			let y: i32 = 3;
			println("The value of x is {} and value of y is {}", x, y);
		}
		println("The value of x is {} and value of y is {}", x, y); // This won't work
	}
	
第一个println!会打印"The value of x is 17 and the value of y is 3", 但实际上该例子并不能被正常编译, 因为第二个println!不能获取到值y, 因为它不在作用域内部. 我们会得到下面的错误:

	$ cargo build
   		Compiling hello v0.1.0 (file:///home/you/projects/hello_world)
	main.rs:7:62: 7:63 error: unresolved name `y`. Did you mean `x`? [E0425]
	main.rs:7     println!("The value of x is {} and value of y is {}", x, y); // This won't work
                                                                       ^
	note: in expansion of format_args!
	<std macros>:2:25: 2:56 note: expansion site
	<std macros>:1:1: 2:62 note: in expansion of print!
	<std macros>:3:1: 3:54 note: expansion site
	<std macros>:1:1: 3:58 note: in expansion of println!
	main.rs:7:5: 7:65 note: expansion site
	main.rs:7:62: 7:63 help: run `rustc --explain E0425` to see a detailed explanation
	error: aborting due to previous error
	Could not compile `hello`.

	To learn more, run the command again with --verbose.
	
此外, 变量绑定也可能被shadow. 这意味着同一个名字上后来的变量绑定在作用域内部会覆盖之前的绑定.

	let x: i32 = 8;
	{
		println!("{}", x); // Prints "8"
		let x = 12;
		println!("{}", x); // Prints "12"
	}
	println!("{}", x); // Prints "8"
	let x = 42;
	println!("{}", x); // Prints "42"
	
Shadowing和可变绑定可能出现在同一个硬币两面, 但是他们是两个完全不同的概念, 不会被交叉使用. 其中shadowing允许我们将一个名字重新绑定到一个不同类型的值上, 也可以改变绑定的不可变性.

	let mut x: i32 = 1;
	x = 7;
	let x = x; // x is now immutable and is bound to 7
	
	let y = 4;
	let y = "I can also bound to text!" // y is now of a different type