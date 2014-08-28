---
layout: post
category : lessons
tagline: "hobbies form"
tags : [intro, beginner, business rules, tutorial, validation, form]
---
{% include JB/setup %}

# Business rules engine

![logo]({{site.url}}/assets/images/form_logo.jpg)

Business rules engine is a lightweight JavaScript library for easy business rules definition of the product, the contract, the form etc.  
  
  
<div class="alert alert-danger" role="alert">There are no dumb questions. Why i should use it?</div>  

## Business rules engine key features

The main benefit is that business rules engine is not tight to HTML DOM or any other UI framework.
This validation engine is **UI agnostic** and that is why it can be used as **an independent representation of business rules** of a product, contract, etc.
It can be easily reused by different types of applications, libraries.

+   It enables to decorate custom objects and its properties with validation rules.
+   It supports composition of validation rules, that enables to validate custom object with nested structures.
+   It is ease to create your own custom validators.
+   It supports asynchronous validation rules (uses promises).
+   It supports shared validation rules.
+   It supports assigning validation rules to collection-based structures - arrays and lists.
+   It supports localization of error messages with TranslateArgs.
+   It deploys as AMD, CommonJS or plain script module pattern.
+   It offers basic build-in constrains validators. Other custom validators can be find in extensible repository of custom validators (work in progress).

<div class="alert alert-info" role="alert">It looks promising and want to see more.</div>

## Useful information

+ [Business rules engine - Tutorial](https://github.com/rsamec/business-rules-engine/wiki)
+ [Business rules engine - API](http://rsamec.github.io/business-rules-engine/docs/globals.html)
+ [Business rules repository - sources](https://github.com/rsamec/business-rules)
+ [Business rules repository - API](http://rsamec.github.io/business-rules/docs/globals.html)
+ [NodeJS Example](https://github.com/rsamec/node-form-app)
+ [AngularJS Example](https://github.com/rsamec/angular-form-app)
+ [AngularJS Demo - Forms app](http://nodejs-formvalidation.rhcloud.com/)
    + [Vacation Request form](http://nodejs-formvalidation.rhcloud.com/#/vacationApproval/new)
    + [Hobbies form](http://nodejs-formvalidation.rhcloud.com/#/hobbies/new)

<div class="alert alert-danger" role="alert">Stop! I urgently need some concrete example.</div>

## To create business rules for hobbies

The executive decided that they need information about all employees hobbies. You are selected as the best programmer to create the business rules.

#### Requirements

The executive requirements:

+   each employee must give at least 2 hobbies, maximum hobbies is 4
+   each hobby can have optional information about
    +   How often do you participate in this hobby
    +   Is a paid hobby
    +   Would recommend this hobby to a friend
+   your firm has foreign employees - form must support czech, english and german languages
+   the GUI should look like - [Hobbies form](http://nodejs-formvalidation.rhcloud.com/#/hobbies/new)

<div class="alert alert-warning" role="alert">Someone has already solved this problem.</div>

#### Define your business rules

It would be nice if there is a huge business rules repository and you can find your business rules.
But back in reality, your business rules are too special so that you have to create you own hobbies business rules.

#### Describe data structure 

First we try to describe what hobbies data we have to pick up from employees 

+ create data structure for hobbies data

{% highlight js %}
        
    /**
     * Data structure for hobbies data.
     */
    export interface IHobbiesData {

        /**
         * Person identification
         */
        Person?:Shared.IPerson

        /**
         *  The things you enjoy doing.
         */
        Hobbies?:Array<IHobby>;
    }
    
{% endhighlight %}

+ create hobby structure with all optional information

{% highlight js %}   
    
    /**
     * The things you enjoy doing.
     */
    export interface IHobby
    {
        /**
         * Description of your hobby.
         */
        HobbyName?:string;

        /**
         * How often do you participate in this hobby.
         */
        Frequency?:HobbyFrequency;

        /**
         * Return true if this is a paid hobby, otherwise false.
         */
        Paid?:boolean;


        /**
         * Return true if you would recommend this hobby to a friend, otherwise false.
         */
        Recommedation?:boolean;

    }
{% endhighlight %}

+ create options for hobby frequency 

{% highlight js %}
    /**
     * How often do you participate in this hobby.
     */
    export enum HobbyFrequency {
        Daily, Weekly, Monthly

    }
    
{% endhighlight %}

#### Define business rules 

To define new business rule for an object, you have to create abstract validator.
You can assign some constraints (required,maxlength,email, etc.) to properties of an object. 

+   create validator for person object
{% highlight js %}
        private createPersonValidator():Validation.IAbstractValidator<Shared.IPerson> {
       
                   //create custom composite validator
                   var personValidator = new Validation.AbstractValidator<Shared.IPerson>();
       
                   //create validators
                   var required = new Validators.RequiredValidator();
                   var email = new Validators.EmailValidator();
                   var maxLength = new Validators.MaxLengthValidator();
                   maxLength.MaxLength = 15;
       
                   //assign validators to properties
                   personValidator.RuleFor("FirstName", required);
                   personValidator.RuleFor("FirstName", maxLength);
       
                   personValidator.RuleFor("LastName", required);
                   personValidator.RuleFor("LastName", maxLength);
       
                   personValidator.RuleFor("Email", required);
                   personValidator.RuleFor("Email", email);
       
                   return personValidator;
               }
{% endhighlight %}

+   create validator for hobby object
{% highlight js %}
         private createItemValidator():Validation.IAbstractValidator<IHobby> {
        
                    //create custom validator
                    var validator = new Validation.AbstractValidator<IHobby>();
        
                    var required = new Validators.RequiredValidator();
                    validator.RuleFor("HobbyName",required);
        
                    return validator;
                }
{% endhighlight %}

+   compose both created validators to one main validator
+   create custom constraint for minimum and maximum number of hobbies
{% highlight js %}
        private createMainValidator():Validation.IAbstractValidator<IHobbiesData> {
        
                    //create custom validator
                    var validator = new Validation.AbstractValidator<IHobbiesData>();
        
                    validator.ValidatorFor("Person",this.createPersonValidator());
                    validator.ValidatorFor("Hobbies",this.createItemValidator(),true);
        
                    var hobbiesCountFce:Validation.IValidate = function(args:Validation.IError){
                        args.HasError = false;
                        args.ErrorMessage = "";
        
                        if (this.Hobbies === undefined || this.Hobbies.length < 2){
                            args.HasError = true;
                            args.ErrorMessage = "Come on, speak up. Tell us at least two things you enjoy doing";
                            args.TranslateArgs = {TranslateId:'HobbiesCountMin', MessageArgs:this.Hobbies.length};
                            return;
                        }
                        if (this.Hobbies.length > 4){
                            args.HasError = true;
                            args.ErrorMessage = "'Do not be greedy. Four hobbies are probably enough!'";
                            args.TranslateArgs = {TranslateId:'HobbiesCountMax',MessageArgs:this.Hobbies.length};
                            return;
                        }
                    };
        
                    validator.Validation({Name:"HobbiesCount",ValidationFce:hobbiesCountFce});
        
                    return validator;
                }
{% endhighlight %}

+ create method that executes all business rules
{% highlight js %}
          /**
                 * Executes all business rules.
                 */
                public Validate():Q.Promise<Validation.IValidationResult> {
                    this.MainValidator.ValidateAll(this.Data);
                    return this.MainValidator.ValidateAsync(this.Data);
                }

{% endhighlight %}
 
<div class="alert alert-success" role="alert">Well done - your hobbies business rules are done.</div>

Look at the [API documentation](http://rsamec.github.io/business-rules/docs/modules/hobbies.html).

