# Scaffolding

One of the interesting features Rails provides is scaffolding.

## Examples

```rails create model Book```

This will generate : 

* a class containing the definition of the Book class with full inheritance and stuff
* a db migration script to reflect the changes to the existing production application
* some testing utilities (at least for Grails, don't know about Rails)

Same goes for the other layers: views and controllers.

There are alot of other useful commands for creating fixtures, installing plugins, ... But to get started, these are the most interesting ones.

## Technical implementation

Let's set db migration aside for now. Just remember projects like [Flyway](http://flywaydb.org/documentation/) or [Liquibase](http://www.liquibase.org/) exist in the Java world.

The best way to implement these commands in a Java environment imo is a Maven or Gradle plugin.

I think writing a Gradle plugin should be easier to start with in the current state of both projects.
I also think it could be easier for end-users to customize scaffolding through a DSL than XML configuration of a Maven plugin.

This way, we could just add some Gradle tasks like : 

```gradle createController io.vertx.myexample.MyController```

And I guess that's the choice Grails developpers made for Grails 3.0.

For end-users, they'd just have to add a Gradle plugin on top of their application to use scaffolding. 
Moreover, this must be a completely different project than the core MVC API. (in a perfect world, there would be both a Gradle plugin and a Maven plugin).

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
* buildAssets (would generate production-ready front-end stuff like minified JS, CSS generated from SASS files, ...)
* start(environment) with shortcuts like startDev, startProd, ...
* migrateDBSchema(environment)