一些JS题目的解答
====

在[这里](http://davidshariff.com/quiz/)看到一些测试题，我HTML和CSS比较一般，尝试把里面的JS题目都解答一下：

#1.
    "1" + 2 + "3" + 4

- 10
- 1234
- 37

答案：1234，加法优先级等同，从左往右，数字与字符串相加，数字转换成字符串进行运算，结果等同于："12"+"3"+4 = "123"+4 = "1234"。
<!--more-->
#2.

    4 + 3 + 2 + "1"

- 10
- 4321
- 91

答案：91，优先级同上，从左往右，等同于：7+2+"1" = 9+"1" = "91"。

#3.
    var foo = 1;
    function bar() {
    	foo = 10;
    	return;
    	function foo() {}
    }
    bar();
    alert(foo);

- 1
- 10
- Function
- undefined
- Error

答案：1，function的定义会提前到当前作用域之前，所以等同于：

    var foo = 1;
    function bar() {
    	function foo() {}
    	foo = 10;
    	return;
    }
    bar();
    alert(foo);

所以，在foo=10的时候，foo是有定义的，属于局部变量，影响不到外层的foo。

参见：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FFunctions_and_function_scope

> Unlike functions defined by function expressions or by the Function constructor, a function defined by a function declaration can be used before the function declaration itself.

#4.

    function bar() {
        return foo;
        foo = 10;
        function foo() {}
        var foo = 11;
    }
    alert(typeof bar());

- number
- function
- undefined
- Error

答案：function，与上题类似，等同于：

    function bar() {
        function foo() {}
        return foo;
        foo = 10;
        var foo = 11;
    }
    alert(typeof bar());

在return之后声明和赋值的foo都无效，所以返回了function。

参见：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/return

> A function immediately stops at the point where return is called.

补充，这个解答有问题：

> @尤里卡Eureka：JS中function声明和var声明都会被提前，最终得到结果为function，是因为*名称解析顺序-Name Resolution Order*(http://t.cn/8kcIRts导致的function声明优先级大于var声明，而不是由return语句退出导致最后的结果~

#5.

    var x = 3;

    var foo = {
        x: 2,
        baz: {
            x: 1,
            bar: function() {
                return this.x;
            }
        }
    }

    var go = foo.baz.bar;

    alert(go());
    alert(foo.baz.bar());

- 1,2
- 1,3
- 2,1
- 2,3
- 3,1
- 3,2

答案：3,1
this指向执行时刻的作用域，go的作用域是全局，所以相当于window，取到的就是window.x，也就是var x=3;这里定义的x。而foo.baz.bar()里面，this指向foo.baz，所以取到的是这个上面的x，也就是1。

参见：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FOperators%2Fthis

#6.

    var x = 4,
        obj = {
            x: 3,
            bar: function() {
                var x = 2;
                setTimeout(function() {
                    var x = 1;
                    alert(this.x);
                }, 1000);
            }
        };
    obj.bar();

- 1
- 2
- 3
- 4
- undefined

答案：4，不管有这个setTimeout还是把这个函数立即执行，它里面这个function都是孤立的，this只能是全局的window，即使不延时，改成立即执行结果同样是4。

#7.

    x = 1;
    function bar() {
        this.x = 2;
        return x;
    }
    var foo = new bar();
    alert(foo.x);

- 1
- 2
- undefined

答案：2，这里主要问题是最外面x的定义，试试把x=1改成x={}，结果会不同的。这是为什么呢？在把函数当作构造器使用的时候，如果手动返回了一个值，要看这个值是否简单类型，如果是，等同于不写返回，如果不是简单类型，得到的就是手动返回的值。如果，不手动写返回值，就会默认从原型创建一个对象用于返回。

参见：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new

#8.

    function foo(a) {
        alert(arguments.length);
    }
    foo(1, 2, 3);

- 1
- 2
- 3
- undefined

答案3，arguments取的是实参的个数，而foo.length取的是形参个数。

参见：
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/arguments/length?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FFunctions_and_function_scope%2Farguments%2Flength

> arguments.length provides the number of arguments actually passed to a function. This can be more or less than the defined parameter count (See Function.length).

#9.

    var foo = function bar() {};
    alert(typeof bar);

- function
- object
- undefined

答案：undefined，这种情况下bar的名字从外部不可见，那是不是这个名字别人就没法知道了呢？不是，toString就可以看到它，比如说alert(foo)，可以看看能打出什么。

参见：
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FFunctions_and_function_scope

> The function name can be used only within the function's body. Attempting to use it outside the function's body results in an error (or undefined if the function name was previously declared via a var statement).

#10.

    var arr = [];
    arr[0]  = 'a';
    arr[1]  = 'b';
    arr.foo = 'c';
    alert(arr.length);

- 1
- 2
- 3
- undefined

答案：2，数组的原型是Object，所以可以像其他类型一样附加属性，不影响其固有性质。

#11.

    function foo(a) {
        arguments[0] = 2;
        alert(a);
    }
    foo(1);

- 1
- 2
- undefined

答案：2，实参可以直接从arguments数组中修改。

参见：
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/arguments?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FFunctions_and_function_scope%2Farguments

> The arguments can also be set

#12.

    function foo(){}
    delete foo.length;
    alert(typeof foo.length);

- number
- undefined
- object
- Error

答案：number，foo.length是无法删除的，它在Function原型上，重点它的configurable是false。

参见：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete

> delete can't remove certain properties of predefined objects (like Object, Array, Math etc). These are described in ECMAScript 5 and later as non-configurable
