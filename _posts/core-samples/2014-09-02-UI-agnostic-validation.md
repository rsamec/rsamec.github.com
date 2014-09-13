---
layout: post
category : lessons
tagline: "separation of concern"
tags : [tutorial, validation, form]
---
{% include JB/setup %}

# Validation rules

There are many validation engines.

+   [jquery validation plugin](http://jqueryvalidation.org/)
+   [angularjs form validation](http://scotch.io/tutorials/javascript/angularjs-form-validation)
+   [angular-form-for](http://bvaughn.github.io/angular-form-for/#/index)
+   [backbone validation](https://github.com/thedersen/backbone.validation)

These libraries are great and simplifies a lot of hard work of validating user input.  

<div class="alert alert-danger" role="alert">There are no silver bullet. What is the problem with these libraries?</div> 

All of these libraries are more or less tight to HTML DOM or any other UI framework.
If you want to use the same validation rules with no-UI scenario, then problems starts to encounter.
Some non-UI scenario

+   server validation - revalidate data on server -> do not trust any data sent from client
+   validate arbitrary data send from third-party
+   bulk imports and validation - e.g. generate invoices, bankbooks, etc. from DB with need to validate its data before generation or printing
+   multiple third-party clients - e.g. runs its own system (different UIs) and use service to send data to you, service is responsible for enforcing the same business rules 
+   multiple UIs - service to enforce the same business rules for different UI clients 


See lightweight JavaScript library that represents UI agnostic validation engine. [Business rules engine](https://github.com/rsamec/business-rules-engine/wiki)
  

<div class="alert alert-info" role="alert">The rule of thumb. Minimize dependency. Especially on UI.</div>  

## Validation rules should be UI agnostic

The validation engine should be **UI agnostic** and only then it can be used as **an independent representation of business rules** of a product, contract, etc.
It can be easily reused by different types of applications, libraries.


<div class="alert alert-danger" role="alert">Computers are for humans. Each application needs some UI.</div>

## How to bind validations and error messages to UI (AngularJs framework)

<div class="alert alert-info" role="alert">Separation of concern. Distinguish between UI and non-UI factors.</div>

It is important to distinguish between

+   UI aspects (validation rule definition, validation result (hasError,isValid), error message content, error message localization)
+   non-UI aspects (validation execution (input changed, save button, etc.), dirty and pristine flags, error message display, error message translation mechanism)


This is an example how to **bind validation rules to UI** via angular simple **rule directive**.

{% highlight html %}

 <input type="text" ng-model="model.Data.Employee.FirstName" rule="model.EmployeeValidator">

{% endhighlight %}

This directive executes validation specified by directive rule (rule="model.EmployeeValidator") for the property "FirstName" used in ngModel directive (ng-model="model.Data.Employee.FirstName").  

{% highlight js %}

uiControls.directive('rule', function ($parse) {
    return {
        require:'ngModel',
        restrict: 'A',
        link: function (scope, element, attrs, ctrl) {
            if (ctrl===undefined) return;
            
            //parse ngModel expression
            var lastIndexOf = attrs.ngModel.lastIndexOf('.');
            if (lastIndexOf === -1) return;
            
            //set model expression
            var parentModel = attrs.ngModel.substr(0,lastIndexOf);
            //set property name
            var propertyName = attrs.ngModel.substr(lastIndexOf + 1);
            
            //create rule
            var rule = scope.$eval(attrs.rule);
            
            //create parent getter
            var getter = $parse(parentModel);

            //when value changed then validate property 
            ctrl.$viewChangeListeners.push(function(){
                
                //execute validation
                rule.ValidateProperty(getter(scope),propertyName);
                
                //set dirty flag for correct display
                rule.ValidationResult.Errors[propertyName].IsDirty = true;
            });
        }
    };
});

{% endhighlight %}
 
Another example solves the **display of error messsages and its translations** via **error directive**.
For translation it is used [angular-translate](https://github.com/angular-translate/angular-translate) service.

{% highlight html %}

<div error="model.HobbiesNumberValidator.Error"></div>

{% endhighlight %}

{% highlight js %}
uiControls.directive('error', function ($translate) {
    return {
        restrict: 'A',
        scope:{
            error:'='
        },
        template:'<div class="validation-error">{{errMsg}}</div>',
        link: function (scope, element, attrs) {
            var setErrMsg = function() {
                if (!scope.error.HasError) {
                    scope.errMsg = undefined;
                    return
                }

                if (scope.error.TranslateArgs == undefined) {
                    scope.errMsg = scope.error.ErrorMessage;
                    return;

                }
                $translate(scope.error.TranslateArgs.TranslateId).then(function (errMsg) {
                    if (scope.error.TranslateArgs.CustomMessage != undefined){
                        scope.errMsg = scope.error.TranslateArgs.CustomMessage(errMsg, scope.error.TranslateArgs.MessageArgs);
                    }else {
                        scope.errMsg = Utils.StringFce.format(errMsg, scope.error.TranslateArgs.MessageArgs);
                    }},
                    function (reason) {
                        //fallback to default error message
                        scope.errMsg = scope.error.ErrorMessage;
                    }
                )
            };

            setErrMsg();
            scope.$watch('error.ErrorMessage', function (newValue, oldValue, scope) {
                setErrMsg();
            }, true);
       }
    };
});
{% endhighlight %}

See it in action 

   + [Vacation Request form](http://nodejs-formvalidation.rhcloud.com/#/vacationApproval/new)
   + [Hobbies form](http://nodejs-formvalidation.rhcloud.com/#/hobbies/new)

<div class="alert alert-success" role="alert">Well done. If you want to know more, see resources below.</div>

## UI agnostic validation framework - more information

+ [Business rules engine - Tutorial](https://github.com/rsamec/business-rules-engine/wiki)
+ [Business rules engine - API](http://rsamec.github.io/business-rules-engine/docs/globals.html)
+ [Business rules repository - sources](https://github.com/rsamec/business-rules)
+ [Business rules repository - API](http://rsamec.github.io/business-rules/docs/globals.html)
+ [NodeJS Example](https://github.com/rsamec/node-form-app)
+ [AngularJS Example](https://github.com/rsamec/angular-form-app)
+ [AngularJS Demo - Forms app](http://nodejs-formvalidation.rhcloud.com/)
    + [Vacation Request form](http://nodejs-formvalidation.rhcloud.com/#/vacationApproval/new)
    + [Hobbies form](http://nodejs-formvalidation.rhcloud.com/#/hobbies/new)


