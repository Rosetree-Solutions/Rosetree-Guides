# Apex Style Guide

## Table of Contents

- [Introduction](#Introduction)
- [Naming Conventions](#Naming-Conventions)
- [Formatting](#Formatting)
- [Best Practices](#Best-Practices)
- [Testing](#Testing)
- [Error Handling](#Error-Handling)

### Introduction

### Naming Conventions

#### Class Names

Class names should be nouns or noun phrases that are written in UpperCamelCase. The name should be made up of whole words avoiding acronyms and abbreviations when possible with the exception of widely used acronyms such as PDF and HTML. The names should be descriptive of the classes purpose while being as short as possible.

##### Examples

| Class Name            | Correct            | Reason                                                   |
| --------------------- | ------------------ | -------------------------------------------------------- |
| BECAccount            | :x:                | Abbreviations should be avoided                          |
| GenerateInvoice       | :x:                | Class names should be nouns or noun phrases.             |
| OppHndlr              | :x:                | Whole words should be used in the place of abbreviations |
| postSandboxRefresh    | :x:                | Class names should start with a capital letter.          |
| XProxyFactory         | :x:                | Class name is not descriptive of its purpose.            |
| ProjectClone          | :white_check_mark: |
| AccountTaxCalculation | :white_check_mark: |

##### Class Suffixes

Suffixes will be used to designate special types of classes. The suffix will be preceded by an underscore which will further highlight the class type, **this is the only time you would use an underscore in a class name**.

| Class Type      | Suffix           | Example                       |
| --------------- | ---------------- | ----------------------------- |
| Test            | \_Test           | ProjectPricing_Test           |
| Trigger         | \_Trigger        | ProjectPricing_Trigger        |
| Trigger Handler | \_TriggerHandler | ProjectPricing_TriggerHandler |
| Schedulable     | \_Schedule       | ProjectPricing_Schedule       |
| Queueable       | \_Queueable      | ProjectPricing_Queueable      |
| Batch           | \_Batch          | ProjectPricing_Batch          |
| Controller      | \_Controller     | ProjectPricing_Controller     |

#### Method Names

Method names should be verbs, written in lowerCamelCase. The name should be made up of whole words avoiding acronyms and abbreviations when possible with the exception of widely used acronyms such as PDF and HTML.

| Method Name    | Correct            | Reason                                                                                             |
| -------------- | ------------------ | -------------------------------------------------------------------------------------------------- |
| q1BECCalc      | :x:                | Name is not descriptive of what the method does                                                    |
| house          | :x:                | Not a verb                                                                                         |
| ProcessPayment | :x:                | Name isn't written in lowerCamelCase, processPayment is the correct way to write this method name. |
| generatePDF    | :white_check_mark: |
| calculateTax   | :white_check_mark: |

#### Variable Names

Variable names are written in lowerCamelCase. The names should be short yet descriptive. One character names should be avoided with the exception of temporary variables like those used within a **short** For Loop (ex. i,j,k,m,n). A variable's purpose should be clear by its name, a programmer shouldn't have to navigate to where a variable is declared to understand what it is used for.

##### Constant Names

Constants are the only types of variables that should be written in uppercase with words separated by underscores ex. MIN_AMOUNT.

| Method Name       | Correct            | Reason                                                                                                                                                                                                                    |
| ----------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Z                 | :x:                | What is Z referring to? This would only be acceptable if it was used within a very short section of code as a throwaway variable, but even then a more common temporary variable name like i or j should be used instead. |
| OriginalQuantity  | :x:                | The variable is not written in lowerCamelCase, originalQuantity is the correct way to write this.                                                                                                                         |
| agDEC             | :x:                | It is not clear what the intent of this variable is.                                                                                                                                                                      |
| locationInventory | :white_check_mark: |
| accountID         | :white_check_mark: |

### Formatting

We use the [Prettier Code Formatter](https://developer.salesforce.com/tools/vscode/en/user-guide/prettier/) for APEX which is available through this [plugin](https://github.com/dangmai/prettier-plugin-apex). Most developers are auto formatting their code upon save through the use of a code formatting engine. If multiple developers are working on the same file it is very beneficial that they use the same code formatter. For this reason the use of this plugin is encouraged but not required.

From a code review perspective the following standards will be enforced.

##### Braces are Used Where Optional

Braces should be used with `if`, `else`, `for`, `do`, and `while` statements even if the body includes a single statement or is empty.

###### Incorrect Example

```apex
if(condition)
    statement;
```

###### Correct Example

```apex
if(condition){
    statement;
}
```

##### Variables Should be Declared Where Needed

Local variables should be declared close to the point in the code where they are first used. Do not declare all local variables at the top of their containing block.

###### Incorrect Example

```apex
private static void doSomething(){
    private static String dogName;
    private static String catName;

    /*
     * All the code in the do something method....
     */
}
```

###### Correct Example

```apex
private static void doSomething(){

    private static String dogName;
    /*
     * The code that uses the dogName variable
     */

    private static String catName;
    /*
     * The code that uses the catName variable
     */
}
```

##### Maximum Line Length 120 Characters

The maximum line length should generally not exceed 120 characters. Once the limit has been reached the line should be wrapped onto another line. There aren't strict rules for how a line should be wrapped in all situations as there are many valid approaches. Below are the common line-wrapping techniques that we use.

###### SOQL Line-Wrapping

Short SOQL queries do not need to be wrapped.

```Apex
List<Account> accountList = [SELECT Id, Name FROM Account LIMIT 10];
```

Longer queries should be wrapped before reserved words like `SELECT`, `FROM`, `WHERE`, `LIMIT`, etc.

```Apex
    List<Contact> contacts = [
        SELECT id, salutation, firstname, lastname, email
        FROM Contact
        WHERE accountId = :a.Id
    ];
```

Yet, another example showing how to break up longer queries with an inner query. Note, if you would like to break up a long `SELECT` statement you should break the line after each `,`.

```Apex
    List<Project_Pricing__c> record = [
      SELECT
        Id,
        Product__c,
        Cost__c,
        Vendor__c,
        (
          SELECT Id, Product__c, Project_Pricing__c, Cost_Price__c
          FROM Plan_Type_Items__r
        )
      FROM Project_Pricing__c
      WHERE
        Product__c = :product
        AND Vendor__c = :supplier
        AND Project__c = :project
      LIMIT 10
    ];
```

### Best Practices

##### No DML in For Loops

There are governor limits that enforce the maximum number of DML statements (insert, update, delete, undelete) used within an Apex transaction. When DML statements are placed inside of a `for` loop they are run once per iteration of the loop making it easy to hit the limit quickly.

Instead DML operations should be moved out of `for` loops by collecting the records to modify inside of a collection and invoking the DML operation once on the collection of data.

###### Incorrect Example

```apex
private static void updateAccounts(List<Account> accountList) {
    for(Account a: accountList) {
        a.updated__c = true;
        /*
         * The following DML statement is not efficient
         * because it is run once for every iteration
         * of the loop. If more than 150 accounts were
         * included in the list the limit would be hit
         * due to this one statement alone.
         */
        update a;
    }
}
```

###### Correct Example

```apex
private static void updateAccounts(List<Account> accountList) {
    List<Account> accountsToUpdate = new List<Account>();
    for(Account a: accountList) {
        a.updated__c = true;
        accountsToUpdate.add(a);
    }
    /*
     * All of the accounts were collected in one list
     * and updated via the single DML statement below.
     */
    update accountsToUpdate;
}
```

##### No SOQL in For Loops

There are governor limits that enforce the maximum number of SOQL queries used within an Apex transaction. When SOQL queries are placed inside of a `for` loop they are run once per iteration of the loop making it easy to hit the limit quickly. Instead SOQL queries should be moved out of `for` loops.

###### Incorrect Example

```apex
trigger AccountTrigger on Account(before update) {
  for (Account a : Trigger.new) {
    /*
     * The following query is not efficient as it
     * is run once for every account within Trigger.new.
     * If more than 100 accounts were included in Trigger.new
     * the queries would exceed the governor limit.
     */
    List<Contact> contacts = [
      SELECT id, salutation, firstname, lastname, email
      FROM Contact
      WHERE accountId = :a.Id
    ];
  }
}
```

###### Correct Example

```apex
trigger AccountTrigger on Account(before update) {
  list<Id> accountIDs = new List<Id>();
  for (Account a : Trigger.new) {
    accountIDs.add(a.Id);
  }
  /*
   * The IDs of the accounts were first collected
   * inside of a list. The SOQL query uses the list to
   * find all of the contacts associated with the accounts.
   */
  List<Contact> contacts = [
    SELECT id, salutation, firstname, lastname, email
    FROM Contact
    WHERE accountId IN :accountIDs
  ];
}
```

##### Use Relationships to Limit Queries

##### Use Database Methods to Allow Partial Success

##### Only Use One Trigger Per Object

##### Keep Logic Out of Triggers

##### Avoid Hardcoding IDs

Most IDs change between sandbox and production environments. For this reason we should not place IDs directly in the Apex code.

###### Incorrect Example

```apex
trigger AccountTrigger on Account(before update) {
  for (Account a : Trigger.new) {
    /* This would fail if the Trigger was copied to a
     * different environment as the new environment
     * would have a different ID for the Record Type.
     */
    if (a.RecordTypeId == '012500000009WAr') {
      // logic here...
    }
  }
}
```

###### Correct Example

```apex
trigger AccountTrigger on Account(before update) {
  /*
   * We are getting the account record type ID by name
   * instead of hardcoding the ID. As an added benefit
   * we can get the record type Id without using a SOQL
   * query.
   */
  Id commercialRecordTypeId = Schema.SObjectType.Account.getRecordTypeInfosByName()
    .get('Commercial')
    .getRecordTypeId();

  for (Account a : Trigger.new) {
    if (a.RecordTypeId == commercialRecordTypeId) {
      // logic here...
    }
  }
}
```

##### Write Clean Code

##### Bulkify Code

##### Map Best Practices

### Testing

- 90%
- system assert

### Error Handling
