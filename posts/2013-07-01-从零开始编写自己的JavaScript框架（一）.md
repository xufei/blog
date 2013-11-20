从零开始编写自己的JavaScript框架（一）
====

#1. 模块的定义和加载

##1.1 模块的定义

一个框架想要能支撑较大的应用，首先要考虑怎么做模块化。有了内核和模块加载系统，外围的模块就可以一个一个增加。不同的JavaScript框架，实现模块化方式各有不同，我们来选择一种比较优雅的方式作个讲解。

先问个问题：我们做模块系统的目的是什么？如果觉得这个问题难以回答，可以从反面来考虑：假如不做模块系统，有什么样的坏处？

我们经历过比较粗放、混乱的前端开发阶段，页面里充满了全局变量，全局函数。那时候要复用js文件，就是把某些js函数放到一个文件里，然后让多个页面都来引用。

考虑到一个页面可以引用多个这样的js，这些js互相又不知道别人里面写了什么，很容易造成命名的冲突，而产生这种冲突的时候，又没有哪里能够提示出来。所以我们要有一种办法，把作用域比较好地隔开。
<!--more-->
JavaScript这种语言比较奇怪，奇怪在哪里呢，它的现有版本里没package跟class，要是有，我们也没必要来考虑什么自己做模块化了。那它是要用什么东西来隔绝作用域呢？

在很多传统高级语言里，变量作用域的边界是大括号，在{}里面定义的变量，作用域不会传到外面去，但我们的JavaScript大人不是这样的，他的边界是function。所以我们这段代码，i仍然能打出值：

    for (var i=0; i<5; i++) {
        //do something
    }
    alert(i);

那么，我们只能选用function做变量的容器，把每个模块封装到一个function里。现在问题又来了，这个function本身的作用域是全局的，怎么办？我们想不到办法，拔剑四顾心茫然。

我们有没有什么可参照的东西呢？这时候，脑海中一群语言飘过：
C语言飘过：“我不是面向对象语言哦~不需要像你这么组织哦~”，“死开！”
Java飘过：“我是纯面向对象语言哦，连main都要在类中哦，编译的时候通过装箱清单指定入口哦~”，“死开！”
C++飘过：“我也是纯面向对象语言哦”，等等，C++是纯面向对象的语言吗？你的main是什么？？？main是特例，不在任何类中！

啊，我们发现了什么，既然无法避免全局的作用域，那与其让100个function都全局，不如只让一个来全局，其他的都由它管理。

本来我们打算自己当上帝的，现在只好改行先当个工商局长。你想开店吗？先来注册，不然封杀你！于是良民们纷纷来注册。店名叫什么，从哪进货，卖什么的，一一登记在案，为了方便下面的讨论，我们连进货的过程都让工商局管理起来。

店名，指的就是这里的模块名，从哪里进货，代表它依赖什么其他模块，卖什么，表示它对外提供一些什么特性。

好了，考虑到我们的这个注册管理机构是个全局作用域，我们还得把它挂在window上作为属性，然后再用一个function隔离出来，要不然，别人也定义一个同名的，就把我们覆盖掉了。

    (function() {
        window.thin = {
            define: function(name, dependencies, factory) {
                //register a module
            }
        };
    })();

在这个module方法内部，应当怎么去实现呢？我们的module应当有一个地方存储，但存储是要在工商局内部的，不是随便什么人都可以看到的，所以，这个存储结构也放在工商局同样的作用域里。

用什么结构去存储呢？工商局备案的时候，店名不能跟已有的重复，所以我们发现这是用map的很好场景，考虑到JavaScript语言层面没有map，我们弄个Object来存。

    (function() {
        var moduleMap = {};
        window.thin = {
            define: function(name, dependencies, factory) {
				if (!moduleMap[name]) {
					var module = {
						name: name,
						dependencies: dependencies,
						factory: factory
					};
					moduleMap[name] = module;
				}
				return moduleMap[name];
			}
        };
    })();

现在，模块的存储结构就搞好了。

##1.2 模块的使用

存的部分搞好了，我们来看看怎么取。现在来了一个商家，卖木器的，他需要从一个卖钉子的那边进货，卖钉子的已经来注册过了，现在要让这个木器厂能买到钉子。现在的问题是，两个商家处于不同的作用域，也就是说，它们互相不可见，那通过什么方式，我们才能让他们产生调用关系呢？

个人解决不了的问题还是得靠政府，有困难要坚决克服，没有困难就制造困难来克服。现在困难有了，该克服了。商家说，我能不能给你我的进货名单，你帮我查一下它们在哪家店，然后告诉我？这么简单的要求当然一口答应下来，但是采用什么方式传递给你呢？这可犯难了。

我们参考AngularJS框架，写了一个类似的代码：

	thin.define("A", [], function() {
		//module A
	});

	thin.define("B", ["A"], function(A) {
		//module B
		var a = new A();
	});

看这段代码特别在哪里呢？模块A的定义，毫无特别之处，主要看模块B。它在依赖关系里写了一个字符串的A，然后在工厂方法的形参写了一个真真切切的A类型。嗯？这个有些奇怪啊，你的A类型要怎么传递过来呢？其实是很简单的，因为我们声明了依赖项的数组，所以可以从依赖项，挨个得到对应的工厂方法，然后创建实例，传进来。

	use: function(name) {
		var module = moduleMap[name];

		if (!module.entity) {
			var args = [];
			for (var i=0; i<module.dependencies.length; i++) {
				if (moduleMap[module.dependencies[i]].entity) {
					args.push(moduleMap[module.dependencies[i]].entity);
				}
				else {
					args.push(this.use(module.dependencies[i]));
				}
			}

			module.entity = module.factory.apply(noop, args);
		}

		return module.entity;
	}

我们可以看到，这里面递归获取了依赖项，然后当作参数，用这个模块的工厂方法来实例化了一下。这里我们多做了一个判断，如果模块工厂已经执行过，就缓存在entity属性上，不需要每次都创建。以此类推，假如一个模块有多个依赖项，也可以用类似的方式写，毫无压力：

	thin.define("D", ["A", "B", "C"], function(A, B, C) {
		//module D
		var a = new A();
		var b = new B();
		var c = new C();
	});

注意了，D模块的工厂，实参的名称未必就要是跟依赖项一致，比如，以后我们代码较多，可以给依赖项和模块名称加命名空间，可能变成这样：

	thin.define("foo.D", ["foo.A", "foo.B", "foo.C"], function(A, B, C) {
		//module D
		var a = new A();
		var b = new B();
		var c = new C();
	});

这段代码仍然可以正常运行。我们来做另外一个测试，改变形参的顺序：

	thin.define("A", [], function() {
		return "a";
	});

	thin.define("B", [], function() {
		return "b";
	});

	thin.define("C", [], function() {
		return "c";
	});

	thin.define("D", ["A", "B", "C"], function(B, A, C) {
		return B + A + C;
	});

	var D = thin.use("D");
	alert(D);

试试看，我们的D打出什么结果呢？结果是"abc"，所以说，模块工厂的实参只跟依赖项的定义有关，跟形参的顺序无关。我们看到，在AngularJS里面，并非如此，实参的顺序是跟形参一致的，这是怎么做到的呢？

我们先离开代码，思考这么一个问题：如何得知函数的形参名数组？对，我们是可以用func.length得到形参个数，但无法得到每个形参的变量名，那怎么办呢？

AngularJS使用了一种比较极端的办法，分析了函数的字面量。众所周知，在JavaScript中，任何对象都隐含了toString方法，对于一个函数来说，它的toString就是自己的实现代码，包含函数签名和注释。下面我贴一下AngularJS里面的这部分代码：

	var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
	var FN_ARG_SPLIT = /,/;
	var FN_ARG = /^\s*(_?)(\S+?)\1\s*$/;
	var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;
	function annotate(fn) {
	  var $inject,
		  fnText,
		  argDecl,
		  last;

	  if (typeof fn == 'function') {
		if (!($inject = fn.$inject)) {
		  $inject = [];
		  fnText = fn.toString().replace(STRIP_COMMENTS, '');
		  argDecl = fnText.match(FN_ARGS);
		  forEach(argDecl[1].split(FN_ARG_SPLIT), function(arg){
			arg.replace(FN_ARG, function(all, underscore, name){
			  $inject.push(name);
			});
		  });
		  fn.$inject = $inject;
		}
	  } else if (isArray(fn)) {
		last = fn.length - 1;
		assertArgFn(fn[last], 'fn');
		$inject = fn.slice(0, last);
	  } else {
		assertArgFn(fn, 'fn', true);
	  }
	  return $inject;
	}

可以看到，这个代码也不长，重点是类型为function的那段，首先去除了注释，然后获取了形参列表字符串，这段正则能获取到两个结果，第一个是全函数的实现，第二个才是真正的形参列表，取第二个出来split，就得到了形参的字符串列表了，然后按照这个顺序再去加载依赖模块，就可以让形参列表不对应于依赖项数组了。

AngularJS的这段代码很强大，但是要损耗一些性能，考虑到我们的框架首要原则是简单，甚至可以为此牺牲一些灵活性，我们不做这么复杂的事情了。

##1.3 模块的加载

到目前为止，我们可以把多个模块都定义在一个文件中，然后手动引入这个js文件，但是如果一个页面要引用很多个模块，引入工作就变得比较麻烦，比如说，单页应用程序（SPA）一般比较复杂，往往包含数以万计行数的js代码，这些代码至少分布在几十个甚至成百上千的模块中，如果我们也在主界面就加载它们，载入时间会非常难以接受。但我们可以这样看：主界面加载的时候，并不是用到了所有这些功能，能否先加载那些必须的，而把剩下的放在需要用的时候再去加载？

所以我们可以考虑万能的AJAX，从服务端获取一个js的内容，然后……，怎么办，你当然说不能eval了，因为据说eval很evil啦，但是它evil在哪里呢？主要是破坏全局作用域啦，怎么怎么，但是如果这些文件里面都是按照我们规定的模块格式写，好像也没有什么在全局作用域的……，好吧。

算了，我们还是用最简单的方式了，就是动态创建script标签，然后设置src，添加到document.head里，然后监听它们的完成事件，做后续操作。真的很简单，因为我们的框架不需要考虑那么多种情况，不需要AMD，不需要require那么麻烦，用这框架的人必须按照这里的原则写。

所以，说真的我们这里没那么复杂啦，要是你们想看更详细原理的不如去看这个，解释得比我好哎：http://coolshell.cn/articles/9749.html#jtss-tsina

我也偷懒了，只是贴一下代码，顺便解释一下，界面把所依赖的js文件路径放在数组里，然后挨个创建script标签，src设置为路径，添加到head中，监听它们的完成事件。在这个完成时间里，我们要做这么一些事情：在fileMap里记录当前js文件的路径，防止以后重复加载，检查列表中所有文件，看看是否全部加载完了，如果全加载好了，就执行回调。

	require: function (pathArr, callback) {
		for (var i = 0; i < pathArr.length; i++) {
			var path = pathArr[i];

			if (!fileMap[path]) {
				var head = document.getElementsByTagName('head')[0];
				var node = document.createElement('script');
				node.type = 'text/javascript';
				node.async = 'true';
				node.src = path + '.js';
				node.onload = function () {
					fileMap[path] = true;
					head.removeChild(node);
					checkAllFiles();
				};
				head.appendChild(node);
			}
		}

		function checkAllFiles() {
			var allLoaded = true;
			for (var i = 0; i < pathArr.length; i++) {
				if (!fileMap[pathArr[i]]) {
					allLoaded = false;
					break;
				}
			}

			if (allLoaded) {
				callback();
			}
		}
	}

##1.4 小结

到此为止，我们的简易框架的模块定义系统就完成了。完整的代码如下：

	(function () {
		var moduleMap = {};
		var fileMap = {};

		var noop = function () {
		};

		var thin = {
			define: function(name, dependencies, factory) {
				if (!moduleMap[name]) {
					var module = {
						name: name,
						dependencies: dependencies,
						factory: factory
					};

					moduleMap[name] = module;
				}

				return moduleMap[name];
			},

			use: function(name) {
				var module = moduleMap[name];

				if (!module.entity) {
					var args = [];
					for (var i=0; i<module.dependencies.length; i++) {
						if (moduleMap[module.dependencies[i]].entity) {
							args.push(moduleMap[module.dependencies[i]].entity);
						}
						else {
							args.push(this.use(module.dependencies[i]));
						}
					}

					module.entity = module.factory.apply(noop, args);
				}

				return module.entity;
			},

			require: function (pathArr, callback) {
                for (var i = 0; i < pathArr.length; i++) {
                    var path = pathArr[i];

                    if (!fileMap[path]) {
                        var head = document.getElementsByTagName('head')[0];
                        var node = document.createElement('script');
                        node.type = 'text/javascript';
                        node.async = 'true';
                        node.src = path + '.js';
                        node.onload = function () {
                            fileMap[path] = true;
                            head.removeChild(node);
                            checkAllFiles();
                        };
                        head.appendChild(node);
                    }
                }

                function checkAllFiles() {
                    var allLoaded = true;
                    for (var i = 0; i < pathArr.length; i++) {
                        if (!fileMap[pathArr[i]]) {
                            allLoaded = false;
                            break;
                        }
                    }

                    if (allLoaded) {
                        callback();
                    }
                }
            }
		};

		window.thin = thin;
	})();

测试代码如下：

	thin.define("constant.PI", [], function() {
		return 3.14159;
	});

	thin.define("shape.Circle", ["constant.PI"], function(pi) {
		var Circle = function(r) {
			this.r = r;
		};

		Circle.prototype = {
			area : function() {
				return pi * this.r * this.r;
			}
		}

		return Circle;
	});

	thin.define("shape.Rectangle", [], function() {
		var Rectangle = function(l, w) {
			this.l = l;
			this.w = w;
		};

		Rectangle.prototype = {
			area: function() {
				return this.l * this.w;
			}
		};

		return Rectangle;
	});

	thin.define("ShapeTypes", ["shape.Circle", "shape.Rectangle"], function(Circle, Rectangle) {
		return {
			CIRCLE: Circle,
			RECTANGLE: Rectangle
		};
	});

	thin.define("ShapeFactory", ["ShapeTypes"], function(ShapeTypes) {
		return {
			getShape: function(type) {
				var shape;

				switch (type) {
					case "CIRCLE": {
						shape = new ShapeTypes[type](arguments[1]);
						break;
					}
					case "RECTANGLE":  {
						shape = new ShapeTypes[type](arguments[1], arguments[2]);
						break;
					}
				}

				return shape;
			}
		};
	});

	var ShapeFactory = thin.use("ShapeFactory");
	alert(ShapeFactory.getShape("CIRCLE", 5).area());
	alert(ShapeFactory.getShape("RECTANGLE", 3, 4).area());

在这个例子里定义了四个模块，每个模块只需要定义自己所直接依赖的模块，其他的可以不必定义。也可以来这里看测试链接：http://xufei.github.io/thin/demo/demo.0.1.html
