---
layout: post
category : lessons
tagline: "clever improve"
tags : [principals]
---
{% include JB/setup %}

Form validations in AngularJS 1.3 have been improved

+   native HTML5 validators have been resolved
+   introduction asynchronous validations
+   support for nested forms
+   new ngMessages module

You can read one great [tutorial about angular form](http://www.yearofmoo.com/2014/09/taming-forms-in-angularjs-1-3.html).

But let's have a look how to improved the form and validation in angular more.

+   enable to reuse validation rules other than angular by definition of UI (angular) agnostic validations
+   optional validations - angular validations can be optional with ng-if replacement (not usable when you need to cache UI state)
+   shared validations (validations that are dependent on more then one field (ng-model))

# To be UI agnostic

Validations are often see as the task of validating user input. Validations are part of UI.
That is right, but they can be perfectly usable without UI too. So i see,validations are part of business rules.

<div class="alert alert-info" role="alert">Business rules can be defined as some UI abstractions of validation rules.</div>

That is why, the validation rules should be defined in UI agnostic way to enable reuse of validation in non-UI scenarios.
Imagine you need to test your business rule or you need to run your validation rules at server or you want to expose some validation API.
In all cases it is simpler to execute validations completely without DOM. I wrote an post about [UI agnostic validation]({% post_url /core-samples/2014-09-02-UI-agnostic-validation %}).


So i don't want to define my validations in HTML or in javascript in angular way.
You can think that means that I have to write my own validation engine instead of what we have in angular 1.3.
But that is not the case. I can have UI agnostic validation rules and use angular validation engine with no restriction.

So my goal is to have

+   declaration of validation rules - UI agnostic (angular agnostic)
+   execution of validation rules - for angular app - using standard angular validations, forms

You can see some [examples of various ways for validation rules definition]({% post_url /core-samples/2014-09-21-declarative-vs-imperative-validation %}) in my previous post.
I choose imperative style for validation rules definitions using [API](http://rsamec.github.io/business-rules-engine/docs/modules/_validation_validation_.html)

{% highlight js %}
  angular.module('myApp').factory('businessRules',function(){
    var BusinessRules = (function () {

          function BusinessRules() {

              this.MainValidator = this.createMainValidator();
          }


         BusinessRules.prototype.createMainValidator = function () {
              //create custom validator
              var validator = new Validation.AbstractValidator();

              validator.ValidatorFor("Person", this.createPersonValidator());
              validator.ValidatorFor("Hobbies", this.createItemValidator(), true);

              var hobbiesCountFce = function (args) {
                  args.HasError = false;
                  args.ErrorMessage = "";

                  if (this.Hobbies === undefined || this.Hobbies.length < 2) {
                      args.HasError = true;
                      args.ErrorMessage = "Come on, speak up. Tell us at least two things you enjoy doing";
                      args.TranslateArgs = { TranslateId: 'HobbiesCountMin' };
                      return;
                  }
                  if (this.Hobbies.length > 4) {
                      args.HasError = true;
                      args.ErrorMessage = "'Do not be greedy. Four hobbies are probably enough!'";
                      args.TranslateArgs = { TranslateId: 'HobbiesCountMax'};
                      return;
                  }
              };

              validator.Validation({ Name: "Hobbies", ValidationFce: hobbiesCountFce });

              return validator;
          };
          BusinessRules.prototype.createPersonValidator = function () {
              //create custom composite validator
              var personValidator = new Validation.AbstractValidator();

              //create validators
              var required = new Validators.RequiredValidator();
              var maxLength = new Validators.MaxLengthValidator(15);

              //assign validators to properties
              personValidator.RuleFor("FirstName", required);
              personValidator.RuleFor("FirstName", maxLength);

              personValidator.RuleFor("LastName", required);
              personValidator.RuleFor("LastName", maxLength);

              personValidator.ValidatorFor("Contact", this.createContactValidator());

              return personValidator;
          };
          BusinessRules.prototype.createContactValidator = function () {
              //create custom validator
              var validator = new Validation.AbstractValidator();
              validator.RuleFor("Email", new Validators.RequiredValidator());
              validator.RuleFor("Email", new Validators.MaxLengthValidator(100));
              validator.RuleFor("Email", new Validators.EmailValidator());
              return validator;
          };
          BusinessRules.prototype.createItemValidator = function () {
              //create custom validator
              var validator = new Validation.AbstractValidator();
              validator.RuleFor("HobbyName", new Validators.RequiredValidator());
              return validator;
          };
          return BusinessRules;
      })();
      return new BusinessRules().MainValidator;
  })
{% endhighlight %}

The examples of usage of MainRule using rule directive

{% highlight html %}
<input type="text" ng-model="va.data.Person.LastName" rule="va.MainRule.AbstractValidators['Person']" />
<input type="text" ng-model="va.data.Person.FirstName" rule="va.MainRule.AbstractValidators['Person']" />
<input type="text" ng-model="va.data.Person.Contact.Email" rule="va.MainRule.AbstractValidators['Person'].AbstractValidators['Contact']" />
{% endhighlight %}

Rule directive binds validation rules definitions to angular validations.
This simple directive reads the validations rules definitions and dynamically create angular validation functions.

{% highlight js %}

app.directive('rule', function ($parse) {
    return {
        require:'ngModel',
        restrict: 'A',
        link: function (scope, element, attrs, ngModel) {
          if (ngModel === undefined) return;

                //parse ngModel expression
                var lastIndexOf = attrs.ngModel.lastIndexOf('.');
                if (lastIndexOf === -1) return;

                //set model expression
                var parentModel = attrs.ngModel.substr(0, lastIndexOf);
                //set property name
                var propertyName = attrs.ngModel.substr(lastIndexOf + 1);

                //create rule
                var rule = scope.$eval(attrs.rule);
                if (rule === undefined) return;

                var propRule = rule.Validators[propertyName];
                if (propRule === undefined) return;


                angular.forEach(propRule, function (validator) {
                    if (validator.isAsync){
                      //async validators
                      ngModel.$asyncValidators[validator.tagName] = function (value) {
                          if (value === undefined) return false;
                          return validator.isAcceptable(value)
                      };
                    }
                    else{
                      //sync validators
                      ngModel.$validators[validator.tagName] = function (value) {
                          if (value === undefined) return false;
                          return validator.isAcceptable(value)
                      };
                    }
                })
        }
    };
});

{% endhighlight %}

You can  play with these examples here:

<iframe style="border: 1px solid #999;width: 100%; height: 420px"
        src="http://embed.plnkr.co/zBjfdp3W3hjzQ9DblY2K/preview" frameborder="0"
        allowfullscreen="allowfullscreen">
  Loading plunk...
</iframe>



TODO:

+   Optional validations - angular validations can be optional with ng-if replacement (not usable when you need to cache UI state)
+   shared validations (validations that are dependent on more then one field (ng-model))

<div class="alert alert-success" role="alert">Well done - we have finished.</div>