# The Controller layer

## State of the art
Apex already provides us a very nice stack to build the framwork on.

The way routes are described, especially with wildcards and named parameters has to be used. We should also take benefit of chaining handlers to implement filters (beforeFilter, afterFilter).

Let's think about this layer as a simple "other" way to describe routes than the one (programmatic) Apex uses. This means end-users should always be able to use Apex directly without any issue.

In MVC frameworks (Spring MVC for instance), routes are usually defined in three ways :

* naming conventions : `SnoopyController` will automatically handle requests hitting "/snoopy"
* annotations : `@Controller` for Spring, `@Path` for resteasy, etc. We should define our own set of annotations and can discuss which approach we'd prefer.
* a routing file or DSL. Let's forget about the good old `web.xml` and think about [that kind of stuff](https://github.com/resthub/springmvc-router). Or [what Django does](https://docs.djangoproject.com/en/1.7/topics/http/urls/#example). 

This could be the first brick to implement since I think it's gonna be the easiest one. And also since it's the pivot of any MVC application.

## Implementations

The easiest way to start with is imo using annotations and reflection. Even though it's Java-dependent. 

That's a very important decision : that means the vertx-mvc framework won't be available for other languages Vert.x supports. (except Groovy, maybe Scala ? idk)

The user defines Controllers and routes through a set of annotations provided by the vertx-mvc library. Say `io.vertx.vertxmvc.annotations`.

In some configuration file the user will define the packages in which controllers are written.

(warning:last time I checked, reflective recursion through packages was kinda messsy to deal with... have to re-check).

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
    if (route.isChainable()){
      context.next() // depends on the type of route !! some will just do some stuff, some will end the response (think about render(...) )
    }
  }
}
```

## Main issues

1. Dependency injection : 

Both Spring and Resteasy inject parameters dynamically into the user's-defined method. 

Spring example:
```
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
}
```
Same goes for resteasy:it gives access to alot of objects through annotations (request parameters, headers, the httpserverequest, cookies, ...)

In the case of Vert.x, everything is present under the `RoutingContext` object.

First guess: we could just expose the RoutingContext directly.

But it might be a bad idea since it gives access to some primitives we'd like to hide from an end-users (like `context.next()` for instance).

So I need your opinion on that point : 

* Should we expose the RoutingContext directly ? (easier to start with : one single parameter for every user's method, might be dangerous)
* Should we use a *Facade* for the RoutingContext ? To avoid exposing it. And then let users use annotation to inject the context attributes they need (like request, response, ...)

The first one is kinda easy. The second one is pretty hard to achieve through reflection but is maybe what users are used to. (since Spring does that, Reasteasy too, maybe Grails too).

### Expose the Vert.x API

RoutingContext is one thing, the Vert.x objects (especially the eventBus) will also be needed for end-users to do some practical stuff.
If we instantiate everything and hide completely the Verticle from the end-user (see code example above) he won't be able to interact with the Vert.x API (apart from the RoutingContext, see above).

End-users will need a way to access at least the verticle configuration and the eventbus.

First idea: we just let him instanciate his own verticle and write `bootstrapMVC(myControllersPackages)`.
But then, he still won't be able to access the Vert.x API from within his controllers.

So maybe the second approach exposed above is a better guess. The *facade* will be responsible for injecting the eventbus if the user asks for it, etc.

But still, should we let the user write his own verticle and bootstrap from here ? 

### Asynchronous work

That kind of approach (Spring, RestEasy) works perfectly in a synchronous world. But in our case, users will want to do some asynchronous stuff, say ask for something over the eventBus, or access the file system through vertx.fileSystem() (non-blocking).

In order to achieve that, we should let controller's methods return some kind of promise and execute context.next (and afterFilters, etc...) only once the promise is fulfilled.

Either we provide a promise API or rely on Vertx's Futures. 

And let us be smart with users. If the methods returns nothing we assume it's some easy/blocking stuff and should call next() right after the method is invoked through reflection. If the method returns a promise, then we should wait for the promise to succeed to call the next handlers ? (but we might block the event loop ?)

## Roadmap

* First, we should specify the kind of annotations we need.
* Then, create a simple example with no filter, just ending the response saying "Hello world"
* Then, chaining filters and let it deal with analyzing the request headers, preparing response payload, and in some afterFilter sending the payload in a proper http response.

Once this is done, we could have a simple example of a JsonApiController which fails when no `Accept:application/json` header is present, then puts something as a payload (say "Hello world") then sends the result as a Json String in the http response with the correct headers set.

Then, we'll have to deal with asynchronous stuff and promises to chain handlers (filters/payload) properly. And maybe adjust the facade interface.
