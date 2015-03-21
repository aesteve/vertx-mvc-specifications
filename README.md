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


## [Scaffolding](/Scaffolding.md)

## [The Controller layer](/ControllerLayer.md)

## [The View layer](/ViewLayer.md)

## The domain layer

TODO : and this is where I'll need a lot of help.

## [Handling environments](/Environments.md)
