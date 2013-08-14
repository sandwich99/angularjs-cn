# 其他关注点

在这一章中, 我们将看一切目前Angular所实现的其他有用的特性, 但是我们不会涵盖所有的或者深入的章节和例子.

## $location

到现在为止, 你已经看到了不少使用AngularJS中的`$location`服务的例子. 它们大多数都只是短暂的一撇--在这里访问, 那里设置. 在这一小节, 我们将深入研究AngularJS中的`$location`服务时什么, 以及什么时候你应该使用它, 什么时候不应该使用它.

`$location`服务是一个存在于任何浏览器中的`window.location`的包装器. 那么为什么你应该使用它而不是直接使用`window.location`呢?

**不再使用全局状态**

`window.location`是一个使用全局状态的很好的例子(实际上, 浏览器中的`window`和`document`对象都是很好的例子). 一旦你的应用程序中有全局的状态(通常我们都说全局变量), 它的测试, 维护和工作都会变得困难(即使不是现在, 从长远来看它肯定是一个潜在的隐患). `$location`服务隐藏了这个潜在的隐患(也就是我们所谓的全局状态), 并且允许你通过注入mocks到你的单元测试中来测试你的浏览器位置信息.

**API**

`window.location`让你能够完全访问浏览器位置信息的内容. 也就是说, `window.location`给你一个字符串而`$location`服务给你提供了更好的服务, 它提供了类似于jQuery的setters和getters让你能够使用它以一个干净的方式工作.

**AngularJS集成**

如果你使用`$location`, 你可以在任何你希望使用的时候使用它. 但是如果直接使用`window.location`, 在有变化时你必须负责通知给AngularJS, 并且还要监听这些改变/变化.

**HTML5集成**

`$location`服务会在HTML5 APIs在浏览器中可用时智能的识别并使用它们. 如果它们不可用, 它会降级使用默认的用法.

那么什么时候你应该使用`$location`服务呢? 任何你想反应URL变化的时候(它并不是通过`$routes`来覆盖的, 而且你应该主要用于基于URL工作的视图中), 以及在浏览器中响应当前URL变化的时候使用.

让我们考虑使用一个小例子来看看你应该如何在一个实际的应用程序中使用`$location`服务. 想象一下我们有一个`datepicker`, 并且当我们选择一个日期时, 应用程序导航到某个URL. 让我们一起来看看它看起来可能是什么样子:

	// Assume that the datepicker calls $scope.dateSelected with the date
	$scope.dateSelected = function(dateTxt) {
		$location.path('filteredResults?startDate=' + dateTxt);
		// If this were being done in the callback for
		// an external library, like jQuery, then we would have to
		$scope.$apply();
	};
	
####用或者不用$apply?

对于AngularJS开发者来说什么时候调用`$scope.$apply()`, 什么时候不能调用它是比较混乱的. 互联网上的建议和谣言非常猖獗. 在本小节我们将让它变得非常清楚.

但是首先让我们先尝试以一个简单的形式使用`$apply`.

`Scope.$apply`就像一个延迟的worker. 我们会告诉它有很多工作要做, 它负责响应并确保更新绑定和所有变化的视图效果. 但并不是所有的时间都只做这项工作, 它只会在它觉得有足够的工作要做时才会做. 在所有的其他情况下, 它只是点点头并标记在稍候处理. 它只是在你给它指示时并显示的告诉它处理实际的工作. AngularJS只是定期在它的声明周期内做这些, 但是如果调用来自于外部(比如说一个jQuery UI事件), `scope.$apply`只是做一个标记, 但并不会做任何事. 这就是为什么要调用`scope.$apply`来告诉它"嘿!你现在需要做这件事, 而不是等待!".

这里有四个快速的提示告诉你应该什么时候(以及如何)调用`$apply`.

+ **不要**始终调用它. 当AngularJS发现它将导致一个异常(在其`$digest`周期内, 我们调用它)时调用`$apply`. 因此"有备无患"并不是你希望使用的方法.

+ 当控制器在AngularJS外部(DOM时间, 外部回调函数如jQuery UI控制器等等)调用AngularJS函数时**调用**它. 对于这一点, 你希望告诉AngularJS来更新它自身(模型, 视图等等), 而`$apply`就是做这个的.

+ 只要可能, 通过传递给`$apply`来执行你的代码或者函数, 而不是执行函数, 然后调用`$apply()`. 例如, 执行下面的代码:

	$scope.$apply(function(){
		$scope.variable1 = 'some value';
		excuteSomeAction();
	});
	
而不是下面的代码:

	$scope.variable1 = 'some value';
	excuteSomeAction();
	$scope.$apply();
	
尽管这两种方式将有相同的效果, 但是它们的方式明显不同.

第一个会在`excuteSomeAction`被调用时将捕获发生的任何错误,  而后者则会瞧瞧的忽略此类错误. 只有使用第一种方式时你才会从AngularJS中获取错误的提示.

+ kaov使用类似的`safeApply`:

	$scope.safeApply = function(fn){
		var phase = this.$root.$$phase;
		if(phase == '$apply' || phase == '$digest') {
			if(fn && (typeof(fn) === 'function')) {
				fn();
			}
		}else{
			this.$apply(fn);
		}
	};
	
你可以在顶层作用域或者根作用域中捕获到它, 然后在任何地方使用`$scope.$safeApply`函数. 一直都在讨论这个, 希望在未来的版本中这会称为默认的行为.

是否那些其他的方法也可以在`$location`对象中使用呢? 表7-1包含了一个快速的参考用于让你绑定使用.

让我们来看看`$location`服务是如何表现的, 如果浏览器中的URL时`http://www.host.com/base/index.html#!/path?param1=value1#hashValue`.

Table 7-1 Functions on the $location service

<table>
	<thead>
		<tr>
			<th>Getter Function</th>
			<th>Getter Value</th>
			<th>Setter Function</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>absUrl()</td>
			<td><i>http://www.host.com/base/index.html#!/path?param1=value1#hashValue,</i></td>
			<td>N/A</td>
		</tr>
		<tr>
			<td>hash()</td>
			<td>hashValue</td>
			<td>hash('newHash')</td>
		</tr>
		<tr>
			<td>host()</td>
			<td>www.host.com</td>
			<td>N/A</td>
		</tr>
		<tr>
			<td>path()</td>
			<td>/path</td>
			<td>path('/newPath')</td>
		</tr>
		<tr>
			<td>protocol()</td>
			<td>http</td>
			<td>N/A</td>
		</tr>
		<tr>
			<td>search()</td>
			<td>{'a':'b'}</td>
			<td>search({'c':'def'})</td>
		</tr>
		<tr>
			<td>url()</td>
			<td>/path?param1=value1?hashValue</td>
			<td>url('/newPath?p2=v2')</td>
		</tr>
	</tbody>
</table>

表7-1的Setter Function一列提供了一个值样本表示setter函数与其的对象类型.

注意`search()`setter函数还有一些操作模式:

+ 基于一个`object<string, string>`调用`search(searchObj)`表示所有的参数和它们的值.
+ 调用`search(string)`将直接在URL上设置URL的参数为`q=String`.
+ 使用一个字符串和值调用`search(param, value)`来设置URL中一个特定的收缩参数(或者使用null调用来移除参数).

使用任意一个这些setter函数并不意味着window.location将立即获得改变. `$location`服务会在Angular生命周期内运行,  所有的位置改变将积累在一起并在周期的后期应用. 所以可以随时作出改变, 一个借一个的, 而不用担心用户会看到一个不断闪烁和不断变更的URL的情况.

