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
