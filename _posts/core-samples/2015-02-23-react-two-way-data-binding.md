---
layout: post
category : lessons
tagline: "complex forms with simple objects"
tags : [intro, beginner, react, tutorial]
---
{% include JB/setup %}

# Introduction

<div style="float:right;margin:0 10px 10px 0" markdown="1">
  ![logo]({{site.url}}/assets/images/hobby-form-react-material-ui.jpg)
</div>

In this tutorial, I’ll build a small interactive form in 3 different CSS frameworks.

I want to demonstrate how to use two-way data binding for complex objects in [react] using [react-binding][react-binding].

React-binding is lightweight mixin called __BindToMixin__ in [react].


# Features

[BindToMixin](https://github.com/rsamec/react-binding) offers two-way data binding support for:

+   object properties with path expression (dot notation)
    +   this.bindToState("data","Employee.FirstName");
    +   this.bindToState("data","Employee.Contact.Email");
+   complex objects (json) with nested properties
    +   this.bindTo(employee,"FirstName");
    +   this.bindTo(employee,"Contact.Email");
+   collection-based structures - arrays and lists
    +   model={this.bindTo(employee,"FirstName")}
        +   this.props.model.items.map(function(item){ return (<Hobby model={hobby}/>);})
        +   this.props.model.add()
        +   this.props.model.remove(item)
+   supports for "value/requestChange" interface also to enable to use [ReactLink][valueLink] attribute
    +   valueLink={this.bindTo(employee,"FirstName")}
+   usable with any css frameworks
    +   [react-bootstrap][reactBootstrap]
    +   [material-ui][materialUi]

# Basic principle

Each bindTo return and uses interface called "value/onChange".
Each bindTo component is passed a value (to render it to UI) as well as setter to a value that triggers a re-render (typically at the top location).

The re-render is done in the component where you bind to the state via (bindToState, bindArrayToState).

BindTo can be nested - composed to support components composition. Then path is concatenate according to parent-child relationship.

# Example usage

If you design a typical business application, you can distinguish among 3 concerns - UI, logic, data.

+   __UI__ should be generally agnostic towards your data and your logic. It's for rendering UI and dealing with UI events.
+   __logic__ should be generally agnostic towards your UI and your data. It's for execution of business rules (validations) and business processes.
+   __data__ should be generally agnostic of UI and logic. It's a persistent storage.

Libraries used for implementation

+   UI - [react][react] - react is agnostic towards data and data -> you could think of React as the __View__ in MVC.
+   Logic - [business-rules-engine][bre] is agnostic towards UI -> you could think of React as the __Controller__ in MVC.
+   Data - plain JSON -> you could think of React as the __Model__ in MVC.


## 1. define structure for data
In all cases we want to capture data from user in this form

{% highlight json %}
{
  "Person": {
    "LastName": "Smith",
    "FirstName": "Adam",
    "Contact": {
      "Email": "smith@gmail.com"
    }
  },
  "Hobbies": [
      {
        "HobbyName": "Bandbington",
        "Frequency": "Daily",
        "Paid": true,
        "Recommendation": true
      },
      {
        "HobbyName": "Cycling",
        "Frequency": "Daily",
        "Recommendation": false,
        "Paid": false
      }
    ]
}

{% endhighlight %}

## 2. describe logic - business rules

We use [business-rules-engine][bre] for defining business rules.

I choose __JSON schema declarative__ description. For other possible formats read [declarative-vs-imperative-validation]({% post_url core-samples/2014-09-21-declarative-vs-imperative-validation %}).

{% highlight json %}
{
    "Person": {
        "type": "object",
        "properties": {
            "FirstName": {
                "type": "string",
                "title": "First name",
                "required": "true",
                "maxLength": "15"
            },
            "LastName": {
                "type": "string",
                "title": "Last name",
                "required": "true",
                "maxLength": "15"
            },
            "Contact": {
                "type": "object",
                "properties": {
                    "Email": {
                        "type": "string",
                        "title": "Email",
                        "required": "true",
                        "maxLength": 100,
                        "email": "true"
                    }
                }
            }
        }
    },
    "Hobbies": {
        "type": "array",
        "items": {
            "type": "object",
            "properties": {
                "HobbyName": {
                    "type": "string",
                    "title": "HobbyName",
                    "required": "true",
                    "maxLength": 100
                },
                "Frequency": {
                    "type": "string",
                    "title": "Frequency",
                    "enum": ["Daily", "Weekly", "Monthly"]
                },
                "Paid": {
                    "type": "boolean",
                    "title": "Paid"
                },
                "Recommedation": {
                    "type": "boolean",
                    "title": "Recommedation"
                }
            }
        },
        "maxItems": 4,
        "minItems": 2
    }
}
{% endhighlight %}

It is a simple task to use business rules in our form.

+   add rules to state -> create validation rules new FormSchema.JsonSchemaRuleFactory(BusinessRulesJson).CreateRule("Main")
+   execute all validation rules - this.state.rules.Validate(this.state.data) to get results
+   using Utils.CompositeDotObject.Transform as helper function that enables to access to validation result comfortably by dot notation (with path expressions) - see one-way binding to error messages below

{% highlight js %}
var HobbyForm = React.createClass({
    mixins: [BindToMixin],
    getInitialState: function() {
        return {
            data: {},
            rules:new FormSchema.JsonSchemaRuleFactory(BusinessRulesJson).CreateRule("Main")
        };
    },
    result:function(){
        var validationResult = this.state.rules.Validate(this.state.data);
        return Utils.CompositeDotObject.Transform(validationResult).Main;
    },

    _handleDialogOpen: function() {
        if (this.result().HasError){
            //TODO: show error dialog;
            //return;
        }

        //send to server to persist
        //for demo -> show data dialog
        this.refs.customDialog.show();
    },
}
{% endhighlight %}

## 3. UI - using React

+   using [BindToMixin][react-binding] for two-way data binding.
+   using [react] one-way binding to display error messages.

Summary for __two-way data bindings__ for HobbyForm - using __model__ and __valueLink__ property

+   PersonComponent - model={__this.bindToState("data", "Person")__}
    +   TextBoxInput - valueLink={__this.bindTo(this.props.model, "FirstName")__}
    +   TextBoxInput - valueLink={__this.bindTo(this.props.model, "LastName")__}
    +   TextBoxInput - valueLink={__this.bindTo(this.props.model, "Contact.Email")__}
+   Button - onClick={__this.addHobby__}
+   HobbyList component - model={__this.bindArrayToState("data", "Hobbies")__}
    +   Hobby component
        +   TextBoxInput - valueLink={__this.bindTo(this.props.model, "HobbyName"__)
        +   Button - onClick={__this.removeHobby__}
        +   RadioGroup - valueLink={__this.bindTo(this.props.model, "Frequency"__)
        +   CheckBoxInput model={__this.bindTo(this.props.model, "Paid")__}
        +   CheckBoxInput model={__this.bindTo(this.props.model, "Recommendation")__}

Summary for __one-way binding to error messages__ - using __error__ property for error messages

+   PersonComponent - error={__this.result().Person__}
    +   TextBoxInput - error={__this.props.error.FirstName.ErrorMessage__}
    +   TextBoxInput - error={__this.props.error.LastName.ErrorMessage__}
    +   TextBoxInput - error={__this.props.error.Contact.Email.ErrorMessage__}
+   Span - {__this.result().Hobbies.PropRule.ErrorMessage__}
+   HobbyList component - errors={__this.result().Hobbies.Children__}
    +   Hobby component - error={__this.props.errors[index]__}
        +   TextBoxInput - error={__this.props.error.HobbyName.ErrorMessage__}


### 3.1. minimal - without any CSS framework

<iframe style="border: 1px solid #999;width: 100%; height: 350px"
        src="http://embed.plnkr.co/qXlUQ7a3YLEypwT2vvSb/preview" frameborder="0"
        allowfullscreen="allowfullscreen">
  Loading plunk...
</iframe>


### 3.2. [react-bootstrap][reactBootstrap]

<iframe style="border: 1px solid #999;width: 100%; height: 350px"
        src="http://embed.plnkr.co/6hoCCd7Bl1PHnb57rQbT/preview" frameborder="0"
        allowfullscreen="allowfullscreen">
  Loading plunk...
</iframe>

__remarks__

+   simple html controls replaced with ReactBootstrap controls - (button -> ReactBootstrap.Button,input -> ReactBoostrap.Input)
+   showing error messages
    +   using __help__ property for error messages
    +   using __bsStyle__ property to style input (bsStyle={__this.validationState(this.props.error.HobbyName)__})


### 3.3. [react material UI][materialUi]

+   You can see the end result at the top of this post.
+   You can play with the [demo](http://polymer-formvalidation.rhcloud.com/dist/index.html).
+   You can browse the [sources](https://github.com/rsamec/react-hobby-form-app).

__remarks__

+   simple html controls replaced with material ui controls - (button -> RaisedButton,input -> TextField)
+   showing error messages using __errorText__ property for error messages

<div class="alert alert-success" role="alert">Well done - your hobbies form is done.</div>

[git]: http://git-scm.com/
[bower]: http://bower.io
[npm]: https://www.npmjs.org/
[node]: http://nodejs.org
[browserify]: http://browserify.org/
[blog]: http://rsamec.github.io/
[valueLink]: http://facebook.github.io/react/docs/two-way-binding-helpers.html
[materialUi]: https://github.com/callemall/material-ui
[react]:http://facebook.github.io/react/
[reactBootstrap]: http://react-bootstrap.github.io/
[react-binding]: https://github.com/rsamec/react-binding
[bre]: https://github.com/rsamec/business-rules-engine
