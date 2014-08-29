---
layout: post
category : lessons
tagline: "generate valid documents"
tags : [intro, beginner, business rules, tutorial, validation, form]
---
{% include JB/setup %}

# Use hobbies business rules

In [last post]({% post_url core-samples/2014-08-27-business-rules-engine-intro %}) hobbies business rules were created. 
You can see hobbies business rules documentation + npm package.

+   [API](http://rsamec.github.io/business-rules/docs/modules/hobbies.html)
+   [br-hobbies](https://www.npmjs.org/package/br-hobbies) - npm package 

Today example demonstrates how to generate hobbies documents for each employee and enforce that each
 document complies already defined hobbies business rules.

## Generate many hobbies documents

<div class="alert alert-danger" role="alert">Do not repeat yourself. We use existing libraries.</div>
{% highlight js %}

import _ = require('underscore');
import Q = require('q');
var Hobbies = require('br-hobbies');
var mongoose = require('mongoose');

{% endhighlight %}
<div class="alert alert-info" role="alert">Thinking about program decomposition. Define steps to program.</div>

+   get list of employees
+   iterate through each employee
    +   create hobbies data
    +   create hobbies business rules
    +   execute hobbies business rules
    +   verify results
    +   if OK insert to documents DB
    +   if NOT OK log business rules errors



{% highlight js %}

//fake our employee db
var employees = [
    {FirstName:'John',LastName:'Smith'},
    {FirstName:'Goerge',LastName:'Podolsky'},
    {FirstName:'Jan',LastName:'Novak'},
    {FirstName:'Karel',LastName:'Abraham'},
    {FirstName:'Josef',LastName:'Blaha'}
]

{% endhighlight %}

{% highlight js %}
//connect to mongo
mongoose.connect('mongodb://rsamec:[password]@ds059908.mongolab.com:59908/documents');

//create a mongoose model
var Doc = mongoose.model('docs', {
    shortName:String,
    name: String,
    desc: String,
    data: Object,
    created: Date,
    updated: Date});

//create document header
var docHeader = {
    shortName: 'hobbies',
    name:'Hobbies',
    desc:'Hobbies business rules.',
    created:new Date()
};

{% endhighlight %}

{% highlight js %}

//iterate list of employee
_.each(employees, function(employee){

    //create  data
    var data:Hobbies.IHobbiesData = {
        Person:{
            FirstName: employee.FirstName,
            LastName: employee.LastName,
            Email: employee.FirstName.charAt(1) + employee.LastName + "@gmail.com"
        },
        Hobbies:[
            {HobbyName:"English", Frequency:Hobbies.HobbyFrequency.Weekly , Paid:true,Recommendation:true},
            {HobbyName:"Swimming", Frequency:Hobbies.HobbyFrequency.Monthly , Paid:false,Recommendation:true}
        ]
    };

    //create business rules
    var businessRules = new Hobbies.BusinessRules(data);

    //execute business rules
    var promise = businessRules.Validate();

    //verify results
    return promise.then(function (result) {

        //log if any errors encounters
        if (result.HasErrors){
            console.log('error encounters at employee: ' + employee.LastName);
            console.log(result.ErrorMessage);
            return;
        }

        //create mongo document -> combine document header with document data
        var document = new Doc(_.extend(docHeader, {data:data}));

        //save to Db
        document.save(function (err) {
            if (err) {console.log('error encounters at employee: ' + employee.LastName)};
        });
    });
})
{% endhighlight %}

<div class="alert alert-success" role="alert">Well done.You have just finished your task.</div> 

You can find wor
