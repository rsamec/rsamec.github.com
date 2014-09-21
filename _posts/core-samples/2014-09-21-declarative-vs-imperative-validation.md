---
layout: post
category : lessons
tagline: "going deep"
tags : [business rules, validation, form]
---
{% include JB/setup %}

<div style="float:right;margin:0 10px 10px 0" markdown="1">
  ![logo]({{site.url}}/assets/images/form_logo.jpg)
</div>


I would like to demonstrate 4 ways how to define some validation rules. I use as an example the validation rules for the hobbies form.

1. non-formal textual representation - typically requirements from customer
2. declarative - [JSON schema](http://json-schema.org/) with validation keywords [JSON Schema Validation](http://json-schema.org/latest/json-schema-validation.html)
3. declarative - raw JSON data annotated with meta data - using keywords from [JQuery validation plugin](http://jqueryvalidation.org/)
4. imperative - [validation API](http://rsamec.github.io/business-rules-engine/docs/modules/validation-validation.validation.html)

Next i would like to show how all these representation are implemented in [business rules engine](https://github.com/rsamec/business-rules-engine).

You can  play with these examples here:

<iframe style="border: 1px solid #999;width: 100%; height: 600px"
        src="http://embed.plnkr.co/vbp9aFw7Lc2EyNyhUf5y/?t=code?f=br-api" frameborder="0"
        allowfullscreen="allowfullscreen">
  Loading plunk...
</iframe>


#### 1. Non-formal textual representation:

+   person
    +   first name + last name are required and both have maximal 15 characters
    +   email is required, maximal length is 100 characters and it is a valid Internet email address as defined by RFC 5322, section 3.4.1 
+   hobbies
    +   each person must give at least 2 hobbies, maximum hobbies is 4
    +   each hobby must have name name that is maximal 100 characters long
    +   each hobby can have optional information about
        +   How often do you participate in this hobby
        +   Is a paid hobby
        +   Would recommend this hobby to a friend

<div class="alert alert-success" role="alert">This is very short and compact representation, but it is not formal, it is be parsed and used by computers.</div>
<div class="alert alert-danger" role="alert">It is not formal, it can not be parsed and used by computers.</div>
     
<hr/>

#### 2. JSON schema validation
[JSON schema](http://json-schema.org/) with validation keywords [JSON Schema Validation](http://json-schema.org/latest/json-schema-validation.html)

{% highlight js %}
 angular.module('myApp').factory('businessRules',function(){
    var json = {
         Person: {
             type: "object",
             properties: {
                 FirstName: { type: "string", title: "First name", required: true, maxLength: 15 },
                 LastName: { type: "string", title: "Last name", required: true, maxLength: 15 },
                 Contact: {
                     type: "object",
                     properties: {
                         Email: {
                             type: "string", title: "Email",
                             required: true,
                             maxLength: 100,
                             email: true
                         }
                     }
                 }
             }
         },
         Hobbies: {
             type: "array",
             items: {
                 type: "object",
                 properties: {
                     HobbyName: { type: "string", title: "HobbyName", required: true, maxLength: 100 },
                     Frequency: { type: "string", title: "Frequency", enum: ["Daily", "Weekly", "Monthly"] },
                     Paid: { type: "boolean", title: "Paid" },
                     Recommedation: { type: "boolean", title: "Recommedation" }
                 }
             },
             maxItems: 4,
             minItems: 2
         }
     };
     return new FormSchema.JsonSchemaRuleFactory(json).CreateRule("Main");
 })
{% endhighlight %}

<div class="alert alert-success" role="alert">Well defined formal specification. The data are automatically valid against to JSON Schema.</div>
<div class="alert alert-danger" role="alert">Declarative representation is more restrictive in expressiveness to imperative representation.</div>

<hr/>

#### 3. JSON data annotated with meta data

raw JSON data  annotated with meta data - using keywords from [JQuery validation plugin](http://jqueryvalidation.org/)

{% highlight js %}
angular.module('myApp').factory('businessRules',function(){
   var json = {
        
        Person: {
            FirstName: {
                rules: {required: true, maxlength: 15}
            },
            LastName: {
                rules: { required: true, maxlength: 15 }
            },
            Contact: {
                Email: {
                    rules: {
                        required: true,
                        maxlength: 100,
                        email: true
                    }
                }
            }
        },
        Hobbies: [
            {
                HobbyName: {
                    rules: { required: true, maxlength: 100 }
                },
                Frequency: {
                    rules: { enum: ["Daily", "Weekly", "Monthly"]
                    }
                }},
            { maxItems: 4, minItems: 2}
        ]
    };
    
    return new FormSchema.JQueryValidationRuleFactory(json).CreateRule("Main");
}){% endhighlight %}

<div class="alert alert-success" role="alert">Compact formal declarative syntax.</div>
<div class="alert alert-danger" role="alert">Declarative representation is more restrictive in expressiveness to imperative representation.</div>

<hr/>

#### 4. Validation API
 
[validation API](http://rsamec.github.io/business-rules-engine/docs/modules/validation-validation.validation.html)


{% highlight js %}
  angular.module('myApp').factory('businessRules',function(){
    var BusinessRules = (function () {
       
          function BusinessRules() {
              
              this.MainValidator = this.createMainValidator().CreateRule("Main");
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

Optional for typescript you can define data structure to ensure strong-type support when coding.

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
    
{% endhighlight %}


<div class="alert alert-success" role="alert">Formal declarative syntax, simpler reuse, easily extensible.</div>
<div class="alert alert-danger" role="alert">For simple examples is not as compact as declarative representation.</div>

<hr/>

## Going deep

To add declarative support in [business rules engine](https://github.com/rsamec/business-rules-engine) i use [validation API](http://rsamec.github.io/business-rules-engine/docs/modules/validation-validation.validation.html).
I share common validators for [basic build-in constrains](http://rsamec.github.io/business-rules-engine/docs/modules/validation-basicvalidators.validators.html).

I write one factory for each syntax

+ JsonSchemaRuleFactory - [JSON schema](http://json-schema.org/) with validation keywords [JSON Schema Validation](http://json-schema.org/latest/json-schema-validation.html)
+ JQueryValidationRuleFactory - raw JSON data annotated with meta data - using keywords from [JQuery validation plugin](http://jqueryvalidation.org/)

{% highlight js %}

    /**
     * It represents the JSON schema factory for creating validation rules based on JSON form schema.
     * It uses constraints keywords from JSON Schema Validation specification.
     */
    export class JsonSchemaRuleFactory implements IValidationRuleFactory{

        /**
         * Default constructor
         * @param jsonSchema JSON schema for business rules.
         */
        constructor(private jsonSchema:any){
        }

        /**
         * Return concrete validation rule structured according to JSON schema.
         * @param name validation rule name
         * @returns {IAbstractValidationRule<any>} return validation rule
         */
        public CreateRule(name:string):Validation.IAbstractValidationRule<any>{
            return this.ParseAbstractRule(this.jsonSchema).CreateRule(name);
        }


        /**
         * Returns an concrete validation rules structured according to JSON schema.
         */
        private ParseAbstractRule(formSchema:any):Validation.IAbstractValidator<any> {

            var rule = new Validation.AbstractValidator<any>();

            for (var key in formSchema) {
                var item = formSchema[key];
                var type = item[Util.TYPE_KEY];
                if (type === "object") {
                    rule.ValidatorFor(key, this.ParseAbstractRule(item[Util.PROPERTIES_KEY]));
                }
                else if (type === "array") {
                    _.each(this.ParseValidationAttribute(item),function(validator){ rule.RuleFor(key,validator)});
                    rule.ValidatorFor(key, this.ParseAbstractRule(item[Util.ARRAY_KEY][Util.PROPERTIES_KEY]), true);
                }
                else {
                    _.each(this.ParseValidationAttribute(item),function(validator){ rule.RuleFor(key,validator)});
                }
            }
            return rule;
        }
        /**
         * Return list of property validators that corresponds json items for JSON form validation tags.
         * See keywords specifications -> http://json-schema.org/latest/json-schema-validation.html
         */
        private ParseValidationAttribute(item:any):Array<Validation.IPropertyValidator> {

            var validators = new Array<Validation.IPropertyValidator>();
            if (item === undefined) return validators;

            //5.  Validation keywords sorted by instance types
            //http://json-schema.org/latest/json-schema-validation.html

            //5.1. - Validation keywords for numeric instances (number and integer)
            // multipleOf validation
            validation = item["multipleOf"];
            if (validation !== undefined) {
                validators.push(new Validators.MultipleOfValidator(validation));
            }

            // maximum validation
            validation = item["maximum"];
            if (validation !== undefined) {
                validators.push(new Validators.MaxValidator(validation,item["exclusiveMaximum"]));
            }

            // minimum validation
            validation = item["minimum"];
            if (validation !== undefined) {
                validators.push(new Validators.MinValidator(validation,item["exclusiveMinimum"]));
            }

            //5.2. - Validation keywords for strings

            // maxLength validation
            validation = item["maxLength"];
            if (validation !== undefined) {
                validators.push(new Validators.MaxLengthValidator(validation));
            }

            // minLength validation
            validation = item["minLength"];
            if (validation !== undefined) {
                validators.push(new Validators.MinLengthValidator(validation));
            }
            // pattern validation
            validation = item["pattern"];
            if (validation !== undefined) {
                validators.push(new Validators.PatternValidator(validation));
            }


            //5.3.  Validation keywords for arrays
            //TODO: additionalItems and items

            // min items validation
            validation= item["minItems"];
            if (validation !== undefined) {
                validators.push( new Validators.MinItemsValidator(validation))
            }

            // max items validation
            validation = item["maxItems"];
            if (validation !== undefined) {
                validators.push( new Validators.MaxItemsValidator(validation))
            }

            // uniqueItems validation
            validation = item["uniqueItems"];
            if (validation !== undefined) {
                validators.push( new Validators.UniqItemsValidator(validation))
            }

            //5.4.  Validation keywords for objects
            //TODO: maxProperties, minProperties, additionalProperties, properties and patternProperties, dependencies

            // required validation
            var validation = item["required"];
            if (validation !== undefined && validation) {
                validators.push(new Validators.RequiredValidator());
            }

            //5.5.  Validation keywords for any instance type
            // enum validation
            validation = item["enum"];
            if (validation !== undefined) {
                validators.push(new Validators.EnumValidator(validation))
            }

            // type validation
            var validation = item["type"];
            if (validation !== undefined) {
                validators.push(new Validators.TypeValidator(validation));
            }
            //7.3.2 email
            validation = item["email"];
            if (validation !== undefined) {
                validators.push(new Validators.EmailValidator())
            }

            //7.3.6 url
            validation = item["uri"];
            if (validation !== undefined) {
                validators.push(new Validators.UrlValidator())
            }

            //TODO: allOf,anyOf,oneOf,not,definitions

            return validators;
        }
    }

{% endhighlight %}



{% highlight js %}
    /**
     * It represents the JSON schema factory for creating validation rules based on raw JSON data annotated by validation rules.
     * It uses constraints keywords from JQuery validation plugin.
     */
    export class JQueryValidationRuleFactory implements IValidationRuleFactory  {

       static RULES_KEY = "rules";
       static DEFAULT_KEY = "default";

        /**
         * Default constructor
         * @param metaData -  raw JSON data annotated by validation rules
         */
        constructor(private metaData:any){
        }

        /**
         * Return an concrete validation rule by traversing raw JSON data annotated by validation rules.
         * @param name validation rule name
         * @returns {IAbstractValidationRule<any>} return validation rule
         */
        public CreateRule(name:string):Validation.IAbstractValidationRule<any>{
            return this.ParseAbstractRule(this.metaData).CreateRule(name);
        }

        /**
         * Returns an concrete validation rule structured according to JSON schema.
         */
        private ParseAbstractRule(metaData:any):Validation.IAbstractValidator<any> {

            var rule = new Validation.AbstractValidator<any>();

            for (var key in metaData) {
                var item = metaData[key];
                var rules = item[JQueryValidationRuleFactory.RULES_KEY];

                if ( _.isArray(item)) {
                    if (item[1] !== undefined) {
                        _.each(this.ParseValidationAttribute(item[1]), function (validator) {
                            rule.RuleFor(key, validator)
                        });
                    }
                    rule.ValidatorFor(key, this.ParseAbstractRule(item[0]), true);
                }
                else if (rules !== undefined) {
                    _.each(this.ParseValidationAttribute(rules),function(validator){ rule.RuleFor(key,validator)})
                }
                else {
                    rule.ValidatorFor(key, this.ParseAbstractRule(item));
                }
            }
            return rule;
        }

        /**
         * Return list of property validators that corresponds json items for JQuery validation pluging tags.
         * See specification - http://jqueryvalidation.org/documentation/
         */
        private ParseValidationAttribute(item:any):Array<Validation.IPropertyValidator> {

           var validators = new Array<Validation.IPropertyValidator>();
           if (item === undefined) return validators;

           var validation = item["required"];
           if (validation !== undefined && validation) {
               validators.push(new Validators.RequiredValidator());
           }

            var validation = item["remote"];
            if (validation !== undefined && validation) {
                validators.push(new Validators.RemoteValidator(validation));
            }

           // maxLength validation
           validation = item["maxlength"];
           if (validation !== undefined) {
               validators.push(new Validators.MaxLengthValidator(validation))
           }

           // minLength validation
           validation = item["minlength"];
           if (validation !== undefined) {
               validators.push(new Validators.MinLengthValidator(validation))
           }

            // rangelength validation
            validation = item["rangelength"];
            if (validation !== undefined) {
                validators.push(new Validators.RangeLengthValidator(validation))
            }

            // maximum validation
            validation = item["max"];
            if (validation !== undefined) {
                validators.push(new Validators.MaxValidator(validation));
            }

            // minimum validation
            validation = item["min"];
            if (validation !== undefined) {
                validators.push(new Validators.MinValidator(validation));
            }

            // range validation
            validation = item["range"];
            if (validation !== undefined) {
                validators.push(new Validators.RangeValidator(validation));
            }

           validation = item["email"];
           if (validation !== undefined) {
               validators.push(new Validators.EmailValidator())
           }

            validation = item["url"];
            if (validation !== undefined) {
                validators.push(new Validators.UrlValidator())
            }

            validation = item["date"];
            if (validation !== undefined) {
                validators.push(new Validators.DateValidator())
            }

            validation = item["dateISO"];
            if (validation !== undefined) {
                validators.push(new Validators.DateISOValidator())
            }

            validation = item["number"];
            if (validation !== undefined) {
                validators.push(new Validators.NumberValidator())
            }


            validation = item["digits"];
            if (validation !== undefined) {
                validators.push(new Validators.DigitValidator())
            }

            validation = item["creditcard"];
            if (validation !== undefined) {
                validators.push(new Validators.CreditCardValidator())
            }

            validation = item["equalTo"];
            if (validation !== undefined) {
                validators.push(new Validators.EqualToValidator(validation))
            }


            // min items validation
            validation= item["minItems"];
            if (validation !== undefined) {
                validators.push( new Validators.MinItemsValidator(validation))
            }

            // max items validation
            validation = item["maxItems"];
            if (validation !== undefined) {
                validators.push( new Validators.MaxItemsValidator(validation))
            }

            // uniqueItems validation
            validation = item["uniqueItems"];
            if (validation !== undefined) {
                validators.push( new Validators.UniqItemsValidator(validation))
            }

            // enum validation
            validation = item["enum"];
            if (validation !== undefined) {
                validators.push(new Validators.EnumValidator(validation))
            }


           return validators;
       }
    }
    
{% endhighlight %}


<div class="alert alert-success" role="alert">Well done - it is so simple to be declarative.
If you have your own definition of validation rules (xml, html) you can write your custom adapter (factory).</div>


### Summary

I am not a big fan of declarative validation. It works really nice for simple example, but for real-world examples problems start to encounter.
There are a lot of small, tiny details to address that are not possible to express in declarative way.
There is always many exception against typical scenario. This exceptional business rule scenario is typically not covered by declarative syntax that is restrictive by its nature.
It is difficult to describe a lot of possibilities that can be found in business in real world.
The reuse of declarative validation rule is also limited.

<div class="alert alert-info" role="alert">Using declarative approach I feel not to have so power in my hands. What do you prefer declarative or imperative approach to validation rules definition?</div>
 