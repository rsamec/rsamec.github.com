---
layout: post
category : lessons
tagline: "Business rules engine introduction"
tags : [intro, beginner, business rules, tutorial, validation, form]
---
{% include JB/setup %}

# Business rules engine

![logo](https://github.com/rsamec/form/blob/master/form_logo.jpg)

Business rules engine is a lightweight JavaScript library for easy business rules definition of the product, the contract, the form etc.

+ [Business rules engine - Tutorial] (https://github.com/rsamec/business-rules-engine/wiki)
+ [Business rules engine - API] (http://rsamec.github.io/business-rules-engine/docs/globals.html)
+ [Business rules repository - sources] (https://github.com/rsamec/business-rules)
+ [Business rules repository - API] (http://rsamec.github.io/business-rules/docs/globals.html)
+ [NodeJS Example] (https://github.com/rsamec/node-form-app)
+ [AngularJS Example] (https://github.com/rsamec/angular-form-app)
+ [AngularJS Demo] (http://nodejs-formvalidation.rhcloud.com/)

## Key features
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



## Example Usage
``` typescript
module Hobbies {

    /**
     * Data structure for generic invoice.
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

    /**
     * How often do you participate in this hobby.
     */
    export enum HobbyFrequency {
        Daily, Weekly, Monthly

    }
}
```

To define business rules for some object, you have to create abstract validator.
``` js
          //create new validator for object with structure<IPerson>
          var personValidator = new Validation.AbstractValidator();

          //basic validators
          var required = new Validators.RequiredValidator();
          var maxLength = new Validators.MaxLengthValidator(15);

          //assigned validators to property
          personValidator.RuleFor("FirstName", required);
          personValidator.RuleFor("FirstName",maxLength);

          //assigned validators to property
          personValidator.RuleFor("LastName", required);
          personValidator.RuleFor("LastName",maxLength);

```

To use business rules and execute them on particular data
```js
          //create test data
          var data = {
                Person1:
                {
                    FirstName:'John',
                    LastName: 'Smith'
                },
                Person2:{}

          }

          //create concrete rule
          var person1Validator = personValidator.CreateRule("Person 1");

          //execute validation
          var result = person1Validator.Validate(this.Data.Person1);

          //verify results
          if (result.HasErrors){
              console.log(result.ErrorMessage);
          }
          //---------
          //--outputs
          //---------

          //create concrete rule
          var person2Validator = personValidator.CreateRule("Person 2");

          //execute validation
          var result = person2Validator.Validate(this.Data.Person1);

           //verify results
          if (result.HasErrors){
              console.log(result.ErrorMessage);
          }

          //---------
          //--outputs
          //---------
          // FirstName: Field is required.
          // LastName: Field is required.

```