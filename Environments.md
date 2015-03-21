# Handling multiple environments

One thing every developer finds himself doing at one point is handling the same application running on different environments.

Such framework often provide an easy way for the developer to configure the current environment he's working on, and what the specifities of this environment are.

CI build systems will then switch take the source code of the application simply switch the variable describing the current environment (from pre-production to production for instance), then deploy the application.

The developer will also have an easy way to test his application on production environment for example, simply be changing one single setting.


## Needs

Here are the common things that depend on the environment you're building your web applications on : 

* Database access (url, network, ...) and credentials
* Host, port, ... SAAS for instance define custom host/port to listen to. On the developer computer, port can be different since he can work simultaneously on different web servers. Also, on some machines ports under a certain number aren't accessible with being root
* Cache settings. Many developers (especially front-end developers) won't like caching to be effective on a development environment. Assets and resources should be hot-reloaded asap. On a test or pre-production environment though, cache must be activated (not only in production, since potential side effects in caching should be detected asap).
* Assets compilation / bundling / minification. In a debug/development environment resources shouldn't be minified. Whether it's SASS/Less stylsheets or Javascript scripts. That's what web framework usually call assets pipelines.

## Implementation

Vert.x already provides a way to describe and read the configuration through conf.json file and the `JsonObject config()` method in a Verticle.

First : 