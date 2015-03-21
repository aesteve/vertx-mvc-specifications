# The view layer

## State of the art

A lot of frameworks come with an opinionated way of writing your views. I don't think it's a very good approach. Some are messy to deal with once things get complicated.

JSP/JSTL is pretty much outdated, JSF is a complete mess, Struts2 views aren't very popular... GSPs / Django templates / RoR are fine but pretty verbose once you have some corner-cases, and moreover, are completely specific to the framework.

With Apex being unopinionated, and handling alot of template engines (handlebars, jade, ...) out of the box, we don't need  anything specific here. Almost better : most template engines will be written and added to Apex in the future. We have to take advantage of that.

That could be a key feature of such an MVC framework to provide an agnostic approach for the view layer.

As mentionned in introduction, the framework should be smart enough to provide convention over configuration to prevent the user the pain to handle a template engine and have it render his views.

A view should just be named and its extension should be used to determine the correct template engine the framework should use. 

In existing frameworks, controllers have a `render` method which is used either to render some template or render direct html, or string. Even though I think it could be defined in a less confusing way (render for everything... meh), I guess users are used to it. Let's not forget about context data too. (Think about Spring's `ModelAndView`).

The kind of primitives we could use (even though we don't name them this way) : 

* `render(String contentType, Object content)`
* `renderHtml(String)`
* `renderXml(String)`
* `renderJson(String)`
* `renderYaml(String)`
* `renderTemplate(String templateName, Map<String, Object> data)` templateEngine would be guessed from templateName
* `renderTemplate(String templateName, Map<String, Object> data, TemplateEngine engine)` let users use their own template engine if they want to

## Implementation

Users' controllers should simply inherit from a `BaseController` or `AbstractController` which defines the render method.

Keep in mind that once the render method has been called, the response has been written and sent, so we won't be able to do anything with it. This doesn't mean we shouldn't continue to chain handlers (afterFilters) since some stuff could be achieved once the HttpReponse is sent. (update some stats objects, notify some service on the eventbus that we handled some kind of request, ...).

We should provide (and instantiate) the common template engines used in Apex. This must be done lazily since 99% of users won't use every template engine.

We could do some interesting stuff with the `render` method like handling ETags, etc.
