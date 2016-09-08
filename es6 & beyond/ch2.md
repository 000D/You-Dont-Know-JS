# 你不懂JS：ES6与未来
# 第二章：语法

无论多少，如果你曾经写过JS，那么你很可能对它的语法感到十分熟悉。当然有一些奇怪之处，但是总体来讲这是一种与其他语言有很多相似之处的，相当合理而且直接的语法。

然而，ES6增加了好几种需要费些功夫才能习惯的新语法形式。在这一章中，我们将遍历它们来看看葫芦里到底卖的什么药。

**提示：** 在写作本书时，这本书中所讨论的特性中的一些已经被各种浏览器（Firefox，Chrome，等等）实现了，但是有一些仅仅被实现了一部分，而另一些根本就没实现。如果直接尝试这些例子，你的体验可能会夹杂着三种情况。如果是这样，就在转译其中尝试吧，这些特性中的大多数都被那些工具涵盖了。ES6Fiddle（http://www.es6fiddle.net/）是一个了不起的尝试ES6的游乐场，简单易用，它是一个Babel转译器的在线REPL（http://babeljs.io/repl/）。

## Block-Scoped Declarations

你可能知道在JavaScript中变量作用域的基本单位总是`function`。如果你需要创建一个作用域块儿，除了普通的函数声明以外最流行的方法就是立即被调用的函数表达式（IIFE）。例如：

```js
var a = 2;

(function IIFE(){
	var a = 3;
	console.log( a );	// 3
})();

console.log( a );		// 2
```

### `let` Declarations

然而，现在我们可以创建绑定到任意的块儿上的声明了，它（勿庸置疑地）称为*块儿作用域*。这意味着一对`{ .. }`就是我们用来创建一个作用域所需要的全部。`var`总是声明附着在外围函数（或者全局，如果在顶层的话）上的变量，取而代之的是，使用`let`：

```js
var a = 2;

{
	let a = 3;
	console.log( a );	// 3
}

console.log( a );		// 2
```

迄今为止，在JS中使用独立的`{ .. }`块儿不是很常见，也不是惯用模式，但它总是合法的。而且那些来自拥有 *块儿作用域* 语言的开发者将很容易认出这种模式。

我相信使用一个专门的`{ .. }`块儿是创建块儿作用域变量的最佳方法。但是，你应该总是将`let`声明放在块儿的最顶端。如果你有多于一个的声明，我推荐只使用一个`let`。

从文体上说，我甚至喜欢将`let`放在开放的`{`的同一行中，以便更清楚地表示这个块儿的目的仅仅是为了这些变量声明作用域。

```js
{	let a = 2, b, c;
	// ..
}
```

它现在看起来很奇怪，而且不大可能与其他大多数ES6文献中的推荐的文法吻合。但是我的疯狂是有原因的。

这是另一种实验性的（不是标准化的）`let`声明形式，称为`let`块儿，看起来就像这样：

```js
let (a = 2, b, c) {
	// ..
}
```

我称这种形式为 *明确的* 块儿作用域，而与`var`相似的`let`声明形式更像是 *隐含的*，因为它在某种意义上劫持了它所处的`{ .. }`。一般来说开发者们认为 *明确的* 机制要比 *隐含的* 机制更好一些，我主张这种情况就是其中之一。

如果你比较前面两个形式的代码段，它们非常相似，而且我个人认为两种形式都有资格在文体上称为 *明确的* 块儿作用域。不幸的是，两者中最 *明确的* `let (..) { .. }`形式没有被ES6所采用。它可能会在后ES6时代被重新提起，但我想目前为止前者是我们的最佳选择。

为了增强对`let ..`声明的 *隐含* 性质的理解，考虑一下这些用法：

```js
let a = 2;

if (a > 1) {
	let b = a * 3;
	console.log( b );		// 6

	for (let i = a; i <= b; i++) {
		let j = i + 10;
		console.log( j );
	}
	// 12 13 14 15 16

	let c = a + b;
	console.log( c );		// 8
}
```

不要回头去看这个代码段，小测验：哪些变量仅存在于`if`语句内部？哪些变量仅存在于`for`循环内部？

答案：`if`语句包含块儿作用域变量`b`和`c`，而`for`循环包含块儿作用域变量`i`和`j`。

你有任何迟疑吗？`i`没有被加入外围的`if`语句的作用域让你惊讶吗？思维上的停顿和疑问 —— 我称之为“思维税” —— 不仅源自于`let`机制对我们来说是新东西的事实，还因为它是 *隐含的*。

还有一个灾难是`let c = ..`声明出现在作用域中太过靠下的地方。传统的被`var`声明的变量，无论它们出现在何处，都会被附着在整个外围的函数作用域中；与此不同的是，`let`声明附着在块儿作用域，而且在它们出现在块儿中之前是不会被初始化的。

在一个`let ..`声明/初始化之前访问一个用`let`声明的变量会导致一个错误，而对于`var`声明来说这个顺序无关紧要（除了文体上的区别）。

考虑如下代码：

```js
{
	console.log( a );	// undefined
	console.log( b );	// ReferenceError!

	var a;
	let b;
}
```

**警告：** 这个由于过早访问被`let`声明的引用而引起的`ReferenceError`在技术上称为一个 *临时死区（Temporal Dead Zone —— TDZ）* 错误 —— 你在访问一个已经被声明但还没被初始化的变量。这将不是我们唯一能够见到TDZ错误的地方 —— 在ES6中它们会在几种地方意外地发生。另外，注意“初始化”并不要求在你的代码中明确地赋一个值，比如`let b;`是完全合法的。一个在声明是没有被赋值的变量被认为已经被赋予了`undefined`值，所以`let b;`和`let b = undefined;`是一样的。无论是否明确赋值，在`let b`语句运行之前你都不能访问`b`。

最后一个坑：对于TDZ变量和未声明的（或声明的！）变量，`typeof`的行为是不同的。例如：

```js
{
	// `a` is not declared
	if (typeof a === "undefined") {
		console.log( "cool" );
	}

	// `b` is declared, but in its TDZ
	if (typeof b === "undefined") {		// ReferenceError!
		// ..
	}

	// ..

	let b;
}
```

`a`没有被声明，所以`typeof`是检查它是否存在的唯一安全的方法。但是`typeof b`抛出了TDZ错误，因为在代码下面很远的地方偶然出现了一个`let b`声明。噢。

现在你应当清楚为什么我坚持认为所有的`let`声明都应该位于它们作用域的顶部了。这完全避免了过偶然早访问的错误。当你观察一个块儿，或任何块儿的开始部分时，它还更 *明确* 地指出这个块儿中含有什么变量。

你的块儿（`if`语句，`while`循环，等等）不一定要与作用域行为共享它们原有的行为。

这种明确性要由你负责，由你用代码规则来维护，这将为你省去许多重构时的头疼和后续的恶果。

**注意：** 更多关于`let`和块儿作用域的信息，参见本系列的 *作用域与闭包* 的第三章。

#### `let` + `for`

我偏好 *明确* 形式的`let`声明块儿，但对此的唯一例外是出现在`for`循环头部的`let`。这里的原因看起来很微妙，但我相信它是更重要的ES6特性中的一个。

考虑如下代码：

```js
var funcs = [];

for (let i = 0; i < 5; i++) {
	funcs.push( function(){
		console.log( i );
	} );
}

funcs[3]();		// 3
```

在`for`头部中的`let i`不仅是为`for`循环本身声明了一个`i`，而且它为循环的每一次迭代都声明了一个新的`i`。这意味着在循环迭代内部创建的闭包都分别闭合着那些在每次迭代中创建的变量，正如你期望的那样。

如果你尝试在这段相同代码的`for`循环头部使用`var i`，那么你会得到`5`而不是`3`，因为在被闭合的外部作用域中只有一个`i`，而不是为每次迭代的函数都有一个`i`被闭合。

你也可以稍稍繁冗地实现相同的东西：

```js
var funcs = [];

for (var i = 0; i < 5; i++) {
	let j = i;
	funcs.push( function(){
		console.log( j );
	} );
}

funcs[3]();		// 3
```

在这里，我们强制地为每次迭代都创建一个新的`j`，然后闭包以相同的方式工作。我喜欢前一种形式；那种额外的特殊能力正是我支持`for(let .. ) ..`形式的原因。可能有人会争论说它有点儿 *隐晦*，但是对我的口味来说，它足够 *明确* 了，也足够有用。

`let`在`for..in`和`for..of`（参见“`for..of`循环”）循环中也以形同的方式工作。

### `const` Declarations

还有另一种需要考虑的块儿作用域声明：`const`，它创建 *常量*。

到底什么是一个常量？它是一个在初始值被设定后就成为只读的变量。考虑如下代码：

```js
{
	const a = 2;
	console.log( a );	// 2

	a = 3;				// TypeError!
}
```

变量持有的值一旦在声明时被设定就不允许改变了。一个`const`声明必须拥有一个明确的初始化。如果想要一个持有`undefined`值的 *常量*，你必须声明`const a = undefined`来得到它。

常量不是一个作用于值本身的制约，而是作用变量对这个值的赋值。换句话说，值不会因为`const`而冻结或不可变，只是它的赋值被冻结了。如果这个值是一个复杂值，比如对象或数组，那么这个值的内容仍然是可以被修改的：

```js
{
	const a = [1,2,3];
	a.push( 4 );
	console.log( a );		// [1,2,3,4]

	a = 42;					// TypeError!
}
```

变量`a`实际上没有持有一个恒定的数组；而是持有一个指向数组的恒定的引用。数组本身可以自由变化。

**警告：** 将一个对象或数组作为常量赋值意味着这个值在常量的词法作用域消失以前是不能够被垃圾回收的，因为指向这个值的引用是永远不能解除的。这可能是你期望的，但如果不是你就要小心！

实质上，`const`声明强制实行了我们许多年来在代码中用文体来表明的东西：我们声明一个名称全由大写字母组成的变量并赋予它某些字面值，我们小心照看它们以使它们永不改变。`var`赋值没有强制性，但是现在`const`赋值上有了，它可以帮你发现不经意的改变。

`const`*可以* 被用于`for`，`for..in`，和`for..of`循环（参见“`for..of`循环”）的变量声明。然而，如果有任何重新赋值的企图，一个错误就会被抛出，例如在`for`循环中常见的`i++`子句。

#### `const` Or Not

有些流传的猜测认为在特定的场景下，与`let`或`var`相比一个`const`可能会被JS引擎进行更多的优化。理论上，引擎可以更容易地知道变量的值/类型将永远不会改变，所以它可以免除一些可能的追踪工作。

无论`const`在这方面是否真的有帮助，还是这仅仅是我们的幻想和直觉，你要做的更重要的决定是你是否打算使用常量的行为。记住：源代码扮演的一个最重要的角色是为了明确地交流你的意图是什么，不仅是与你自己，而且还是与未来的你和其他代码的协作者。

一些开发者喜欢在一开始将每个变量都声明为一个`const`然后当它的值在代码中有必要发生变化的时候将声明放松至一个`let`。这是一个有趣的角度，但是不清楚这是否真正能偶改善代码的可读性或可推理性。

就像许多人认为的那样，它不是一种真正的 *保护*，因为任何后来的想要改变一个`const`值的开发者都可以盲目地将声明从`const`改为`let`。它至多是防止意外的改变。但是同样地，除了我们的直觉和感觉以外，似乎没有客观和明确的标准可以衡量什么构成了“意外”或预防措施。这与类型强制上的思维模式类似。

我的建议：为了避免潜在的令人糊涂的代码，仅将`const`用于那些你有意地并且明显标识为不会改变的变量。换言之，不要为了代码行为而 *依靠* `const`，而是在意图可以被清楚地表明是，将它作为一个表明意图的工具。

### Block-scoped Functions

从ES6开始，发生在块儿内部的函数声明现在被明确规定属于那个块儿的作用域。在ES6之前，语言规范没有要求这一点，但是许多实现不管怎样都是这么做的。所以现在语言规范和现实吻合了。

```js
{
	foo();					// works!

	function foo() {
		// ..
	}
}

foo();						// ReferenceError
```

函数`foo()`在`{ .. }`块儿内部被声明的，由于ES6的原因它是属于那里的块儿作用域的。所以在那个块儿的外部是不可用的。但是还要注意它在块儿里面被提升了，这与早先提到的遭受TDZ错误陷阱的`let`声明是相反的。

如果你以前曾经写过这样的代码，并依赖于老旧的非块儿作用域行为的话，那么函数声明的块儿作用域可能是一个问题：

```js
if (something) {
	function foo() {
		console.log( "1" );
	}
}
else {
	function foo() {
		console.log( "2" );
	}
}

foo();		// ??
```

在前ES6环境下，无论`something`的值是什么`foo()`都将会打印`"2"`，因为两个函数声明被提升到了块儿的顶端，而且总是第二个有效。

在ES6中，最后一行将抛出一个`ReferenceError`。

## Spread/Rest

ES6引入了一个新的`...`操作符，根据你在何处以及如何使用它，它一般被称作 *扩散（spread）* 或 *剩余（rest）* 操作符。让我们看一看：

```js
function foo(x,y,z) {
	console.log( x, y, z );
}

foo( ...[1,2,3] );				// 1 2 3
```

当`...`在一个数组（实际上，是我们将在第三章中讲解的任何的 *可迭代* 对象）前面被使用时，它就将数组“扩散”为它的个别的值。

通常你将会在前面所展示的那样的代码段中看到这种用法，也就是将一个数组扩散为函数调用的一组参数。

```js
foo.apply( null, [1,2,3] );		// 1 2 3
```

但`...`也可以在其他上下文环境中被用于扩散/展开一个值，比如在另一个数组声明内部：

```js
var a = [2,3,4];
var b = [ 1, ...a, 5 ];

console.log( b );					// [1,2,3,4,5]
```

在这种用法中，`...`取代了`concat(..)`，它在这里的行为就像`[1].concat( a, [5] )`。

另一种`...`的用法常见于一种实质上相反的操作；与将值散开不同，`...`将一组值 *收集* 到一个数组中。

```js
function foo(x, y, ...z) {
	console.log( x, y, z );
}

foo( 1, 2, 3, 4, 5 );			// 1 2 [3,4,5]
```

这个代码段中的`...z`实质上是在说：“将 *剩余的* 参数值（如果有的话）收集到一个称为`z`的数组中。” 因为`x`被赋值为`1`，而`y`被赋值为`2`，所以剩余的参数值`3`，`4`，和`5`被收集进了`z`。

当然，如果你没有任何命名参数，`...`会收集所有的参数值：

```js
function foo(...args) {
	console.log( args );
}

foo( 1, 2, 3, 4, 5);			// [1,2,3,4,5]
```

**注意：** 在`foo(..)`函数声明中的`...args`经常因为你向其中收集参数的剩余部分而被称为“剩余参数”。我喜欢使用“收集”这个词，因为它描述了它做什么而不是它包含什么。

这种用法最棒的地方是，它为被废弃了很久的`arguments`数组 —— 实际上它不是一个真正的数组，而是一个类数组对象 —— 提供了一种非常稳固的替代方案。因为`args`（无论你叫它什么 —— 许多人喜欢叫它`r`或者`rest`）是一个真正的数组，我们可以摆脱许多愚蠢的前ES6技巧，我们曾经通过这些技巧尽全力去使`arguments`变成我们可以视之为数组的东西。

考虑如下代码：

```js
// doing things the new ES6 way
function foo(...args) {
	// `args` is already a real array

	// discard first element in `args`
	args.shift();

	// pass along all of `args` as arguments
	// to `console.log(..)`
	console.log( ...args );
}

// doing things the old-school pre-ES6 way
function bar() {
	// turn `arguments` into a real array
	var args = Array.prototype.slice.call( arguments );

	// add some elements on the end
	args.push( 4, 5 );

	// filter out odd numbers
	args = args.filter( function(v){
		return v % 2 == 0;
	} );

	// pass along all of `args` as arguments
	// to `foo(..)`
	foo.apply( null, args );
}

bar( 0, 1, 2, 3 );					// 2 4
```

在函数`foo(..)`声明中的`...args`收集参数值，而在`console.log(..)`调用中的`...args`将它们扩散开。这个例子很好地展示了`...`操作符平行但相反的用途。

除了在函数声明中`...`的用法以外，还有另一种`...`被用于收集值的情况，我们将在本章稍后的“太多，太少，正合适”一节中检视它。

## Default Parameter Values

也许在JavaScript中最常见的惯用法之一就是为函数参数设置默认值。我们多年来一直使用的方法应当看起来很熟悉：

```js
function foo(x,y) {
	x = x || 11;
	y = y || 31;

	console.log( x + y );
}

foo();				// 42
foo( 5, 6 );		// 11
foo( 5 );			// 36
foo( null, 6 );		// 17
```

当然，如果你曾经用过这种模式，你就会知道它既有用又有点儿危险，例如如果你需要能够为其中一个参数传入一个可能被认为是falsy的值。考虑下面的代码：

```js
foo( 0, 42 );		// 53 <-- Oops, not 42
```

为什么？因为`0`是falsy，因此`x || 11`的结果为`11`，而不是直接被传入的`0`。

为了填这个坑，一些人会像这样更加啰嗦地编写检查：

```js
function foo(x,y) {
	x = (x !== undefined) ? x : 11;
	y = (y !== undefined) ? y : 31;

	console.log( x + y );
}

foo( 0, 42 );			// 42
foo( undefined, 6 );	// 17
```

当然，这意味着除了`undefined`以外的任何值都可以直接传入。然而，`undefined`将被假定是这样一种信号，“我没有传入这个值。” 除非你实际需要能够传入`undefined`，它就工作的很好。

在那样的情况下，你可以通过测试参数值是否没有出现在`arguments`数组中，来看它是否实际上被省略了，也许是像这样：

```js
function foo(x,y) {
	x = (0 in arguments) ? x : 11;
	y = (1 in arguments) ? y : 31;

	console.log( x + y );
}

foo( 5 );				// 36
foo( 5, undefined );	// NaN
```

但是在没有能力传入意味着“我省略了这个参数值”的任何种类的值的情况下，你如何才能省略第一个参数值`x`呢？

`foo(,5)`很诱人，但它不是合法的语法。`foo.apply(null,[,5])`看起来应该可以实现这个技巧，但是`apply(..)`的奇怪之处意味着这组参数值将被视为`[undefined,5]`，显然它没有被省略。

如果你深入调查下去，你将发现你只能通过简单地传入比“期望的”参数值个数少的参数值来省略末尾的参数值，但是你不能省略在参数值列表中间或者开头的参数值。这就是不可能的。

这里有一个施用于JavaScript设计的重要原则需要被记住：`undefined`意味着 *丢失*。也就是，在`undefined`和 *丢失* 之间没有区别，至少是就函数参数值而言。

**注意：** 容易令人糊涂的是，JS中有其他的地方不适用这种特殊的设计原则，比如带有空值槽的数组。更多信息参见本系列的 *类型与文法*。

带着所有这些意识，现在我们可以检视在ES6中新增的一种有用的好语法，来简化对丢失的参数值进行默认值的赋值。

```js
function foo(x = 11, y = 31) {
	console.log( x + y );
}

foo();					// 42
foo( 5, 6 );			// 11
foo( 0, 42 );			// 42

foo( 5 );				// 36
foo( 5, undefined );	// 36 <-- `undefined` is missing
foo( 5, null );			// 5  <-- null coerces to `0`

foo( undefined, 6 );	// 17 <-- `undefined` is missing
foo( null, 6 );			// 6  <-- null coerces to `0`
```

注意这些结果，和它们如何暗示了与前面的方式的微妙区别和相似之处。

与常见得多的`x || 11`惯用法相比，在一个函数声明中的`x = 11`更像`x !== undefined ? x : 11`，所以在将你的前ES6代码转换为这种ES6默认参数值语法时要多加小心。

**注意：** 一个剩余/收集参数（参见“扩散/剩余”）不能拥有默认值。所以，虽然`function foo(...vals=[1,2,3]) {`看起来是一种迷人的能力，但它不是合法的语法。有必要的话你需要继续手动实施那种逻辑。

### Default Value Expressions

函数默认值可以比像`31`这样的简单值复杂得多；它们可以是任何合法的表达式，甚至是函数调用：

```js
function bar(val) {
	console.log( "bar called!" );
	return y + val;
}

function foo(x = y + 3, z = bar( x )) {
	console.log( x, z );
}

var y = 5;
foo();								// "bar called"
									// 8 13
foo( 10 );							// "bar called"
									// 10 15
y = 6;
foo( undefined, 10 );				// 9 10
```

如你所见，默认值表达式是被懒惰地求值的，这意味着他们仅在被需要时运行 —— 也就是，当一个参数的参数值被省略或者为`undefined`。

这是一个微妙的细节，但是在一个函数声明中的正式参数是在它们自己的作用域中的（将它想象为一个仅仅围绕在函数声明的`(..)`外面的一个作用域气泡），不是在函数体的作用域中。这意味着在一个默认值表达式中的标识符引用会在首先在正式参数的作用域中查找标识符，然后再查找一个外部作用域。更多信息参见本系列的 *作用域与闭包*。

考虑如下代码：

```js
var w = 1, z = 2;

function foo( x = w + 1, y = x + 1, z = z + 1 ) {
	console.log( x, y, z );
}

foo();					// ReferenceError
```

在默认值表达式`w + 1`中的`w`在正式参数作用域中查找`w`，但没有找到，所以外部作用域的`w`被使用了。接下来，在默认值表达式`x + 1`中的`x`在正式参数的作用域中找到了`x`，而且走运的是`x`已经被初始化了，所以对`y`的赋值工作的很好。

然而，`z + 1`中的`z`找到了一个在那个时刻还没有被初始化的参数变量`z`，所以它绝不会试着在外部作用域中寻找`z`。

正如我们在本章早先的“`let`声明”一节中提到过的那样，ES6拥有一个TDZ，它会防止一个变量在它还没有被初始化的状态下被访问。因此，`z + 1`默认值表达式抛出一个TDZ`ReferenceError`错误。

虽然对于代码的清晰度来说不见得是一个好主意，一个默认值表达式甚至可以是一个内联的函数表达式调用 —— 通常被称为一个立即被调用的函数表达式（IIFE）：

```js
function foo( x =
	(function(v){ return v + 11; })( 31 )
) {
	console.log( x );
}

foo();			// 42
```

一个IIFE（或者任何其他被执行的内联函数表达式）作为默认值表示来说很合适是非常少见的。如果你发现自己试图这么做，那么就退一步再考虑一下！

**警告：** 如果一个IIFE试图访问标识符`x`，而且还没有声明自己的`x`，那么这也将是一个TDZ错误，就像我们刚才讨论的一样。

前一个代码段的默认值表达式是一个IIFE，这是因为它是通过`(31)`在内联时立即被执行。如果我们去掉这一部分，赋予`x`的默认值将会仅仅是一个函数的引用，也许像一个默认的回调。可能有一些情况这种模式将十分有用，比如：

```js
function ajax(url, cb = function(){}) {
	// ..
}

ajax( "http://some.url.1" );
```

这种情况下，我们实质上想在没有其他值被指定时，让默认的`cb`是一个没有操作的空函数。这个函数表达式只是一个函数引用，不是一个调用它自己（在它末尾没有调用的`()`）以达成自己目的的函数。

从JS的早些年开始，就有一个少为人知但是十分有用的奇怪之处可供我们使用：`Function.prototype`本身就是一个没有操作的空函数。这样，这个声明可以是`cb = Function.prototype`而省去内联函数表达式的创建。

## Destructuring

ES6引入了一个称为 *解构* 的新语法特性，如果你将它考虑为 *结构化赋值* 那么它令人困惑的程度可能会小一些。为了理解它的含义，考虑如下代码：

```js
function foo() {
	return [1,2,3];
}

var tmp = foo(),
	a = tmp[0], b = tmp[1], c = tmp[2];

console.log( a, b, c );				// 1 2 3
```

如你所见，我们创建了一个手动赋值：从`foo()`返回的数组中的值到个别的变量`a`，`b`，和`c`，而且这么做我们就（不幸地）需要`tmp`变量。

相似地，我们也可以用对象这么做：

```js
function bar() {
	return {
		x: 4,
		y: 5,
		z: 6
	};
}

var tmp = bar(),
	x = tmp.x, y = tmp.y, z = tmp.z;

console.log( x, y, z );				// 4 5 6
```

属性值`tmp.x`被赋值给变量`x`，`tmp.y`到`y`和`tmp.z`到`z`也一样。

从一个数组中索引的值，或从一个对象的属性中手动赋值可以被认为是 *结构化赋值*。ES6为 *解构* 增加了一种专门的语法，明确地称为 *数组解构* 和 *对象结构*。这种语法消灭了前一个代码段中对变量`tmp`的需要，使它们更加干净。考虑如下代码：

```js
var [ a, b, c ] = foo();
var { x: x, y: y, z: z } = bar();

console.log( a, b, c );				// 1 2 3
console.log( x, y, z );				// 4 5 6
```

你很可能更加习惯于看到像`[a,b,c]`这样的东西出现在一个`=`赋值的右手边的语法，即作为被赋予的值。

解构对称地翻转了这个模式，所以在`=`赋值左手边的`[a,b,c]`被看作是为了将右手边的数组拆解为分离的变量赋值的某种“模式”。

相似地，`{ x: x, y: y, z: z }`指明了一种“模式”把来自于`bar()`的对象拆解为分离的变量赋值。

### Object Property Assignment Pattern

让我们深入前一个代码段中的`{ x: x, .. }`语法。如果属性名与你想要声明的变量名一致，你实际上可以缩写这个语法：

```js
var { x, y, z } = bar();

console.log( x, y, z );				// 4 5 6
```

很酷，对吧？

但`{ x, .. }`是省略了`x: `部分还是省略了` : x`部分？当我们使用这种缩写语法时，我们实际上省略了`x: `部分。这看起来可能不是一个重要的细节，但是一会儿你就会了解它的重要性。

如果你能写缩写形式，那为什么你还要写出更长的形式呢？因为更长的形式事实上允许你将一个属性赋值给一个不同的变量名称，这有时很有用：

```js
var { x: bam, y: baz, z: bap } = bar();

console.log( bam, baz, bap );		// 4 5 6
console.log( x, y, z );				// ReferenceError
```

关于这种对象结构形式有一个微妙但超级重要的怪异之处需要理解。为了展示为什么它可能是一个你需要注意的坑，让我们考虑一下普通对象字面量的“模式”是如何被指定的：

```js
var X = 10, Y = 20;

var o = { a: X, b: Y };

console.log( o.a, o.b );			// 10 20
```

在`{ a: X, b: Y }`中，我们知道`a`是对象属性，而`X`是被赋值给它的源值。换句话说，它的语义模式是`目标: 源`，或者更明显地，`属性别名: 值`。我们能直观地明白这一点，因为它和`=`赋值是一样的，而它的模式就是`目标 = 源`。

然而，当你使用对象解构赋值时 —— 也就是，将看起来像是对象字面量的`{ .. }`语法放在`=`操作符的左手边 —— 你反转了这个`目标: 源`的模式。

回想一下：

```js
var { x: bam, y: baz, z: bap } = bar();
```

这里面对称的模式是`源: 目标`（或者`值: 属性别名`）。`x: bam`意味着属性`x`是源值而`ban`是被赋值的目标变量。换句话说，对象字面量是`target <-- source`，而对象解构赋值是`source --> target`。看到它是如何反转的了吗？

有另外一种考虑这种语法的方式，它可能有助于缓和这种困惑。考虑如下代码：

```js
var aa = 10, bb = 20;

var o = { x: aa, y: bb };
var     { x: AA, y: BB } = o;

console.log( AA, BB );				// 10 20
```

在`{ x: aa, y: bb }`这一行中，`x`和`y`代表对象属性。在`{ x: AA, y: BB }`这一行，`x`和`y` *也* 代表对象属性。

还记得刚才我是如何断言`{ x, .. }`省去了`x: `部分的吗？在这两行中，如果你在代码段中擦掉`x: `和`y: `部分，仅留下`aa, bb`和`AA, BB`，它的效果 —— 从概念上讲，实际上不是 —— 将是从`aa`赋值到`AA`和从`bb`赋值到`BB`。

所以，这种平行性也许有助于解释为什么对于这种ES6特性，语法模式被故意地反转了。

*注意：* 对于解构赋值来说我更喜欢它的语法是`{ AA: x , BB: y }`，因为那样的话可以在两种用法中一致地我们更熟悉的`target: source`模式。唉，我已经被迫训练自己的大脑去习惯这种反转了，就像一些读者也不得不去做的那样。

### Not Just Declarations

至此，我们一直将解构赋值与`var`声明（当然，它们也可以使用`let`和`const`）一起使用，但是解构是一种一般意义上的赋值操作，不仅是一种声明。

考虑如下代码：

```js
var a, b, c, x, y, z;

[a,b,c] = foo();
( { x, y, z } = bar() );

console.log( a, b, c );				// 1 2 3
console.log( x, y, z );				// 4 5 6
```

变量可以是已经被定义好的，然后解构仅仅负责赋值，正如我们已经看到的那样。

**注意：** 特别对于对象解构形式来说，当我们省略了`var`/`let`/`const`声明符时，就必须将整个赋值表达式包含在`()`中，因为如果不这样做的话左手边作为语句第一个元素的`{ .. }`将被视为一个语句块儿而不是一个对象。

事实上，变量表达式（`a`，`y`，等等）不必是一个变量标识符。任何合法的赋值表达式都是允许的。例如：

```js
var o = {};

[o.a, o.b, o.c] = foo();
( { x: o.x, y: o.y, z: o.z } = bar() );

console.log( o.a, o.b, o.c );		// 1 2 3
console.log( o.x, o.y, o.z );		// 4 5 6
```

你甚至可以在解构中使用可计算属性名。考虑如下代码：

```js
var which = "x",
	o = {};

( { [which]: o[which] } = bar() );

console.log( o.x );					// 4
```

`[which]:`的部分是可计算属性名，它的结果是`x` —— 将从当前的对象中拆解出来作为赋值的源头的属性。`o[which]`的部分只是一个普通的对象键引用，作为赋值的目标来说它与`o.x`是等价的。

你可以使用普通的赋值来创建对象映射/变形，例如：

```js
var o1 = { a: 1, b: 2, c: 3 },
	o2 = {};

( { a: o2.x, b: o2.y, c: o2.z } = o1 );

console.log( o2.x, o2.y, o2.z );	// 1 2 3
```

或者你可以将对象映射进一个数组，例如：

```js
var o1 = { a: 1, b: 2, c: 3 },
	a2 = [];

( { a: a2[0], b: a2[1], c: a2[2] } = o1 );

console.log( a2 );					// [1,2,3]
```

或者用另一种方式：

```js
var a1 = [ 1, 2, 3 ],
	o2 = {};

[ o2.a, o2.b, o2.c ] = a1;

console.log( o2.a, o2.b, o2.c );	// 1 2 3
```

或者你可以将一个数组重排到另一个数组中：

```js
var a1 = [ 1, 2, 3 ],
	a2 = [];

[ a2[2], a2[0], a2[1] ] = a1;

console.log( a2 );					// [2,3,1]
```

你甚至可以不使用临时变量来解决传统的“交换两个变量”的问题：

```js
var x = 10, y = 20;

[ y, x ] = [ x, y ];

console.log( x, y );				// 20 10
```

**警告：** 小心：你不应该将声明和赋值混在一起，除非你想要所有的赋值表达式 *也* 被视为声明。否则，你会得到一个语法错误。这就是为什么在刚才的例子中我必须将`var a2 = []`与`[ a2[0], .. ] = ..`解构赋值分开做。尝试`var [ a2[0], .. ] = ..`没有任何意义，因为`a2[0]`不是一个合法的声明标识符；很显然它也不能隐含地创建一个`var a2 = []`声明来使用。

### Repeated Assignments

对象解构形式允许源属性（持有任意值的类型）被罗列多次。例如：

```js
var { a: X, a: Y } = { a: 1 };

X;	// 1
Y;	// 1
```

这意味着你既可以解构一个子对象/数组属性，也可以捕获这个子对象/数组的值本身。考虑如下代码：

```js
var { a: { x: X, x: Y }, a } = { a: { x: 1 } };

X;	// 1
Y;	// 1
a;	// { x: 1 }

( { a: X, a: Y, a: [ Z ] } = { a: [ 1 ] } );

X.push( 2 );
Y[0] = 10;

X;	// [10,2]
Y;	// [10,2]
Z;	// 1
```

关于解构有一句话要提醒：像我们到目前为止的讨论中做的那样，将所有的解构赋值都罗列在单独一行中的方式可能很诱人。然而，一个好得多的主意是使用恰当的缩进将解构赋值的模式分散在多行中 —— 和你在JSON或对象字面量中做的事非常相似 —— 为了可读性。

```js
// harder to read:
var { a: { b: [ c, d ], e: { f } }, g } = obj;

// better:
var {
	a: {
		b: [ c, d ],
		e: { f }
	},
	g
} = obj;
```

记住：**解构的目的不仅是为了少打些字，更多是为了陈述的可读性**

#### Destructuring Assignment Expressions

带有对象或数组解构的赋值表达式的完成值是右手边完整的对象/数组值。考虑如下代码：

```js
var o = { a:1, b:2, c:3 },
	a, b, c, p;

p = { a, b, c } = o;

console.log( a, b, c );			// 1 2 3
p === o;						// true
```

在前面的代码段中，`p`被赋值为对象`o`的引用，而不是`a`，`b`，或`c`的值。数组解构也是一样：

```js
var o = [1,2,3],
	a, b, c, p;

p = [ a, b, c ] = o;

console.log( a, b, c );			// 1 2 3
p === o;						// true
```

通过将这个对象/数组作为完成值传递下去，你可将解构赋值表达式链接在一起：

```js
var o = { a:1, b:2, c:3 },
	p = [4,5,6],
	a, b, c, x, y, z;

( {a} = {b,c} = o );
[x,y] = [z] = p;

console.log( a, b, c );			// 1 2 3
console.log( x, y, z );			// 4 5 4
```

### Too Many, Too Few, Just Enough

对于数组解构赋值和对象解构赋值两者来说，你不必分配所有出现的值。例如：

```js
var [,b] = foo();
var { x, z } = bar();

console.log( b, x, z );				// 2 4 6
```

从`foo()`返回的值`1`和`3`被丢弃了，从`bar()`返回的值`5`也是。

相似地，如果你试着分配比你正在解构/拆解的值要多的值时，它们会如你所想的那样退回到`undefined`：

```js
var [,,c,d] = foo();
var { w, z } = bar();

console.log( c, z );				// 3 6
console.log( d, w );				// undefined undefined
```

这种行为平行地遵循早先提到的“`undefined`意味着丢失”原则。

我们在本章早先检视了`...`操作符，并看到了它有时可以用于将一个数组值扩散为它的分离值，而有时它可以被用于相反的操作：将一组值收集进一个数组。

除了在函数声明中的收集/剩余用法以外，`...`可以在解构赋值中实施相同的行为。为了展示这一点，让我们回想一下本章早先的一个代码段：

```js
var a = [2,3,4];
var b = [ 1, ...a, 5 ];

console.log( b );					// [1,2,3,4,5]
```

我们在这里看到因为`...a`出现在数组`[ .. ]`中值的位置，所以它将`a`扩散开。如果`...a`出现一个数组解构的位置，它会实施收集行为：

```js
var a = [2,3,4];
var [ b, ...c ] = a;

console.log( b, c );				// 2 [3,4]
```

解构赋值`var [ .. ] = a`为了将`a`赋值给在`[ .. ]`中描述的模式而将它扩散开。第一部分的名称`b`对应`a`中的第一个值(`2`)。然后`...c`将剩余的值（`3`和`4`）收集到一个称为`c`的数组中。

**注意：** 我们已经看到`...`是如何与数组一起工作的，但是对象呢？那不是一个ES6特性，但是参看第八章中关于一种可能的“ES6之后”的特性的讨论，它可以让`...`扩散或者收集对象。

### Default Value Assignment

两种形式的解构都可以为赋值提供默认值选项，它使用和早先讨论过的默认函数参数值相似的`=`语法。

考虑如下代码：

```js
var [ a = 3, b = 6, c = 9, d = 12 ] = foo();
var { x = 5, y = 10, z = 15, w = 20 } = bar();

console.log( a, b, c, d );			// 1 2 3 12
console.log( x, y, z, w );			// 4 5 6 20
```

你可以将默认值赋值与前面讲过的赋值表达式语法组合在一起。例如：

```js
var { x, y, z, w: WW = 20 } = bar();

console.log( x, y, z, WW );			// 4 5 6 20
```

如果你在一个解构中使用一个对象或者数组作为默认值，那么要小心不要把自己（或者读你的代码的其他开发者）搞糊涂了。你可能会创建一些非常难理解的代码：

```js
var x = 200, y = 300, z = 100;
var o1 = { x: { y: 42 }, z: { y: z } };

( { y: x = { y: y } } = o1 );
( { z: y = { y: z } } = o1 );
( { x: z = { y: x } } = o1 );
```

你能从这个代码段中看出`x`，`y`和`z`最终是什么值吗？花点儿时间深思熟虑一下，我能想象你的样子。我会终结这个悬念：

```js
console.log( x.y, y.y, z.y );		// 300 100 42
```

这里的要点是：解构很棒也可以很有用，但是如果使用的不明智，它也是一把可以伤人（某人的大脑）的利剑。

### Nested Destructuring

如果你正在解构的值拥有嵌套的对象或数组，你也可以结构这些嵌套的值：

```js
var a1 = [ 1, [2, 3, 4], 5 ];
var o1 = { x: { y: { z: 6 } } };

var [ a, [ b, c, d ], e ] = a1;
var { x: { y: { z: w } } } = o1;

console.log( a, b, c, d, e );		// 1 2 3 4 5
console.log( w );					// 6
```

嵌套的结构可以是一种将对象名称空间扁平哈的简单方法。例如：

```js
var App = {
	model: {
		User: function(){ .. }
	}
};

// instead of:
// var User = App.model.User;

var { model: { User } } = App;
```

### Destructuring Parameters

你能在下面的代码段中发现赋值吗？

```js
function foo(x) {
	console.log( x );
}

foo( 42 );
```

其中的赋值有点儿被隐藏的感觉：当`foo(42)`被执行时`42`（参数值）被赋值给`x`（参数）。如果参数/参数值对是一种赋值，那么按常理说它是一个可以被解构的赋值，对吧？当然！

考虑参数的数组解构：

```js
function foo( [ x, y ] ) {
	console.log( x, y );
}

foo( [ 1, 2 ] );					// 1 2
foo( [ 1 ] );						// 1 undefined
foo( [] );							// undefined undefined
```

参数也可以进行对象解构：

```js
function foo( { x, y } ) {
	console.log( x, y );
}

foo( { y: 1, x: 2 } );				// 2 1
foo( { y: 42 } );					// undefined 42
foo( {} );							// undefined undefined
```

这种技术是命名参数值（一个长期以来被要求的JS特性！）的一种近似解法：对象上的属性映射到被解构的同名参数上。这也意味着我们免费地（在任何位置）得到了可选参数，如你所见，省去“参数”`x`可以如我们期望的那样工作。

当然，先前讨论过的所有解构的种类对于参数解构来说都是可用的，包括嵌套解构，默认值，和其他。解构也可以和其他ES6函数参数功能很好地混合在一起，比如默认参数值和剩余/收集参数。

考虑这些快速的示例（当然这没有穷尽所有可能的种类）：

```js
function f1([ x=2, y=3, z ]) { .. }
function f2([ x, y, ...z], w) { .. }
function f3([ x, y, ...z], ...w) { .. }

function f4({ x: X, y }) { .. }
function f5({ x: X = 10, y = 20 }) { .. }
function f6({ x = 10 } = {}, { y } = { y: 10 }) { .. }
```

为了展示的目的，让我们从这个代码段中取一个例子来检视：

```js
function f3([ x, y, ...z], ...w) {
	console.log( x, y, z, w );
}

f3( [] );							// undefined undefined [] []
f3( [1,2,3,4], 5, 6 );				// 1 2 [3,4] [5,6]
```

这里使用了两个`...`操作符，他们都是将值收集到数组中（`z`和`w`），虽然`...z`是从第一个数组参数值的剩余值中收集，而`...w`是从第一个之后的剩余主参数值中收集的。

#### Destructuring Defaults + Parameter Defaults

有一个微妙的地方你应当注意要特别小心 —— 解构默认值与函数参数默认值的行为之间的不同。例如：

```js
function f6({ x = 10 } = {}, { y } = { y: 10 }) {
	console.log( x, y );
}

f6();								// 10 10
```

首先，看起来我们用两种不同的方法为参数`x`和`y`都声明了默认值`10`。然而，这两种不同的方式会在特定的情况下表现出不同的行为，而且这种区别极其微妙。

考虑如下代码：

```js
f6( {}, {} );						// 10 undefined
```

等等，为什么会这样？十分清楚，如果在第一个参数值的对象中没有一个同名属性被传递，那么命名参数`x`将默认为`10`。

但`y`是`undefined`是怎么回事儿？值`{ y: 10 }`是一个作为函数参数默认值的对象，不是结构默认值。因此，它仅在第二个参数根本没有被传递，或者`undefined`被传递时生效，

在前面的代码段中，我们传递了第二个参数（`{}`），所以默认值`{ y: 10 }`不被使用，而解构`{ y }`会针对被传入的空对象值`{}`发生。

现在，将`{ y } = { y: 10 }`与`{ x = 10 } = {}`比较一下。

对于`x`的使用形式来说，如果第一个函数参数值被省略或者是`undefined`，会默认地使用空对象`{}`。然后，不管在第一个参数值的位置上是什么值 —— 要么是默认的`{}`，要么是你传入的 —— 都会被`{ x = 10 }`解构，它会检查属性`x`是否被找到，如果没有找到（或者是`undefined`），默认值`10`会被设置到命名参数`x`上。

深呼吸。回过头去把最后几段多读几遍。让我们用代码复习一下：

```js
function f6({ x = 10 } = {}, { y } = { y: 10 }) {
	console.log( x, y );
}

f6();								// 10 10
f6( undefined, undefined );			// 10 10
f6( {}, undefined );				// 10 10

f6( {}, {} );						// 10 undefined
f6( undefined, {} );				// 10 undefined

f6( { x: 2 }, { y: 3 } );			// 2 3
```

一般来说，与参数`y`的默认行为比起来，参数`x`的默认行为可能看起来更可取也更合理。因此，理解`{ x = 10 } = {}`形式与`{ y } = { y: 10 }`形式为何与如何不同是很重要的。

如果这仍然有点儿模糊，回头再把它读一遍，并亲自把它玩弄一番。未来的你将会感谢你花了时间把这种非常微妙的晦涩的细节的坑搞明白。

#### Nested Defaults: Destructured and Restructured

Although it may at first be difficult to grasp, an interesting idiom emerges for setting defaults for a nested object's properties: using object destructuring along with what I'd call *restructuring*.

虽然一开始可能很难掌握，但是为一个嵌套的对象的属性设置默认值产生了一种有趣的惯用法：将对象解构与一种我成为 *重构* 的东西一起使用。

Consider a set of defaults in a nested object structure, like the following:

考虑在一个嵌套的对象解构中的一组默认值，就像下面这样：

```js
// taken from: http://es-discourse.com/t/partial-default-arguments/120/7

var defaults = {
	options: {
		remove: true,
		enable: false,
		instance: {}
	},
	log: {
		warn: true,
		error: true
	}
};
```

Now, let's say that you have an object called `config`, which has some of these applied, but perhaps not all, and you'd like to set all the defaults into this object in the missing spots, but not override specific settings already present:

现在，我们假定你有一个称为`config`的对象，它有一些这其中的值，但也许不全有，而且你想要将所有的默认值设置到这个对象的缺失点上，但不覆盖已经存在的特定设置：

```js
var config = {
	options: {
		remove: false,
		instance: null
	}
};
```

You can of course do so manually, as you might have done in the past:

你当然可以手动这样做，就像你可能曾经做过的那样：

```js
config.options = config.options || {};
config.options.remove = (config.options.remove !== undefined) ?
	config.options.remove : defaults.options.remove;
config.options.enable = (config.options.enable !== undefined) ?
	config.options.enable : defaults.options.enable;
...
```

Yuck.
讨厌。

Others may prefer the assign-overwrite approach to this task. You might be tempted by the ES6 `Object.assign(..)` utility (see Chapter 6) to clone the properties first from `defaults` and then overwritten with the cloned properties from `config`, as so:

另一些人可能喜欢用覆盖赋值的方式来完成这个任务。你可能会被ES6的`Object.assign(..)`工具（见第六章）所吸引，来首先克隆`defaults`中的属性然后使用从`config`中克隆的属性覆盖它，像这样：

```js
config = Object.assign( {}, defaults, config );
```

That looks way nicer, huh? But there's a major problem! `Object.assign(..)` is shallow, which means when it copies `defaults.options`, it just copies that object reference, not deep cloning that object's properties to a `config.options` object. `Object.assign(..)` would need to be applied (sort of "recursively") at all levels of your object's tree to get the deep cloning you're expecting.

这看起来好多了，是吧？但是这里有一个重大问题！`Object.assign(..)`是浅拷贝，这意味着当它拷贝`defaults.options`时，它仅仅拷贝这个对象的引用，而不是深度克隆这个对象的属性到一个`config.options`对象。`Object.assign(..)`需要在你的对象树的每一层中实施才能得到你期望的深度克隆。

**Note:** Many JS utility libraries/frameworks provide their own option for deep cloning of an object, but those approaches and their gotchas are beyond our scope to discuss here.

**注意：** 许多JS工具库/框架都为对象的深度克隆提供它们自己的选项，但是那些方式和它们的坑超出了我们在这里的讨论范围。

So let's examine if ES6 object destructuring with defaults can help at all:

那么让我们检视一下ES6的带有默认值的对象解构能否帮到我们：

```js
config.options = config.options || {};
config.log = config.log || {};
({
	options: {
		remove: config.options.remove = defaults.options.remove,
		enable: config.options.enable = defaults.options.enable,
		instance: config.options.instance = defaults.options.instance
	} = {},
	log: {
		warn: config.log.warn = defaults.log.warn,
		error: config.log.error = defaults.log.error
	} = {}
} = config);
```

Not as nice as the false promise of `Object.assign(..)` (being that it's shallow only), but it's better than the manual approach by a fair bit, I think. It is still unfortunately verbose and repetitive, though.

不像`Object.assign(..)`的虚假诺言（因为它只是浅拷贝）那么好，但是我想它要比手动的方式强多了。虽然它仍然很不幸地带有冗余和重复。

The previous snippet's approach works because I'm hacking the destructuring and defaults mechanism to do the property `=== undefined` checks and assignment decisions for me. It's a trick in that I'm destructuring `config` (see the `= config` at the end of the snippet), but I'm reassigning all the destructured values right back into `config`, with the `config.options.enable` assignment references.

前面的代码段的方式可以工作，因为我黑进了结构和默认机制来为我做属性的`=== undefined`检查和赋值的决定。这里的技巧是，我解构了`config`（看看在代码段末尾的`= config`），但是我将所有解构出来的值又立即赋值回`config`，带着`config.options.enable`赋值引用。

Still too much, though. Let's see if we can make anything better.

但还是太多了。让我们看看能否做得更好。

The following trick works best if you know that all the various properties you're destructuring are uniquely named. You can still do it even if that's not the case, but it's not as nice -- you'll have to do the destructuring in stages, or create unique local variables as temporary aliases.

下面的技巧在你知道你正在解构的所有属性的名称都是唯一的情况下工作得最好。但即使不是这样的情况你也仍然可以使用它，只是没有那么好 —— 你将不得不分阶段解构，或者创建独一无二的本地变量作为临时别名。

If we fully destructure all the properties into top-level variables, we can then immediately restructure to reconstitute the original nested object structure.

如果我们将所有的属性完全解构为顶层变量，那么我们就可以立即重构来重组原本的嵌套对象解构。

But all those temporary variables hanging around would pollute scope. So, let's use block scoping (see "Block-Scoped Declarations" earlier in this chapter) with a general `{ }` enclosing block:



```js
// merge `defaults` into `config`
{
	// destructure (with default value assignments)
	let {
		options: {
			remove = defaults.options.remove,
			enable = defaults.options.enable,
			instance = defaults.options.instance
		} = {},
		log: {
			warn = defaults.log.warn,
			error = defaults.log.error
		} = {}
	} = config;

	// restructure
	config = {
		options: { remove, enable, instance },
		log: { warn, error }
	};
}
```

That seems a fair bit nicer, huh?

**Note:** You could also accomplish the scope enclosure with an arrow IIFE instead of the general `{ }` block and `let` declarations. Your destructuring assignments/defaults would be in the parameter list and your restructuring would be the `return` statement in the function body.

The `{ warn, error }` syntax in the restructuring part may look new to you; that's called "concise properties" and we cover it in the next section!

## Object Literal Extensions

ES6 adds a number of important convenience extensions to the humble `{ .. }` object literal.

### Concise Properties

You're certainly familiar with declaring object literals in this form:

```js
var x = 2, y = 3,
	o = {
		x: x,
		y: y
	};
```

If it's always felt redundant to say `x: x` all over, there's good news. If you need to define a property that is the same name as a lexical identifier, you can shorten it from `x: x` to `x`. Consider:

```js
var x = 2, y = 3,
	o = {
		x,
		y
	};
```

### Concise Methods

In a similar spirit to concise properties we just examined, functions attached to properties in object literals also have a concise form, for convenience.

The old way:

```js
var o = {
	x: function(){
		// ..
	},
	y: function(){
		// ..
	}
}
```

And as of ES6:

```js
var o = {
	x() {
		// ..
	},
	y() {
		// ..
	}
}
```

**Warning:** While `x() { .. }` seems to just be shorthand for `x: function(){ .. }`, concise methods have special behaviors that their older counterparts don't; specifically, the allowance for `super` (see "Object `super`" later in this chapter).

Generators (see Chapter 4) also have a concise method form:

```js
var o = {
	*foo() { .. }
};
```

#### Concisely Unnamed

While that convenience shorthand is quite attractive, there's a subtle gotcha to be aware of. To illustrate, let's examine pre-ES6 code like the following, which you might try to refactor to use concise methods:

```js
function runSomething(o) {
	var x = Math.random(),
		y = Math.random();

	return o.something( x, y );
}

runSomething( {
	something: function something(x,y) {
		if (x > y) {
			// recursively call with `x`
			// and `y` swapped
			return something( y, x );
		}

		return y - x;
	}
} );
```

This obviously silly code just generates two random numbers and subtracts the smaller from the bigger. But what's important here isn't what it does, but rather how it's defined. Let's focus on the object literal and function definition, as we see here:

```js
runSomething( {
	something: function something(x,y) {
		// ..
	}
} );
```

Why do we say both `something:` and `function something`? Isn't that redundant? Actually, no, both are needed for different purposes. The property `something` is how we can call `o.something(..)`, sort of like its public name. But the second `something` is a lexical name to refer to the function from inside itself, for recursion purposes.

Can you see why the line `return something(y,x)` needs the name `something` to refer to the function? There's no lexical name for the object, such that it could have said `return o.something(y,x)` or something of that sort.

That's actually a pretty common practice when the object literal does have an identifying name, such as:

```js
var controller = {
	makeRequest: function(..){
		// ..
		controller.makeRequest(..);
	}
};
```

Is this a good idea? Perhaps, perhaps not. You're assuming that the name `controller` will always point to the object in question. But it very well may not -- the `makeRequest(..)` function doesn't control the outer code and so can't force that to be the case. This could come back to bite you.

Others prefer to use `this` to define such things:

```js
var controller = {
	makeRequest: function(..){
		// ..
		this.makeRequest(..);
	}
};
```

That looks fine, and should work if you always invoke the method as `controller.makeRequest(..)`. But you now have a `this` binding gotcha if you do something like:

```js
btn.addEventListener( "click", controller.makeRequest, false );
```

Of course, you can solve that by passing `controller.makeRequest.bind(controller)` as the handler reference to bind the event to. But yuck -- it isn't very appealing.

Or what if your inner `this.makeRequest(..)` call needs to be made from a nested function? You'll have another `this` binding hazard, which people will often solve with the hacky `var self = this`, such as:

```js
var controller = {
	makeRequest: function(..){
		var self = this;

		btn.addEventListener( "click", function(){
			// ..
			self.makeRequest(..);
		}, false );
	}
};
```

More yuck.

**Note:** For more information on `this` binding rules and gotchas, see Chapters 1-2 of the *this & Object Prototypes* title of this series.

OK, what does all this have to do with concise methods? Recall our `something(..)` method definition:

```js
runSomething( {
	something: function something(x,y) {
		// ..
	}
} );
```

The second `something` here provides a super convenient lexical identifier that will always point to the function itself, giving us the perfect reference for recursion, event binding/unbinding, and so on -- no messing around with `this` or trying to use an untrustable object reference.

Great!

So, now we try to refactor that function reference to this ES6 concise method form:

```js
runSomething( {
	something(x,y) {
		if (x > y) {
			return something( y, x );
		}

		return y - x;
	}
} );
```

Seems fine at first glance, except this code will break. The `return something(..)` call will not find a `something` identifier, so you'll get a `ReferenceError`. Oops. But why?

The above ES6 snippet is interpreted as meaning:

```js
runSomething( {
	something: function(x,y){
		if (x > y) {
			return something( y, x );
		}

		return y - x;
	}
} );
```

Look closely. Do you see the problem? The concise method definition implies `something: function(x,y)`. See how the second `something` we were relying on has been omitted? In other words, concise methods imply anonymous function expressions.

Yeah, yuck.

**Note:** You may be tempted to think that `=>` arrow functions are a good solution here, but they're equally insufficient, as they're also anonymous function expressions. We'll cover them in "Arrow Functions" later in this chapter.

The partially redeeming news is that our `something(x,y)` concise method won't be totally anonymous. See "Function Names" in Chapter 7 for information about ES6 function name inference rules. That won't help us for our recursion, but it helps with debugging at least.

So what are we left to conclude about concise methods? They're short and sweet, and a nice convenience. But you should only use them if you're never going to need them to do recursion or event binding/unbinding. Otherwise, stick to your old-school `something: function something(..)` method definitions.

A lot of your methods are probably going to benefit from concise method definitions, so that's great news! Just be careful of the few where there's an un-naming hazard.

#### ES5 Getter/Setter

Technically, ES5 defined getter/setter literals forms, but they didn't seem to get used much, mostly due to the lack of transpilers to handle that new syntax (the only major new syntax added in ES5, really). So while it's not a new ES6 feature, we'll briefly refresh on that form, as it's probably going to be much more useful with ES6 going forward.

Consider:

```js
var o = {
	__id: 10,
	get id() { return this.__id++; },
	set id(v) { this.__id = v; }
}

o.id;			// 10
o.id;			// 11
o.id = 20;
o.id;			// 20

// and:
o.__id;			// 21
o.__id;			// 21 -- still!
```

These getter and setter literal forms are also present in classes; see Chapter 3.

**Warning:** It may not be obvious, but the setter literal must have exactly one declared parameter; omitting it or listing others is illegal syntax. The single required parameter *can* use destructuring and defaults (e.g., `set id({ id: v = 0 }) { .. }`), but the gather/rest `...` is not allowed (`set id(...v) { .. }`).

### Computed Property Names

You've probably been in a situation like the following snippet, where you have one or more property names that come from some sort of expression and thus can't be put into the object literal:

```js
var prefix = "user_";

var o = {
	baz: function(..){ .. }
};

o[ prefix + "foo" ] = function(..){ .. };
o[ prefix + "bar" ] = function(..){ .. };
..
```

ES6 adds a syntax to the object literal definition which allows you to specify an expression that should be computed, whose result is the property name assigned. Consider:

```js
var prefix = "user_";

var o = {
	baz: function(..){ .. },
	[ prefix + "foo" ]: function(..){ .. },
	[ prefix + "bar" ]: function(..){ .. }
	..
};
```

Any valid expression can appear inside the `[ .. ]` that sits in the property name position of the object literal definition.

Probably the most common use of computed property names will be with `Symbol`s (which we cover in "Symbols" later in this chapter), such as:

```js
var o = {
	[Symbol.toStringTag]: "really cool thing",
	..
};
```

`Symbol.toStringTag` is a special built-in value, which we evaluate with the `[ .. ]` syntax, so we can assign the `"really cool thing"` value to the special property name.

Computed property names can also appear as the name of a concise method or a concise generator:

```js
var o = {
	["f" + "oo"]() { .. }	// computed concise method
	*["b" + "ar"]() { .. }	// computed concise generator
};
```

### Setting `[[Prototype]]`

We won't cover prototypes in detail here, so for more information, see the *this & Object Prototypes* title of this series.

Sometimes it will be helpful to assign the `[[Prototype]]` of an object at the same time you're declaring its object literal. The following has been a nonstandard extension in many JS engines for a while, but is standardized as of ES6:

```js
var o1 = {
	// ..
};

var o2 = {
	__proto__: o1,
	// ..
};
```

`o2` is declared with a normal object literal, but it's also `[[Prototype]]`-linked to `o1`. The `__proto__` property name here can also be a string `"__proto__"`, but note that it *cannot* be the result of a computed property name (see the previous section).

`__proto__` is controversial, to say the least. It's a decades-old proprietary extension to JS that is finally standardized, somewhat begrudgingly it seems, in ES6. Many developers feel it shouldn't ever be used. In fact, it's in "Annex B" of ES6, which is the section that lists things JS feels it has to standardize for compatibility reasons only.

**Warning:** Though I'm narrowly endorsing `__proto__` as a key in an object literal definition, I definitely do not endorse using it in its object property form, like `o.__proto__`. That form is both a getter and setter (again for compatibility reasons), but there are definitely better options. See the *this & Object Prototypes* title of this series for more information.

For setting the `[[Prototype]]` of an existing object, you can use the ES6 utility `Object.setPrototypeOf(..)`. Consider:

```js
var o1 = {
	// ..
};

var o2 = {
	// ..
};

Object.setPrototypeOf( o2, o1 );
```

**Note:** We'll discuss `Object` again in Chapter 6. "`Object.setPrototypeOf(..)` Static Function" provides additional details on `Object.setPrototypeOf(..)`. Also see "`Object.assign(..)` Static Function" for another form that relates `o2` prototypically to `o1`.

### Object `super`

`super` is typically thought of as being only related to classes. However, due to JS's classless-objects-with-prototypes nature, `super` is equally effective, and nearly the same in behavior, with plain objects' concise methods.

Consider:

```js
var o1 = {
	foo() {
		console.log( "o1:foo" );
	}
};

var o2 = {
	foo() {
		super.foo();
		console.log( "o2:foo" );
	}
};

Object.setPrototypeOf( o2, o1 );

o2.foo();		// o1:foo
				// o2:foo
```

**Warning:** `super` is only allowed in concise methods, not regular function expression properties. It also is only allowed in `super.XXX` form (for property/method access), not in `super()` form.

The `super` reference in the `o2.foo()` method is locked statically to `o2`, and specifically to the `[[Prototype]]` of `o2`. `super` here would basically be `Object.getPrototypeOf(o2)` -- resolves to `o1` of course -- which is how it finds and calls `o1.foo()`.

For complete details on `super`, see "Classes" in Chapter 3.

## Template Literals

At the very outset of this section, I'm going to have to call out the name of this ES6 feature as being awfully... misleading, depending on your experiences with what the word *template* means.

Many developers think of templates as being reusable renderable pieces of text, such as the capability provided by most template engines (Mustache, Handlebars, etc.). ES6's use of the word *template* would imply something similar, like a way to declare inline template literals that can be re-rendered. However, that's not at all the right way to think about this feature.

So, before we go on, I'm renaming to what it should have been called: *interpolated string literals* (or *interpoliterals* for short).

You're already well aware of declaring string literals with `"` or `'` delimiters, and you also know that these are not *smart strings* (as some languages have), where the contents would be parsed for interpolation expressions.

However, ES6 introduces a new type of string literal, using the `` ` `` backtick as the delimiter. These string literals allow basic string interpolation expressions to be embedded, which are then automatically parsed and evaluated.

Here's the old pre-ES6 way:

```js
var name = "Kyle";

var greeting = "Hello " + name + "!";

console.log( greeting );			// "Hello Kyle!"
console.log( typeof greeting );		// "string"
```

Now, consider the new ES6 way:

```js
var name = "Kyle";

var greeting = `Hello ${name}!`;

console.log( greeting );			// "Hello Kyle!"
console.log( typeof greeting );		// "string"
```

As you can see, we used the `` `..` `` around a series of characters, which are interpreted as a string literal, but any expressions of the form `${..}` are parsed and evaluated inline immediately. The fancy term for such parsing and evaluating is *interpolation* (much more accurate than templating).

The result of the interpolated string literal expression is just a plain old normal string, assigned to the `greeting` variable.

**Warning:** `typeof greeting == "string"` illustrates why it's important not to think of these entities as special template values, as you cannot assign the unevaluated form of the literal to something and reuse it. The `` `..` `` string literal is more like an IIFE in the sense that it's automatically evaluated inline. The result of a `` `..` `` string literal is, simply, just a string.

One really nice benefit of interpolated string literals is they are allowed to split across multiple lines:

```js
var text =
`Now is the time for all good men
to come to the aid of their
country!`;

console.log( text );
// Now is the time for all good men
// to come to the aid of their
// country!
```

The line breaks (newlines) in the interpolated string literal were preserved in the string value.

Unless appearing as explicit escape sequences in the literal value, the value of the `\r` carriage return character (code point `U+000D`) or the value of the `\r\n` carriage return + line feed sequence (code points `U+000D` and `U+000A`) are both normalized to a `\n` line feed character (code point `U+000A`). Don't worry though; this normalization is rare and would likely only happen if copy-pasting text into your JS file.

### Interpolated Expressions

Any valid expression is allowed to appear inside `${..}` in an interpolated string literal, including function calls, inline function expression calls, and even other interpolated string literals!

Consider:

```js
function upper(s) {
	return s.toUpperCase();
}

var who = "reader";

var text =
`A very ${upper( "warm" )} welcome
to all of you ${upper( `${who}s` )}!`;

console.log( text );
// A very WARM welcome
// to all of you READERS!
```

Here, the inner `` `${who}s` `` interpolated string literal was a little bit nicer convenience for us when combining the `who` variable with the `"s"` string, as opposed to `who + "s"`. There will be cases that nesting interpolated string literals is helpful, but be wary if you find yourself doing that kind of thing often, or if you find yourself nesting several levels deep.

If that's the case, the odds are good that your string value production could benefit from some abstractions.

**Warning:** As a word of caution, be very careful about the readability of your code with such new found power. Just like with default value expressions and destructuring assignment expressions, just because you *can* do something doesn't mean you *should* do it. Never go so overboard with new ES6 tricks that your code becomes more clever than you or your other team members.

#### Expression Scope

One quick note about the scope that is used to resolve variables in expressions. I mentioned earlier that an interpolated string literal is kind of like an IIFE, and it turns out thinking about it like that explains the scoping behavior as well.

Consider:

```js
function foo(str) {
	var name = "foo";
	console.log( str );
}

function bar() {
	var name = "bar";
	foo( `Hello from ${name}!` );
}

var name = "global";

bar();					// "Hello from bar!"
```

At the moment the `` `..` `` string literal is expressed, inside the `bar()` function, the scope available to it finds `bar()`'s `name` variable with value `"bar"`. Neither the global `name` nor `foo(..)`'s `name` matter. In other words, an interpolated string literal is just lexically scoped where it appears, not dynamically scoped in any way.

### Tagged Template Literals

Again, renaming the feature for sanity sake: *tagged string literals*.

To be honest, this is one of the cooler tricks that ES6 offers. It may seem a little strange, and perhaps not all that generally practical at first. But once you've spent some time with it, tagged string literals may just surprise you in their usefulness.

For example:

```js
function foo(strings, ...values) {
	console.log( strings );
	console.log( values );
}

var desc = "awesome";

foo`Everything is ${desc}!`;
// [ "Everything is ", "!"]
// [ "awesome" ]
```

Let's take a moment to consider what's happening in the previous snippet. First, the most jarring thing that jumps out is ``foo`Everything...`;``. That doesn't look like anything we've seen before. What is it?

It's essentially a special kind of function call that doesn't need the `( .. )`. The *tag* -- the `foo` part before the `` `..` `` string literal -- is a function value that should be called. Actually, it can be any expression that results in a function, even a function call that returns another function, like:

```js
function bar() {
	return function foo(strings, ...values) {
		console.log( strings );
		console.log( values );
	}
}

var desc = "awesome";

bar()`Everything is ${desc}!`;
// [ "Everything is ", "!"]
// [ "awesome" ]
```

But what gets passed to the `foo(..)` function when invoked as a tag for a string literal?

The first argument -- we called it `strings` -- is an array of all the plain strings (the stuff between any interpolated expressions). We get two values in the `strings` array: `"Everything is "` and `"!"`.

For convenience sake in our example, we then gather up all subsequent arguments into an array called `values` using the `...` gather/rest operator (see the "Spread/Rest" section earlier in this chapter), though you could of course have left them as individual named parameters following the `strings` parameter.

The argument(s) gathered into our `values` array are the results of the already-evaluated interpolation expressions found in the string literal. So obviously the only element in `values` in our example is `"awesome"`.

You can think of these two arrays as: the values in `values` are the separators if you were to splice them in between the values in `strings`, and then if you joined everything together, you'd get the complete interpolated string value.

A tagged string literal is like a processing step after the interpolation expressions are evaluated but before the final string value is compiled, allowing you more control over generating the string from the literal.

Typically, the string literal tag function (`foo(..)` in the previous snippets) should compute an appropriate string value and return it, so that you can use the tagged string literal as a value just like untagged string literals:

```js
function tag(strings, ...values) {
	return strings.reduce( function(s,v,idx){
		return s + (idx > 0 ? values[idx-1] : "") + v;
	}, "" );
}

var desc = "awesome";

var text = tag`Everything is ${desc}!`;

console.log( text );			// Everything is awesome!
```

In this snippet, `tag(..)` is a pass-through operation, in that it doesn't perform any special modifications, but just uses `reduce(..)` to loop over and splice/interleave `strings` and `values` together the same way an untagged string literal would have done.

So what are some practical uses? There are many advanced ones that are beyond our scope to discuss here. But here's a simple idea that formats numbers as U.S. dollars (sort of like basic localization):

```js
function dollabillsyall(strings, ...values) {
	return strings.reduce( function(s,v,idx){
		if (idx > 0) {
			if (typeof values[idx-1] == "number") {
				// look, also using interpolated
				// string literals!
				s += `$${values[idx-1].toFixed( 2 )}`;
			}
			else {
				s += values[idx-1];
			}
		}

		return s + v;
	}, "" );
}

var amt1 = 11.99,
	amt2 = amt1 * 1.08,
	name = "Kyle";

var text = dollabillsyall
`Thanks for your purchase, ${name}! Your
product cost was ${amt1}, which with tax
comes out to ${amt2}.`

console.log( text );
// Thanks for your purchase, Kyle! Your
// product cost was $11.99, which with tax
// comes out to $12.95.
```

If a `number` value is encountered in the `values` array, we put `"$"` in front of it and format it to two decimal places with `toFixed(2)`. Otherwise, we let the value pass-through untouched.

#### Raw Strings

In the previous snippets, our tag functions receive the first argument we called `strings`, which is an array. But there's an additional bit of data included: the raw unprocessed versions of all the strings. You can access those raw string values using the `.raw` property, like this:

```js
function showraw(strings, ...values) {
	console.log( strings );
	console.log( strings.raw );
}

showraw`Hello\nWorld`;
// [ "Hello
// World" ]
// [ "Hello\nWorld" ]
```

The raw version of the value preserves the raw escaped `\n` sequence (the `\` and the `n` are separate characters), while the processed version considers it a single newline character. However, the earlier mentioned line-ending normalization is applied to both values.

ES6 comes with a built-in function that can be used as a string literal tag: `String.raw(..)`. It simply passes through the raw versions of the `strings` values:

```js
console.log( `Hello\nWorld` );
// Hello
// World

console.log( String.raw`Hello\nWorld` );
// Hello\nWorld

String.raw`Hello\nWorld`.length;
// 12
```

Other uses for string literal tags included special processing for internationalization, localization, and more!

## Arrow Functions

We've touched on `this` binding complications with functions earlier in this chapter, and they're covered at length in the *this & Object Prototypes* title of this series. It's important to understand the frustrations that `this`-based programming with normal functions brings, because that is the primary motivation for the new ES6 `=>` arrow function feature.

Let's first illustrate what an arrow function looks like, as compared to normal functions:

```js
function foo(x,y) {
	return x + y;
}

// versus

var foo = (x,y) => x + y;
```

The arrow function definition consists of a parameter list (of zero or more parameters, and surrounding `( .. )` if there's not exactly one parameter), followed by the `=>` marker, followed by a function body.

So, in the previous snippet, the arrow function is just the `(x,y) => x + y` part, and that function reference happens to be assigned to the variable `foo`.

The body only needs to be enclosed by `{ .. }` if there's more than one expression, or if the body consists of a non-expression statement. If there's only one expression, and you omit the surrounding `{ .. }`, there's an implied `return` in front of the expression, as illustrated in the previous snippet.

Here's some other arrow function variations to consider:

```js
var f1 = () => 12;
var f2 = x => x * 2;
var f3 = (x,y) => {
	var z = x * 2 + y;
	y++;
	x *= 3;
	return (x + y + z) / 2;
};
```

Arrow functions are *always* function expressions; there is no arrow function declaration. It also should be clear that they are anonymous function expressions -- they have no named reference for the purposes of recursion or event binding/unbinding -- though "Function Names" in Chapter 7 will describe ES6's function name inference rules for debugging purposes.

**Note:** All the capabilities of normal function parameters are available to arrow functions, including default values, destructuring, rest parameters, and so on.

Arrow functions have a nice, shorter syntax, which makes them on the surface very attractive for writing terser code. Indeed, nearly all literature on ES6 (other than the titles in this series) seems to immediately and exclusively adopt the arrow function as "the new function."

It is telling that nearly all examples in discussion of arrow functions are short single statement utilities, such as those passed as callbacks to various utilities. For example:

```js
var a = [1,2,3,4,5];

a = a.map( v => v * 2 );

console.log( a );				// [2,4,6,8,10]
```

In those cases, where you have such inline function expressions, and they fit the pattern of computing a quick calculation in a single statement and returning that result, arrow functions indeed look to be an attractive and lightweight alternative to the more verbose `function` keyword and syntax.

Most people tend to *ooh and aah* at nice terse examples like that, as I imagine you just did!

However, I would caution you that it would seem to me somewhat a misapplication of this feature to use arrow function syntax with otherwise normal, multistatement functions, especially those that would otherwise be naturally expressed as function declarations.

Recall the `dollabillsyall(..)` string literal tag function from earlier in this chapter -- let's change it to use `=>` syntax:

```js
var dollabillsyall = (strings, ...values) =>
	strings.reduce( (s,v,idx) => {
		if (idx > 0) {
			if (typeof values[idx-1] == "number") {
				// look, also using interpolated
				// string literals!
				s += `$${values[idx-1].toFixed( 2 )}`;
			}
			else {
				s += values[idx-1];
			}
		}

		return s + v;
	}, "" );
```

In this example,  the only modifications I made were the removal of `function`, `return`, and some `{ .. }`, and then the insertion of `=>` and a `var`. Is this a significant improvement in the readability of the code? Meh.

I'd actually argue that the lack of `return` and outer `{ .. }` partially obscures the fact that the `reduce(..)` call is the only statement in the `dollabillsyall(..)` function and that its result is the intended result of the call. Also, the trained eye that is so used to hunting for the word `function` in code to find scope boundaries now needs to look for the `=>` marker, which can definitely be harder to find in the thick of the code.

While not a hard-and-fast rule, I'd say that the readability gains from `=>` arrow function conversion are inversely proportional to the length of the function being converted. The longer the function, the less `=>` helps; the shorter the function, the more `=>` can shine.

I think it's probably more sensible and reasonable to adopt `=>` for the places in code where you do need short inline function expressions, but leave your normal-length main functions as is.

### Not Just Shorter Syntax, But `this`

Most of the popular attention toward `=>` has been on saving those precious keystrokes by dropping `function`, `return`, and `{ .. }` from your code.

But there's a big detail we've skipped over so far. I said at the beginning of the section that `=>` functions are closely related to `this` binding behavior. In fact, `=>` arrow functions are *primarily designed* to alter `this` behavior in a specific way, solving a particular and common pain point with `this`-aware coding.

The saving of keystrokes is a red herring, a misleading sideshow at best.

Let's revisit another example from earlier in this chapter:

```js
var controller = {
	makeRequest: function(..){
		var self = this;

		btn.addEventListener( "click", function(){
			// ..
			self.makeRequest(..);
		}, false );
	}
};
```

We used the `var self = this` hack, and then referenced `self.makeRequest(..)`, because inside the callback function we're passing to `addEventListener(..)`, the `this` binding will not be the same as it is in `makeRequest(..)` itself. In other words, because `this` bindings are dynamic, we fall back to the predictability of lexical scope via the `self` variable.

Herein we finally can see the primary design characteristic of `=>` arrow functions. Inside arrow functions, the `this` binding is not dynamic, but is instead lexical. In the previous snippet, if we used an arrow function for the callback, `this` will be predictably what we wanted it to be.

Consider:

```js
var controller = {
	makeRequest: function(..){
		btn.addEventListener( "click", () => {
			// ..
			this.makeRequest(..);
		}, false );
	}
};
```

Lexical `this` in the arrow function callback in the previous snippet now points to the same value as in the enclosing `makeRequest(..)` function. In other words, `=>` is a syntactic stand-in for `var self = this`.

In cases where `var self = this` (or, alternatively, a function `.bind(this)` call) would normally be helpful, `=>` arrow functions are a nicer alternative operating on the same prinicple. Sounds great, right?

Not quite so simple.

If `=>` replaces `var self = this` or `.bind(this)` and it helps, guess what happens if you use `=>` with a `this`-aware function that *doesn't* need `var self = this` to work? You might be able to guess that it's going to mess things up. Yeah.

Consider:

```js
var controller = {
	makeRequest: (..) => {
		// ..
		this.helper(..);
	},
	helper: (..) => {
		// ..
	}
};

controller.makeRequest(..);
```

Although we invoke as `controller.makeRequest(..)`, the `this.helper` reference fails, because `this` here doesn't point to `controller` as it normally would. Where does it point? It lexically inherits `this` from the surrounding scope. In this previous snippet, that's the global scope, where `this` points to the global object. Ugh.

In addition to lexical `this`, arrow functions also have lexical `arguments` -- they don't have their own `arguments` array but instead inherit from their parent -- as well as lexical `super` and `new.target` (see "Classes" in Chapter 3).

So now we can conclude a more nuanced set of rules for when `=>` is appropriate and not:

* If you have a short, single-statement inline function expression, where the only statement is a `return` of some computed value, *and* that function doesn't already make a `this` reference inside it, *and* there's no self-reference (recursion, event binding/unbinding), *and* you don't reasonably expect the function to ever be that way, you can probably safely refactor it to be an `=>` arrow function.
* If you have an inner function expression that's relying on a `var self = this` hack or a `.bind(this)` call on it in the enclosing function to ensure proper `this` binding, that inner function expression can probably safely become an `=>` arrow function.
* If you have an inner function expression that's relying on something like `var args = Array.prototype.slice.call(arguments)` in the enclosing function to make a lexical copy of `arguments`, that inner function expression can probably safely become an `=>` arrow function.
* For everything else -- normal function declarations, longer multistatement function expressions, functions that need a lexical name identifier self-reference (recursion, etc.), and any other function that doesn't fit the previous characteristics -- you should probably avoid `=>` function syntax.

Bottom line: `=>` is about lexical binding of `this`, `arguments`, and `super`. These are intentional features designed to fix some common problems, not bugs, quirks, or mistakes in ES6.

Don't believe any hype that `=>` is primarily, or even mostly, about fewer keystrokes. Whether you save keystrokes or waste them, you should know exactly what you are intentionally doing with every character typed.

**Tip:** If you have a function that for any of these articulated reasons is not a good match for an `=>` arrow function, but it's being declared as part of an object literal, recall from "Concise Methods" earlier in this chapter that there's another option for shorter function syntax.

If you prefer a visual decision chart for how/why to pick an arrow function:

<img src="fig1.png">

## `for..of` Loops

Joining the `for` and `for..in` loops from the JavaScript we're all familiar with, ES6 adds a `for..of` loop, which loops over the set of values produced by an *iterator*.

The value you loop over with `for..of` must be an *iterable*, or it must be a value which can be coerced/boxed to an object (see the *Types & Grammar* title of this series) that is an iterable. An iterable is simply an object that is able to produce an iterator, which the loop then uses.

Let's compare `for..of` to `for..in` to illustrate the difference:

```js
var a = ["a","b","c","d","e"];

for (var idx in a) {
	console.log( idx );
}
// 0 1 2 3 4

for (var val of a) {
	console.log( val );
}
// "a" "b" "c" "d" "e"
```

As you can see, `for..in` loops over the keys/indexes in the `a` array, while `for..of` loops over the values in `a`.

Here's the pre-ES6 version of the `for..of` from that previous snippet:

```js
var a = ["a","b","c","d","e"],
	k = Object.keys( a );

for (var val, i = 0; i < k.length; i++) {
	val = a[ k[i] ];
	console.log( val );
}
// "a" "b" "c" "d" "e"
```

And here's the ES6 but non-`for..of` equivalent, which also gives a glimpse at manually iterating an iterator (see "Iterators" in Chapter 3):

```js
var a = ["a","b","c","d","e"];

for (var val, ret, it = a[Symbol.iterator]();
	(ret = it.next()) && !ret.done;
) {
	val = ret.value;
	console.log( val );
}
// "a" "b" "c" "d" "e"
```

Under the covers, the `for..of` loop asks the iterable for an iterator (using the built-in `Symbol.iterator`; see "Well-Known Symbols" in Chapter 7), then it repeatedly calls the iterator and assigns its produced value to the loop iteration variable.

Standard built-in values in JavaScript that are by default iterables (or provide them) include:

* Arrays
* Strings
* Generators (see Chapter 3)
* Collections / TypedArrays (see Chapter 5)

**Warning:** Plain objects are not by default suitable for `for..of` looping. That's because they don't have a default iterator, which is intentional, not a mistake. However, we won't go any further into those nuanced reasonings here. In "Iterators" in Chapter 3, we'll see how to define iterators for our own objects, which lets `for..of` loop over any object to get a set of values we define.

Here's how to loop over the characters in a primitive string:

```js
for (var c of "hello") {
	console.log( c );
}
// "h" "e" "l" "l" "o"
```

The `"hello"` primitive string value is coerced/boxed to the `String` object wrapper equivalent, which is an iterable by default.

In `for (XYZ of ABC)..`, the `XYZ` clause can either be an assignment expression or a declaration, identical to that same clause in `for` and `for..in` loops. So you can do stuff like this:

```js
var o = {};

for (o.a of [1,2,3]) {
	console.log( o.a );
}
// 1 2 3

for ({x: o.a} of [ {x: 1}, {x: 2}, {x: 3} ]) {
  console.log( o.a );
}
// 1 2 3
```

`for..of` loops can be prematurely stopped, just like other loops, with `break`, `continue`, `return` (if in a function), and thrown exceptions. In any of these cases, the iterator's `return(..)` function is automatically called (if one exists) to let the iterator perform cleanup tasks, if necessary.

**Note:** See "Iterators" in Chapter 3 for more complete coverage on iterables and iterators.

## Regular Expressions

Let's face it: regular expressions haven't changed much in JS in a long time. So it's a great thing that they've finally learned a couple of new tricks in ES6. We'll briefly cover the additions here, but the overall topic of regular expressions is so dense that you'll need to turn to chapters/books dedicated to it (of which there are many!) if you need a refresher.

### Unicode Flag

We'll cover the topic of Unicode in more detail in "Unicode" later in this chapter. Here, we'll just look briefly at the new `u` flag for ES6+ regular expressions, which turns on Unicode matching for that expression.

JavaScript strings are typically interpreted as sequences of 16-bit characters, which correspond to the characters in the *Basic Multilingual Plane (BMP)* (http://en.wikipedia.org/wiki/Plane_%28Unicode%29). But there are many UTF-16 characters that fall outside this range, and so strings may have these multibyte characters in them.

Prior to ES6, regular expressions could only match based on BMP characters, which means that those extended characters were treated as two separate characters for matching purposes. This is often not ideal.

So, as of ES6, the `u` flag tells a regular expression to process a string with the interpretation of Unicode (UTF-16) characters, such that such an extended character will be matched as a single entity.

**Warning:** Despite the name implication, "UTF-16" doesn't strictly mean 16 bits. Modern Unicode uses 21 bits, and standards like UTF-8 and UTF-16 refer roughly to how many bits are used in the representation of a character.

An example (straight from the ES6 specification): 𝄞 (the musical symbol G-clef) is Unicode point U+1D11E (0x1D11E).

If this character appears in a regular expression pattern (like `/𝄞/`), the standard BMP interpretation would be that it's two separate characters (0xD834 and 0xDD1E) to match with. But the new ES6 Unicode-aware mode means that `/𝄞/u` (or the escaped Unicode form `/\u{1D11E}/u`) will match `"𝄞"` in a string as a single matched character.

You might be wondering why this matters? In non-Unicode BMP mode, the pattern is treated as two separate characters, but would still find the match in a string with the `"𝄞"` character in it, as you can see if you try:

```js
/𝄞/.test( "𝄞-clef" );			// true
```

The length of the match is what matters. For example:

```js
/^.-clef/ .test( "𝄞-clef" );		// false
/^.-clef/u.test( "𝄞-clef" );		// true
```

The `^.-clef` in the pattern says to match only a single character at the beginning before the normal `"-clef"` text. In standard BMP mode, the match fails (two characters), but with `u` Unicode mode flagged on, the match succeeds (one character).

It's also important to note that `u` makes quantifiers like `+` and `*` apply to the entire Unicode code point as a single character, not just the *lower surrogate* (aka rightmost half of the symbol) of the character. The same goes for Unicode characters appearing in character classes, like `/[💩-💫]/u`.

**Note:** There's plenty more nitty-gritty details about `u` behavior in regular expressions, which Mathias Bynens (https://twitter.com/mathias) has written extensively about (https://mathiasbynens.be/notes/es6-unicode-regex).

### Sticky Flag

Another flag mode added to ES6 regular expressions is `y`, which is often called "sticky mode." *Sticky* essentially means the regular expression has a virtual anchor at its beginning that keeps it rooted to matching at only the position indicated by the regular expression's `lastIndex` property.

To illustrate, let's consider two regular expressions, the first without sticky mode and the second with:

```js
var re1 = /foo/,
	str = "++foo++";

re1.lastIndex;			// 0
re1.test( str );		// true
re1.lastIndex;			// 0 -- not updated

re1.lastIndex = 4;
re1.test( str );		// true -- ignored `lastIndex`
re1.lastIndex;			// 4 -- not updated
```

Three things to observe about this snippet:

* `test(..)` doesn't pay any attention to `lastIndex`'s value, and always just performs its match from the beginning of the input string.
* Because our pattern does not have a `^` start-of-input anchor, the search for `"foo"` is free to move ahead through the whole string looking for a match.
* `lastIndex` is not updated by `test(..)`.

Now, let's try a sticky mode regular expression:

```js
var re2 = /foo/y,		// <-- notice the `y` sticky flag
	str = "++foo++";

re2.lastIndex;			// 0
re2.test( str );		// false -- "foo" not found at `0`
re2.lastIndex;			// 0

re2.lastIndex = 2;
re2.test( str );		// true
re2.lastIndex;			// 5 -- updated to after previous match

re2.test( str );		// false
re2.lastIndex;			// 0 -- reset after previous match failure
```

And so our new observations about sticky mode:

* `test(..)` uses `lastIndex` as the exact and only position in `str` to look to make a match. There is no moving ahead to look for the match -- it's either there at the `lastIndex` position or not.
* If a match is made, `test(..)` updates `lastIndex` to point to the character immediately following the match. If a match fails, `test(..)` resets `lastIndex` back to `0`.

Normal non-sticky patterns that aren't otherwise `^`-rooted to the start-of-input are free to move ahead in the input string looking for a match. But sticky mode restricts the pattern to matching just at the position of `lastIndex`.

As I suggested at the beginning of this section, another way of looking at this is that `y` implies a virtual anchor at the beginning of the pattern that is relative (aka constrains the start of the match) to exactly the `lastIndex` position.

**Warning:** In previous literature on the topic, it has alternatively been asserted that this behavior is like `y` implying a `^` (start-of-input) anchor in the pattern. This is inaccurate. We'll explain in further detail in "Anchored Sticky" later.

#### Sticky Positioning

It may seem strangely limiting that to use `y` for repeated matches, you have to manually ensure `lastIndex` is in the exact right position, as it has no move-ahead capability for matching.

Here's one possible scenario: if you know that the match you care about is always going to be at a position that's a multiple of a number (e.g., `0`, `10`, `20`, etc.), you can just construct a limited pattern matching what you care about, but then manually set `lastIndex` each time before match to those fixed positions.

Consider:

```js
var re = /f../y,
	str = "foo       far       fad";

str.match( re );		// ["foo"]

re.lastIndex = 10;
str.match( re );		// ["far"]

re.lastIndex = 20;
str.match( re );		// ["fad"]
```

However, if you're parsing a string that isn't formatted in fixed positions like that, figuring out what to set `lastIndex` to before each match is likely going to be untenable.

There's a saving nuance to consider here. `y` requires that `lastIndex` be in the exact position for a match to occur. But it doesn't strictly require that *you* manually set `lastIndex`.

Instead, you can construct your expressions in such a way that they capture in each main match everything before and after the thing you care about, up to right before the next thing you'll care to match.

Because `lastIndex` will set to the next character beyond the end of a match, if you've matched everything up to that point, `lastIndex` will always be in the correct position for the `y` pattern to start from the next time.

**Warning:** If you can't predict the structure of the input string in a sufficiently patterned way like that, this technique may not be suitable and you may not be able to use `y`.

Having structured string input is likely the most practical scenario where `y` will be capable of performing repeated matching throughout a string. Consider:

```js
var re = /\d+\.\s(.*?)(?:\s|$)/y
	str = "1. foo 2. bar 3. baz";

str.match( re );		// [ "1. foo ", "foo" ]

re.lastIndex;			// 7 -- correct position!
str.match( re );		// [ "2. bar ", "bar" ]

re.lastIndex;			// 14 -- correct position!
str.match( re );		// ["3. baz", "baz"]
```

This works because I knew something ahead of time about the structure of the input string: there is always a numeral prefix like `"1. "` before the desired match (`"foo"`, etc.), and either a space after it, or the end of the string (`$` anchor). So the regular expression I constructed captures all of that in each main match, and then I use a matching group `( )` so that the stuff I really care about is separated out for convenience.

After the first match (`"1. foo "`), the `lastIndex` is `7`, which is already the position needed to start the next match, for `"2. bar "`, and so on.

If you're going to use `y` sticky mode for repeated matches, you'll probably want to look for opportunities to have `lastIndex` automatically positioned as we've just demonstrated.

#### Sticky Versus Global

Some readers may be aware that you can emulate something like this `lastIndex`-relative matching with the `g` global match flag and the `exec(..)` method, as so:

```js
var re = /o+./g,		// <-- look, `g`!
	str = "foot book more";

re.exec( str );			// ["oot"]
re.lastIndex;			// 4

re.exec( str );			// ["ook"]
re.lastIndex;			// 9

re.exec( str );			// ["or"]
re.lastIndex;			// 13

re.exec( str );			// null -- no more matches!
re.lastIndex;			// 0 -- starts over now!
```

While it's true that `g` pattern matches with `exec(..)` start their matching from `lastIndex`'s current value, and also update `lastIndex` after each match (or failure), this is not the same thing as `y`'s behavior.

Notice in the previous snippet that `"ook"`, located at position `6`, was matched and found by the second `exec(..)` call, even though at the time, `lastIndex` was `4` (from the end of the previous match). Why? Because as we said earlier, non-sticky matches are free to move ahead in their matching. A sticky mode expression would have failed here, because it would not be allowed to move ahead.

In addition to perhaps undesired move-ahead matching behavior, another downside to just using `g` instead of `y` is that `g` changes the behavior of some matching methods, like `str.match(re)`.

Consider:

```js
var re = /o+./g,		// <-- look, `g`!
	str = "foot book more";

str.match( re );		// ["oot","ook","or"]
```

See how all the matches were returned at once? Sometimes that's OK, but sometimes that's not what you want.

The `y` sticky flag will give you one-at-a-time progressive matching with utilities like `test(..)` and `match(..)`. Just make sure the `lastIndex` is always in the right position for each match!

#### Anchored Sticky

As we warned earlier, it's inaccurate to think of sticky mode as implying a pattern starts with `^`. The `^` anchor has a distinct meaning in regular expressions, which is *not altered* by sticky mode. `^` is an anchor that *always* refers to the beginning of the input, and *is not* in any way relative to `lastIndex`.

Besides poor/inaccurate documentation on this topic, the confusion is unfortunately strengthened further because an older pre-ES6 experiment with sticky mode in Firefox *did* make `^` relative to `lastIndex`, so that behavior has been around for years.

ES6 elected not to do it that way. `^` in a pattern means start-of-input absolutely and only.

As a consequence, a pattern like `/^foo/y` will always and only find a `"foo"` match at the beginning of a string, *if it's allowed to match there*. If `lastIndex` is not `0`, the match will fail. Consider:

```js
var re = /^foo/y,
	str = "foo";

re.test( str );			// true
re.test( str );			// false
re.lastIndex;			// 0 -- reset after failure

re.lastIndex = 1;
re.test( str );			// false -- failed for positioning
re.lastIndex;			// 0 -- reset after failure
```

Bottom line: `y` plus `^` plus `lastIndex > 0` is an incompatible combination that will always cause a failed match.

**Note:** While `y` does not alter the meaning of `^` in any way, the `m` multiline mode *does*, such that `^` means start-of-input *or* start of text after a newline. So, if you combine `y` and `m` flags together for a pattern, you can find multiple `^`-rooted matches in a string. But remember: because it's `y` sticky, you'll have to make sure `lastIndex` is pointing at the correct new line position (likely by matching to the end of the line) each subsequent time, or no subsequent matches will be made.

### Regular Expression `flags`

Prior to ES6, if you wanted to examine a regular expression object to see what flags it had applied, you needed to parse them out -- ironically, probably with another regular expression -- from the content of the `source` property, such as:

```js
var re = /foo/ig;

re.toString();			// "/foo/ig"

var flags = re.toString().match( /\/([gim]*)$/ )[1];

flags;					// "ig"
```

As of ES6, you can now get these values directly, with the new `flags` property:

```js
var re = /foo/ig;

re.flags;				// "gi"
```

It's a small nuance, but the ES6 specification calls for the expression's flags to be listed in this order: `"gimuy"`, regardless of what order the original pattern was specified with. That's the reason for the difference between `/ig` and `"gi"`.

No, the order of flags specified or listed doesn't matter.

Another tweak from ES6 is that the `RegExp(..)` constructor is now `flags`-aware if you pass it an existing regular expression:

```js
var re1 = /foo*/y;
re1.source;							// "foo*"
re1.flags;							// "y"

var re2 = new RegExp( re1 );
re2.source;							// "foo*"
re2.flags;							// "y"

var re3 = new RegExp( re1, "ig" );
re3.source;							// "foo*"
re3.flags;							// "gi"
```

Prior to ES6, the `re3` construction would throw an error, but as of ES6 you can override the flags when duplicating.

## Number Literal Extensions

Prior to ES5, number literals looked like the following -- the octal form was not officially specified, only allowed as an extension that browsers had come to de facto agreement on:

```js
var dec = 42,
	oct = 052,
	hex = 0x2a;
```

**Note:** Though you are specifying a number in different bases, the number's mathematic value is what is stored, and the default output interpretation is always base-10. The three variables in the previous snippet all have the `42` value stored in them.

To further illustrate that `052` was a nonstandard form extension, consider:

```js
Number( "42" );				// 42
Number( "052" );			// 52
Number( "0x2a" );			// 42
```

ES5 continued to permit the browser-extended octal form (including such inconsistencies), except that in strict mode, the octal literal (`052`) form is disallowed. This restriction was done mainly because many developers had the habit (from other languages) of seemingly innocuously prefixing otherwise base-10 numbers with `0`'s for code alignment purposes, and then running into the accidental fact that they'd changed the number value entirely!

ES6 continues the legacy of changes/variations to how number literals outside base-10 numbers can be represented. There's now an official octal form, an amended hexadecimal form, and a brand-new binary form. For web compatibility reasons, the old octal `052` form will continue to be legal (though unspecified) in non-strict mode, but should really never be used anymore.

Here are the new ES6 number literal forms:

```js
var dec = 42,
	oct = 0o52,			// or `0O52` :(
	hex = 0x2a,			// or `0X2a` :/
	bin = 0b101010;		// or `0B101010` :/
```

The only decimal form allowed is base-10. Octal, hexadecimal, and binary are all integer forms.

And the string representations of these forms are all able to be coerced/converted to their number equivalent:

```js
Number( "42" );			// 42
Number( "0o52" );		// 42
Number( "0x2a" );		// 42
Number( "0b101010" );	// 42
```

Though not strictly new to ES6, it's a little-known fact that you can actually go the opposite direction of conversion (well, sort of):

```js
var a = 42;

a.toString();			// "42" -- also `a.toString( 10 )`
a.toString( 8 );		// "52"
a.toString( 16 );		// "2a"
a.toString( 2 );		// "101010"
```

In fact, you can represent a number this way in any base from `2` to `36`, though it'd be rare that you'd go outside the standard bases: 2, 8, 10, and 16.

## Unicode

Let me just say that this section is not an exhaustive everything-you-ever-wanted-to-know-about-Unicode resource. I want to cover what you need to know that's *changing* for Unicode in ES6, but we won't go much deeper than that. Mathias Bynens (http://twitter.com/mathias) has written/spoken extensively and brilliantly about JS and Unicode (see https://mathiasbynens.be/notes/javascript-unicode and http://fluentconf.com/javascript-html-2015/public/content/2015/02/18-javascript-loves-unicode).

The Unicode characters that range from `0x0000` to `0xFFFF` contain all the standard printed characters (in various languages) that you're likely to have seen or interacted with. This group of characters is called the *Basic Multilingual Plane (BMP)*. The BMP even contains fun symbols like this cool snowman: ☃ (U+2603).

There are lots of other extended Unicode characters beyond this BMP set, which range up to `0x10FFFF`. These symbols are often referred to as *astral* symbols, as that's the name given to the set of 16 *planes* (e.g., layers/groupings) of characters beyond the BMP. Examples of astral symbols include 𝄞 (U+1D11E) and 💩 (U+1F4A9).

Prior to ES6, JavaScript strings could specify Unicode characters using Unicode escaping, such as:

```js
var snowman = "\u2603";
console.log( snowman );			// "☃"
```

However, the `\uXXXX` Unicode escaping only supports four hexadecimal characters, so you can only represent the BMP set of characters in this way. To represent an astral character using Unicode escaping prior to ES6, you need to use a *surrogate pair* -- basically two specially calculated Unicode-escaped characters side by side, which JS interprets together as a single astral character:

```js
var gclef = "\uD834\uDD1E";
console.log( gclef );			// "𝄞"
```

As of ES6, we now have a new form for Unicode escaping (in strings and regular expressions), called Unicode *code point escaping*:

```js
var gclef = "\u{1D11E}";
console.log( gclef );			// "𝄞"
```

As you can see, the difference is the presence of the `{ }` in the escape sequence, which allows it to contain any number of hexadecimal characters. Because you only need six to represent the highest possible code point value in Unicode (i.e., 0x10FFFF), this is sufficient.

### Unicode-Aware String Operations

By default, JavaScript string operations and methods are not sensitive to astral symbols in string values. So, they treat each BMP character individually, even the two surrogate halves that make up an otherwise single astral character. Consider:

```js
var snowman = "☃";
snowman.length;					// 1

var gclef = "𝄞";
gclef.length;					// 2
```

So, how do we accurately calculate the length of such a string? In this scenario, the following trick will work:

```js
var gclef = "𝄞";

[...gclef].length;				// 1
Array.from( gclef ).length;		// 1
```

Recall from the "`for..of` Loops" section earlier in this chapter that ES6 strings have built-in iterators. This iterator happens to be Unicode-aware, meaning it will automatically output an astral symbol as a single value. We take advantage of that using the `...` spread operator in an array literal, which creates an array of the string's symbols. Then we just inspect the length of that resultant array. ES6's `Array.from(..)` does basically the same thing as `[...XYZ]`, but we'll cover that utility in detail in Chapter 6.

**Warning:** It should be noted that constructing and exhausting an iterator just to get the length of a string is quite expensive on performance, relatively speaking, compared to what a theoretically optimized native utility/property would do.

Unfortunately, the full answer is not as simple or straightforward. In addition to the surrogate pairs (which the string iterator takes care of), there are special Unicode code points that behave in other special ways, which is much harder to account for. For example, there's a set of code points that modify the previous adjacent character, known as *Combining Diacritical Marks*.

Consider these two string outputs:

```js
console.log( s1 );				// "é"
console.log( s2 );				// "é"
```

They look the same, but they're not! Here's how we created `s1` and `s2`:

```js
var s1 = "\xE9",
	s2 = "e\u0301";
```

As you can probably guess, our previous `length` trick doesn't work with `s2`:

```js
[...s1].length;					// 1
[...s2].length;					// 2
```

So what can we do? In this case, we can perform a *Unicode normalization* on the value before inquiring about its length, using the ES6 `String#normalize(..)` utility (which we'll cover more in Chapter 6):

```js
var s1 = "\xE9",
	s2 = "e\u0301";

s1.normalize().length;			// 1
s2.normalize().length;			// 1

s1 === s2;						// false
s1 === s2.normalize();			// true
```

Essentially, `normalize(..)` takes a sequence like `"e\u0301"` and normalizes it to `"\xE9"`. Normalization can even combine multiple adjacent combining marks if there's a suitable Unicode character they combine to:

```js
var s1 = "o\u0302\u0300",
	s2 = s1.normalize(),
	s3 = "ồ";

s1.length;						// 3
s2.length;						// 1
s3.length;						// 1

s2 === s3;						// true
```

Unfortunately, normalization isn't fully perfect here, either. If you have multiple combining marks modifying a single character, you may not get the length count you'd expect, because there may not be a single defined normalized character that represents the combination of all the marks. For example:

```js
var s1 = "e\u0301\u0330";

console.log( s1 );				// "ḛ́"

s1.normalize().length;			// 2
```

The further you go down this rabbit hole, the more you realize that it's difficult to get one precise definition for "length." What we see visually rendered as a single character -- more precisely called a *grapheme* -- doesn't always strictly relate to a single "character" in the program processing sense.

**Tip:** If you want to see just how deep this rabbit hole goes, check out the "Grapheme Cluster Boundaries" algorithm (http://www.Unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries).

### Character Positioning

Similar to length complications, what does it actually mean to ask, "what is the character at position 2?" The naive pre-ES6 answer comes from `charAt(..)`, which will not respect the atomicity of an astral character, nor will it take into account combining marks.

Consider:

```js
var s1 = "abc\u0301d",
	s2 = "ab\u0107d",
	s3 = "ab\u{1d49e}d";

console.log( s1 );				// "abćd"
console.log( s2 );				// "abćd"
console.log( s3 );				// "ab𝒞d"

s1.charAt( 2 );					// "c"
s2.charAt( 2 );					// "ć"
s3.charAt( 2 );					// "" <-- unprintable surrogate
s3.charAt( 3 );					// "" <-- unprintable surrogate
```

So, is ES6 giving us a Unicode-aware version of `charAt(..)`? Unfortunately, no. At the time of this writing, there's a proposal for such a utility that's under consideration for post-ES6.

But with what we explored in the previous section (and of course with the limitations noted thereof!), we can hack an ES6 answer:

```js
var s1 = "abc\u0301d",
	s2 = "ab\u0107d",
	s3 = "ab\u{1d49e}d";

[...s1.normalize()][2];			// "ć"
[...s2.normalize()][2];			// "ć"
[...s3.normalize()][2];			// "𝒞"
```

**Warning:** Reminder of an earlier warning: constructing and exhausting an iterator each time you want to get at a single character is... very not ideal, performance wise. Let's hope we get a built-in and optimized utility for this soon, post-ES6.

What about a Unicode-aware version of the `charCodeAt(..)` utility? ES6 gives us `codePointAt(..)`:

```js
var s1 = "abc\u0301d",
	s2 = "ab\u0107d",
	s3 = "ab\u{1d49e}d";

s1.normalize().codePointAt( 2 ).toString( 16 );
// "107"

s2.normalize().codePointAt( 2 ).toString( 16 );
// "107"

s3.normalize().codePointAt( 2 ).toString( 16 );
// "1d49e"
```

What about the other direction? A Unicode-aware version of `String.fromCharCode(..)` is ES6's `String.fromCodePoint(..)`:

```js
String.fromCodePoint( 0x107 );		// "ć"

String.fromCodePoint( 0x1d49e );	// "𝒞"
```

So wait, can we just combine `String.fromCodePoint(..)` and `codePointAt(..)` to get a better version of a Unicode-aware `charAt(..)` from earlier? Yep!

```js
var s1 = "abc\u0301d",
	s2 = "ab\u0107d",
	s3 = "ab\u{1d49e}d";

String.fromCodePoint( s1.normalize().codePointAt( 2 ) );
// "ć"

String.fromCodePoint( s2.normalize().codePointAt( 2 ) );
// "ć"

String.fromCodePoint( s3.normalize().codePointAt( 2 ) );
// "𝒞"
```

There's quite a few other string methods we haven't addressed here, including `toUpperCase()`, `toLowerCase()`, `substring(..)`, `indexOf(..)`, `slice(..)`, and a dozen others. None of these have been changed or augmented for full Unicode awareness, so you should be very careful -- probably just avoid them! -- when working with strings containing astral symbols.

There are also several string methods that use regular expressions for their behavior, like `replace(..)` and `match(..)`. Thankfully, ES6 brings Unicode awareness to regular expressions, as we covered in "Unicode Flag" earlier in this chapter.

OK, there we have it! JavaScript's Unicode string support is significantly better over pre-ES6 (though still not perfect) with the various additions we've just covered.

### Unicode Identifier Names

Unicode can also be used in identifier names (variables, properties, etc.). Prior to ES6, you could do this with Unicode-escapes, like:

```js
var \u03A9 = 42;

// same as: var Ω = 42;
```

As of ES6, you can also use the earlier explained code point escape syntax:

```js
var \u{2B400} = 42;

// same as: var 𫐀 = 42;
```

There's a complex set of rules around exactly which Unicode characters are allowed. Furthermore, some are allowed only if they're not the first character of the identifier name.

**Note:** Mathias Bynens has a great post (https://mathiasbynens.be/notes/javascript-identifiers-es6) on all the nitty-gritty details.

The reasons for using such unusual characters in identifier names are rather rare and academic. You typically won't be best served by writing code that relies on these esoteric capabilities.

## Symbols

With ES6, for the first time in quite a while, a new primitive type has been added to JavaScript: the `symbol`. Unlike the other primitive types, however, symbols don't have a literal form.

Here's how you create a symbol:

```js
var sym = Symbol( "some optional description" );

typeof sym;		// "symbol"
```

Some things to note:

* You cannot and should not use `new` with `Symbol(..)`. It's not a constructor, nor are you producing an object.
* The parameter passed to `Symbol(..)` is optional. If passed, it should be a string that gives a friendly description for the symbol's purpose.
* The `typeof` output is a new value (`"symbol"`) that is the primary way to identify a symbol.

The description, if provided, is solely used for the stringification representation of the symbol:

```js
sym.toString();		// "Symbol(some optional description)"
```

Similar to how primitive string values are not instances of `String`, symbols are also not instances of `Symbol`. If, for some reason, you want to construct a boxed wrapper object form of a symbol value, you can do the following:

```js
sym instanceof Symbol;		// false

var symObj = Object( sym );
symObj instanceof Symbol;	// true

symObj.valueOf() === sym;	// true
```

**Note:** `symObj` in this snippet is interchangeable with `sym`; either form can be used in all places symbols are utilized. There's not much reason to use the boxed wrapper object form (`symObj`) instead of the primitive form (`sym`). Keeping with similar advice for other primitives, it's probably best to prefer `sym` over `symObj`.

The internal value of a symbol itself -- referred to as its `name` -- is hidden from the code and cannot be obtained. You can think of this symbol value as an automatically generated, unique (within your application) string value.

But if the value is hidden and unobtainable, what's the point of having a symbol at all?

The main point of a symbol is to create a string-like value that can't collide with any other value. So, for example, consider using a symbol as a constant representing an event name:

```js
const EVT_LOGIN = Symbol( "event.login" );
```

You'd then use `EVT_LOGIN` in place of a generic string literal like `"event.login"`:

```js
evthub.listen( EVT_LOGIN, function(data){
	// ..
} );
```

The benefit here is that `EVT_LOGIN` holds a value that cannot be duplicated (accidentally or otherwise) by any other value, so it is impossible for there to be any confusion of which event is being dispatched or handled.

**Note:** Under the covers, the `evthub` utility assumed in the previous snippet would almost certainly be using the symbol value from the `EVT_LOGIN` argument directly as the property/key in some internal object (hash) that tracks event handlers. If `evthub` instead needed to use the symbol value as a real string, it would need to explicitly coerce with `String(..)` or `toString()`, as implicit string coercion of symbols is not allowed.

You may use a symbol directly as a property name/key in an object, such as a special property that you want to treat as hidden or meta in usage. It's important to know that although you intend to treat it as such, it is not *actually* a hidden or untouchable property.

Consider this module that implements the *singleton* pattern behavior -- that is, it only allows itself to be created once:

```js
const INSTANCE = Symbol( "instance" );

function HappyFace() {
	if (HappyFace[INSTANCE]) return HappyFace[INSTANCE];

	function smile() { .. }

	return HappyFace[INSTANCE] = {
		smile: smile
	};
}

var me = HappyFace(),
	you = HappyFace();

me === you;			// true
```

The `INSTANCE` symbol value here is a special, almost hidden, meta-like property stored statically on the `HappyFace()` function object.

It could alternatively have been a plain old property like `__instance`, and the behavior would have been identical. The usage of a symbol simply improves the metaprogramming style, keeping this `INSTANCE` property set apart from any other normal properties.

### Symbol Registry

One mild downside to using symbols as in the last few examples is that the `EVT_LOGIN` and `INSTANCE` variables had to be stored in an outer scope (perhaps even the global scope), or otherwise somehow stored in a publicly available location, so that all parts of the code that need to use the symbols can access them.

To aid in organizing code with access to these symbols, you can create symbol values with the *global symbol registry*. For example:

```js
const EVT_LOGIN = Symbol.for( "event.login" );

console.log( EVT_LOGIN );		// Symbol(event.login)
```

And:

```js
function HappyFace() {
	const INSTANCE = Symbol.for( "instance" );

	if (HappyFace[INSTANCE]) return HappyFace[INSTANCE];

	// ..

	return HappyFace[INSTANCE] = { .. };
}
```

`Symbol.for(..)` looks in the global symbol registry to see if a symbol is already stored with the provided description text, and returns it if so. If not, it creates one to return. In other words, the global symbol registry treats symbol values, by description text, as singletons themselves.

But that also means that any part of your application can retrieve the symbol from the registry using `Symbol.for(..)`, as long as the matching description name is used.

Ironically, symbols are basically intended to replace the use of *magic strings* (arbitrary string values given special meaning) in your application. But you precisely use *magic* description string values to uniquely identify/locate them in the global symbol registry!

To avoid accidental collisions, you'll probably want to make your symbol descriptions quite unique. One easy way of doing that is to include prefix/context/namespacing information in them.

For example, consider a utility such as the following:

```js
function extractValues(str) {
	var key = Symbol.for( "extractValues.parse" ),
		re = extractValues[key] ||
			/[^=&]+?=([^&]+?)(?=&|$)/g,
		values = [], match;

	while (match = re.exec( str )) {
		values.push( match[1] );
	}

	return values;
}
```

We use the magic string value `"extractValues.parse"` because it's quite unlikely that any other symbol in the registry would ever collide with that description.

If a user of this utility wants to override the parsing regular expression, they can also use the symbol registry:

```js
extractValues[Symbol.for( "extractValues.parse" )] =
	/..some pattern../g;

extractValues( "..some string.." );
```

Aside from the assistance the symbol registry provides in globally storing these values, everything we're seeing here could have been done by just actually using the magic string `"extractValues.parse"` as the key, rather than the symbol. The improvements exist at the metaprogramming level more than the functional level.

You may have occasion to use a symbol value that has been stored in the registry to look up what description text (key) it's stored under. For example, you may need to signal to another part of your application how to locate a symbol in the registry because you cannot pass the symbol value itself.

You can retrieve a registered symbol's description text (key) using `Symbol.keyFor(..)`:

```js
var s = Symbol.for( "something cool" );

var desc = Symbol.keyFor( s );
console.log( desc );			// "something cool"

// get the symbol from the registry again
var s2 = Symbol.for( desc );

s2 === s;						// true
```

### Symbols as Object Properties

If a symbol is used as a property/key of an object, it's stored in a special way so that the property will not show up in a normal enumeration of the object's properties:

```js
var o = {
	foo: 42,
	[ Symbol( "bar" ) ]: "hello world",
	baz: true
};

Object.getOwnPropertyNames( o );	// [ "foo","baz" ]
```

To retrieve an object's symbol properties:

```js
Object.getOwnPropertySymbols( o );	// [ Symbol(bar) ]
```

This makes it clear that a property symbol is not actually hidden or inaccessible, as you can always see it in the `Object.getOwnPropertySymbols(..)` list.

#### Built-In Symbols

ES6 comes with a number of predefined built-in symbols that expose various meta behaviors on JavaScript object values. However, these symbols are *not* registered in the global symbol registry, as one might expect.

Instead, they're stored as properties on the `Symbol` function object. For example, in the "`for..of`" section earlier in this chapter, we introduced the `Symbol.iterator` value:

```js
var a = [1,2,3];

a[Symbol.iterator];			// native function
```

The specification uses the `@@` prefix notation to refer to the built-in symbols, the most common ones being: `@@iterator`, `@@toStringTag`, `@@toPrimitive`. Several others are defined as well, though they probably won't be used as often.

**Note:** See "Well Known Symbols" in Chapter 7 for detailed information about how these built-in symbols are used for meta programming purposes.

## Review

ES6 adds a heap of new syntax forms to JavaScript, so there's plenty to learn!

Most of these are designed to ease the pain points of common programming idioms, such as setting default values to function parameters and gathering the "rest" of the parameters into an array. Destructuring is a powerful tool for more concisely expressing assignments of values from arrays and nested objects.

While features like `=>` arrow functions appear to also be all about shorter and nicer-looking syntax, they actually have very specific behaviors that you should intentionally use only in appropriate situations.

Expanded Unicode support, new tricks for regular expressions, and even a new primitive `symbol` type round out the syntactic evolution of ES6.
