# Apex Style Guide

## Table of Contents
* [Introduction](#Introduction)
* [Naming Conventions](#Naming-Conventions)
* [Formatting](#Formatting)
* [Best Practices](#Best-Practices)
* [Testing](#Testing)
* [Error Handling](#Error-Handling)


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
q1BECCalc | :x: | Name is not descriptive of what the method does
house|:x:| Not a verb
ProcessPayment |:x: | Name isn't written in lowerCamelCase, processPayment is the correct way to write this method name. 
generatePDF|:white_check_mark:|
calculateTax|:white_check_mark:|

#### Variable Names
Variable names are written in lowerCamelCase. The names should be short yet descriptive. One character names should be avoided with the exception of temporary variables like those used within a **short** For Loop (ex. i,j,k,m,n). A variable's purpose should be clear by its name, a programmer shouldn't have to navigate to where a variable is declared to understand what it is used for. 

##### Constant Names
Constants are the only types of variables that should be written in uppercase with words separated by underscores ex. MIN_AMOUNT. 


Method Name | Correct | Reason
-----------|---------|-------|
Z | :x: | What is Z referring to? This would only be acceptable if it was used within a very short section of code as a throwaway variable, but even then a more common temporary variable name like i or j should be used instead. 
OriginalQuantity|:x:| The variable is not written in lowerCamelCase, originalQuantity is the correct way to write this. 
agDEC |:x: | It is not clear what the intent of this variable is. 
locationInventory|:white_check_mark:|
accountID|:white_check_mark:|

### Formatting
We use the [Prettier Code Formatter](https://developer.salesforce.com/tools/vscode/en/user-guide/prettier/) for APEX which is available through this [plugin](https://github.com/dangmai/prettier-plugin-apex).  Most developers are auto formatting their code upon save through the use of a code formatting engine. If multiple developers are working on the same file it is very beneficial that they use the same code formatter. For this reason the use of this plugin is encouraged but not required.

From a code review perspective the following standards will be enforced.

#### Braces are Used Where Optional
Braces should be used with `if`, `else`, `for`, `do`, and `while` statements even if the body includes a single statement or is empty.
##### Incorrect Example 
```apex
if(condition)
    statement;
```
##### Correct Example
```apex
if(condition){
    statement;
}
```

#### Maximum Line Length 120 Characters
The maximum line length should generally not exceed 120 characters. Once the limit has been reached the line should be wrapped onto another line. 


### Best Practices
##### No DML in For Loops
##### No SOQL in For Loops
##### Use SOQL For Loops
##### Use Relationships to Limit Queries
##### Use Database Methods to Allow Partial Success
##### Only Use One Trigger Per Object 
##### Keep Logic Out of Triggers
##### Avoid Hardcoding IDs
##### Write Clean Code
##### Bulkify Code

### Testing
### Error Handling
