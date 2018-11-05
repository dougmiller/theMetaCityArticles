# A reasonable approach to growing an AngularJS site/app

Its just JavaScript and HTML.

The pain point is everything on one page. All the examples are setup like this. This is fine for spikes and proofs of concepts however once you move into any kind of serious development stage you are going to need a better approach. This is my approach which I think is quite reasonable and seems to be what other people who are far better than me seem to do to.

The general idea is break out each controller, factory, directive, filter, etc into it's own file and each type into it's own folder. Simple enough.

	app/
    	js/
    	    controllers/
    	        PersonCtrl.js
    	        HouseCtrl.js
    	    services/
    	        PersonSrvs.js
    	        HouseSrvs.js
    	    directives/
    	         FinancialGraph.js
    	         UtilitiesUsageGraph.js
    	     filters/
    	          SearchAddresses.js
    	          SearchPeoplesNames.js
    	     controllers.js
    	     services.js
    	     directives.js
    	     filters.js


Each of the <code>controllers.js</code>, <code>services.js</code> etc setup the DI point to the respective controllers and services etc:

	angular.module('theApp.controllers', []);

and then in the subfile:

	angular.module('theApp.controllers').controller('PersonCtrl', ['$scope', '$http', function($scope, $http){
	...

This approach has a few obvious advantages: it is easy to reason about what each file does and where/how it will affect the app. This keeps concerns separated quite well.

What is with the Crtl and Sevs suffix? There is a fair chance of a collision between the names of functions in controllers and services. By adding a suffix on the file, this will help when your IDE tries to auto-complete and there is ambiguity about where the function is, that's all.

There is a little bit more setup to have this work properly.

You need to add each file to the <code>index.html</code> so that it is loaded.

	<script src="js/controllers.js">
	<script src="js/controllers/PersonCtrl.js">
	<script src="js/controllers/HouseCtrl.js">

	<script src="js/services.js">
	<script src="js/services/PersonSrvs.js">
	<script src="js/services/HouseSrvs.js">

	<script src="js/directives.js">
	<script src="js/directives/FinancialGraph.js">
	<script src="js/directives/UtilitiesUsageGraph.js">

	<script src="js/filters.js">
	<script src="js/filters/SearchAddresses.js">
	<script src="js/filters/SearchPeoplesNames.js">

Don't put in <code>defer</code> or <code>async</code> on these as the order of loading these matter greatly. If you are running a hybrid app, you want to block on loading these files.

What is with the <code>controllers.js</code> file? If we are separating out these into individual files why do we still need this? This will act as the point that the other files will use for dependency injection. In the <code>controllers.js</code> file we setup an empty injection point with any dependencies we need.

In each of the controllers we insert ourselves as a dependency.

Do this for all the controllers and all of the other files in the other types (services, directives, filters etc).

Congratulations you now have included all the files you need.
