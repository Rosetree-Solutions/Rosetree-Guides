# Apex Style Guide

## Table of Contents
* [Introduction](#Introduction)
* [Naming Conventions](#Naming-Conventions)
* [Formatting](#Formatting)
* [Testing](#Testing)
* [Error Handling](#Error-Handling)
* [Best Practices](#Best-Practices)

### Introduction
### Naming Conventions
#### Class Names
Class names should be nouns or noun phrases that are written in UpperCamelCase. The name should be made up of whole words avoiding acronyms and abbreviations when possible with the exception of widely used acronyms such as PDF and HTML. The names should be descriptive of the classes purpose while being as short as possible. 

##### Examples
Class Name | Correct | Reason
-----------|---------|-------|
BECAccount|:x:|Abbreviations should be avoided
GenerateInvoice|:x:|Class names should be nouns or noun phrases. 
OppHndlr|:x:|Whole words should be used in the place of abbreviations
postSandboxRefresh | :x:| Class names should start with a capital letter. 
XProxyFactory|:x:|Class name is not descriptive of its purpose.
ProjectClone|:white_check_mark:	
AccountTaxCalculation|:white_check_mark:	


##### Class Suffixes
Suffixes will be used to designate special types of classes. The suffix will be preceded by an underscore which will further highlight the class type, **this is the only time you would use an underscore in a class name**. 

Class Type | Suffix | Example
-----------|--------|---------
Test|_Test|ProjectPricing_Test
Trigger | _Trigger|ProjectPricing_Trigger
Trigger Handler | _TriggerHandler | ProjectPricing_TriggerHandler
Schedulable | _Schedule|ProjectPricing_Schedule
Queueable | _Queueable|ProjectPricing_Queueable
Batch | _Batch| ProjectPricing_Batch
Controller | _Controller | ProjectPricing_Controller

#### Method Names
Method names should be verbs, written in lowerCamelCase. The name should be made up of whole words avoiding acronyms and abbreviations when possible with the exception of widely used acronyms such as PDF and HTML. 

Method Name | Correct | Reason
-----------|---------|-------|
generatePDF|:white_check_mark:|
calculateTax|:white_check_mark:|
Q1BECCalc | :x: | Name is not descriptive of what the method does



### Formatting
### Testing
### Error Handling
### Best Practices