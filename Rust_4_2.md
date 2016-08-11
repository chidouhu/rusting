## 4.2 函数
每个Rust程序都至少有一个函数, main函数:

	fn main() {
	}
	
这是最简单的函数声明. 正如前面所提的, fn意味着"这是一个函数", 后面跟着函数名, 两个圆括号表示函数不接受参数, 然后大括号包裹的是函数体. 比如一个名为foo的函数:

	fn foo() {
	}
	
那么如果需要参数怎么办? 这里有一个打印数字的函数:

	fn print_number(x: 32) {
		println!("x is {}", x);
	}
	
这里有一个使用print_number的完整程序:

	fn main() {
		print_number(5);
	}
	
	fn print_number(x: i32) {
		println!("x is {}", x);
	}
	
如你所见, 函数参数和let声明很像: 你需要为参数名添加一个类型.

下面有一个完整的例子, 将两个数字加起来并打印:

	fn main() {
		print_num(5, 6);
	}
	
	fn print_sum(x: i32, y: i32) {
		println!("sum is: {}", x + y);
	}
	
你用逗号将参数隔开, 在函数调用的时候也这么做即可.

和let不一样的是, 你***必须***函数参数的类型. 下面这样的代码不能编译:

	fn print_sum(x, y) {
		println!("sum is: {}", x + y);
	}
	
你会得到下面错误:

	expected one of `!`, `:`, or `@`, found `)`
	fn print_sum(x, y) {
	
这是一个精巧的设计. 当某个语言在整个程序层面上可以推导时(例如Haskell), 通常建议你显示的标明你的类型. 我们认同强制函数声明类型. 尽管在函数体内部允许类型推导. (TODO)

那么返回值怎么办? 这里有一个函数将整型值加一:

	fn add_one(x: i32) -> i32 {
		x + 1
	}
	
Rust函数返回一个值, 你需要在一个"箭头"后面声明这个值的类型. 函数体的最后一行决定了它的返回值. 你会注意到这里少了一个分号. 如果我们添加了分号:

	fn add_one(x: i32) -> i32 {
		x + 1;
	}
	
我们会得到下面的错误:

	error: not all control paths return a value
	fn add_one(x: i32) -> i32 {
     	x + 1;
	}

	help: consider removing this semicolon:
     	x + 1;
          	^
          	
这揭示了Rust两个有意思的地方: 这是一种基于expression的语言, 分号的含义与其他基于分号和大括号的语言不一样. 这两点是相关的.

### 表达和声明
Rust是一种基于表达的语言. 只有两种声明, 其它一起都是表达.

那么区别在哪里呢? 表达式会返回一个值, 而声明不会. 这就是为什么我们会得到一个"not all control paths return a value": 声明x + 1;并不会返回一个值. 在Rust中有两种声明: "declaration声明"和"表达式声明", 其他一切都是表达. 我们先讨论declaration声明.

在某些语言中, 变量绑定可以用表达式来写, 而不是声明. 比如Ruby:

	x = y = 5;
	
然而在Rust中, 需要用let来引入绑定, 而不是表达式. 下面的代码会编译失败:

	let x = (let y = 5); // expected identifier, found keyword `let`
	
编译器在告诉我们这里期望看到一个表达式的起始, 而let只能开启一个声明, 而不是表达式.

注意往一个已经绑定了的变量赋值(e.g. y = 5)仍然是一个表达式, 尽管它的值并没有实际用处. 其他语言中赋值的计算结果是一个指定的值(在前面的例子里就是5), 而在Rust里赋值的值是一个空tuple(), 因为指定的值只有一个owner.

	let mut y = 5;
	let x = (y = 6); // x has value `()`, not `6`
	
Rust中的第二种声明是表达式声明. 它的目的是将表达式转为声明. 在实际术语中, Rust的语法期望声明后紧跟其它声明. 这意味着你需要使用分号来分割表达式. 这意味着Rust和很多其它语言很像, 它要求你在每一行的末尾使用分号, 你会在Rust代码的几乎每一行末尾看到分号.

为什么我们说"几乎"呢? 例外就是下面的代码:

	fn add_one(x: i32) -> i32 {
		x + 1
	}
	
我们的函数要求返回一个i32, 如果加上分号, 返回的就是一个(). Rust认为这不是我们想要的, 因此建议我们去掉这个分号.

### 提前return
如果提前return的话会怎么样? Rust确实有一个return关键字:

	fn foo(x: i32) -> i32 {
		return x;
		
		// we never run this code!
		x + 1
	}
	
在函数的最后一行用return也可以, 但是不建议这么写:

	fn foo(x: i32) -> i32 {
		return x + 1;
	}
	
前一种没有return的定义可能看起来有些奇怪, 但是看起来更直观.

### 发散函数
Rust有一些特殊的"发散函数"的语法, 这些函数不会return:

	fn diverges() -> ! {
		panic!("This function never returns!");
	}
	
panic!是一个宏, 和前面的println!宏类似. 不同于println!()的是, panic!()会让当前的执行线程crash. 由于该函数会crash, 它永远不会return, 因此有一个类型"!", 称之为"发散".

如果你添加一个主函数调用diverges()并且执行, 你会得到下面输出:

	thread ‘<main>’ panicked at ‘This function never returns!’, hello.rs:2
	
如果你想要更多的信息, 你可以设置RUST_BACKTRACE环境变量来得到backtrace:

	$ RUST_BACKTRACE=1 ./diverges
	thread '<main>' panicked at 'This function never returns!', hello.rs:2
	stack backtrace:
   		1:     0x7f402773a829 - sys::backtrace::write::h0942de78b6c02817K8r
   		2:     0x7f402773d7fc - panicking::on_panic::h3f23f9d0b5f4c91bu9w
   		3:     0x7f402773960e - rt::unwind::begin_unwind_inner::h2844b8c5e81e79558Bw
   		4:     0x7f4027738893 - rt::unwind::begin_unwind::h4375279447423903650
   		5:     0x7f4027738809 - diverges::h2266b4c4b850236beaa
   		6:     0x7f40277389e5 - main::h19bb1149c2f00ecfBaa
   		7:     0x7f402773f514 - rt::unwind::try::try_fn::h13186883479104382231
   		8:     0x7f402773d1d8 - __rust_try
   		9:     0x7f402773f201 - rt::lang_start::ha172a3ce74bb453aK5w
  		10:     0x7f4027738a19 - main
  		11:     0x7f402694ab44 - __libc_start_main
  		12:     0x7f40277386c8 - <unknown>
  		13:                0x0 - <unknown>
  		
如果你想覆盖一个已存在的RUST_BACKTRACE, 这时候不要只是unset这个变量, 把它设置成0可以避免获得backtrace. 所有其它的值(甚至没有值)都会打开backstrace.

RUST_BACKSTRACE也可以和Cargo的run命令一起工作:

	$ RUST_BACKTRACE=1 cargo run
     	Running `target/debug/diverges`
	thread '<main>' panicked at 'This function never returns!', hello.rs:2
	stack backtrace:
   		1:     0x7f402773a829 - sys::backtrace::write::h0942de78b6c02817K8r
   		2:     0x7f402773d7fc - panicking::on_panic::h3f23f9d0b5f4c91bu9w
   		3:     0x7f402773960e - rt::unwind::begin_unwind_inner::h2844b8c5e81e79558Bw
   		4:     0x7f4027738893 - rt::unwind::begin_unwind::h4375279447423903650
   		5:     0x7f4027738809 - diverges::h2266b4c4b850236beaa
   		6:     0x7f40277389e5 - main::h19bb1149c2f00ecfBaa
   		7:     0x7f402773f514 - rt::unwind::try::try_fn::h13186883479104382231
   		8:     0x7f402773d1d8 - __rust_try
   		9:     0x7f402773f201 - rt::lang_start::ha172a3ce74bb453aK5w
  		10:     0x7f4027738a19 - main
  		11:     0x7f402694ab44 - __libc_start_main
  		12:     0x7f40277386c8 - <unknown>
  		13:                0x0 - <unknown>
  		
发散函数可以用于任意类型:

	let x: i32 = diverges();
	let y: String = diverges();
	
### 函数指针
我们也可以创建变量绑定来指向函数:

	let f: fn(i32) -> i32;
	
f是一个变量绑定, 指向某个函数, 该函数接受i32作为参数并返回i32. 例如:

	fn plush_one(i: i32) -> i32 {
		i + 1
	}
	
	// without type inference
	let f: fn(i32) -> i32 = plus_one;
	
	// with type inference
	let f = plus_one;
	
我们也可以用f来调用该函数:

	let six = f(5);