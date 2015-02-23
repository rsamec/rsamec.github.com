---
layout: post
category : lessons
tagline: "too many revolutions"
tags : [principals]
---
{% include JB/setup %}

I personally don't like the posts that describe general author's feelings without arguments supported author's statements.
I like post with concrete examples, with description of technical implementation, with code snippets.
This post will be an exception. I promise it will be short and I'll go straight to the point.

# Too many revolutions

There are many revolutions in developing a typical web application.

+   Web components revolutions
+   SPA applications using various client MVC* frameworks (angular, knockout, backbone, ember, etc.)
+   language revolutions - TypeScript, Dart, Closure, ATScript
+   DOM - object observe revolutions
+   ES6 revolutions - class constructs, generators, promise
+   React
+   isomorfic applications
+   no backend applications

These standards, components, libraries are great and simplifies a lot work needed to be done when developing a typical business line application.

<div class="alert alert-success" role="alert">The only true revolution is the shift from programmers to users when building web applications.</div>

These technologies, languages, components enables to increase the level of abstraction when expressing the intent of your web application.
Many technical details are hidden to users.

You can look at web components as lego blocks that enables to users with knowledge of HTML markup to build applications from them.
The typical user don't know markup languages, it will be not able to use web components. 
 

## Final notes

I think it is a great news for people people working in industries

This shift is good for companies (no in house programmers needed), but i think it is good also for programmers (they can concentrate on "real" software).


## Final note

Whatever SPA (UI) framework you use to build some application, you have to create some reusable components (widgets).
The final application consists of tree of the components.

Toolbar Navigation -> Workspace -> Views -> MainContainer -> List -> Fields(TextField, ComboBoxField) + Actions Buttons

The difficult part of you application is to identify and create the quality components at the right granularity.

<div class="alert alert-success" role="alert">
Your application as good as your components are.
</div>

#### The key concept:

+   Components is reusable unit. Application consists of simple tree of these components.
+   Components is modeled as a standalone MVC entity.
    +   model -> props (immutable) + state(mutated) - immutability from components perspective
    +   controller -> code shared over all instances of component
    +   view -> template shared for all instances of component
+   Assign the state of application to the right component. To avoid cascading updates.
+   Components should never save state in the DOM.

<div class="alert alert-success" role="alert">
These concepts are general and usable in any UI frameworks.
</div>

When the state of the component is changed, the changes must be propageted to UI (typically) DOM.

There are many UI frameworks (angular, react, polymer, ember, knockout, jquery UI, etc.).
It is an implementation detail how these changes are propagated to UI

+   react - re-render virtual DOM and apply diff changes to DOM
+   angular - dirty checking - if changed -> apply change to component node in DOM
+   polymer web components - object.observe -> change event handler -> apply change to component node in DOM
+   knockout - observable -> change event handler -> apply change to component node in DOM
+   WPF - Binding.INotifyChange -> re-render diff changes do UI (XAML)

## Example:
We need model TextField to capture user input.

TextField component:

+   model
    +   user typed value
    +   validation error

Is this a mutable state?
would any other component of your app other than the text field want to know that the value user entered?
would any other component of your app other than the text field want to know that the user typed value is valid?

If yes, then it should be props, otherwise it is mutable state.
