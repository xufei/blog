前端开发技术的发展
==

前端开发技术，从狭义的定义来看，是指围绕HTML、JavaScript、CSS这样一套体系的开发技术，它的运行宿主是浏览器。从广义的定义来看，包括了：

- 专门为手持终端设计的类似WML这样的类HTML语言，类似WMLScript这样的类JavaScript语言。
- VML和SVG等基于XML的描述图形的语言。
- 从属于XML体系的XML，XPath，DTD等技术。
- 用于支撑后端的ASP，JSP，ASP.net，PHP，nodejs等语言或者技术。
- 被第三方程序打包的一种类似浏览器的宿主环境，比如Adobe AIR和使用HyBird方式的一些开发技术，如PhoneGap（它使用Android中的WebView等技术，让开发人员使用传统Web开发技术来开发本地应用）
- Adobe Flash，Flex，Microsoft Silverlight，Java Applet，JavaFx等RIA开发技术。
	
本文从狭义的前端定义出发，探讨一下这方面开发技术的发展过程。
<!--more-->
从前端开发技术的发展来看，大致可以分为以下几个阶段：

#一. 刀耕火种
##1. 静态页面

最早期的Web界面基本都是在互联网上使用，人们浏览某些内容，填写几个表单，并且提交。当时的界面以浏览为主，基本都是HTML代码，有时候穿插一些JavaScript，作为客户端校验这样的基础功能。代码的组织比较简单，而且CSS的运用也是比较少的。
	
最简单的是这样一个文件：

	<html>
		<head>
			<title>测试一</title>
		</head>
		<body>
			<h1>主标题</h1>
			<p>段落内容</p>
		</body>
	</html>

##2. 带有简单逻辑的界面
这个界面带有一段JavaScript代码，用于拼接两个输入框中的字符串，并且弹出窗口显示。

	<html>
		<head>
			<title>测试二</title>
		</head>
		<body>
			<input id="firstNameInput" type="text" /> 
			<input id="lastNameInput" type="text" /> 
			<input type="button" onclick="greet()" />
			<script language="JavaScript">
			function greet() {
				var firstName = document.getElementById("firstNameInput").value;
				var lastName = document.getElementById("lastNameInput").value;
				alert("Hello, " + firstName + "." + lastName);
			}
			</script> 
		</body>
	</html>

##3. 结合了服务端技术的混合编程

由于静态界面不能实现保存数据等功能，出现了很多服务端技术，早期的有CGI（Common Gateway Interface，多数用C语言或者Perl实现的），ASP（使用VBScript或者JScript），JSP（使用Java），PHP等等，Python和Ruby等语言也常被用于这类用途。

有了这类技术，在HTML中就可以使用表单的post功能提交数据了，比如：

	<form method="post" action="username.asp">
		<p>First Name: <input type="text" name="firstName" /></p>
		<p>Last Name: <input type="text" name="lastName" /></p>
		<input type="submit" value="Submit" />
	</form>

在这个阶段，由于客户端和服务端的职责未作明确的划分，比如生成一个字符串，可以由前端的JavaScript做，也可以由服务端语言做，所以通常在一个界面里，会有两种语言混杂在一起，用<%和%>标记的部分会在服务端执行，输出结果，甚至经常有把数据库连接的代码跟页面代码混杂在一起的情况，给维护带来较大的不便。

	<html>
		<body>
			<p>Hello world!</p>
			<p>
			<%
				response.write("Hello world from server!")
			%>
			</p>
		</body>
	</html>

##4.组件化的萌芽

这个时代，也逐渐出现了组件化的萌芽。比较常见的有服务端的组件化，比如把某一类服务端功能单独做成片段，然后其他需要的地方来include进来，典型的有：ASP里面数据库连接的地方，把数据源连接的部分写成conn.asp，然后其他每个需要操作数据库的asp文件包含它。

上面所说的是在服务端做的，浏览器端通常有针对JavaScript的，把某一类的Javascript代码写到单独的js文件中，界面根据需要，引用不同的js文件。针对界面的组件方式，通常利用frameset和iframe这两个标签。某一大块有独立功能的界面写到一个html文件，然后在主界面里面把它当作一个frame来载入，一般的B/S系统集成菜单的方式都是这样的。

此外，还出现了一些基于特定浏览器的客户端组件技术，比如IE浏览器的HTC（HTML Component）。这种技术最初是为了对已有的常用元素附加行为的，后来有些场合也用它来实现控件。微软ASP.net的一些版本里，使用这种技术提供了树形列表，日历，选项卡等功能。HTC的优点是允许用户自行扩展HTML标签，可以在自己的命名空间里定义元素，然后，使用HTML，JavaScript和CSS来实现它的布局、行为和观感。这种技术因为是微软的私有技术，所以逐渐变得不那么流行。

Firefox浏览器里面推出过一种叫XUL的技术，也没有流行起来。

#二. 铁器时代

这个时代的典型特征是Ajax的出现。

##1. AJAX
AJAX其实是一系列已有技术的组合，早在这个名词出现之前，这些技术的使用就已经比较广泛了，GMail因为恰当地应用了这些技术，获得了很好的用户体验。

由于Ajax的出现，规模更大，效果更好的Web程序逐渐出现，在这些程序中，JavaScript代码的数量迅速增加。出于代码组织的需要，“JavaScript框架”这个概念逐步形成，当时的主流是prototype和mootools，这两者各有千秋，提供了各自方式的面向对象组织思路。

##2. JavaScript基础库

Prototype框架主要是为JavaScript代码提供了一种组织方式，对一些原生的JavaScript类型提供了一些扩展，比如数组、字符串，又额外提供了一些实用的数据结构，如：枚举，Hash等，除此之外，还对dom操作，事件，表单和Ajax做了一些封装。

Mootools框架的思路跟Prototype很接近，它对JavaScript类型扩展的方式别具一格，所以在这类框架中，经常被称作“最优雅的”对象扩展体系。

从这两个框架的所提供的功能来看，它们的定位是核心库，在使用的时候一般需要配合一些外围的库来完成。

jQuery与这两者有所不同，它着眼于简化DOM相关的代码。
例如：

- DOM的选择

jQuery提供了一系列选择器用于选取界面元素，在其他一些框架中也有类似功能，但是一般没有它的简洁、强大。

	$("*")				//选取所有元素
	$("#lastname")		//选取id为lastname的元素
	$(".intro")			//选取所有class="intro"的元素
	$("p")				//选取所有&lt;p&gt;元素
	$(".intro.demo")	//选取所有 class="intro"且class="demo"的元素

- 链式表达式：

在jQuery中，可以使用链式表达式来连续操作dom，比如下面这个例子：

如果不使用链式表达式，可能我们需要这么写：

	var neat = $("p.neat");
	neat.addClass("ohmy");
	neat.show("slow");

但是有了链式表达式，我们只需要这么一行代码就可以完成这些：

	$("p.neat").addClass("ohmy").show("slow");

除此之外，jQuery还提供了一些动画方面的特效代码，也有大量的外围库，比如jQuery UI这样的控件库，jQuery mobile这样的移动开发库等等。

##3. 模块代码加载方式

以上这些框架提供了代码的组织能力，但是未能提供代码的动态加载能力。动态加载JavaScript为什么重要呢？因为随着Ajax的普及，jQuery等辅助库的出现，Web上可以做很复杂的功能，因此，单页面应用程序（SPA，Single Page Application）也逐渐多了起来。

单个的界面想要做很多功能，需要写的代码是会比较多的，但是，并非所有的功能都需要在界面加载的时候就全部引入，如果能够在需要的时候才加载那些代码，就把加载的压力分担了，在这个背景下，出现了一些用于动态加载JavaScript的框架，也出现了一些定义这类可被动态加载代码的规范。

在这些框架里，知名度比较高的是RequireJS，它遵循一种称为AMD（Asynchronous Module Definition）的规范。

比如下面这段，定义了一个动态的匿名模块，它依赖math模块

	define(["math"], function(math) {
		return {
			addTen : function(x) {
				return math.add(x, 10);
			}
		};
	}); 

假设上面的代码存放于adder.js中，当需要使用这个模块的时候，通过如下代码来引入adder：

	<script src="require.js"></script>
	<script>
		require(["adder"], function(adder) {
			//使用这个adder
		}); 
	</script>

RequireJS除了提供异步加载方式，也可以使用同步方式加载模块代码。AMD规范除了使用在前端浏览器环境中，也可以运行于nodejs等服务端环境，nodejs的模块就是基于这套规范定义的。（修订，这里弄错了，nodejs是基于类似的CMD规范的）

#三. 工业革命

这个时期，随着Web端功能的日益复杂，人们开始考虑这样一些问题：

- 如何更好地模块化开发
- 业务数据如何组织
- 界面和业务数据之间通过何种方式进行交互

在这种背景下，出现了一些前端MVC、MVP、MVVM框架，我们把这些框架统称为MV*框架。这些框架的出现，都是为了解决上面这些问题，具体的实现思路各有不同，主流的有Backbone，AngularJS，Ember，Spine等等，本文主要选用Backbone和AngularJS来讲述以下场景。

##1. 数据模型

在这些框架里，定义数据模型的方式与以往有些差异，主要在于数据的get和set更加有意义了，比如说，可以把某个实体的get和set绑定到RESTful的服务上，这样，对某个实体的读写可以更新到数据库中。另外一个特点是，它们一般都提供一个事件，用于监控数据的变化，这个机制使得数据绑定成为可能。

在一些框架中，数据模型需要在原生的JavaScript类型上做一层封装，比如Backbone的方式是这样：

	var Todo = Backbone.Model.extend({
		// Default attributes for the todo item.
		defaults : function() {
			return {
				title : "empty todo...",
				order : Todos.nextOrder(),
				done : false
			};
		},

		// Ensure that each todo created has `title`.
		initialize : function() {
			if (!this.get("title")) {
				this.set({
					"title" : this.defaults().title
				});
			}
		},

		// Toggle the 'done' state of this todo item.
		toggle : function() {
			this.save({
				done : !this.get("done")
			});
		}
	});

上述例子中，defaults方法用于提供模型的默认值，initialize方法用于做一些初始化工作，这两个都是约定的方法，toggle是自定义的，用于保存todo的选中状态。

除了对象，Backbone也支持集合类型，集合类型在定义的时候要通过model属性指定其中的元素类型。

	// The collection of todos is backed by *localStorage* instead of a remote server.
	var TodoList = Backbone.Collection.extend({
		// Reference to this collection's model.
		model : Todo,

		// Save all of the todo items under the '"todos-backbone"' namespace.
		localStorage : new Backbone.LocalStorage("todos-backbone"),

		// Filter down the list of all todo items that are finished.
		done : function() {
			return this.filter(function(todo) {
				return todo.get('done');
			});
		},

		// Filter down the list to only todo items that are still not finished.
		remaining : function() {
			return this.without.apply(this, this.done());
		},

		// We keep the Todos in sequential order, despite being saved by unordered 
		//GUID in the database. This generates the next order number for new items.
		nextOrder : function() {
			if (!this.length)
				return 1;
			return this.last().get('order') + 1;
		},

		// Todos are sorted by their original insertion order.
		comparator : function(todo) {
			return todo.get('order');
		}
	});


数据模型也可以包含一些方法，比如自身的校验，或者跟后端的通讯、数据的存取等等，在上面两个例子中，也都有体现。

AngularJS的模型定义方式与Backbone不同，可以不需要经过一层封装，直接使用原生的JavaScript简单数据、对象、数组，相对来说比较简便。

##2. 控制器

在Backbone中，是没有独立的控制器的，它的一些控制的职责都放在了视图里，所以其实这是一种MVP（Model View Presentation）模式，而AngularJS有很清晰的控制器层。

还是以这个todo为例，在AngularJS中，会有一些约定的注入，比如$scope，它是控制器、模型和视图之间的桥梁。在控制器定义的时候，将$scope作为参数，然后，就可以在控制器里面为它添加模型的支持。

	function TodoCtrl($scope) {
		$scope.todos = [{
			text : 'learn angular',
			done : true
		}, {
			text : 'build an angular app',
			done : false
		}];

		$scope.addTodo = function() {
			$scope.todos.push({
				text : $scope.todoText,
				done : false
			});
			$scope.todoText = '';
		};

		$scope.remaining = function() {
			var count = 0;
			angular.forEach($scope.todos, function(todo) {
				count += todo.done ? 0 : 1;
			});
			return count;
		};

		$scope.archive = function() {
			var oldTodos = $scope.todos;
			$scope.todos = [];
			angular.forEach(oldTodos, function(todo) {
				if (!todo.done)
					$scope.todos.push(todo);
			});
		};
	}

本例中为$scope添加了todos这个数组，addTodo，remaining和archive三个方法，然后，可以在视图中对他们进行绑定。

##3. 视图
在这些主流的MV*框架中，一般都提供了定义视图的功能。在Backbone中，是这样定义视图的：

	// The DOM element for a todo item...
	var TodoView = Backbone.View.extend({
		//... is a list tag.
		tagName : "li",

		// Cache the template function for a single item.
		template : _.template($('#item-template').html()),

		// The DOM events specific to an item.
		events : {
			"click .toggle" : "toggleDone",
			"dblclick .view" : "edit",
			"click a.destroy" : "clear",
			"keypress .edit" : "updateOnEnter",
			"blur .edit" : "close"
		},

		// The TodoView listens for changes to its model, re-rendering. Since there's
		// a one-to-one correspondence between a **Todo** and a **TodoView** in this
		// app, we set a direct reference on the model for convenience.
		initialize : function() {
			this.listenTo(this.model, 'change', this.render);
			this.listenTo(this.model, 'destroy', this.remove);
		},

		// Re-render the titles of the todo item.
		render : function() {
			this.$el.html(this.template(this.model.toJSON()));
			this.$el.toggleClass('done', this.model.get('done'));
			this.input = this.$('.edit');
			return this;
		},

		//......

		// Remove the item, destroy the model.
		clear : function() {
			this.model.destroy();
		}
	});

上面这个例子是一个典型的“部件”视图，它对于界面上的已有元素没有依赖。也有那么一些视图，需要依赖于界面上的已有元素，比如下面这个，它通过el属性，指定了HTML中id为todoapp的元素，并且还在initialize方法中引用了另外一些元素，通常，需要直接放置到界面的顶层试图会采用这种方式，而“部件”视图一般由主视图来创建、布局。

	// Our overall **AppView** is the top-level piece of UI.
	var AppView = Backbone.View.extend({
		// Instead of generating a new element, bind to the existing skeleton of
		// the App already present in the HTML.
		el : $("#todoapp"),

		// Our template for the line of statistics at the bottom of the app.
		statsTemplate : _.template($('#stats-template').html()),

		// Delegated events for creating new items, and clearing completed ones.
		events : {
			"keypress #new-todo" : "createOnEnter",
			"click #clear-completed" : "clearCompleted",
			"click #toggle-all" : "toggleAllComplete"
		},

		// At initialization we bind to the relevant events on the `Todos`
		// collection, when items are added or changed. Kick things off by
		// loading any preexisting todos that might be saved in *localStorage*.
		initialize : function() {
			this.input = this.$("#new-todo");
			this.allCheckbox = this.$("#toggle-all")[0];

			this.listenTo(Todos, 'add', this.addOne);
			this.listenTo(Todos, 'reset', this.addAll);
			this.listenTo(Todos, 'all', this.render);

			this.footer = this.$('footer');
			this.main = $('#main');

			Todos.fetch();
		},

		// Re-rendering the App just means refreshing the statistics -- the rest
		// of the app doesn't change.
		render : function() {
			var done = Todos.done().length;
			var remaining = Todos.remaining().length;

			if (Todos.length) {
				this.main.show();
				this.footer.show();
				this.footer.html(this.statsTemplate({
					done : done,
					remaining : remaining
				}));
			} else {
				this.main.hide();
				this.footer.hide();
			}

			this.allCheckbox.checked = !remaining;
		},

		//......
	});

对于AngularJS来说，基本不需要有额外的视图定义，它采用的是直接定义在HTML上的方式，比如：

	<div ng-controller="TodoCtrl">
		<span>{{remaining()}} of {{todos.length}} remaining</span>
		<a href="" ng-click="archive()">archive</a>
		<ul class="unstyled">
			<li ng-repeat="todo in todos">
				<input type="checkbox" ng-model="todo.done">
				<span class="done-{{todo.done}}">{{todo.text}}</span>
			</li>
		</ul>
		<form ng-submit="addTodo()">
			<input type="text" ng-model="todoText"  size="30"
			placeholder="add new todo here">
			<input class="btn-primary" type="submit" value="add">
		</form>
	</div>

在这个例子中，使用ng-controller注入了一个TodoCtrl的实例，然后，在TodoCtrl的$scope中附加的那些变量和方法都可以直接访问了。注意到其中的ng-repeat部分，它遍历了todos数组，然后使用其中的单个todo对象创建了一些HTML元素，把相应的值填到里面。这种做法和ng-model一样，都创造了双向绑定，即：

- 改变模型可以随时反映到界面上
- 在界面上做的操作（输入，选择等等）可以实时反映到模型里。

而且，这种绑定都会自动忽略其中可能因为空数据而引起的异常情况。

##4. 模板

模板是这个时期一种很典型的解决方案。我们常常有这样的场景：在一个界面上重复展示类似的DOM片段，例如微博。以传统的开发方式，也可以轻松实现出来，比如：

	var feedsDiv = $("#feedsDiv");

	for (var i = 0; i < 5; i++) {
		var feedDiv = $("<div class='post'></div>");
	
		var authorDiv = $("<div class='author'></div>");
		var authorLink = $("<a></a>")
			.attr("href", "/user.html?user='" + "Test" + "'")
			.html("@" + "Test")
			.appendTo(authorDiv);
		authorDiv.appendTo(feedDiv);
	
		var contentDiv = $("<div></div>")
			.html("Hello, world!")
			.appendTo(feedDiv);
		var dateDiv = $("<div></div>")
			.html("发布日期：" + new Date().toString())
			.appendTo(feedDiv);
	
		feedDiv.appendTo(feedsDiv);
	}

但是使用模板技术，这一切可以更加优雅，以常用的模板框架UnderScore为例，实现这段功能的代码为：

	var templateStr = '<div class="post">'
		+'<div class="author">'
		+	'<a href="/user.html?user={{creatorName}}">@{{creatorName}}</a>'
		+'</div>'
		+'<div>{{content}}</div>'
		+'<div>{{postedDate}}</div>'
		+'</div>';
	var template = _.template(templateStr);
	template({
		createName : "Xufei",
		content: "Hello, world",
		postedDate: new Date().toString()
	});

也可以这么定义：

	<script type="text/template" id="feedTemplate">
	<% _.each(feeds, function (item) { %>
		<div class="post">
			<div class="author">
				<a href="/user.html?user=<%= item.creatorName %>">@<%= item.creatorName %></a>
			</div>
			<div><%= item.content %></div>
			<div><%= item.postedData %></div>
		</div>
	<% }); %>
	</script>
	
	<script>
	$('#feedsDiv').html( _.template($('#feedTemplate').html(), feeds));
	</script>

除此之外，UnderScore还提供了一些很方便的集合操作，使得模板的使用更加方便。如果你打算使用BackBone框架，并且需要用到模板功能，那么UnderScore是一个很好的选择，当然，也可以选用其它的模板库，比如Mustache等等。

如果使用AngularJS，可以不需要额外的模板库，它自身就提供了类似的功能，比如上面这个例子可以改写成这样：

	<div class="post" ng-repeat="post in feeds">
		<div class="author">
			<a ng-href="/user.html?user={{post.creatorName}}">@{{post.creatorName}}</a>
		</div>
		<div>{{post.content}}</div>
		<div>
			发布日期：{{post.postedTime | date:'medium'}}
		</div>
	</div>

主流的模板技术都提供了一些特定的语法，有些功能很强。值得注意的是，他们虽然与JSP之类的代码写法类似甚至相同，但原理差别很大，这些模板框架都是在浏览器端执行的，不依赖任何服务端技术，即使界面文件是.html也可以，而传统比如JSP模板是需要后端支持的，执行时间是在服务端。

##5. 路由

通常路由是定义在后端的，但是在这类MV*框架的帮助下，路由可以由前端来解析执行。比如下面这个Backbone的路由示例：

	var Workspace = Backbone.Router.extend({
		routes: {
			"help":				 "help",	// #help
			"search/:query":		"search",  // #search/kiwis
			"search/:query/p:page": "search"   // #search/kiwis/p7
		},
		
		help: function() {
			...
		},
		
		search: function(query, page) {
			...
		}	
	});

在上述例子中，定义了一些路由的映射关系，那么，在实际访问的时候，如果在地址栏输入"#search/obama/p2"，就会匹配到"search/:query/p:page"这条路由，然后，把"obama"和"2"当作参数，传递给search方法。

AngularJS中定义路由的方式有些区别，它使用一个$routeProvider来提供路由的存取，每一个when表达式配置一条路由信息，otherwise配置默认路由，在配置路由的时候，可以指定一个额外的控制器，用于控制这条路由对应的html界面：

	app.config(['$routeProvider',
	function($routeProvider) {
		$routeProvider.when('/phones', {
			templateUrl : 'partials/phone-list.html',
			controller : PhoneListCtrl
		}).when('/phones/:phoneId', {
			templateUrl : 'partials/phone-detail.html',
			controller : PhoneDetailCtrl
		}).otherwise({
			redirectTo : '/phones'
		});
	}]); 

注意，在AngularJS中，路由的template并非一个完整的html文件，而是其中的一段，文件的头尾都可以不要，也可以不要那些包含的外部样式和JavaScript文件，这些在主界面中载入就可以了。

##6. 自定义标签

用过XAML或者MXML的人一定会对其中的可扩充标签印象深刻，对于前端开发人员而言，基于标签的组件定义方式一定是优于其他任何方式的，看下面这段HTML：

	<div>
		<input type="text" value="hello, world"/>
		<button>test</button>
	</div>

即使是刚刚接触这种东西的新手，也能够理解它的意思，并且能够照着做出类似的东西，如果使用传统的面向对象语言去描述界面，效率远远没有这么高，这就是在界面开发领域，声明式编程比命令式编程适合的最重要原因。

但是，HTML的标签是有限的，如果我们需要的功能不在其中，怎么办？在开发过程中，我们可能需要一个选项卡的功能，但是，HTML里面不提供选项卡标签，所以，一般来说，会使用一些li元素和div的组合，加上一些css，来实现选项卡的效果，也有的框架使用JavaScript来完成这些功能。总的来说，这些代码都不够简洁直观。

如果能够有一种技术，能够提供类似这样的方式，该多么好呢？

	<tabs>
		<tab name="Tab 1">content 1</tab>
		<tab name="Tab 2">content 2</tab>
	</tabs>

回忆一下，我们在章节1.4 组件化的萌芽 里面，提到过一种叫做HTC的技术，这种技术提供了类似的功能，而且使用起来也比较简便，问题是，它属于一种正在消亡的技术，于是我们的目光投向了更为现代的前端世界，AngularJS拯救了我们。

在AngularJS的首页，可以看到这么一个区块“Create Components”，在它的演示代码里，能够看到类似的一段：

	<tabs>
		<pane title="Localization">
			...
		</pane>
		<pane title="Pluralization">
			...
		</pane>
	</tabs>

那么，它是怎么做到的呢？秘密在这里：
	
	angular.module('components', []).directive('tabs', function() {
		return {
			restrict : 'E',
			transclude : true,
			scope : {},
			controller : function($scope, $element) {
				var panes = $scope.panes = [];
	
				$scope.select = function(pane) {
					angular.forEach(panes, function(pane) {
						pane.selected = false;
					});
					pane.selected = true;
				}
	
				this.addPane = function(pane) {
					if (panes.length == 0)
						$scope.select(pane);
					panes.push(pane);
				}
			},
			template : '<div class="tabbable">'
				+ '<ul class="nav nav-tabs">' 
				+ '<li ng-repeat="pane in panes" ng-class="{active:pane.selected}">' 
				+ '<a href="" ng-click="select(pane)">{{pane.title}}</a>' 
				+ '</li>' 
				+ '</ul>' 
				+ '<div class="tab-content" ng-transclude></div>' 
				+ '</div>',
			replace : true
		};
	}).directive('pane', function() {
		return {
			require : '^tabs',
			restrict : 'E',
			transclude : true,
			scope : {
				title : '@'
			},
			link : function(scope, element, attrs, tabsCtrl) {
				tabsCtrl.addPane(scope);
			},
			template : '<div class="tab-pane" ng-class="{active: selected}" ng-transclude>' + '</div>',
			replace : true
		};
	})

这段代码里，定义了tabs和pane两个标签，并且限定了pane标签不能脱离tabs而单独存在，tabs的controller定义了它的行为，两者的template定义了实际生成的html，通过这种方式，开发者可以扩展出自己需要的新元素，对于使用者而言，这不会增加任何额外的负担。

#四. 一些想说的话

###关于ExtJS

注意到在本文中，并未提及这样一个比较流行的前端框架，主要是因为他自成一系，思路跟其他框架不同，所做的事情，层次介于文中的二和三之间，所以没有单独列出。

###写作目的

在我10多年的Web开发生涯中，经历了Web相关技术的各种变革，从2003年开始，接触并使用到了HTC，VML，XMLHTTP等当时比较先进的技术，目睹了网景浏览器的衰落，IE的后来居上，Firefox和Chrome的逆袭，各类RIA技术的风起云涌，对JavaScript的模块化有过持续的思考。未来究竟是什么样子？我说不清楚，只能凭自己的一些认识，把这些年一些比较主流的发展过程总结一下，供有需要了解的朋友们作个参考，错漏在所难免，欢迎大家指教。

个人邮箱：xu.fei@outlook.com  
新浪微博：http://weibo.com/sharpmaster
