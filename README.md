# vertx-mvc-specifications
Specification wiki for a MVC-layer on top of Vert.x 3 stack


## Objectives

The vertx-mvc project aims at porting some of the key features existing in MVC frameworks such as Rails or Grails into the Vert.x environment.

Some of the most useful features we can think of by now are (non-exhaustive list obviously) : 

* Scaffolding an application without writing code
* Managing different environments : development, production, etc. out of the box. So that users can benefit from a specific environment (database access/credentials, minified assets, etc.) without having a ton of configuration files of boilerplate code.
* Providing a simple way to test your code with real data (fixtures) without having to import/export database snapshots
* Organizing code in a MVC way (action controllers, view controllers) in a descriptive way, not hard-coded way
* Providing some convention-over-configuration features. By naming a controller the right way, you'll be guaranteed the right view will be bound to it
* Having a set of useful methods in the controller layer like Grails' `render as` method
* Being able to bind with the minimum amount of code the domain-layer to an end-user REST API
* Providing some of the most useful ActiveRecord, GORM features like findBy, save, update directly on model objects or classes to users
* hot-reload, at least for views
* packaging the whole thing into one single deployment file (thanks fatJar! we shouldn't be in trouble with this one)
* i18n out of the box, especially for views, without having to redeploy or rebuild the application. Language detection through Accept-Language, Cookies etc.

Some of the advanced features that would be awesome to deal with :

* Handling easy data migration when the domain (model) layer changes so that users can just 
* TODO

Keep in mind that this project brings an opiniated framework (MVC) on-top of an unoponiated one. 
But the key features of Vert.x or Apex should still remain accessible under the hood.

This way, end-users could benefit from the MVC-layer to quickly scaffold an MVC application, but could still benefit from the unopiniated way of thinking from Vert.x once they're stuck in some corner-case.

For example : we shouldn't be using a custom view layer like GSP but benefit from the existing template engines in Apex to let users decide which template engine they'd like to use.
In this kind of example, we should decide which would be the default template, but provide some easy ways (convention over configuration) to change templates. In this example, the controller could find the files matching the correct name and, depending on the extension, use the right template engine (.jade => jade, etc.)


And this should be the kind of rule to keep in mind for every feature in vertx-mvc. 

Another example : An APIController could return data as json by default, but could easily be switched to XML. The pagination of data in an API could be the same as GitHub by default but could be changed to another way of paginating data (not through HTTP headers but in HTTP payload, ...)


## Scaffolding

One of the interesting features Rails provides is scaffolding.

### Examples

```rails create model Book```

This will generate : 

* a class containing the definition of the Book class with full inheritance and stuff
* a db migration script to reflect the changes to the existing production application

### Technical implementation

Let's set db migration aside for now. Just remember projects like Flyweight or Liquibase exist in the Java world.

The best way to achieve that in a Java environment imo is a Maven or Gradle plugin.

I think writing a Gradle plugin should be easier to start with in the current state of both projects.
I think it could be easier for end-users to customize scaffolding through a DSL than XML configuration of a Maven plugin.

This way, we could just add some Gradle tasks like : 

```gradle createController io.vertx.myexample.MyController```

And I guess that's the choice Grails developpers made for Grails 3.0.

For end-users, they'd just have to add a Gradle plugin on top of their application to use scaffolding.

```
plugins {
  id 'io.vertx.vertxmvc' version '0.1'
}
```

And they could just customize the way code generation works like this :

```
generateController {
  defaultImplementation "APIController" // maybe default would be ViewController
  basePath "myOwnControllers/theOneIGenerated" // instead of "/controllers" for example
  prefix "com.peanuts.controllers" // would prefix every generated controllers' package
}
```

This could give us a very nice way to implement convention over configuration for the scaffolding part of the framework.


WIP : list of actions to implement for version 1.0

* createController
* createView
* createDomain
* createFixture
* buildAssets (would generate productio-ready front-end stuff like minified JS, CSS generated from SASS files, ...)
* start(environment) with shortcuts like startDev, startProd, ...
* migrateDBSchema(environment)

## The Controller layer

### State of the art
Apex already provides us a very nice stack to build the framwork on.

The way routes are described, especially with wildcards and named parameters has to be used. We should also take benefit of chaining handlers to implement filters (beforeFilter, afterFilter).

Let's think about this layer as a simple "other" way to describe routes than the one (programmatic, since unopinionated) Apex uses. This means end-users should always be able to use Apex directly without any issue.

In MVC frameworks (Spring MVC for instance), routes are usually defined in three ways :

* naming conventions : `SnoopyController` will automatically handle requests hitting "/snoopy"
* annotations : `@Controller` for Spring, `@Path` for resteasy, etc. We should define our own set of annotations and can discuss which approach we'd prefer.
* a routing file or DSL. Let's forget about the good old `web.xml` and think about [that kind of stuff](https://github.com/resthub/springmvc-router). Or [what Django does](https://docs.djangoproject.com/en/1.7/topics/http/urls/#example). 

This could be the first brick to implement since I think it's gonna be the easiest one. And also since it's the pivot of any MVC application.

### Implementations

The easiest way to start with is imo using annotations and reflection. Even though it's Java-dependent. 

That's a very important decision : that means the vertx-mvc framework won't be available for other languages Vert.x supports. (except Groovy, maybe Scala ? idk)

The user defines Controllers and routes through a set of annotations provided by the vertx-mvc library. Say `io.vertx.vertxmvc.annotations`.

In some configuration file the user will define the packages in which controllers are written.

(warning:last time I checked, reflective recursion through packages was kinda messsy to deal with... re-check).

At startup, verticle will look for annotated controllers and instantiate every one of them (singleton). It's gonna read routes and dynamically create Vert.x handlers.

```
class MVCRoute implements Handler<RoutingContext> {
  String path
  HttpMethod method
  Set<Method> beforeFilters
  Set<Method> afterFilters
  Method mainHandler
  Object controllerSingleton
  
  static MVCRoute fromAnnotatedController(Clazz controller, Method mainMethod) {
    // constructor
  }
  
  addFilter(Method filter, Filter.Type type = Filter.Type.BEFORE){
    // ...
  }
  
  @Override
  handle(RoutingContext context) {
    try {
      beforeFilters.each {
        // invoke
      }
      // invoke mainHandler
      afterFilters.each {
        // invoke
      }
    } catch (ReflectionExceptionStuff all) {
      context.fail(all)
      context.reponse().setStatusCode(500) 
      // and have a clean way to display exceptions
    }
  }
  
  // ...
}
```


```
// At startup
List<MVCRoute> routes = scanRoutes(userDefinedPackages)
routes.each { MVCRoute route ->
  Router.route(route.toVertxRoute()).handler { RoutingContext context ->
    route.invoke(context);
    context.next() // depends on the type of route !! some will just do some stuff, some will end the response (think about render(...) )
  }
}
```

#### Main issues

1. Dependency injection : 

Both Spring and Resteasy inject parameters dynamically into the user's-defined method. 

```
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
}
```
Same goes for resteasy.
It gives access to alot of objects through annotations (request parameters, headers, the httpserverequest, ...)

In the case of Vert.x, everything is present under the `RoutingContext` object.

In a first implementation, we could just expose the RoutingContext directly but it might be a bad idea since it gives access to some primitives we'd like to hide from an end-users (like `context.next()` for instance).

So I need your opinion on that point : 

* Should we expose the RoutingContext directly ? (easier to start with : one single parameter for every user's method, might be dangerous)
* Should we use a *Facade* for the RoutingContext ? To avoid exposing it. And then let users use annotation to inject context attributes (like request, response, ...)

The first one is kinda easy. The second one is pretty hard to achieve through reflection but is maybe what users are used to.

#### Expose the Vert.x API

RoutingContext is one thing, the Vert.x objects (especially the eventBus) will also be needed for end-users to do some practical stuff.
If we instantiate everything and hide completely the Verticle from the end-user he won't be able to interact with the Vert.x API (apart from the RoutingContext, see above).

End-users will need a way to access at least the verticle configuration and the eventbus.

Either we just let him instanciate his own verticle and say `bootstrapMVC(myControllersPackages)` but then he still won't be able to access the Vert.x API from within his controllers.

So maybe the second approach exposed above is a better guess. The facade will be responsible for injecting the eventbus if the user asks for it, etc.

#### Asynchronous work

That kind of approach works perfectly in a synchronous world. But in our case, users will want to do some synchronous stuff, say ask for something over the eventBus to handle an HTTP request properly, or access the file system through vertx.fileSystem() (non-blocking).

In order to achieve that, we should let controller's methods return some kind of promise and execute context.next (and afterFilters, etc...) only once the promise is fulfilled.

Either we provide a promise API or rely on Vertx's Futures. 
And let us be smart with users. If the methods returns nothing we assume it's some eays/blocking stuff and should call next() right after the method is invoked through reflection.

### Roadmap

* First, we should specify the kinf od annotations we need.
* Then, create a simple example with no filter, just ending the response saying "Hello world"
* Then, chaining filters and let the it deal with analyzing the request headers, preparing response payload, and in some afterFilter sending the payload in a proper HTTPResponse

Once this is done, we could have a simple example of a JsonApiController which fails when no `Accept:application/json` header is present, then puts something as a payload (say "Hello world") then sends the result as a Json String in the http response with the correct headers set.

Then, we'll have to deal with asynchronous stuff and promises to chain handlers (filters/payload) properly. And maybe adjust the facade interface.



## The domain layer
