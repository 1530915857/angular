@title
TypeScript to JavaScript

@intro
Convert Angular TypeScript examples into ES6 and ES5 JavaScript.

@description
Anything you can do with Angular in _TypeScript_, you can also do
in JavaScript. Translating from one language to the other is mostly a
matter of changing the way you organize your code and access Angular APIs.

_TypeScript_ is a popular language option for Angular development. 
Most code examples on the Internet as well as on this site are written in _TypeScript_. 
This cookbook contains recipes for translating _TypeScript_
code examples to _ES6_ and to _ES5_ so that JavaScript developers
can read and write Angular apps in their preferred dialect.


{@a toc}
## Table of contents

[_TypeScript_ to _ES6_ to _ES5_](guide/ts-to-js#from-ts)<br>
[Modularity: imports and exports](guide/ts-to-js#modularity)<br>
[Classes and Class Metadata](guide/ts-to-js#class-metadata)<br>
[_ES5_ DSL](guide/ts-to-js#dsl)<br>
[Interfaces](guide/ts-to-js#interfaces)<br>
[Input and Output Metadata](guide/ts-to-js#io-decorators)<br>
[Dependency Injection](guide/ts-to-js#dependency-injection)<br>
[Host Binding](guide/ts-to-js#host-binding)<br>
[View and Child Decorators](guide/ts-to-js#view-child-decorators)<br>
[AOT compilation in _TypeScript_ Only](guide/ts-to-js#aot)<br>

**Run and compare the live <live-example name="cb-ts-to-js">_TypeScript_</live-example> and <live-example name="cb-ts-to-js" lang="js">JavaScript</live-example>
code shown in this cookbook.**


{@a from-ts}

## _TypeScript_ to _ES6_ to _ES5_

_TypeScript_ 
<a href="https://www.typescriptlang.org" target="_blank" title='"TypeScript is a typed, superset of JavaScript"'>is a typed superset of _ES6 JavaScript_</a>.
_ES6 JavaScript_ is a superset of _ES5 JavaScript_. _ES5_ is the kind of JavaScript that runs natively in all modern browsers.
The transformation of _TypeScript_ code all the way down to _ES5_ code can be seen as "shedding" features.

The downgrade progression is
* _TypeScript_ to _ES6-with-decorators_
* _ES6-with-decorators_ to _ES6-without-decorators_ ("_plain ES6_")
* _ES6-without-decorators_ to _ES5_

When translating from _TypeScript_ to _ES6-with-decorators_, remove 
[class property access modifiers](http://www.typescriptlang.org/docs/handbook/classes.html#public-private-and-protected-modifiers)
such as `public` and `private`.
Remove most of the 
[type declarations](https://www.typescriptlang.org/docs/handbook/basic-types.html), 
such as `:string` and `:boolean`
but **keep the constructor parameter types which are used for dependency injection**.
 

From _ES6-with-decorators_ to _plain ES6_, remove all 
[decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
and the remaining types.
You must declare properties in the class constructor (`this.title = '...'`) rather than in the body of the class.

Finally, from _plain ES6_ to _ES5_, the main missing features are `import`
statements and `class` declarations. 

For _plain ES6_ transpilation you can _start_ with a setup similar to the 
[_TypeScript_ quickstart](https://github.com/angular/quickstart) and adjust the application code accordingly. 
Transpile with [Babel](https://babeljs.io/) using the `es2015` preset. 
To use decorators and annotations with Babel, install the 
[`angular2`](https://github.com/shuhei/babel-plugin-angular2-annotations) preset as well.
 


{@a modularity}

## Importing and Exporting

### Importing Angular Code

In both _TypeScript_ and _ES6_, you import Angular classes, functions, and other members with _ES6_ `import` statements.

In _ES5_, you access the Angular entities of the [the Angular packages](glossary)
through the global `ng` object. 
Anything you can import from `@angular` is a nested member of this `ng` object:

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/app.module.ts' region='ng2import'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/app.module.es6' region='ng2import'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/app.module.es6' region='ng2import'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/app.module.js' region='ng2import'}
  </md-tab>


</md-tab-group>

### Exporting Application Code

Each file in a _TypeScript_ or _ES6_ Angular application constitutes an _ES6_ module.
When you want to make something available to other modules, you `export` it.

_ES5_ lacks native support for modules. 
In an Angular _ES5_ application, you load each file manually by adding a `<script>` tag to `index.html`. 

~~~ {.alert.is-important}

The order of `<script>` tags is often significant. 
You must load a file that defines a public JavaScript entity before a file that references that entity.

~~~

The best practice in _ES5_ is to create a form of modularity that avoids polluting the global scope. 
Add one application namespace object such as `app` to the global `document`.
Then each code file "exports" public entities by attaching them to that namespace object, e.g., `app.HeroComponent`.
You could factor a large application into several sub-namespaces 
which leads to "exports" along the lines of `app.heroQueries.HeroComponent`.

Every _ES5_ file should wrap code in an 
[Immediately Invoked Function Expression (IIFE)](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)
to limit unintentional leaking of private symbols into the global scope.

Here is a `HeroComponent` as it might be defined and "exported" in each of the four language variants.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero.component.ts' region='appexport'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero.component.es6' region='appexport'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero.component.es6' region='appexport'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero.component.js' region='appexport'}
  </md-tab>


</md-tab-group>

### Importing Application Code

In _TypeScript_ and _ES6_ apps, you `import` things that have been exported from other modules.

In _ES5_ you use the shared namespace object to access "exported" entities from other files.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/app.module.ts' region='appimport'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/app.module.es6' region='appimport'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/app.module.es6' region='appimport'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/app.module.js' region='appimport'}
  </md-tab>


</md-tab-group>



~~~ {.alert.is-helpful}

Alternatively, you can use a module loader such as Webpack or
Browserify in an Angular JavaScript project. In such a project, you would
use _CommonJS_ modules and the `require` function to load Angular framework code.
Then use `module.exports` and `require` to export and import application  code.



~~~



{@a class-metadata}

## Classes and Class Metadata

### Classes

Most Angular _TypeScript_ and _ES6_ code is written as classes.

Properties and method parameters of _TypeScript_ classes may be marked with the access modifiers
`private`, `internal`, and `public`. 
Remove these modifiers when translating to JavaScript.

Most type declarations (e.g, `:string` and `:boolean`) should be removed when translating to JavaScript.
When translating to _ES6-with-decorators_, ***do not remove types from constructor parameters!***

Look for types in _TypeScript_ property declarations.
In general it is better to initialize such properties with default values because
many browser JavaScript engines can generate more performant code.
When _TypeScript_ code follows this same advice, it can infer the property types
and there is nothing to remove during translation. 

In _ES6-without-decorators_, properties of classes must be assigned inside the constructor.

_ES5_ JavaScript has no classes. 
Use the constructor function pattern instead, adding methods to the prototype.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero.component.ts' region='class'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero.component.es6' region='class'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero.component.es6' region='class'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero.component.js' region='constructorproto'}
  </md-tab>


</md-tab-group>

### Metadata

When writing in _TypeScript_ or _ES6-with-decorators_, 
provide configuration and metadata by adorning a class with one or more *decorators*. 
For example, you supply metadata to a component class by preceding its definition with a
[`@Component`](api/core/index/Component-decorator) decorator function whose
argument is an object literal with metadata properties.

In _plain ES6_, you provide metadata by attaching an `annotations` array to the _class_.
Each item in the array is a new instance of a metadata decorator created with a similar metadata object literal.

In _ES5_, you also provide an `annotations` array but you attach it to the _constructor function_ rather than to a class.

See these variations side-by-side:

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero.component.ts' region='metadata'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero.component.es6' region='metadata'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero.component.es6' region='metadata'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero.component.js' region='metadata'}
  </md-tab>


</md-tab-group>

***External Template file***

A large component template is often kept in a separate template file.

{@example 'cb-ts-to-js/ts/src/app/hero-title.component.html'}

The component (`HeroTitleComponent` in this case) then references the template file in its metadata `templateUrl` property:
<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-title.component.ts' region='templateUrl'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-title.component.es6' region='templateUrl'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero-title.component.es6' region='templateUrl'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero-title.component.js' region='templateUrl'}
  </md-tab>


</md-tab-group>

Note that both the _TypeScript_ and _ES6_ `templateUrl` properties identify the location of the template file _relative to the component module_.
All three metadata configurations specify the `moduleId` property 
so that Angular can calculate the proper module address.

The _ES5_ approach shown here does not support modules and therefore there is no way to calculate a _module-relative URL_.
The `templateUrl` for the _ES5_ code must specify the _path from the project root_ and 
omits the irrelevant `moduleId` property. 

With the right tooling, the `moduleId` may not be needed in the other JavaScript dialects either.
But it's safest to provide it anyway.


{@a dsl}

## _ES5_ DSL

This _ES5_ pattern of creating a constructor and annotating it with metadata is so common that Angular 
provides a convenience API to make it a little more compact and locates the metadata above the constructor, 
as you would if you wrote in _TypeScript_ or _ES6-with-decorators_.

This _API_ (_Application Programming Interface_) is commonly known as the _ES5 DSL_ (_Domain Specific Language_).

Set an application namespace property (e.g., `app.HeroDslComponent`) to the result of an `ng.core.Component` function call.
Pass the same metadata object to `ng.core.Component` as you did before.
Then chain a call to the `Class` method which takes an object defining the class constructor and instance methods.

Here is an example of the `HeroComponent`, re-written with the DSL,
next to the original _ES5_ version for comparison:

<md-tab-group>

  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/hero.component.js' region='dsl'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero.component.js'}
  </md-tab>


</md-tab-group>



~~~ {.callout.is-helpful}


<header>
  Name the constructor
</header>

A **named** constructor displays clearly in the console log
if the component throws a runtime error. 
An **unnamed** constructor displays as an anonymous function (e.g., `class0`) 
which is impossible to find in the source code.


~~~

### Properties with getters and setters

_TypeScript_ and _ES6_ support with getters and setters.
Here's an example of a read-only _TypeScript_ property with a getter
that prepares a toggle-button label for the next clicked state:

{@example 'cb-ts-to-js/ts/src/app/hero-queries.component.ts' region='defined-property'}

This _TypeScript_ "getter" property is transpiled to an _ES5_
<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty"
   target="_blank" title="Defined Properties">defined property</a>.
The _ES5 DSL_ does not support _defined properties_ directly
but you can still create them by extracting the "class" prototype and
adding the _defined property_ in raw JavaScript like this:

{@example 'cb-ts-to-js/js/src/app/hero-queries.component.js' region='defined-property'}

### DSL for other classes
There are similar DSLs for other decorated classes. 
You can define a directive with `ng.core.Directive`: 

<code-example>
  app.MyDirective = ng.core.Directive({  
      selector: '[myDirective]'  
    }).Class({  
      ...  
    });
</code-example>

and a pipe with `ng.core.Pipe`:
<code-example>
  app.MyPipe = ng.core.Pipe({  
      name: 'myPipe'  
    }).Class({  
      ...  
    });  
    
</code-example>



{@a interfaces}

## Interfaces

A _TypeScript_ interface helps ensure that a class implements the interface's members correctly.
We strongly recommend Angular interfaces where appropriate. 
For example, the component class that implements the `ngOnInit` lifecycle hook method
should implement the `OnInit` interface.

_TypeScript_ interfaces exist for developer convenience and are not used by Angular at runtime.
They have no physical manifestation in the generated JavaScript code.
Just implement the methods and ignore interfaces when translating code samples from _TypeScript_ to JavaScript.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-lifecycle.component.ts'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-lifecycle.component.es6'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero-lifecycle.component.es6'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero-lifecycle.component.js'}
  </md-tab>


  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/hero-lifecycle.component.js' region='dsl'}
  </md-tab>


</md-tab-group>



{@a io-decorators}

## Input and Output Metadata

### Input and Output Decorators

In _TypeScript_ and _ES6-with-decorators_, you often add metadata to class _properties_ with _property decorators_.
For example, you apply [`@Input` and `@Output` property decorators](guide/template-syntax) 
to public class properties that will be the target of data binding expressions in parent components.

There is no equivalent of a property decorator in _ES5_ or _plain ES6_. 
Fortunately, every property decorator has an equivalent representation in a class decorator metadata property.
A _TypeScript_ `@Input` property decorator can be represented by an item in the `Component` metadata's `inputs` array.

You already know how to add `Component` or `Directive` class metadata in _any_ JavaScript dialect so
there's nothing fundamentally new about adding another property. 
But note that what would have been _separate_ `@Input` and `@Output` property decorators for each class property are
combined in the metadata `inputs` and `outputs` _arrays_.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/confirm.component.ts'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/confirm.component.es6'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/confirm.component.es6'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/confirm.component.js'}
  </md-tab>


  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/confirm.component.js' region='dsl'}
  </md-tab>


</md-tab-group>

In the previous example, one of the public-facing binding names (`cancelMsg`) 
differs from the corresponding class property name (`notOkMsg`).
That's OK but you must tell Angular about it so that it can map an external binding of `cancelMsg`
to the component's `notOkMsg` property.

In _TypeScript_ and _ES6-with-decorators_, 
you specify the special binding name in the argument to the property decorator.

In _ES5_ and _plain ES6_ code, convey this pairing with the `propertyName: bindingName` syntax in the class metadata.

## Dependency Injection
Angular relies heavily on [Dependency Injection](guide/dependency-injection) to provide services to the objects it creates.
When Angular creates a new component, directive, pipe or another service, 
it sets the class constructor parameters to instances of services provided by an _Injector_.

The developer must tell Angular what to inject into each parameter.

### Injection by Class Type

The easiest and most popular technique in _TypeScript_ and _ES6-with-decorators_ is to set the constructor parameter type
to the class associated with the service to inject. 

The _TypeScript_ transpiler writes parameter type information into the generated JavaScript.
Angular reads that information at runtime and locates the corresponding service in the appropriate _Injector_..
The _ES6-with-decorators_ transpiler does essentially the same thing using the same parameter-typing syntax.

_ES5_ and _plain ES6_ lack types so you must identify "injectables" by attaching a **`parameters`** array to the constructor function. 
Each item in the array specifies the service's injection token.

As with _TypeScript_ the most popular token is a class, 
or rather a _constructor function_ that represents a class in _ES5_ and _plain ES6_.
The format of the `parameters` array varies:

* _plain ES6_ &mdash; nest each constructor function in a sub-array.

* _ES5_ &mdash; simply list the constructor functions.

When writing with _ES5 DSL_, set the `Class.constructor` property to
an array whose first parameters are the injectable constructor functions and whose
last parameter is the class constructor itself. 
This format should be familiar to AngularJS developers.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-di.component.ts'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-di.component.es6'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero-di.component.es6'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero-di.component.js'}
  </md-tab>


  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/hero-di.component.js' region='dsl'}
  </md-tab>


</md-tab-group>

### Injection with the @Inject decorator

Sometimes the dependency injection token isn't a class or constructor function.

In _TypeScript_ and _ES6-with-decorators_, you precede the class constructor parameter
by calling the `@Inject()` decorator with the injection token.
In the following example, the token is the string `'heroName'`.

The other JavaScript dialects add a `parameters` array to the class contructor function.
Each item constains a new instance of `Inject`:

* _plain ES6_ &mdash; each item is a new instance of `Inject(token)` in a sub-array.

* _ES5_ &mdash; simply list the string tokens.

When writing with _ES5 DSL_, set the `Class.constructor` property to a function definition
array as before. Create a new instance of `ng.core.Inject(token)` for each parameter. 

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-di-inject.component.ts'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-di-inject.component.es6'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero-di-inject.component.es6'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero-di-inject.component.js'}
  </md-tab>


  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/hero-di-inject.component.js' region='dsl'}
  </md-tab>


</md-tab-group>

### Additional Injection Decorators

You can qualify injection behavior with injection decorators from `@angular/core`.

In _TypeScript_ and _ES6-with-decorators_, 
you precede the constructor parameters with injection qualifiers such as:
* [`@Optional`](api/core/index/Optional-decorator) sets the parameter to `null` if the service is missing
* [`@Attribute`](api/core/index/Attribute-interface) to inject a host element attribute value
* [`@ContentChild`](api/core/index/ContentChild-decorator) to inject a content child
* [`@ViewChild`](api/core/index/ViewChild-decorator) to inject a view child
* [`@Host`](api/core/index/Host-decorator) to inject a service in this component or its host
* [`@SkipSelf`](api/core/index/SkipSelf-decorator) to inject a service provided in an ancestor of this component

In _plain ES6_ and _ES5_, create an instance of the equivalent injection qualifier in a nested array within the `parameters` array.
For example, you'd write `new Optional()` in _plain ES6_ and `new ng.core.Optional()` in _ES5_.


When writing with _ES5 DSL_, set the `Class.constructor` property to a function definition
array as before. Use a nested array to define a parameter's complete injection specification.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-title.component.ts'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-title.component.es6'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero-title.component.es6'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero-title.component.js'}
  </md-tab>


  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/hero-title.component.js' region='dsl'}
  </md-tab>


</md-tab-group>


In the example above, there is no provider for the `'titlePrefix'` token.
Without `Optional`, Angular would raise an error.
With `Optional`, Angular sets the constructor parameter to `null` 
and the component displays the title without a prefix.


{@a host-binding}

## Host Binding
Angular supports bindings to properties and events of the _host element_ which is the
element whose tag matches the component selector.

### Host Decorators

In _TypeScript_ and _ES6-with-decorators_, you can use host property decorators to bind a host
element to a component or directive.
The [`@HostBinding`](api/core/index/HostBinding-interface) decorator
binds host element properties to component data properties. 
The [`@HostListener`](api/core/index/HostListener-interface) decorator binds
host element events to component event handlers.

In _plain ES6_ or _ES5_, add a `host` attribute to the component metadata to achieve the
same effect as `@HostBinding` and `@HostListener`. 

The  `host` value is an object whose properties are host property and listener bindings:

* Each key follows regular Angular binding syntax: `[property]` for host bindings
  or `(event)` for host listeners.
* Each value identifies the corresponding component property or method.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-host.component.ts'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-host.component.es6'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero-host.component.es6'}
  </md-tab>


  <md-tab label="ES5 JavaScript">
    {@example 'cb-ts-to-js/js/src/app/hero-host.component.js'}
  </md-tab>


  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/hero-host.component.js' region='dsl'}
  </md-tab>


</md-tab-group>

### Host Metadata
Some developers prefer to specify host properties and listeners
in the component metadata. 
They'd _rather_ do it the way you _must_ do it _ES5_ and _plain ES6_.

The following re-implementation of the `HeroComponent` reminds us that _any property metadata decorator_
can be expressed as component or directive metadata in both _TypeScript_ and _ES6-with-decorators_.
These particular _TypeScript_ and _ES6_ code snippets happen to be identical.
<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-host-meta.component.ts'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-host-meta.component.es6'}
  </md-tab>


</md-tab-group>



{@a view-child-decorators}

### View and Child Decorators

Several _property_ decorators query a component's nested view and content components.

_View_ children are associated with element tags that appear _within_ the component's template.

_Content_ children are associated with elements that appear _between_ the component's element tags;
they are projected into an `<ng-content>` slot in the component's template.     The [`@ViewChild`](api/core/index/ViewChild-decorator) and
[`@ViewChildren`](api/core/index/ViewChildren-decorator) property decorators
allow a component to query instances of other components that are used in
its view. 

In _ES5_ and _ES6_, you access a component's view children by adding a `queries` property to the component metadata. 
The `queries` property value is a hash map.

* each _key_ is the name of a component property that will hold the view child or children.
* each _value_ is a new instance of either `ViewChild` or `ViewChildren`.

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-queries.component.ts' region='view'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-queries.component.es6' region='view'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero-queries.component.es6' region='view'}
  </md-tab>


  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/hero-queries.component.js' region='view'}
  </md-tab>


</md-tab-group>

The [`@ContentChild`](api/core/index/ContentChild-decorator) and
[`@ContentChildren`](api/core/index/ContentChildren-decorator) property decorators
allow a component to query instances of other components that have been projected
into its view from elsewhere.

They can be added in the same way as [`@ViewChild`](api/core/index/ViewChild-decorator) and
[`@ViewChildren`](api/core/index/ViewChildren-decorator).

<md-tab-group>

  <md-tab label="TypeScript">
    {@example 'cb-ts-to-js/ts/src/app/hero-queries.component.ts' region='content'}
  </md-tab>


  <md-tab label="ES6 JavaScript with decorators">
    {@example 'cb-ts-to-js/js-es6-decorators/src/app/hero-queries.component.es6' region='content'}
  </md-tab>


  <md-tab label="ES6 JavaScript">
    {@example 'cb-ts-to-js/js-es6/src/app/hero-queries.component.es6' region='content'}
  </md-tab>


  <md-tab label="ES5 JavaScript with DSL">
    {@example 'cb-ts-to-js/js/src/app/hero-queries.component.js' region='content'}
  </md-tab>


</md-tab-group>



~~~ {.alert.is-helpful}

In _TypeScript_ and _ES6-with-decorators_ you can also use the `queries` metadata 
instead of the `@ViewChild` and `@ContentChild` property decorators.    


~~~



{@a aot}

## AOT Compilation in _TypeScript_ only

Angular offers two modes of template compilation, JIT (_Just-in-Time_) and 
[AOT (_Ahead-of-Time_)](guide/aot-compiler).
Currently the AOT compiler only works with _TypeScript_ applications because, in part, it generates
_TypeScript_ files as an intermediate result.
**AOT is not an option for pure JavaScript applications** at this time.