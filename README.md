#InsideSales.com AngularJS Best Practices

##Introduction

The goal of this style guide is to present a set of best practices and style guidelines for IS AngularJS applications.

In this style guide you won't find common guidelines for JavaScript development.

For AngularJS development recommended style is [Google's JavaScript style guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml). We also have an existing internal guide which is very similar to the Google guide.

In AngularJS's GitHub wiki there is a similar section by [ProLoser](https://github.com/ProLoser), you can check it [here](https://github.com/angular/angular.js/wiki).

##General

* Use:
    * `$timeout` instead of `setTimeout`
        * `$interval` instead of `setInterval`
	    * `$window` instead of `window`
	        * `$document` instead of `document`
		    * `$http` instead of `$.ajax`

		    This will make your testing easier and in some cases prevent unexpected behaviour (for example, if you missed `$scope.$apply` in `setTimeout`).

		    * Automate your workflow using tools like:
		        * [Yeoman](http://yeoman.io)
			    * [Grunt](http://gruntjs.com)
			        * [Bower](http://bower.io)

				* Use promises (`$q`) instead of callbacks. It will make your code look more elegant and clean, and save you from callback hell.
				* Use `$resource` instead of `$http` when possible. The higher level of abstraction will save you from redundancy.
				* Use an AngularJS pre-minifier (like [ngmin](https://github.com/btford/ngmin) or [ng-annotate](https://github.com/olov/ng-annotate)) for preventing problems after minification.
				* Don't use globals. Resolve all dependencies using Dependency Injection.
				* Do not pollute your `$scope`. Only add functions and variables that are being used in the templates.
				* Prefer the usage of [controllers instead of `ngInit`](https://github.com/angular/angular.js/pull/4366/files). The only appropriate use of `ngInit` is for aliasing special properties of `ngRepeat`. Besides this case, you should use controllers rather than `ngInit` to initialize values on a scope.
				* Do not use `$` prefix for the names of variables, properties and methods. This prefix is reserved for AngularJS usage.

				##Modules

				* Modules should be named with lowerCamelCase. For indicating that module `b` is submodule of module `a` you can nest them by using namespacing like: `a.b`.

				##Controllers

				* Do not manipulate DOM in your controllers, this will make your controllers harder for testing and will violate the [Separation of Concerns principle](https://en.wikipedia.org/wiki/Separation_of_concerns). Use directives instead.
				* The naming of the controller is done using the controller's functionality (for example shopping cart, homepage, admin panel) and the substring `Controller` in the end. The controllers are named UpperCamelCase (`HomePageController`, `ShoppingCartController`, `AdminPanelController`, etc.).
				* The controllers should not be defined as globals (no matter AngularJS allows this, it is a bad practice to pollute the global namespace).
				* Use array syntax for controller definitions:


				```JavaScript
				module.controller('MyCtrl', ['dependency1', 'dependency2', ..., 'dependencyn', function (dependency1, dependency2, ..., dependencyn) {
				  //...body
				  }]);
				  ```


				  Using this type of definition avoids problems with minification. You can automatically generate the array definition from standard one using tools like [ng-annotate](https://github.com/olov/ng-annotate) (and grunt task [grunt-ng-annotate](https://github.com/mzgol/grunt-ng-annotate)).
				  * Use the original names of the controller's dependencies. This will help you produce more readable code:

				  ```JavaScript
				  module.controller('MyCtrl', ['$scope', function (s) {
				    //...body
				    }]);
				    ```

				    is less readable than:

				    ```JavaScript
				    module.controller('MyCtrl', ['$scope', function ($scope) {
				      //...body
				      }]);
				      ```

				      This especially applies to a file that has so much code that you'd need to scroll through. This would possibly cause you to forget which variable is tied to which dependency.

				      * Make the controllers as lean as possible. Abstract commonly used functions into a service.
				      * Communicate within different controllers using method invocation (possible when children wants to communicate with parent) or `$emit`, `$broadcast` and `$on` methods. The emitted and broadcasted messages should be kept to a minimum.
				      * Make a list of all messages which are passed using `$emit`, `$broadcast` and manage it carefully because of name collisions and possible bugs.
				      * When you need to format data encapsulate the formatting logic into a [filter](#filters) and declare it as dependency:

				      ```JavaScript
				      module.filter('myFormat', function () {
				        return function () {
					    //body...
					      };
					      });

					      module.controller('MyCtrl', ['$scope', 'myFormatFilter', function ($scope, myFormatFilter) {
					        //body...
						}]);
						```

						##Directives

						* Name your directives with lowerCamelCase
						* Use `scope` instead of `$scope` in your link function. In the compile, post/pre link functions you have already defined arguments which will be passed when the function is invoked, you won't be able to change them using DI. This style is also used in AngularJS's source code.
						* Use custom prefixes (Power Standings uses the prefix 'ps') for your directives to prevent name collisions with third-party libraries.
						* Do not use `ng` or `ui` prefixes since they are reserved for AngularJS and AngularJS UI usage.
						* DOM manipulations must be done only through directives.
						* Create an isolated scope when you develop reusable components.
						* Use directives as attributes or elements instead of comments or classes, this will make your code more readable.
						* Use `$scope.$on('$destroy', fn)` for cleaning up. This is especially useful when you're wrapping third-party plugins as directives.
						* Do not forget to use `$sce` when you should deal with untrusted content.

						##Filters

						* Name your filters with lowerCamelCase.
						* Make your filters as light as possible. They are called often during the `$digest` loop so creating a slow filter will slow down your app.
						* Do a single thing in your filters, keep them coherent. More complex manipulations can be achieved by piping existing filters.

						##Services

						* Use camelCase (lower) to name your services.
						* Encapsulate business logic in services.
						* Services representing the domain preferably a `service` instead of a `factory`. In this way we can take advantage of the "klassical" inheritance easier:

						```JavaScript
						function Human() {
						  //body
						  }
						  Human.prototype.talk = function () {
						    return "I'm talking";
						    };

						    function Developer() {
						      //body
						      }
						      Developer.prototype = Object.create(Human.prototype);
						      Developer.prototype.code = function () {
						        return "I'm coding";
							};

							myModule.service('Human', Human);
							myModule.service('Developer', Developer);

							```

							* For session-level cache you can use `$cacheFactory`. This should be used to cache results from requests or heavy computations.

							##Templates

							* Think about using `ng-bind` or `ng-cloak` instead of simple `{{ }}` to prevent flashing content on initial pages of multi view apps.
							* Avoid writing complex expressions in the templates.
							* When you need to set the `src` of an image dynamically use `ng-src` instead of `src` with `{{}}` template.
							* Instead of using scope variable as string and using it with `style` attribute with `{{ }}`, use the directive `ng-style` with object-like parameters and scope variables as values:

							```HTML
							<script>
							...
							$scope.divStyle = {
							  width: 200,
							    position: 'relative'
							    };
							    ...
							    </script>

							    <div ng-style="divStyle">my beautifully styled div which will work in IE</div>;
							    ```

							    ##Routing

							    * Use `resolve` to resolve dependencies before the view is shown.

							    ##Testing

							    //ToDo

							    ##Contribution

							    Since the goal of this style guide is to be a team effort, contributions are teh awesome.
							    For example, you can contribute by writing the Testing section
