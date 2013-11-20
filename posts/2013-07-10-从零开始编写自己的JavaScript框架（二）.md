从零开始编写自己的JavaScript框架（二）
====

#2. 数据绑定

##2.1 数据绑定的原理

数据绑定是一种很便捷的特性，一些RIA框架带有双向绑定功能，比如Flex和Silverlight，当某个数据发生变更时，所绑定的界面元素也发生变更，当界面元素的值发生变化时，数据也跟着变化，这种功能在处理表单数据的填充和收集时，是非常有用的。

在HTML中，原生是没有这样的功能的，但有些框架做到了，它们是怎么做到的呢？我们来做个简单的试试，顺便探讨一下其中原理。

先看数据到界面上的的绑定，比如：

    <input vm-value="name"/>
    var person = {
        name: "Tom"
    };

如果我们给name重新赋值，person.name = "Jerry"，怎么才能让界面得到变更？
<!--more-->
从直觉来说，我们需要在name发生改变的时候，触发一个事件，或者调用某个指定的方法，然后才好着手做后面的事情，比如：

    var person = {
        name: "Tom",
        setName: function(newName) {
            this.name = newName;
            //do something
        }
    };

这样我们可以在setName里面去给input赋值。推而广之，为了使得实体包含的多个属性都可以运作，可以这么做：

    var person = {
        name: "Tom",
        gender: 5
        set: function(key, value) {
            this[key] = value;
            //do something
        }
    };

或者合并两个方法，只判断是否传了参数：

    Person.prototype.name = function(value) {
        if (arguments.length == 0) {
            return this._name;
        }
        else {
            this._name = value;
        }
    }

这种情况下，赋值的时候就是person.name("Tom")，取值的时候就是var name = person.name()了。

有一些框架是通过这种方式来变通实现数据绑定的，对数据的写入只能通过方法调用。但这种方式很不直接，我们来想点别的办法。

在C#等一些语言里，有一种东西叫做存取器，比如说：

    class Person
    {
        private string name;

        public string Name
        {
            get
            {
                return name;
            }
            set
            {
                name = value;
            }
        }
    }

用的时候，person.Name = "Jerry"，就会调用到set里，相当于是个方法。

这一点非常好，很符合我们的需要，那JavaScript里面有没有类似存取器的特性呢？老早以前是没有的，但现在有了，那就是Object.defineProperty，它的第三个参数就是可选的存取函数。比如说：

    var person = {};

    // Add an accessor property to the object.
    Object.defineProperty(person, "name", {
        set: function (value) {
            this._name = value;
            //do something
        },
        get: function () {
            return this._name;
        },
        enumerable: true,
        configurable: true
    });

赋值的时候，person.name = "Tom"，取值的时候，var name = person.name，简直太美妙了。注意这里define的时候，是定义在实例上的，如果想要定义到类型里面，可以在构造器里面定义。

现在我们从数据到DOM的绑定可以解决掉了，至少我们能够在变量被更改的时候去做一些自己的事情，比如查找这个属性被绑定到哪些控件了，然后挨个对其赋值。框架怎么知道属性被绑定到哪些控件了呢？这个直接在第二部分的实现过程中讨论。

再看控件到数据的绑定，这个其实很好理解。无非就是给控件添加change之类的事件监听，在这里面把关联到的数据更新掉。到这里，我们在原理方面已经没有什么问题了，现在开始准备把它写出来。

##2.2 数据绑定的实现

我们的框架启动之后，要先把前面所说的这种绑定关系收集起来，这种属性会分布于DOM的各个角落，一个很现实的做法是，递归遍历界面的每个DOM节点，检测该属性，于是我们代码的结构大致如下所示。

    function parseElement(element) {
        for (var i=0; i<element.attributes.length; i++) {
            parseAttribute(element.attributes[i]);
        }

        for (var i=0; i<element.children.length; i++) {
            parseElement(element.children[i]);
        }
    }

但是我们这时候面临一个问题，比如你的输入框绑定在name变量上，这个name应该从属于什么？它是全局变量吗？

我们在开始做这个框架的时候强调了一个原则：业务模块不允许定义全局变量，框架内部也尽量少有全局作用域，到目前为止，我们只暴露了thin一个全局入口，所以在这里不能破坏这个原则。

因此，我们要求业务开发人员去定义一个视图模型，把变量包装起来，所包装的不限于变量，也可以有方法。比如下面，我们定义了一个实体叫Person，带两个变量，两个方法，后面我们来演示一下怎么把它们绑定到HTML界面。

    thin.define("Person", [], function() {
        function Person() {
            this.name = "Tom";
            this.age = 5;
        }

        Person.prototype = {
            growUp: function() {
                this.age++;
            }
        };

        return Person;
    });

模型方面都准备好了，现在来看界面：

    <div vm-model="Person">
        <input type="text" vm-value="name"/>
        <input type="text" vm-value="age"/>
        <input type="button" vm-click="growUp" value="Grow Up"/>
    </div>

为了使得结构更加容易看，我们把界面的无关属性比如样式之类都去掉了，只留下不能再减少的这么一段。现在我们可以看到，在界面的顶层定义一个vm-model属性，值为实体的名称。两个输入框通过vm-value来绑定到实例属性，vm-init绑定界面的初始化方法，vm-click绑定按钮的点击事件。

好了，现在我们可以来扫描这个简单的DOM结构了。想要做这么一个绑定，首先要考虑数据从哪里来？在绑定name和code属性之前，毫无疑问，应当先实例化一个Person，我们怎么才能知道需要把Person模块实例化呢？

当扫描到一个DOM元素的时候，我们要先检测它的vm-model属性，如果有值，就取这个值来实例化，然后，把这个值一直传递下去，在扫描其他属性或者下属DOM元素的时候都带进去。这么一来，parseElement就变成一个递归了，于是它只好有两个参数，变成了这样：

    function parseElement(element, vm) {
        var model = vm;

        if (element.getAttribute("vm-model")) {
            model = bindModel(element.getAttribute("vm-model"));
        }

        for (var i=0; i<element.attributes.length; i++) {
            parseAttribute(element, element.attributes[i], model);
        }

        for (var i=0; i<element.children.length; i++) {
            parseElement(element.children[i], model);
        }
    }

看看我们打算怎么来实例化这个模型，这个bindModel方法的参数是模块名，于是我们先去use一下，从工厂里生成出来，然后new一下，先这么return出去吧。

    function bindModel(modelName) {
        thin.log("model" + modelName);

        var model = thin.use(modelName, true);
        var instance = new model();

        return instance;
    }

现在我们开始关注parseAttribute函数，可能的attribute有哪些种类呢？我列举了一些很常用的：

- init，用于绑定初始化方法
- click，用于绑定点击
- value，绑定变量
- enable和disable，绑定可用状态
- visible和invisible，绑定可见状态

然后就可以实现我们parseAttribute函数了：

    function parseAttribute(element, attr, model) {
        if (attr.name.indexOf("vm-") == 0) {
            var type = attr.name.slice(3);

            switch (type) {
                case "init":
                    bindInit(element, attr.value, model);
                    break;
                case "value":
                    bindValue(element, attr.value, model);
                    break;
                case "click":
                    bindClick(element, attr.value, model);
                    break;
                case "enable":
                    bindEnable(element, attr.value, model, true);
                    break;
                case "disable":
                    bindEnable(element, attr.value, model, false);
                    break;
                case "visible":
                    bindVisible(element, attr.value, model, true);
                    break;
                case "invisible":
                    bindVisible(element, attr.value, model, false);
                    break;
                case "element":
                    model[attr.value] = element;
                    break;
            }
        }
    }

注意到最后还有个element类型，本来可以不要这个，但我们考虑到将来，一切都是组件化的时候，界面上打算不写id，也不依靠选择器，而是用某个标志来定位元素，所以加上了这个，文章最后的示例中使用了它。

这么多绑定，不打算都讲，用bindValue函数来说明一下吧：

    function bindValue(element, key, vm) {
        thin.log("binding value: " + key);

        vm.$watch(key, function (value, oldValue) {
            element.value = value || "";
        });

        element.onkeyup = function () {
            vm[key] = element.value;
        };

        element.onpaste = function () {
            vm[key] = element.value;
        };
    }

我们假定每个模型实例上带有一个$watch方法，用于监控某变量的变化，可以传入一个监听函数，当变量变化的时候，自动调用这个函数，并且把新旧两个值传回来。

在这个代码里，我们使用$watch方法给传入的key添加一个监听，监听器里面给监听元素赋值。我们这里偷懒了一下，假定所有的绑定元素都是输入框，所以直接给element.value设置值，为了防止值为空导致显示undefined，把值跟空字符串用短路表达式做了个转换。

接下来，也对element的几个可能导致值变化的事件进行了监听，在里面把模型上对应的值更新掉。这样双向绑定就做好了。

然后回头来看$watch的实现。很显然这里也要一个map，我们给它取名为$watchers，存放属性的绑定关系，对于每个属性，它的值需要保存一份，供getter获取，同时还有一个数组，存放了该属性绑定的处理函数。当属性发生变更的时候，去挨个把它们调用一下。

    var Binder = {
        $watch: function (key, watcher) {
            if (!this.$watchers[key]) {
                this.$watchers[key] = {
                    value: this[key],
                    list: []
                };

                Object.defineProperty(this, key, {
                    set: function (val) {
                        var oldValue = this.$watchers[key].value;
                        this.$watchers[key].value = val;

                        for (var i = 0; i < this.$watchers[key].list.length; i++) {
                            this.$watchers[key].list[i](val, oldValue);
                        }
                    },

                    get: function () {
                        return this.$watchers[key].value;
                    }
                });
            }

            this.$watchers[key].list.push(watcher);
        }
    };

但是vm怎么就有$watch呢，每个地方都去判断一下非空然后再去创建其实挺麻烦的，所以，这个属性我们可以直接在实例化模型的时候创建出来。

    function bindModel(name) {
        thin.log("binding model: " + name);

        var model = thin.use(name, true);
        var instance = new model().extend(Binder);
        instance.$watchers = {};

        return instance;
    }

看看这里的写法，为什么$watchers要额外设置，而$watch就可以放在Binder里面来extend呢？

先解释extend干了什么，它做的是一个对象的浅拷贝，也就是说，把Binder的属性和方法都复制给了创建出来的model实例，注意，这个所谓的复制，如果是简单类型，那确实复制了，如果是引用类型，那复制的其实只是一个引用，所以如果$watchers也放在Binder里，不同的instance就共享一个$watchers，逻辑就是错误的。那为什么$watch又可以放在这里复制呢？因为它是函数，它的this始终指向当前的执行主体，也就是说，如果放在instance1上执行，指向的就是instance1，放在instance2上执行，指向的就是instance2，我们利用这一点，就可以不用让每个实例都创建一份$watch方法，而是共用同一个。

同理，我们可以把enable，visible，init，click这些都做起来，init的执行时间放在扫描完vm-model那个element之下的所有DOM节点之后。

嗯，我们是不是可以试一下了？来写个代码：

    <!DOCTYPE html>
    <html>
    <head>
        <title>Simple binding demo</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta name="description" content="binding">
        <meta name="author" content="xu.fei@outlook.com">
        <script type="text/javascript" src="../js/thin.js"></script>
    </head>
    <body>
    <div vm-model="test.Person">
        <input type="text" vm-value="name"/>
        <input type="text" vm-value="age"/>
        <input type="text" vm-value="age"/>
        <input type="button" vm-click="growUp" value="Grow Up"/>
    </div>

    <div vm-model="test.Person" vm-init="init">
        <input type="text" vm-value="name"/>
        <input type="text" vm-value="age"/>
        <input type="button" vm-click="growUp" value="Grow Up"/>
    </div>
    <script type="text/javascript">
        thin.define("test.Person", [], function () {
            function Person() {
                this.name = "Tom";
                this.age = 5;
            }

            Person.prototype = {
                init: function () {
                    this.name = "Jerry";
                    this.age = 3;
                },

                growUp: function () {
                    this.age++;
                }
            };

            return Person;
        });
    </script>
    </body>
    </html>

或者访问这里：http://xufei.github.io/thin/demo/simple-binding.html

以刚才文章提到的内容，还不能完全解释这个例子的效果，因为没看到在哪里调用parseElement的。说来也简单，就在thin.js里面，直接写了一个thin.ready，在那边调用了这个函数，去解析了document.body，于是测试页面里面才可以只写绑定和视图模型。

我们还有一个更实际一点的例子，结合了另外一个系列里面写的简单DataGrid控件，做了一个很基础的人员管理界面：http://xufei.github.io/thin/demo/binding.html

##2.3 小结

到此为止，我们的绑定框架勉强能够运行起来了！虽然很简陋，而且要比较新的浏览器才能跑，但毕竟是跑起来了。

注意Object.defineProperty仅在Chrome等浏览器中可用，IE需要9以上才比较正常。在司徒正美的avalon框架中，巧妙使用VBScript绕过这一限制，利用vbs的property和两种语言的互通，实现了低版本IE的兼容。我们这个框架的目标不是兼容，而是为了说明原理，所以感兴趣的朋友可以去看看avalon的源码。 
