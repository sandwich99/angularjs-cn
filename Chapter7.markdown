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