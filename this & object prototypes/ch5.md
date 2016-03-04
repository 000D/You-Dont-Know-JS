# 你不懂JS: *this* & Object Prototypes
# 第五章: Prototypes（原型）

In Chapters 3 and 4, we mentioned the `[[Prototype]]` chain several times, but haven't said what exactly it is. We will now examine prototypes in detail.

在第三，四章中，我们几次提到了`[[Prototype]]`链，但我们没有讨论它到底什么。现在我们就详细考察一下prototypes（原型）。

**Note:** All of the attempts to emulate class-copy behavior, as described previously in Chapter 4, labeled as variations of "mixins", completely circumvent the `[[Prototype]]` chain mechanism we examine here in this chapter.

**注意：** 所有模拟类拷贝行为的企图，也就是我们在前面第四章描述的内容，都被标记为“mixin”，完全和我们要在本章中考察的`[[Prototype]]`链区别开(TODO)。

## `[[Prototype]]`

Objects in JavaScript have an internal property, denoted in the specification as `[[Prototype]]`, which is simply a reference to another object. Almost all objects are given a non-`null` value for this property, at the time of their creation.

JavaScript中的对象有一个内部属性，意味着在语言规范中的`[[Prototype]]`，它只是一个其他对象的引用。几乎所有的对象在被创建时，它的这个属性都被赋予了一个非`null`值。

**Note:** We will see shortly that it *is* possible for an object to have an empty `[[Prototype]]` linkage, though this is somewhat less common.

**注意：** 我们马上就会看到，一个对象拥有一个空的`[[Prototype]]`链接是 *可能* 的，虽然这有些不寻常。

Consider:

考虑：

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

What is the `[[Prototype]]` reference used for? In Chapter 3, we examined the `[[Get]]` operation that is invoked when you reference a property on an object, such as `myObject.a`. For that default `[[Get]]` operation, the first step is to check if the object itself has a property `a` on it, and if so, it's used.

这个`[[Prototype]]`引用有什么用？在第三种中，我们考察了`[[Get]]`操作，它会在你引用一个对象上的属性时被调用，比如`myObject.a`。对于默认的`[[Get]]`操作来说，第一步就是检查对象本身是否拥有一个`a`属性，如果有，就使用它。

**Note:** ES6 Proxies are outside of our discussion scope in this book (will be covered in a later book in the series!), but everything we discuss here about normal `[[Get]]` and `[[Put]]` behavior does not apply if a `Proxy` is involved.

**注意：** ES6的Proxies（代理）超出了我们要在本书内讨论的范围（将会在本系列的后续书目中涵盖！），但是如果加入`Proxy`，我们在这里讨论的关于普通`[[Get]]`和`[[Put]]`的行为都是不被采用的。

But it's what happens if `a` **isn't** present on `myObject` that brings our attention now to the `[[Prototype]]` link of the object.

但是如果`myObject`上 **不** 存在`a`属性，那么这时我们就将注意力转向对象的`[[Prototype]]`链。

The default `[[Get]]` operation proceeds to follow the `[[Prototype]]` **link** of the object if it cannot find the requested property on the object directly.

如果默认的`[[Get]]`操作不能直接在对象上找到被请求的属性，那么会沿着对象的`[[Prototype]]`**链** 继续处理。

```js
var anotherObject = {
	a: 2
};

// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

myObject.a; // 2
```

**Note:** We will explain what `Object.create(..)` does, and how it operates, shortly. For now, just assume it creates an object with the `[[Prototype]]` linkage we're examining to the object specified.

**注意：** 我们马上就会解释`Object.create(..)`是做什么，如何做的。眼下先假设，它创建了一个对象，这个对象带有一个链到指定的对象的`[[Prototype]]`链接，这个链接就是我们要考察的。

So, we have `myObject` that is now `[[Prototype]]` linked to `anotherObject`. Clearly `myObject.a` doesn't actually exist, but nevertheless, the property access succeeds (being found on `anotherObject` instead) and indeed finds the value `2`.

那么，我们现在让`myObject``[[Prototype]]`链到了`anotherObject`。虽然很明显`myObject.a`实际上不存在，但是属性访问成功了（在`anotherObject`中找到了），而且确实找到了值`2`。

But, if `a` weren't found on `anotherObject` either, its `[[Prototype]]` chain, if non-empty, is again consulted and followed.

但是，如果在`anotherObject`上也没由找到`a`，那么如果它的`[[Prototype]]`链不为空，就继续沿着它查找。

This process continues until either a matching property name is found, or the `[[Prototype]]` chain ends. If no matching property is *ever* found by the end of the chain, the return result from the `[[Get]]` operation is `undefined`.

这个处理持续进行，直到找到名称匹配的属性，或者`[[Prototype]]`链终结。如果在链条的末尾都没有找到匹配的属性，那么`[[Get]]`操作的返回结果为`undefined`。

Similar to this `[[Prototype]]` chain look-up process, if you use a `for..in` loop to iterate over an object, any property that can be reached via its chain (and is also `enumerable` -- see Chapter 3) will be enumerated. If you use the `in` operator to test for the existence of a property on an object, `in` will check the entire chain of the object (regardless of *enumerability*).

和这种`[[Prototype]]`链查询处理相似，如果你使用`for..in`循环迭代一个对象，所有在它的链条上可以到达的（并且是`enumerable`——见第三章）属性都会被枚举。如果你使用`in`操作符来测试一个属性在一个对象上的存在性，`in`将会检查对象的整个链条（不管 *可枚举性*）。

```js
var anotherObject = {
	a: 2
};

// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );

for (var k in myObject) {
	console.log("found: " + k);
}
// found: a

("a" in myObject); // true
```

So, the `[[Prototype]]` chain is consulted, one link at a time, when you perform property look-ups in various fashions. The look-up stops once the property is found or the chain ends.

所以，当你以各种方式进行属性查询时，`[[Prototype]]`就会被查询，每次一个链接。一旦找到属性或者链条终结，这样的查询会才会停止。

### `Object.prototype`

But *where* exactly does the `[[Prototype]]` chain "end"?

但是`[[Prototype]]`链到底在 *哪里* “终结”？

The top-end of every *normal* `[[Prototype]]` chain is the built-in `Object.prototype`. This object includes a variety of common utilities used all over JS, because all normal (built-in, not host-specific extension) objects in JavaScript "descend from" (aka, have at the top of their `[[Prototype]]` chain) the `Object.prototype` object.

每个 *普通* 的`[[Prototype]]`链的最顶端，是内建的`Object.prototype`。这个对象包含各种在整个JS中被使用的共通工具，因为JavaScript中所有普通（内建，而非被宿主环境扩展的）的对象都“衍生自”（也就是，使它们的`[[Prototype]]`顶端为）`Object.prototype`对象。

Some utilities found here you may be familiar with include `.toString()` and `.valueOf()`. In Chapter 3, we introduced another: `.hasOwnProperty(..)`. And yet another function on `Object.prototype` you may not be familiar with, but which we'll address later in this chapter, is `.isPrototypeOf(..)`.

你可能会很熟悉这里发现的一些工具，比如`.toString()`和`.valueOf()`。在第三章中，我们介绍了另一个：`.hasOwnProperty(..)`。还有另外一个你可能不太熟悉，但我们将在这一章里讨论的`Object.prototype`上的函数是`.isPrototypeOf(..)`。

### Setting & Shadowing Properties
### 设置与遮蔽属性

Back in Chapter 3, we mentioned that setting properties on an object was more nuanced than just adding a new property to the object or changing an existing property's value. We will now revisit this situation more completely.

回到第三章，我们提到过在对象上设置属性要比仅仅在对象上添加新属性或改变既存属性的值更加微妙。现在我们将更完整地重温这个话题。

```js
myObject.foo = "bar";
```

If the `myObject` object already has a normal data accessor property called `foo` directly present on it, the assignment is as simple as changing the value of the existing property.

如果`myObject`对象已直接经拥有了普通的名为`foo`的数据访问器属性，那么这个赋值就和改变既存属性的值一样简单。

If `foo` is not already present directly on `myObject`, the `[[Prototype]]` chain is traversed, just like for the `[[Get]]` operation. If `foo` is not found anywhere in the chain, the property `foo` is added directly to `myObject` with the specified value, as expected.

如果`foo`还没有直接存在于`myObject`，`[[Prototype]]`就会被遍历，就像`[[Get]]`操作那样。如果在链条的任何地方都没有找到`foo`，那么就会像我们期望的那样，属性`foo`就以指定的值被直接添加到`myObject`上。

However, if `foo` is already present somewhere higher in the chain, nuanced (and perhaps surprising) behavior can occur with the `myObject.foo = "bar"` assignment. We'll examine that more in just a moment.

然而，如果`foo`已经存在于链条更高层的某处，`myObject.foo = "bar"`赋值就可能会发生微妙的（也许令人吃惊的）行为。我们一会儿就详细考察。

If the property name `foo` ends up both on `myObject` itself and at a higher level of the `[[Prototype]]` chain that starts at `myObject`, this is called *shadowing*. The `foo` property directly on `myObject` *shadows* any `foo` property which appears higher in the chain, because the `myObject.foo` look-up would always find the `foo` property that's lowest in the chain.

如果属性名`foo`同时存在于`myObject`本身和从`myObject`开始的`[[Prototype]]`链的更高层，这样的情况称为 *遮蔽*。直接存在于`myObject`上的`foo`属性会 *遮蔽* 所有出现在链条高层的`foo`属性，因为`myObject.foo`查询总是在寻找链条最底层的`foo`属性。

As we just hinted, shadowing `foo` on `myObject` is not as simple as it may seem. We will now examine three scenarios for the `myObject.foo = "bar"` assignment when `foo` is **not** already on `myObject` directly, but **is** at a higher level of `myObject`'s `[[Prototype]]` chain:

正如我们被暗示的那样，在`myObject`上的`foo`遮蔽没有看起来那么简单。我们现在来考察`myObject.foo = "bar"`赋值的三种场景，当`foo` **不直接存在** 于`myObject`，但 **存在** 于`myObject`的`[[Prototype]]`链的更高层：

1. If a normal data accessor (see Chapter 3) property named `foo` is found anywhere higher on the `[[Prototype]]` chain, **and it's not marked as read-only (`writable:false`)** then a new property called `foo` is added directly to `myObject`, resulting in a **shadowed property**.
1. 如果一个普通的名为`foo`的数据访问属性在`[[Prototype]]`链的高层某处被找到，**而且没有被标记为只读（`writable:false`）**，那么一个名为`foo`的新属性就直接添加到`myObject`上，形成一个 **遮蔽属性**。
2. If a `foo` is found higher on the `[[Prototype]]` chain, but it's marked as **read-only (`writable:false`)**, then both the setting of that existing property as well as the creation of the shadowed property on `myObject` **are disallowed**. If the code is running in `strict mode`, an error will be thrown. Otherwise, the setting of the property value will silently be ignored. Either way, **no shadowing occurs**.
2. 如果一个`foo`在`[[Prototype]]`链的高层某处被找到，但是它被标记为 **只读（`writable:false`）** ，那么设置既存属性和在`myObject`上创建遮蔽属性都是 **不允许** 的。如果代码运行在`strict mode`下，一个错误会被抛出。否则，这个设置属性值的操作会被无声地忽略。不论怎样，**没有发生遮蔽**。
3. If a `foo` is found higher on the `[[Prototype]]` chain and it's a setter (see Chapter 3), then the setter will always be called. No `foo` will be added to (aka, shadowed on) `myObject`, nor will the `foo` setter be redefined.
3. 如果一个`foo`在`[[Prototype]]`链的高层某处被找到，而且它是一个setter（见第三章），那么这个setter总是被调用。没有`foo`会被添加到（也就是遮蔽在）`myObject`上，这个`foo`setter也不会被重定义。

Most developers assume that assignment of a property (`[[Put]]`) will always result in shadowing if the property already exists higher on the `[[Prototype]]` chain, but as you can see, that's only true in one (#1) of the three situations just described.

大多数开发者认为，如果一个属性已经存在于`[[Prototype]]`链的高层，那么对它的赋值（`[[Put]]`）将总是造成遮蔽。但如你所见，这仅在刚才描述的3中场景中的一种（第一种）中是对的。

If you want to shadow `foo` in cases #2 and #3, you cannot use `=` assignment, but must instead use `Object.defineProperty(..)` (see Chapter 3) to add `foo` to `myObject`.

如果你想在第二和第三种情况中遮蔽`foo`，那你就不能使用`=`赋值，而必须使用`Object.defineProperty(..)`（见第三章）将`foo`添加到`myObject`。

**Note:** Case #2 may be the most surprising of the three. The presence of a *read-only* property prevents a property of the same name being implicitly created (shadowed) at a lower level of a `[[Prototype]]` chain. The reason for this restriction is primarily to reinforce the illusion of class-inherited properties. If you think of the `foo` at a higher level of the chain as having been inherited (copied down) to `myObject`, then it makes sense to enforce the non-writable nature of that `foo` property on `myObject`. If you however separate the illusion from the fact, and recognize that no such inheritance copying *actually* occured (see Chapters 4 and 5), it's a little unnatural that `myObject` would be prevented from having a `foo` property just because some other object had a non-writable `foo` on it. It's even stranger that this restriction only applies to `=` assignment, but is not enforced when using `Object.defineProperty(..)`.

**注意：** 第二种情况可能是三种情况中最让人吃惊的了。*只读* 属性的存在会阻止同名属性在`[[Prototype]]`链的低层被创建（遮蔽）。这个限制的主要原因是，增强类继承属性的幻觉。如果你想象位于链条高层的`foo`被继承（拷贝）至`myObject`， 那么在`myObject`上强制`foo`属性不可写就有道理。但如果你将幻觉和现实分开，而且认识到 *实际上* 没有这样的继承拷贝发生（见第四，五章），那么仅因为某些其他的对象上拥有不可写的`foo`，而导致`myObject`不能拥有`foo`属性就有些不自然。而且更奇怪的是，这个限制仅限于`=`赋值，当使用`Object.defineProperty(..)`时不被强制。

Shadowing with **methods** leads to ugly *explicit pseudo-polymorphism* (see Chapter 4) if you need to delegate between them. Usually, shadowing is more complicated and nuanced than it's worth, **so you should try to avoid it if possible**. See Chapter 6 for an alternative design pattern, which among other things discourages shadowing in favor of cleaner alternatives.

如果你需要在方法间进行委托，**方法** 的遮蔽会导致难看的 *显式假想多态*（见第四章）。一般来说，遮蔽与它带来的好处相比太过复杂和微妙了，**所以你应当尽量避免它**。第六章有一种可选的设计模式，它提倡干净而不鼓励遮蔽。

Shadowing can even occur implicitly in subtle ways, so care must be taken if trying to avoid it. Consider:

遮蔽甚至会以微妙的方式隐式地发生，所以要想避免它必须小心。考虑：

```js
var anotherObject = {
	a: 2
};

var myObject = Object.create( anotherObject );

anotherObject.a; // 2
myObject.a; // 2

anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false

myObject.a++; // oops, implicit shadowing!

anotherObject.a; // 2
myObject.a; // 3

myObject.hasOwnProperty( "a" ); // true
```

Though it may appear that `myObject.a++` should (via delegation) look-up and just increment the `anotherObject.a` property itself *in place*, instead the `++` operation corresponds to `myObject.a = myObject.a + 1`. The result is `[[Get]]` looking up `a` property via `[[Prototype]]` to get the current value `2` from `anotherObject.a`, incrementing the value by one, then `[[Put]]` assigning the `3` value to a new shadowed property `a` on `myObject`. Oops!

虽然看起来`myObject.a++`应当（通过委托）查询并 *原地* 递增`anotherObject.a`属性，但是`++`操作符相当于`myObject.a = myObject.a + 1`。结果就是在`[[Prototype]]`上进行`a`的`[[Get]]`查询，从`anotherObject.a`得到当前的值`2`，将这个值递增1，然后将值`3`用`[[Put]]`赋值到`myObject`上的新遮蔽属性`a`上。哦！

Be very careful when dealing with delegated properties that you modify. If you wanted to increment `anotherObject.a`, the only proper way is `anotherObject.a++`.

处理你修改的委托属性是要非常小心。如果你想递增`anotherObject.a`， 那么唯一正确的方法是`anotherObject.a++`。

## "Class"

At this point, you might be wondering: "*Why* does one object need to link to another object?" What's the real benefit? That is a very appropriate question to ask, but we must first understand what `[[Prototype]]` is **not** before we can fully understand and appreciate what it *is* and how it's useful.

现在你可能会想知道：“*为什么* 一个对象需要链到另一个对象？”真正的好处是什么？这是一个很恰当的问题，但在我们能够完全理解和体味它是什么和如何有用之前，我们必须首先理解`[[Prototype]]` **不是** 什么。

As we explained in Chapter 4, in JavaScript, there are no abstract patterns/blueprints for objects called "classes" as there are in class-oriented languages. JavaScript **just** has objects.

正如我么在第四章解释的，在JavaScript中，对于对象来说没有抽象模式/蓝图，即没有面向类的语言中那样的称为类的东西。JavaScript **只** 有对象。

In fact, JavaScript is **almost unique** among languages as perhaps the only language with the right to use the label "object oriented", because it's one of a very short list of languages where an object can be created directly, without a class at all.

实际上，在所有语言中，JavaScript **几乎是独一无二的**，也许是唯一的可以被称为“面向对象”的语言，因为可以根本没有类，而直接创建对象的语言很少，而JavaScript就是其中之一。

In JavaScript, classes can't (being that they don't exist!) describe what an object can do. The object defines its own behavior directly. **There's *just* the object.**

在JavaScript中，类不能（因为根本不存在）描述对象可以做到事。对象直接定义它自己的行为。**这里 *仅有* 对象**

### "Class" Functions

There's a peculiar kind of behavior in JavaScript that has been shamelessly abused for years to *hack* something that *looks* like "classes". We'll examine this approach in detail.

在JavaScript中有一种奇异的行为被无耻地滥用了与多年来 *骇* 成某些 *看起来* 像“类”的东西。我们来仔细看看这种方式。

The peculiar "sort-of class" behavior hinges on a strange characteristic of functions: all functions by default get a public, non-enumerable (see Chapter 3) property on them called `prototype`, which points at an otherwise arbitrary object.

“某种程度的类”这种奇特的行为取决于函数的一个奇怪的性质：所有的函数默认都会得到一个公有的，不可枚举的属性，称为`prototype`，它可以指向任意的对象。

```js
function Foo() {
	// ...
}

Foo.prototype; // { }
```

This object is often called "Foo's prototype", because we access it via an unfortunately-named `Foo.prototype` property reference. However, that terminology is hopelessly destined to lead us into confusion, as we'll see shortly. Instead, I will call it "the object formerly known as Foo's prototype". Just kidding. How about: "object arbitrarily labeled 'Foo dot prototype'"?

这个对象经常被称为“Foo的原型”，因为我们通过一个不幸地被命名为`Foo.prototype`的属性引用来访问它。然而，我们马上会看到，这个术语绝望地注定将我们引向困惑。为了取代它，我将它称为“以前被认为是Foo的原型的对象”。只是开个玩笑。“一个被随意标记为‘Foo点儿原型’的对象”，怎么样？

Whatever we call it, what exactly is this object?

不管我们怎么称呼它，这个对象到底是什么？

The most direct way to explain it is that each object created from calling `new Foo()` (see Chapter 2) will end up (somewhat arbitrarily) `[[Prototype]]`-linked to this "Foo dot prototype" object.

解释它的最直接的方法是，每个由调用`new Foo()`（见第二章）而创建的对象将最终（有些随意地）被`[[Prototype]]`链接入这个“Foo点儿原型”对象。

Let's illustrate:

让我们描绘一下：

```js
function Foo() {
	// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

When `a` is created by calling `new Foo()`, one of the things (see Chapter 2 for all *four* steps) that happens is that `a` gets an internal `[[Prototype]]` link to the object that `Foo.prototype` is pointing at.

当通过调用`new Foo()`创建`a`时，会发生的事情之一（见第二章了解所有 *四个* 步骤）是，`a`得到一个内部`[[Prototype]]`链接，此链接链到`Foo.prototype`所指向的对象。

Stop for a moment and ponder the implications of that statement.

停一会来思考一下这句话的含义。

In class-oriented languages, multiple **copies** (aka, "instances") of a class can be made, like stamping something out from a mold. As we saw in Chapter 4, this happens because the process of instantiating (or inheriting from) a class means, "copy the behavior plan from that class into a physical object", and this is done again for each new instance.

在面向类的语言中，可以制造一个类的多个 **拷贝**（即“实例”），就像从模具中冲压出某些东西一样。我们在第四章中看到，这是因为初始化（或者继承）类的处理意味着，“将行为计划从这个类拷贝到物理对象中”，对于每个新实例这都会发生。

But in JavaScript, there are no such copy-actions performed. You don't create multiple instances of a class. You can create multiple objects that `[[Prototype]]` *link* to a common object. But by default, no copying occurs, and thus these objects don't end up totally separate and disconnected from each other, but rather, quite ***linked***.

但是在JavaScript中，没有这样的拷贝动作发生。你不会创建类的多个实例。你可以创建多个对象`[[Prototype]]`连接至一个共通对象。但默认地，没有拷贝发生，如此这些对象彼此间最终不会完全分离和切断关系，而是相当地 ***链接在一起***。

`new Foo()` results in a new object (we called it `a`), and **that** new object `a` is internally `[[Prototype]]` linked to the `Foo.prototype` object.

`new Foo()`得到一个新对象（我们叫他`a`），这个新对象`a`内部地被`[[Prototype]]`链接至`Foo.prototype`对象。

**We end up with two objects, linked to each other.** That's *it*. We didn't instantiate a class. We certainly didn't do any copying of behavior from a "class" into a concrete object. We just caused two objects to be linked to each other.

**结果我们得到两个对象，彼此链接。** 如是而已。我们没有初始化一个对象。当然我们也没有做任何从一个“类”到一个实体对象拷贝。我们只是让两个对象互相链接在一起。

In fact, the secret, which eludes most JS developers, is that the `new Foo()` function calling had really almost nothing *direct* to do with the process of creating the link. **It was sort of an accidental side-effect.** `new Foo()` is an indirect, round-about way to end up with what we want: **a new object linked to another object**.

事实上，这个使大多数JS开发者无法理解的秘密，是因为`new Foo()`函数调用实际上几乎和建立链接的处理没有任何 *直接* 关系。**它是某种偶然的副作用。**`new Foo()`是一个间接的，迂回的方法来得到我们想要的：**一个被链接到另一个对象的对象。**

Can we get what we want in a more *direct* way? **Yes!** The hero is `Object.create(..)`. But we'll get to that in a little bit.

我们能用更直接的方法得到我们想要的吗？**可以！** 这位英雄就是`Object.create(..)`。我们过会儿就谈到它。

#### What's in a name?

In JavaScript, we don't make *copies* from one object ("class") to another ("instance"). We make *links* between objects. For the `[[Prototype]]` mechanism, visually, the arrows move from right to left, and from bottom to top.

在JavaScript中，我们不从一个对象（“类”）向另一个对象（“实例”） *拷贝*。我们在对象之间制造 *链接*。对于`[[Prototype]]`机制，视觉上，箭头的移动方向是从右至左，由下至上。

<img src="fig3.png">

This mechanism is often called "prototypal inheritance" (we'll explore the code in detail shortly), which is commonly said to be the dynamic-language version of "classical inheritance". It's an attempt to piggy-back on the common understanding of what "inheritance" means in the class-oriented world, but *tweak* (**read: pave over**) the understood semantics, to fit dynamic scripting.

这种机制常被称为“prototypal inheritance（原型继承）”（我们和快就用代码探索），它经常被说成是动态语言版的“类继承”。这种说法试图建立在“继承”在面向类世界中的含义的共识上。但是 *弄拧*（**意思是：铲平**） 了语义理解，来适应动态脚本。（TODO）

The word "inheritance" has a very strong meaning (see Chapter 4), with plenty of mental precedent. Merely adding "prototypal" in front to distinguish the *actually nearly opposite* behavior in JavaScript has left in its wake nearly two decades of miry confusion.

先入为主，“继承”这个词有很强烈的含义（见第四章）。仅仅在它前面加入“原型”来区别于JavaScript中 *实际上几乎相反* 的行为，是真相在泥泞般的困惑中沉睡了几乎二十年。（TODO）

I like to say that sticking "prototypal" in front "inheritance" to drastically reverse its actual meaning is like holding an orange in one hand, an apple in the other, and insisting on calling the apple a "red orange". No matter what confusing label I put in front of it, that doesn't change the *fact* that one fruit is an apple and the other is an orange.

我想说，将“原型”贴在“继承”之前很大程度上搞反了它的实际意义，就像一只手拿着一个桔子，另一手拿着一个苹果，而坚持说苹果是一个“红色的桔子”。无论我在它前面放什么糊涂的标签，那都不会改变一个水果是苹果而另一个是桔子的 *事实*。

The better approach is to plainly call an apple an apple -- to use the most accurate and direct terminology. That makes it easier to understand both their similarities and their **many differences**, because we all have a simple, shared understanding of what "apple" means.

更好的方法是直白地将苹果称为苹果——使用最准确的术语。这样能更容易地理解它们的相似之处和 **许多不同之处**，因为我们都对“苹果”的意义有一个简单的，共享的理解。

Because of the confusion and conflation of terms, I believe the label "prototypal inheritance" itself (and trying to mis-apply all its associated class-orientation terminology, like "class", "constructor", "instance", "polymorphism", etc) has done **more harm than good** in explaining how JavaScript's mechanism *really* works.

由于用语的模糊和歧义，我相信，对于解释JavaScript机制真正如何工作来说，“原型继承”这个标签（以及试图错误地应用所有面向类的术语，比如“类”，“构造器”，“实例”，“多态”等）本身带来的 **危害比好处多**。

"Inheritance" implies a *copy* operation, and JavaScript doesn't copy object properties (natively, by default). Instead, JS creates a link between two objects, where one object can essentially *delegate* property/function access to another object. "Delegation" (see Chapter 6) is a much more accurate term for JavaScript's object-linking mechanism.

“继承”意味着 *拷贝* 操作，而JavaScript不拷贝对象属性（原生上，默认地）。取而代之的是，JS在两个对象间建立链接，一个对象实质上可以将对属性/函数的访问 *委托* 到另一个对象上。对于描述JavaScript对象链接机制来说，“委托”是一个准确得多的术语。

Another term which is sometimes thrown around in JavaScript is "differential inheritance". The idea here is that we describe an object's behavior in terms of what is *different* from a more general descriptor. For example, you explain that a car is a kind of vehicle, but one that has exactly 4 wheels, rather than re-describing all the specifics of what makes up a general vehicle (engine, etc).

另一个有时被扔到扔到JavaScript周围的术语是“差分继承”。这里的想法是，我们可以用它与一个更泛化的对象的 *不同* 来描述一个对象的行为。比如，你要解释汽车是一种载具，与其重新描述组成一个一般载具的所有特点，不如只说它有4个轮子。

If you try to think of any given object in JS as the sum total of all behavior that is *available* via delegation, and **in your mind you flatten** all that behavior into one tangible *thing*, then you can (sorta) see how "differential inheritance" might fit.

如果你试着想象，在JS中任何给定的对象都是通过委托可用的所有行为的总和，而且 **在你思维中你扁平化** 所有的行为到一个有形的 *东西* 中，那么你就可以（八九不离十地）看到“差分继承”是如何适应的。

But just like with "prototypal inheritance", "differential inheritance" pretends that your mental model is more important than what is physically happening in the language. It overlooks the fact that object `B` is not actually differentially constructed, but is instead built with specific characteristics defined, alongside "holes" where nothing is defined. It is in these "holes" (gaps in, or lack of, definition) that delegation *can* take over and, on the fly, "fill them in" with delegated behavior.

但正如“原型继承”，“差分继承”假意使你的思维模型比在语言中物理发生的事情更重要。它忽视了这样一个事实：对象`B`实际上不是一个差异结构，而是由一些定义好的特定性质，与一些没有任何定义的“洞”组成的。正是通过这些“洞”，委托可以接管并且，动态地，用委托行为“填补”它们。

The object is not, by native default, flattened into the single differential object, **through copying**, that the mental model of "differential inheritance" implies. As such, "differential inheritance" is just not as natural a fit for describing how JavaScript's `[[Prototype]]` mechanism actually works.

对象不是像“差分继承”的思维模型所暗示的那样，原生默认地，**通过拷贝** 扁平化到一个单独的差异对象中。如此，对于描述JavaScript的`[[Prototype]]`机制如何工作来说，“差分继承”就不是自然合理。

You *can choose* to prefer the "differential inheritance" terminology and mental model, as a matter of taste, but there's no denying the fact that it *only* fits the mental acrobatics in your mind, not the physical behavior in the engine.

你 *可以选择* 偏向“差分继承”这个术语和思维模型，作为个人口味的问题，但是不能否认这个事实：它 *仅仅* 符合你思维中的精神杂耍（TODO），不是引擎的物理行为。

### "Constructors"
### "Constructors"（构造器）

Let's go back to some earlier code:

让我们回到早先的代码：

```js
function Foo() {
	// ...
}

var a = new Foo();
```

What exactly leads us to think `Foo` is a "class"?

到底是什么导致我们认为`Foo`是一个“类”？

For one, we see the use of the `new` keyword, just like class-oriented languages do when they construct class instances. For another, it appears that we are in fact executing a *constructor* method of a class, because `Foo()` is actually a method that gets called, just like how a real class's constructor gets called when you instantiate that class.

其一，我们看到了`new`关键字的使用，就像面向类语言中人们构建类的对象那样。另外，它看起来我们事实上执行了一个类的 *构造器* 方法，因为`Foo()`实际上是个被调用的方法，就像当你初始化一个真实的类时，这个类的构造器被调用的那样。

To further the confusion of "constructor" semantics, the arbitrarily labeled `Foo.prototype` object has another trick up its sleeve. Consider this code:

为了使“构造器”的语义更使人糊涂，被随意贴上标签的`Foo.prototype`对象还有另外一招。考虑这段代码：

```js
function Foo() {
	// ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

The `Foo.prototype` object by default (at declaration time on line 1 of the snippet!) gets a public, non-enumerable (see Chapter 3) property called `.constructor`, and this property is a reference back to the function (`Foo` in this case) that the object is associated with. Moreover, we see that object `a` created by the "constructor" call `new Foo()` *seems* to also have a property on it called `.constructor` which similarly points to "the function which created it".

`Foo.prototype`对象默认地（就在代码段中第一行的声明处！）得到一个公有的，称为`.constructor`的不可枚举（见第三章）属性，而且这个属性回头指向这个对象关联的函数（这里是`Foo`）。另外，我们看到被“构造器”调用`new Foo()`创建的对象`a` *看起来* 也拥有一个称为`.constructor`的属性，也相似地指向“创建它的函数”。

**Note:** This is not actually true. `a` has no `.constructor` property on it, and though `a.constructor` does in fact resolve to the `Foo` function, "constructor" **does not actually mean** "was constructed by", as it appears. We'll explain this strangeness shortly.

**注意：** 这实际上不是真的。`a`上没有`.constructro`属性，而`a.constructor`确实解析成了`Foo`函数，“constructor”并不像它看起来的那样实际意味着“被XX创建”。我们很快就会解释这个奇怪的地方。

Oh, yeah, also... by convention in the JavaScript world, "class"es are named with a capital letter, so the fact that it's `Foo` instead of `foo` is a strong clue that we intend it to be a "class". That's totally obvious to you, right!?

哦，是的，另外……根据JavaScript世界中的惯例，“类”都以大写字母开头的单词命名，所以使用`Foo`而不是`foo`强烈地意味着我们打算让它成为一个“类”。这对你来说太明显了，对吧！？

**Note:** This convention is so strong that many JS linters actually *complain* if you call `new` on a method with a lowercase name, or if we don't call `new` on a function that happens to start with a capital letter. That sort of boggles the mind that we struggle so much to get (fake) "class-orientation" *right* in JavaScript that we create linter rules to ensure we use capital letters, even though the capital letter doesn't mean ** *anything* at all** to the JS engine.

**注意：** 这个惯例是如此强大，以至于如果你在一个小写字母名称的方法上使用`new`调用，或并没有在一个大写字母开头的函数上使用`new`盗用，许多JS语法检查器将会报告错误。这是因为我们如此努力地想要在JavaScript中将（假的）“面向类” *搞对*，所以我们建立了这些语法规则来确保我们使用了大写字母，即便对JS引擎来讲，大写字母根本没有 *任何意义*。

#### Constructor Or Call?
#### 构造器还是调用？

In the above snippet, it's tempting to think that `Foo` is a "constructor", because we call it with `new` and we observe that it "constructs" an object.

上面的代码的段中，我们试图认为`Foo`是一个“构造器”，是因为我们用`new`调用它，而且我们观察到它“构建”了一个对象。

In reality, `Foo` is no more a "constructor" than any other function in your program. Functions themselves are **not** constructors. However, when you put the `new` keyword in front of a normal function call, that makes that function call a "constructor call". In fact, `new` sort of hijacks any normal function and calls it in a fashion that constructs an object, **in addition to whatever else it was going to do**.

在现实中，`Foo`不会比你的程序中的其他任何函数“更像构造器”。函数自身 **不是** 构造器。但是，当你在普通函数调用前面放一个`new`关键字时，这就将函数调用变成了“构造器调用”。事实上，`new`在某种意义上劫持了普通函数将它以另一种方式调用：构建一个对象，**外加它要做的其他任何事**。

For example:

```js
function NothingSpecial() {
	console.log( "Don't mind me!" );
}

var a = new NothingSpecial();
// "Don't mind me!"

a; // {}
```

`NothingSpecial` is just a plain old normal function, but when called with `new`, it *constructs* an object, almost as a side-effect, which we happen to assign to `a`. The **call** was a *constructor call*, but `NothingSpecial` is not, in and of itself, a *constructor*.

`NothingSpecial`仅仅是一个普通的函数，但当用`new`调用时，几乎是一种副作用，它会 *构建* 一个对象，被我赋值到`a`。这个 **调用** 是一个 *构造器调用*，但是`NothingSpecial`本身并不是一个 *构造器*。

In other words, in JavaScript, it's most appropriate to say that a "constructor" is **any function called with the `new` keyword** in front of it.

换句话说，在JavaScript中，更合适的说法是，“构造器”是在它前面 **用`new`关键字调用的函数**。

Functions aren't constructors, but function calls are "constructor calls" if and only if `new` is used.

函数不是构造器，但是当且仅当`new`被使用时，函数调用是一个“构造器调用”。

### Mechanics

Are *those* the only common triggers for ill-fated "class" discussions in JavaScript?

仅仅是这些原因使得JavaScript中关于“类”的讨论变得命运多舛吗？

**Not quite.** JS developers have strived to simulate as much as they can of class-orientation:

**不全是。** JS开发者们努力地尽可能的模拟面向类：

```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

var a = new Foo( "a" );
var b = new Foo( "b" );

a.myName(); // "a"
b.myName(); // "b"
```

This snippet shows two additional "class-orientation" tricks in play:

这段代码展示了另外两种“面向类”的花招：

1. `this.name = name`: adds the `.name` property onto each object (`a` and `b`, respectively; see Chapter 2 about `this` binding), similar to how class instances encapsulate data values.
1. `this.name = name`：在每个对象（分别在`a`和`b`上；参照第二章关于`this`绑定的内容）上添加了`.name`属性，和类的实例包装数据值很相似。

2. `Foo.prototype.myName = ...`: perhaps the more interesting technique, this adds a property (function) to the `Foo.prototype` object. Now, `a.myName()` works, but perhaps surprisingly. How?
2. `Foo.prototype.myName = ...`：这也许是更有趣的技术，它在`Foo.prototype`对象上添加了一个属性（函数）。现在，也许让人惊奇，`a.myName()`可以工作。但是是如何工作的？

In the above snippet, it's strongly tempting to think that when `a` and `b` are created, the properties/functions on the `Foo.prototype` object are *copied* over to each of `a` and `b` objects. **However, that's not what happens.**

在上面的代码段中，有很强的倾向认为当`a`和`b`被创建时，`Foo.prototype`上的属性/函数被 *拷贝* 到了`a`与`b`俩个对象上。**但是，这没有发生。**

At the beginning of this chapter, we explained the `[[Prototype]]` link, and how it provides the fall-back look-up steps if a property reference isn't found directly on an object, as part of the default `[[Get]]` algorithm.

在本章开头，我们解释了`[[Prototype]]`链，和它作为默认的`[[Get]]`算法的一部分,如何在不能直接在对象上找到属性引用时提供后备的查询步骤。

So, by virtue of how they are created, `a` and `b` each end up with an internal `[[Prototype]]` linkage to `Foo.prototype`. When `myName` is not found on `a` or `b`, respectively, it's instead found (through delegation, see Chapter 6) on `Foo.prototype`.

于是，得益于他们被创建的方式，`a`和`b`都最终拥有一个内部的`[[Prototype]]`链接链到`Foo.prototype`。当无法分别在`a`和`b`中找到`myName`时，就会在`Foo.prototype`上找到（通过委托，见第六章）。

#### "Constructor" Redux
#### 终极"构造器"

Recall the discussion from earlier about the `.constructor` property, and how it *seems* like `a.constructor === Foo` being true means that `a` has an actual `.constructor` property on it, pointing at `Foo`? **Not correct.**

回想我们刚才对`.constructor`属性的讨论，怎么看起来`a.constructor === Foo`为真意味着`a`上实际拥有一个`.constructor`属性，指向`Foo`？**不对。**

This is just unfortunate confusion. In actuality, the `.constructor` reference is also *delegated* up to `Foo.prototype`, which **happens to**, by default, have a `.constructor` that points at `Foo`.

这只是一种不行的混淆。实际上，`.constructor`引用也 *委托* 到了`Foo.prototype`，它 **恰好** 有一个指向`Foo`的默认属性。

It *seems* awfully convenient that an object `a` "constructed by" `Foo` would have access to a `.constructor` property that points to `Foo`. But that's nothing more than a false sense of security. It's a happy accident, almost tangentially, that `a.constructor` *happens* to point at `Foo` via this default `[[Prototype]]` delegation. There's actually several ways that the ill-fated assumption of `.constructor` meaning "was constructed by" can come back to bite you.

这 *看起来* 方便得可怕，一个被`Foo`构建的对象可以访问指向`Foo`的`.constructor`属性。但这只不过是安全性上的错觉。它是一个欢乐的巧合，（TODO），通过这个默认的`[[Prototype]]`委托`a.constructor` *恰好* 指向`Foo`。实际上`.construcor`意味着“被XX构建”这种注定失败的臆测会以几种方式来咬到你。

For one, the `.constructor` property on `Foo.prototype` is only there by default on the object created when `Foo` the function is declared. If you create a new object, and replace a function's default `.prototype` object reference, the new object will not by default magically get a `.constructor` on it.

第一，在`Foo.prototype`上的`.constructor`属性仅当`Foo`函数被声明时才出现在对象上。如果你创建一个新对象，并替换函数默认的`.prototype`对象引用，这个新对象上将不会魔法般地得到`.contructor`。

Consider:

考虑：

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

`Object(..)` didn't "construct" `a1` did it? It sure seems like `Foo()` "constructed" it. Many developers think of `Foo()` as doing the construction, but where everything falls apart is when you think "constructor" means "was constructed by", because by that reasoning, `a1.constructor` should be `Foo`, but it isn't!

`Object(..)`没有“构建”`a1`，是吧？看起来确实是`Foo()`“构建了”它。许多开发者认为`Foo()`在执行构建，但当你认为“构造器”意味着“被XX构建”时，一切就都崩塌了，因为如果那样的话，`a1.construcor`应当是`Foo`，但它不是！

What's happening? `a1` has no `.constructor` property, so it delegates up the `[[Prototype]]` chain to `Foo.prototype`. But that object doesn't have a `.constructor` either (like the default `Foo.prototype` object would have had!), so it keeps delegating, this time up to `Object.prototype`, the top of the delegation chain. *That* object indeed has a `.constructor` on it, which points to the built-in `Object(..)` function.

发生了什么？`a1`没有`.constructor`属性，所以它沿`[[Prototype]]`链向上委托到了`Foo.prototype`。但是这个对象也没有`.constructor`（默认的`Foo.prototype`对象就会有！），所以它继续委托，这次轮到了`Object.prototype`，委托链的最顶端。*那个* 对象上确实拥有`.constructor`，它指向内建的`Object(..)`函数。

**Misconception, busted.**

**误解，消除了。**

Of course, you can add `.constructor` back to the `Foo.prototype` object, but this takes manual work, especially if you want to match native behavior and have it be non-enumerable (see Chapter 3).

当然，你可以把`.constructor`加回到`Foo.prototype`对象上，但是做一些手动工作，特别是如果你想要它与原生的行为吻合，并不可枚举时（见第三章）。

For example:

举例来说：

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

// Need to properly "fix" the missing `.constructor`
// property on the new object serving as `Foo.prototype`.
// See Chapter 3 for `defineProperty(..)`.
Object.defineProperty( Foo.prototype, "constructor" , {
	enumerable: false,
	writable: true,
	configurable: true,
	value: Foo    // point `.constructor` at `Foo`
} );
```

That's a lot of manual work to fix `.constructor`. Moreover, all we're really doing is perpetuating the misconception that "constructor" means "was constructed by". That's an *expensive* illusion.

要修复`.constructor`要花不少功夫。而且，我们做的一切是为了延续“构造器”意味着“被XX构建”的误解。这是一种昂贵的假象。

The fact is, `.constructor` on an object arbitrarily points, by default, at a function who, reciprocally, has a reference back to the object -- a reference which it calls `.prototype`. The words "constructor" and "prototype" only have a loose default meaning that might or might not hold true later. The best thing to do is remind yourself, "constructor does not mean constructed by".

事实上，一个对象上的`.construcor`默认地随意指向一个函数，而这个函数反过来拥有一个指向这个对象的引用——这个引用被称为`.prototype`。“构造器”和“原型”这两个词仅有松散的默认含义，可能是真的也可能不是真的。最佳方案是提醒你自己，“构造器不是意味着被XX构建”。

`.constructor` is not a magic immutable property. It *is* non-enumerable (see snippet above), but its value is writable (can be changed), and moreover, you can add or overwrite (intentionally or accidentally) a property of the name `constructor` on any object in any `[[Prototype]]` chain, with any value you see fit.

`.constructor`不是一个魔法般不可变的属性。它是不可枚举的（见上面的代码段），但是它的值是可写的（可以改变），而且，你可以在`[[Prototype]]`链上的任何对象上添加或覆盖（有意或无意地）名为`constructor`的属性，用你感觉合适的任何值。

By virtue of how the `[[Get]]` algorithm traverses the `[[Prototype]]` chain, a `.constructor` property reference found anywhere may resolve quite differently than you'd expect.

根据`[[Get]]`算法如何遍历`[[Prototype]]`链，在任何地方找到的一个`.constructor`属性引用解析的结果可能与你期望的十分不同。

See how arbitrary its meaning actually is?

看到它的实际意义有多随便了吗？

The result? Some arbitrary object-property reference like `a1.constructor` cannot actually be *trusted* to be the assumed default function reference. Moreover, as we'll see shortly, just by simple omission, `a1.constructor` can even end up pointing somewhere quite surprising and insensible.

结果？某些像`a1.constructor`这样随意的对象属性引用实际上不能被信任，认为它是默认的函数引用。还有，我们马上就会看到，通过一个简单的省略，`a1.constructor`可以最终指向某些令人惊讶，没道理的地方。

`a1.constructor` is extremely unreliable, and an unsafe reference to rely upon in your code. **Generally, such references should be avoided where possible.**

`a1.constructor`是极其不可靠的，在你的代码中不应依赖的不安全引用。**一般来说，这样的引用应当尽量避免。**

## "(Prototypal) Inheritance"
## “（原型）继承”

We've seen some approximations of "class" mechanics as typically hacked into JavaScript programs. But JavaScript "class"es would be rather hollow if we didn't have an approximation of "inheritance".

我们已经看到了一些近似的“类”机制骇进JavaScript程序。但是如果我们没有一种近似的“继承”，JavaScript的“类”将会更空洞。

Actually, we've already seen the mechanism which is commonly called "prototypal inheritance" at work when `a` was able to "inherit from" `Foo.prototype`, and thus get access to the `myName()` function. But we traditionally think of "inheritance" as being a relationship between two "classes", rather than between "class" and "instance".

实际上，我们已经看到了一个常被称为“原型继承”的机制如何工作：`a`可以“继承自”`Foo.prototype`，并因此可以访问`myName()`函数。但是我们传统的想法认为“继承”是两个“类”间的关系，而非“类”与“实例”的关系。

<img src="fig3.png">

Recall this figure from earlier, which shows not only delegation from an object (aka, "instance") `a1` to object `Foo.prototype`, but from `Bar.prototype` to `Foo.prototype`, which somewhat resembles the concept of Parent-Child class inheritance. *Resembles*, except of course for the direction of the arrows, which show these are delegation links rather than copy operations.

回想之前这幅图，它不仅展示了从对象（也就是“实例”）`a1`到对象`Foo.prototype`的委托，而且从`Bar.prototype`到`Foo.prototype`，这酷似类继承的亲自概念。*酷似*，除了方向，箭头表示的是委托链接，而不是拷贝操作。

And, here's the typical "prototype style" code that creates such links:

这里是一段典型的创建这样的链接的“原型风格”代码：

```js
function Foo(name) {
	this.name = name;
}

Foo.prototype.myName = function() {
	return this.name;
};

function Bar(name,label) {
	Foo.call( this, name );
	this.label = label;
}

// 这里，我们创建一个新的`Bar.prototype`链接链到`Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// 注意！现在`Bar.prototype.constructor`不存在了，
// 如果你有依赖这个属性的习惯的话，可以被手动“修复”。

Bar.prototype.myLabel = function() {
	return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```

**Note:** To understand why `this` points to `a` in the above code snippet, see Chapter 2.

**注意：** 要想知道为什么上面代码中的`this`指向`a`，参见第二章。

The important part is `Bar.prototype = Object.create( Foo.prototype )`. `Object.create(..)` *creates* a "new" object out of thin air, and links that new object's internal `[[Prototype]]` to the object you specify (`Foo.prototype` in this case).

重要的部分是`Bar.prototype = Object.create( Foo.prototype )`。`Object.create(..)`凭空 *创建* 了一个“新”对象，并将这个新对象内部的`[[Prototype]]`链接到你指定的对象上（在这里是`Foo.prototype`）。

In other words, that line says: "make a *new* 'Bar dot prototype' object that's linked to 'Foo dot prototype'."

换句话说，这一行的意思是：“做一个 *新的* 链接到‘Foo点儿prototype’的‘Bar点儿prototype’对象”。

When `function Bar() { .. }` is declared, `Bar`, like any other function, has a `.prototype` link to its default object. But *that* object is not linked to `Foo.prototype` like we want. So, we create a *new* object that *is* linked as we want, effectively throwing away the original incorrectly-linked object.

当`function Bar() { .. }`被声明时，就像其他函数一样，拥有一个链到默认对象的`.prototype`链接。但是 *那个* 对象没有链到我们希望的`Foo.prototype`。所以，我们创建了一个 *新* 对象，链到我们希望的地方，并将原来的错误链接的对象扔掉。

**Note:** A common mis-conception/confusion here is that either of the following approaches would *also* work, but they do not work as you'd expect:

**注意：** 这里一个常见的误解/困惑是，下面两种方法 *也* 能工作，但是他们不会如你期望的那样工作：

```js
// doesn't work like you want!
Bar.prototype = Foo.prototype;

// works kinda like you want, but with
// side-effects you probably don't want :(
Bar.prototype = new Foo();
```

`Bar.prototype = Foo.prototype` doesn't create a new object for `Bar.prototype` to be linked to. It just makes `Bar.prototype` be another reference to `Foo.prototype`, which effectively links `Bar` directly to **the same object as** `Foo` links to: `Foo.prototype`. This means when you start assigning, like `Bar.prototype.myLabel = ...`, you're modifying **not a separate object** but *the* shared `Foo.prototype` object itself, which would affect any objects linked to `Foo.prototype`. This is almost certainly not what you want. If it *is* what you want, then you likely don't need `Bar` at all, and should just use only `Foo` and make your code simpler.

`Bar.prototype = Foo.prototype`不会创建新对象让`Bar.prototype`链接。它只是让`Bar.prototype`成为`Foo.prototype`的另一个引用，将`Bar`直接链到`Foo`链着的 **同一个对象**：`Foo.prototype`。这意味着当你开始赋值时，比如`Bar.prototype.myLabel = ...`，你修改的 **不是一个分离的对象** 而是那个被分享的`Foo.prototype`对象本身，它将影响到所有链接到`Foo.prototype`的对象。这几乎可以确定不是你想要的。如果这正是你想要的，那么你根本就不需要`Bar`，你应当仅使用`Foo`来使你的代码更简单。

`Bar.prototype = new Foo()` **does in fact** create a new object which is duly linked to `Foo.prototype` as we'd want. But, it uses the `Foo(..)` "constructor call" to do it. If that function has any side-effects (such as logging, changing state, registering against other objects, **adding data properties to `this`**, etc.), those side-effects happen at the time of this linking (and likely against the wrong object!), rather than only when the eventual `Bar()` "descendents" are created, as would likely be expected.

`Bar.prototype = new Foo()`**确实** 创建了一个新的对象，这个新对象也的确链接到了我们希望的`Foo.prototype`。但是，它是用`Foo(..)`“构造器调用”来这样做的。如果这个函数有任何副作用（比如logging，改变状态，注册其他对象，**向`this`添加数据属性**，等等），这些副作用就会在链接时发生（而且很可能是对错误的对象！），而不是像可能希望的那样，仅最终在`Bar()`的“后裔”被创建时发生。

So, we're left with using `Object.create(..)` to make a new object that's properly linked, but without having the side-effects of calling `Foo(..)`. The slight downside is that we have to create a new object, throwing the old one away, instead of modifying the existing default object we're provided.

于是，我们剩下的选择就是使用`Object.create(..)`来制造一个新对象，这个对象被正确地链接，而且没有调用`Foo(..)`时所产生的副作用。一个轻微的缺点是，我们不得不创建新对象，并把旧的扔掉，而不是修改提供给我们的默认既存对象。

It would be *nice* if there was a standard and reliable way to modify the linkage of an existing object. Prior to ES6, there's a non-standard and not fully-cross-browser way, via the `.__proto__` property, which is settable. ES6 adds a `Object.setPrototypeOf(..)` helper utility, which does the trick in a standard and predictable way.

如果有一种标准且可靠地方法来修改既存对象的链接就好了。ES6之前，有一个非标准的，而且不是完全跨浏览器的方法：通过可以设置的`.__proto__`属性。ES6中增加了`Object.setPrototypeOf(..)`辅助工具，它提供了标准且可预见的方法。

Compare the pre-ES6 and ES6-standardized techniques for linking `Bar.prototype` to `Foo.prototype`, side-by-side:

一对一地比较前ES6和ES6标准的技术如何处理将`Bar.prototype`链接至`Foo.prototype`：

```js
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

Ignoring the slight performance disadvantage (throwing away an object that's later garbage collected) of the `Object.create(..)` approach, it's a little bit shorter and may be perhaps a little easier to read than the ES6+ approach. But it's probably a syntactic wash either way.

如果忽略`Object.create(..)`方式在性能上的轻微劣势（扔掉一个对象，然后被回收），它相对短一些而且可能比ES6+的方式更易读。（TODO）。

### Inspecting "Class" Relationships
### 考察“类”关系

What if you have an object like `a` and want to find out what object (if any) it delegates to? Inspecting an instance (just an object in JS) for its inheritance ancestry (delegation linkage in JS) is often called *introspection* (or *reflection*) in traditional class-oriented environments.

如果你有一个对象`a`并且希望找到它委托至哪个对象呢（如果有的话）？考察一个实例（一个JS对象）的继承血统（在JS中是委托链接），在传统的面向类环境中称为 *introspection（自省）*（或 *reflection（反射）*）。

Consider:
考虑：

```js
function Foo() {
	// ...
}

Foo.prototype.blah = ...;

var a = new Foo();
```

How do we then introspect `a` to find out its "ancestry" (delegation linkage)? The first approach embraces the "class" confusion:

那么我们如何内省`a`来找到它的“祖先”（委托链）呢？一种方式是拥抱“类”的困惑：

```js
a instanceof Foo; // true
```

The `instanceof` operator takes a plain object as its left-hand operand and a **function** as its right-hand operand. The question `instanceof` answers is: **in the entire `[[Prototype]]` chain of `a`, does the object arbitrarily pointed to by `Foo.prototype` ever appear?**

`instanceof`操作符的左边操作数接收一个普通对象，右边操作数接收一个 **函数**。`instanceof`回答的问题是：**在`a`的整个`[[Prototype]]`链中，有没有出现被`Foo.prototype`所指向的对象？**（TODO）

Unfortunately, this means that you can only inquire about the "ancestry" of some object (`a`) if you have some **function** (`Foo`, with its attached `.prototype` reference) to test with. If you have two arbitrary objects, say `a` and `b`, and want to find out if *the objects* are related to each other through a `[[Prototype]]` chain, `instanceof` alone can't help.

不幸的是，这意味着如果你拥有可以用于测试的 **函数**（`Foo`，和它带有的`.prototype`引用），你只能查询某些对象（`a`）的“祖先”。如果你有两个任意的对象，比如`a`和`b`，而且你想调查是否 *这些对象* 通过`[[Prototype]]`链相互关联，单靠`instanceof`帮不上什么忙。

**Note:** If you use the built-in `.bind(..)` utility to make a hard-bound function (see Chapter 2), the function created will not have a `.prototype` property. Using `instanceof` with such a function transparently substitutes the `.prototype` of the *target function* that the hard-bound function was created from.

**注意：** 如果你使用内建的`.bind(..)`工具来制造一个硬绑定的函数（见第二章），这个被创建的函数将不会拥有`.prototype`属性。将`instanceof`与这样的函数一起使用时，将会透明地替换为创建这个硬绑定函数的 *目标函数* 的`.prototype`。

It's fairly uncommon to use hard-bound functions as "constructor calls", but if you do, it will behave as if the original *target function* was invoked instead, which means that using `instanceof` with a hard-bound function also behaves according to the original function.

将硬绑定函数用于“构造器调用”十分不常见，但如果你这么做，它会表现得好像是 *目标函数* 被调用了，这意味着将`instanceof`与硬绑定函数一起使用也会参照原版函数。

This snippet illustrates the ridiculousness of trying to reason about relationships between **two objects** using "class" semantics and `instanceof`:

下面这段代码展示了试图通过“类”的语义和`instanceof`来推导 **两个对象** 间的关系是多么荒谬：

```js
// helper utility to see if `o1` is
// related to (delegates to) `o2`
function isRelatedTo(o1, o2) {
	function F(){}
	F.prototype = o2;
	return o1 instanceof F;
}

var a = {};
var b = Object.create( a );

isRelatedTo( b, a ); // true
```

Inside `isRelatedTo(..)`, we borrow a throw-away function `F`, reassign its `.prototype` to arbitrarily point to some object `o2`, then ask if `o1` is an "instance of" `F`. Obviously `o1` isn't *actually* inherited or descended or even constructed from `F`, so it should be clear why this kind of exercise is silly and confusing. **The problem comes down to the awkwardness of class semantics forced upon JavaScript**, in this case as revealed by the indirect semantics of `instanceof`.

The second, and much cleaner, approach to `[[Prototype]]` reflection is:

```js
Foo.prototype.isPrototypeOf( a ); // true
```

Notice that in this case, we don't really care about (or even *need*) `Foo`, we just need an **object** (in our case, arbitrarily labeled `Foo.prototype`) to test against another **object**. The question `isPrototypeOf(..)` answers is: **in the entire `[[Prototype]]` chain of `a`, does `Foo.prototype` ever appear?**

Same question, and exact same answer. But in this second approach, we don't actually need the indirection of referencing a **function** (`Foo`) whose `.prototype` property will automatically be consulted.

We *just need* two **objects** to inspect a relationship between them. For example:

```js
// Simply: does `b` appear anywhere in
// `c`s [[Prototype]] chain?
b.isPrototypeOf( c );
```

Notice, this approach doesn't require a function ("class") at all. It just uses object references directly to `b` and `c`, and inquires about their relationship. In other words, our `isRelatedTo(..)` utility above is built-in to the language, and it's called `isPrototypeOf(..)`.

We can also directly retrieve the `[[Prototype]]` of an object. As of ES5, the standard way to do this is:

```js
Object.getPrototypeOf( a );
```

And you'll notice that object reference is what we'd expect:

```js
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

Most browsers (not all!) have also long supported a non-standard alternate way of accessing the internal `[[Prototype]]`:

```js
a.__proto__ === Foo.prototype; // true
```

The strange `.__proto__` (not standardized until ES6!) property "magically" retrieves the internal `[[Prototype]]` of an object as a reference, which is quite helpful if you want to directly inspect (or even traverse: `.__proto__.__proto__...`) the chain.

Just as we saw earlier with `.constructor`, `.__proto__` doesn't actually exist on the object you're inspecting (`a` in our running example). In fact, it exists (non-enumerable; see Chapter 2) on the built-in `Object.prototype`, along with the other common utilities (`.toString()`, `.isPrototypeOf(..)`, etc).

Moreover, `.__proto__` looks like a property, but it's actually more appropriate to think of it as a getter/setter (see Chapter 3).

Roughly, we could envision `.__proto__` implemented (see Chapter 3 for object property definitions) like this:

```js
Object.defineProperty( Object.prototype, "__proto__", {
	get: function() {
		return Object.getPrototypeOf( this );
	},
	set: function(o) {
		// setPrototypeOf(..) as of ES6
		Object.setPrototypeOf( this, o );
		return o;
	}
} );
```

So, when we access (retrieve the value of) `a.__proto__`, it's like calling `a.__proto__()` (calling the getter function). *That* function call has `a` as its `this` even though the getter function exists on the `Object.prototype` object (see Chapter 2 for `this` binding rules), so it's just like saying `Object.getPrototypeOf( a )`.

`.__proto__` is also a settable property, just like using ES6's `Object.setPrototypeOf(..)` shown earlier. However, generally you **should not change the `[[Prototype]]` of an existing object**.

There are some very complex, advanced techniques used deep in some frameworks that allow tricks like "subclassing" an `Array`, but this is commonly frowned on in general programming practice, as it usually leads to *much* harder to understand/maintain code.

**Note:** As of ES6, the `class` keyword will allow something that approximates "subclassing" of built-in's like `Array`. See Appendix A for discussion of the `class` syntax added in ES6.

The only other narrow exception (as mentioned earlier) would be setting the `[[Prototype]]` of a default function's `.prototype` object to reference some other object (besides `Object.prototype`). That would avoid replacing that default object entirely with a new linked object. Otherwise, **it's best to treat object `[[Prototype]]` linkage as a read-only characteristic** for ease of reading your code later.

**Note:** The JavaScript community unofficially coined a term for the double-underscore, specifically the leading one in properties like `__proto__`: "dunder". So, the "cool kids" in JavaScript would generally pronounce `__proto__` as "dunder proto".

## Object Links

As we've now seen, the `[[Prototype]]` mechanism is an internal link that exists on one object which references some other object.

This linkage is (primarily) exercised when a property/method reference is made against the first object, and no such property/method exists. In that case, the `[[Prototype]]` linkage tells the engine to look for the property/method on the linked-to object. In turn, if that object cannot fulfill the look-up, its `[[Prototype]]` is followed, and so on. This series of links between objects forms what is called the "prototype chain".

### `Create()`ing Links

We've thoroughly debunked why JavaScript's `[[Prototype]]` mechanism is **not** like *classes*, and we've seen how it instead creates **links** between proper objects.

What's the point of the `[[Prototype]]` mechanism? Why is it so common for JS developers to go to so much effort (emulating classes) in their code to wire up these linkages?

Remember we said much earlier in this chapter that `Object.create(..)` would be a hero? Now, we're ready to see how.

```js
var foo = {
	something: function() {
		console.log( "Tell me something good..." );
	}
};

var bar = Object.create( foo );

bar.something(); // Tell me something good...
```

`Object.create(..)` creates a new object (`bar`) linked to the object we specified (`foo`), which gives us all the power (delegation) of the `[[Prototype]]` mechanism, but without any of the unnecessary complication of `new` functions acting as classes and constructor calls, confusing `.prototype` and `.constructor` references, or any of that extra stuff.

**Note:** `Object.create(null)` creates an object that has an empty (aka, `null`) `[[Prototype]]` linkage, and thus the object can't delegate anywhere. Since such an object has no prototype chain, the `instanceof` operator (explained earlier) has nothing to check, so it will always return `false`. These special empty-`[[Prototype]]` objects are often called "dictionaries" as they are typically used purely for storing data in properties, mostly because they have no possible surprise effects from any delegated properties/functions on the `[[Prototype]]` chain, and are thus purely flat data storage.

We don't *need* classes to create meaningful relationships between two objects. The only thing we should **really care about** is objects linked together for delegation, and `Object.create(..)` gives us that linkage without all the class cruft.

#### `Object.create()` Polyfilled

`Object.create(..)` was added in ES5. You may need to support pre-ES5 environments (like older IE's), so let's take a look at a simple **partial** polyfill for `Object.create(..)` that gives us the capability that we need even in those older JS environments:

```js
if (!Object.create) {
	Object.create = function(o) {
		function F(){}
		F.prototype = o;
		return new F();
	};
}
```

This polyfill works by using a throw-away `F` function and overriding its `.prototype` property to point to the object we want to link to. Then we use `new F()` construction to make a new object that will be linked as we specified.

This usage of `Object.create(..)` is by far the most common usage, because it's the part that *can be* polyfilled. There's an additional set of functionality that the standard ES5 built-in `Object.create(..)` provides, which is **not polyfillable** for pre-ES5. As such, this capability is far-less commonly used. For completeness sake, let's look at that additional functionality:

```js
var anotherObject = {
	a: 2
};

var myObject = Object.create( anotherObject, {
	b: {
		enumerable: false,
		writable: true,
		configurable: false,
		value: 3
	},
	c: {
		enumerable: true,
		writable: false,
		configurable: false,
		value: 4
	}
} );

myObject.hasOwnProperty( "a" ); // false
myObject.hasOwnProperty( "b" ); // true
myObject.hasOwnProperty( "c" ); // true

myObject.a; // 2
myObject.b; // 3
myObject.c; // 4
```

The second argument to `Object.create(..)` specifies property names to add to the newly created object, via declaring each new property's *property descriptor* (see Chapter 3). Because polyfilling property descriptors into pre-ES5 is not possible, this additional functionality on `Object.create(..)` also cannot be polyfilled.

The vast majority of usage of `Object.create(..)` uses the polyfill-safe subset of functionality, so most developers are fine with using the **partial polyfill** in pre-ES5 environments.

Some developers take a much stricter view, which is that no function should be polyfilled unless it can be *fully* polyfilled. Since `Object.create(..)` is one of those partial-polyfill'able utilities, this narrower perspective says that if you need to use any of the functionality of `Object.create(..)` in a pre-ES5 environment, instead of polyfilling, you should use a custom utility, and stay away from using the name `Object.create` entirely. You could instead define your own utility, like:

```js
function createAndLinkObject(o) {
	function F(){}
	F.prototype = o;
	return new F();
}

var anotherObject = {
	a: 2
};

var myObject = createAndLinkObject( anotherObject );

myObject.a; // 2
```

I do not share this strict opinion. I fully endorse the common partial-polyfill of `Object.create(..)` as shown above, and using it in your code even in pre-ES5. I'll leave it to you to make your own decision.

### Links As Fallbacks?

It may be tempting to think that these links between objects *primarily* provide a sort of fallback for "missing" properties or methods. While that may be an observed outcome, I don't think it represents the right way of thinking about `[[Prototype]]`.

Consider:

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.cool(); // "cool!"
```

That code will work by virtue of `[[Prototype]]`, but if you wrote it that way so that `anotherObject` was acting as a fallback **just in case** `myObject` couldn't handle some property/method that some developer may try to call, odds are that your software is going to be a bit more "magical" and harder to understand and maintain.

That's not to say there aren't cases where fallbacks are an appropriate design pattern, but it's not very common or idiomatic in JS, so if you find yourself doing so, you might want to take a step back and reconsider if that's really appropriate and sensible design.

**Note:** In ES6, an advanced functionality called `Proxy` is introduced which can provide something of a "method not found" type of behavior. `Proxy` is beyond the scope of this book, but will be covered in detail in a later book in the *"You Don't Know JS"* series.

**Don't miss an important but nuanced point here.**

Designing software where you intend for a developer to, for instance, call `myObject.cool()` and have that work even though there is no `cool()` method on `myObject` introduces some "magic" into your API design that can be surprising for future developers who maintain your software.

You can however design your API with less "magic" to it, but still take advantage of the power of `[[Prototype]]` linkage.

```js
var anotherObject = {
	cool: function() {
		console.log( "cool!" );
	}
};

var myObject = Object.create( anotherObject );

myObject.doCool = function() {
	this.cool(); // internal delegation!
};

myObject.doCool(); // "cool!"
```

Here, we call `myObject.doCool()`, which is a method that *actually exists* on `myObject`, making our API design more explicit (less "magical"). *Internally*, our implementation follows the **delegation design pattern** (see Chapter 6), taking advantage of `[[Prototype]]` delegation to `anotherObject.cool()`.

In other words, delegation will tend to be less surprising/confusing if it's an internal implementation detail rather than plainly exposed in your API interface design. We will expound on **delegation** in great detail in the next chapter.

## Review (TL;DR)

When attempting a property access on an object that doesn't have that property, the object's internal `[[Prototype]]` linkage defines where the `[[Get]]` operation (see Chapter 3) should look next. This cascading linkage from object to object essentially defines a "prototype chain" (somewhat similar to a nested scope chain) of objects to traverse for property resolution.

All normal objects have the built-in `Object.prototype` as the top of the prototype chain (like the global scope in scope look-up), where property resolution will stop if not found anywhere prior in the chain. `toString()`, `valueOf()`, and several other common utilities exist on this `Object.prototype` object, explaining how all objects in the language are able to access them.

The most common way to get two objects linked to each other is using the `new` keyword with a function call, which among its four steps (see Chapter 2), it creates a new object linked to another object.

The "another object" that the new object is linked to happens to be the object referenced by the arbitrarily named `.prototype` property of the function called with `new`. Functions called with `new` are often called "constructors", despite the fact that they are not actually instantiating a class as *constructors* do in traditional class-oriented languages.

While these JavaScript mechanisms can seem to resemble "class instantiation" and "class inheritance" from traditional class-oriented languages, the key distinction is that in JavaScript, no copies are made. Rather, objects end up linked to each other via an internal `[[Prototype]]` chain.

For a variety of reasons, not the least of which is terminology precedent, "inheritance" (and "prototypal inheritance") and all the other OO terms just do not make sense when considering how JavaScript *actually* works (not just applied to our forced mental models).

Instead, "delegation" is a more appropriate term, because these relationships are not *copies* but delegation **links**.
