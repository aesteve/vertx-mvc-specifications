# Handling multiple environments

One thing every developer finds himself doing at one point is handling the same application running on different environments.

Such framework often provide an easy way for the developer to configure the current environment he's working on, and what the specifities of this environment are.

CI build systems will then switch take the source code of the application simply switch the variable describing the current environment (from pre-production to production for instance), then deploy the application.

The developer will also have an easy way to test his application on production environment for example, simply be changing one single setting.


## Needs

Here are the common things that depend on the environment you're building your web applications on : 

* Database access (url, network, ...) and credentials
* Host, port, ... SAAS for instance define custom host/port to listen to. On the developer computer, port can be different since he can work simultaneously on different web servers. Also, on some machines ports under a certain number aren't accessible without being root
* Cache settings. Many developers (especially front-end developers) won't like caching to be effective on a development environment. Assets and resources should be hot-reloaded asap. On a test or pre-production environment though, cache must be activated (not only in production, since potential side effects in caching should be detected asap).
* Assets compilation / bundling / minification. In a debug/development environment resources shouldn't be minified. Whether it's SASS/Less stylsheets or Javascript scripts. That's what web framework usually call assets pipelines.

Users should be responsible for naming their environments as they want to. Some call it `test` whereas some other just call it `pre-production` or `qa`. Same goes for different SAAS providers. Production could refer to Amazon AWS or OpenShift and could be switched from one to another.

## Implementation

Vert.x already provides a way to describe and read the configuration through conf.json file and the `JsonObject config()` method in a Verticle.

## Wrap the config JsonObject

SAAS providers like OpenShift provide some configuration through environment variables. It could be interesting to wrap into the configuration JsonObject some of the environment variables. But since it's a very specific approach : e.g. `OPENSHIFT_IP` should be mapped onto `host` in config, maybe we should just let the user describe these mappings in a Gradle configuration script rather than programmatically.

This way, the application source code would never change, and the user wouldn't need to write some painful code like 
```
if (System.getEnv(...)) {
    //
} 
else if (config().getString("", "default")) {
    // 
}
```

## Provide / generate different conf.json files

The most easy way to provide different configuration to the end-users would be to let him write every config needed and let the gradle task generate the right `conf.json` file for the correct environment.

Example:

Say we have two different environments : 

* DEBUG
* PRODUCTION

Then, the user should define 2 (or 3) files : 

* `conf_DEBUG.json`
* `conf_PRODUCTION.json`
* (optional : `conf_default.json`)

The third file could contain some common stuff between both the environments.

Using Gradle build system (or our own Maven plugin) we would generate the files according to the parameter sent to the `createApp` task (or another name, but `build` already exists).


Then : 

```
gradle createApp DEBUG
```

Would generate the fatJar for the application + a `conf.json` file containing contents from both `conf_default.json` and `conf_DEBUG.json` and put it in the destination defined by the end-user (defaults to build/libs for example).

## Configuration nodes

Let's find a node name and the possible values for every of the "Needs" expressed above.