---
layout: post
category : lessons
tagline: "complex objects"
tags : [intro, beginner, react, tutorial]
---
{% include JB/setup %}

Consider design of typical business application as triad of UI, logic, data.

+   __application UI__ should be generally agnostic towards your data and your logic. It's for rendering UI and dealing with UI events.
+   __application logic__ should be generally agnostic towards your UI and your data. It's for execution of business rules (validations) and business processes.
+   __application data__ should be generally agnostic of UI and logic. It's a persistent storage.

Libraries for implementation

+   UI - [react][react] - react is agnostic towards data and data -> you could think of React as the __View__ in MVC.
+   Logic - [business-rules-engine][bre] is agnostic towards UI -> you could think of React as the __Controller__ in MVC.
+   Data - plain JSON -> you could think of React as the __Model__ in MVC.

+   two-way data binding support - [react-binding][react-binding]

## react-binding


React [LinkedStateMixin][valueLink] is an easy way to express two-way data binding in React.

React-binding is lightweight mixin - [BindToMixin][react-binding].

#### Two-way data binding features

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
    +   [react-bootstrap][reactBoostrap]
    +   [material-ui][materialUi]

#### Base principle

Each bindTo return and uses interface called "value/onChange".
Each bindTo component is passed a value (to render it to UI) as well as setter to a value that triggers a re-render (typically at the central location).
The re-render is done at the component where you bind to state is called (bindToState, bindArrayToState).
BindTo enables to be composed by itself (concatenates path according to parent-child relationship).


### Examples

In this tutorial, I’ll build a small interactive form.

+   You can see the end result on the right.
+   You can play with the demo.
+   You can browse the sources.

Let’s start. Out basic steps are

+   design the UI form + define data + bind to UI
+   define business rules + bind to UI

### 1. design react form and bind to UI

We create hobby form to capture data from user in this form

{% highlight js %}
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


<iframe style="border: 1px solid #999;width: 100%; height: 200px"
        src="http://embed.plnkr.co/aTilRFEJe0gEWaZzr8PC/?t=code?f=script.jsx" frameborder="0"
        allowfullscreen="allowfullscreen">
  Loading plunk...
</iframe>

#### Person component
{% highlight js %}
var Form = React.createClass({
    mixins:[BindToMixin],
    getInitialState: function() {
        return { data: {}};
    },
    render: function() {
        return (
            <div>
               <input type='text' valueLink={this.bindToState("data","Employee.FirstName")} />
               <input type='text' valueLink={this.bindToState("data","Employee.LastName")} />
               <input type='text' valueLink={this.bindToState("data","Employee.Contact.Email")} />
            </div>
        );
    }
});
{% endhighlight %}


### bindTo(parent,pathExpression)

We create reusable component PersonComponent with 3 fields and use twice in our form.

{% highlight js %}
var Form = React.createClass({
    mixins:[BindToMixin],
    getInitialState: function() {
        return { data: {}};
    },
    render: function() {
        return (
            <div>
                <PersonComponent personModel={this.bindToState("data","Employee")} />
                <PersonComponent personModel={this.bindToState("data","Deputy")} />
            </div>
        );
    }
});


var PersonComponent = React.createClass({
  mixins:[BindToMixin],
  render: function() {
    return (
      <div>
        <input type='text' valueLink={this.bindTo(this.props.personModel,"FirstName")} />
        <input type='text' valueLink={this.bindTo(this.props.personModel,"LastName")} />
        <input type='text' valueLink={this.bindTo(this.props.personModel,"Contact.Email")} />
      </div>
    );
  }
});

{% endhighlight %}

### bindToArrayState(key,path)

We create list of hobbies with add and remove buttons.

{% highlight js %}

var HobbyForm = React.createClass({
    mixins:[BindToMixin],
    getInitialState: function() {
        return { data: {}};
    },
    addHobby:function(e){
      if (this.state.data.Hobbies === undefined)
        this.state.data.Hobbies = []
      this.state.data.Hobbies.push({});
      this.setState({data:this.state.data})
    },
    render: function() {
        return (
            <div className="commentBox">
                <div >
                  <button onClick={this.addHobby}>Add</button>
                  <HobbyList model={this.bindArrayToState("data","Hobbies")} />
                </div>
            </div>
        );
    }
});
var HobbyList = React.createClass({
    handleDelete: function(hobby){
        return this.props.model.remove(hobby);
    },
    render: function() {
        if (this.props.model.Items === undefined) return <span>There are no items.</span>;

        var nodes = this.props.model.items.map(function(hobby, index) {
            return (
                <Hobby model={hobby} key={index} onDelete={this.handleDelete} />
            );
        },this);
        return (
            <div>
                {nodes}
            </div>
        );
    }
});
var Hobby = React.createClass({
    mixins:[BindToMixin],
    handleClick: function(e){
        e.preventDefault();
        return this.props.onDelete(this.props.model.value);
    },
    render: function() {
        return (
            <div className="comment">
              <input type='text' valueLink={this.bindTo(this.props.model,"HobbyName")} />
              <button value="Delete" onClick={this.handleClick}>Delete</button>
            </div>
        );
    }
});

{% endhighlight %}
<div class="alert alert-danger" role="alert">There are no dumb questions. Why i should use it?</div>  

+ create method that executes all business rules
{% highlight js %}

{% endhighlight %}

### 2. validation rules

<div class="alert alert-success" role="alert">Well done - your hobbies form is done.</div>



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
