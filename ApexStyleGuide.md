# Apex Style Guide

## Table of Contents

- [Introduction](#Introduction)
- [Best Practices](#Best-Practices)
  - [No DML in For Loops](#no-dml-in-for-loops)
  - [No SOQL in For Loops](#no-soql-in-for-loops)
  - [Use SOQL Queries Efficiently](#use-soql-queries-efficiently)
  - [Use Relationships in Queries](#use-relationships-in-queries)
  - [Be Mindful & Efficient with Limits](#be-mindful--efficient-with-limits)
  - [Know When To Use DML VS Database Methods](#know-when-to-use-dml-vs-database-methods)
  - [Bulkfiy Your Code](#bulkify-your-code)
  - [Avoid Hardcoding IDs](#avoid-hardcoding-ids)
  - [Trigger Best Practices](#trigger-best-practices)
    - [Only Have One Trigger Per Object](#only-have-one-trigger-per-object)
    - [Keep Logic Out of Triggers](#keep-logic-out-of-triggers)
    - [Use the RTS Trigger Framework](#use-the-rts-trigger-framework)
- [Formatting](#Formatting)
- [Clean Code Principles](#Clean-Code-Principles)
  - [Naming Conventions](#Naming-Conventions)
  - [Class & Method Conventions](#class--method-conventions)
- [Test Classes](#test-classes)
- [Error Handling](#Error-Handling)
- [ApexDocs](#ApexDocs)

## Introduction

The purpose of this style guide is to document the best practices and standards that we abide by as an organization when writing Apex code. The code that we write should not only be functional and performant, but it should also be maintainable. At the end of the day, we will be handing our completed solutions over to our clients and fellow developers to maintain. For this reason, we should strive to write clean code that is easily readable and extensible which is as important as making it functional.

This document will serve as the basis for code review comments and suggestions. The content included in this document is certainly not exhaustive of everything one needs to know to write proper Apex code. However, the information included should provide guidance for the most critical areas surrounding best practices, cleanliness, maintainability, and general standards that we follow.

Please note, you are strongly encouraged to share your thoughts on the content within this document. If you disagree with anything in this style guide or you think something should be added simply create a Feature branch, make the change, and submit it via a pull request for review. At the end of the day, we strive to write good code and learning from each other is paramount to us achieving this goal.

## Best Practices

The following best practices should be observed when writing Apex code. These key practices will help to ensure we are building efficient, scalable, and maintainable code on the platform.

#### No DML in For Loops

There are governor limits that enforce the maximum number of DML statements (insert, update, delete, undelete) used within an Apex transaction. When DML statements are placed inside of a `for` loop they are run once per iteration of the loop making it easy to hit the limit quickly.

Instead, DML operations should be moved out of `for` loops by gathering the records to modify inside of a collection and invoking the DML operation once on the collection of data.

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

#### No SOQL in For Loops

There are governor limits that enforce the maximum number of SOQL queries used within an Apex transaction. When SOQL queries are placed inside of a `for` loop they are run once per iteration of the loop making it easy to hit the limit quickly. Instead, SOQL queries should be moved out of `for` loops.

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

#### Use SOQL Queries Efficiently

A developer should always be as efficient as possible when writing queries against the database. The current synchronous SOQL limit is 100 queries issued within one Apex transaction. The 100 query limit may sound like a lot, but as the code base grows and additional solutions are added you can hit the limit easily if queries have not been handled efficiently from the beginning.

##### Combine SOQL Queries Where Possible

###### Incorrect Example

```apex
    /* Here two queries are used when
     * they could be combined into one.
     */
      List<Opportunity> oppsOver100K = [
        SELECT ID, Name, Amount
        FROM Opportunity
        WHERE Amount >= 100000.00
      ];
      List<Opportunity> oppsLessThan100K = [
        SELECT ID, Name, Amount
        FROM Opportunity
        WHERE Amount < 100000.00
      ];
```

###### Correct Example

```apex
      /* Here we are creating lists that we are populating
       * via one query inside a SOQL For Loop
       */
      List<Opportunity> oppsOver100K = new List<Opportunity>();
      List<Opportunity> oppsLessThan100K = new List<Opportunity>();

      for (Opportunity opp : [
        SELECT ID, Name, Amount
        FROM Opportunity
        WHERE Amount >= 100000.00
      ]) {
        if (opp.Amount >= 100000.00) {
          oppsOver100K.add(opp);
        } else {
          oppsLessThan100K.add(opp);
        }
      }
```

#### Use Relationships in Queries

Use relationships in queries to pull in all of the required data and records into one consolidated query. In the one query below we are pulling in Accounts with their closed opportunities including the owner of those opportunities.

```Apex
    List<Account> accountsWithClosedOpportunities = [
    SELECT
        id,
        name,
        (
        SELECT id, name, closedate, stagename, OwnerId, Owner.Name
        FROM Opportunities
        WHERE
            accountId IN :Trigger.newMap.keySet()
            AND (StageName = 'Closed - Lost'
            OR StageName = 'Closed - Won')
        )
    FROM Account
    WHERE Id IN :Trigger.newMap.keySet()
    ];
```

#### Be Mindful & Efficient with Limits

All Apex code is held to specified [governor limits](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm). When writing code the goal is not to simply stay within the limits but to write code efficiently so that it consumes the least number of limits possible. If you can refactor a section of code to consume fewer SOQL queries, DML statements, or callouts it is always better to do so even if it means the code is slightly longer or is a little less easy to read.

#### Know When To Use DML VS Database Methods

When running a DML operation on a list it is common that if one record fails you **do not** want all of the other records to also fail. For this reason, it is often preferred to use [Database methods](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_methods_system_database.htm#apex_System_Database_insert_2) instead of a standard DML operation. When changing data in the database always consider how potential errors should be handled.

- **Use DML When**: You would like an error that is returned during processing to return an exception as well as block all of the other records from being processed.
- **Use Database Methods When**: You would like to allow partial success on bulk operations with the ability to review any errors via the results array response.

```Apex

  /*
   * Two accounts, one of which is missing a required field
   */

  Account[] accounts = new List<Account>{
    new Account(Name = 'Test Account'),
    new Account()
  };
  Database.SaveResult[] srList = Database.insert(accounts, false);

  try {
    insert accounts;
  } catch (Exception e) {
    // Handle the error...
    ErrorHandler log = new ErrorHandler(
      'Account Insert Handler',
      'Error occurred when inserting Account List',
      e
    );
  }

  /*
   * Database.insert Example
   */
  Database.SaveResult[] srList = Database.insert(accounts, false);
  for (Database.SaveResult sr : srList) {
    if (!sr.isSuccess()) {
      for (Database.Error err : sr.getErrors()) {
        // Handle the error...
        ErrorHandler log = new ErrorHandler(
          'Account Insert Handler',
          'Error occurred when inserting Account List ' + err.getMessage()
        );
      }
    }
  }


```

#### Bulkify Your Code

Ensure your code properly handles batches of records. This is especially important with Triggers as it is common for a trigger to be invoked by a batch of records being processed through an import or API call into Salesforce. However when writing any Apex code make sure you understand how the code will be invoked; when code is invoked via a batch of records all of the records should be processed in bulk to ensure the solution is scalable and stays within limits. The sections on [SOQL ](#no-soql-in-for-loops) and [DML ](#no-dml-in-for-loops) statements in `For` loops share some bulk processing techniques.

#### Avoid Hardcoding IDs

Most IDs change between sandbox and production environments. For this reason, we should not place IDs directly in Apex code.

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

#### Trigger Best Practices

##### Only Have One Trigger Per Object

When multiple Triggers are created for the same object you can't control the order in which each Trigger runs. In addition, multiple Triggers are more difficult to debug and maintain. When creating a Trigger for a client always look and see if another trigger exists on the object first; if a Trigger already exists it is very likely their developers would prefer you add on to their existing trigger/handler.

##### Keep Logic Out of Triggers

Placing all of the logic of Triggers inside a Trigger Handler will keep the Trigger itself clean as well as allow one to run the logic of a trigger separately in another class if desired. Breaking out the logic into separate methods within the handler also makes it much easier to maintain and extend the Trigger logic.

##### Use the RTS Trigger Framework

If the client doesn't have an existing Trigger framework you should use the RTS Trigger Framework (after reviewing with the client). Our Trigger framework makes developing new Triggers and Trigger Handlers fast and easy while also including several best practices such as separating the trigger logic into handler classes, Apex bypass functionality via custom metadata records, and recursion prevention through the use of static variables.

## Formatting

We use the [Prettier Code Formatter](https://developer.salesforce.com/tools/vscode/en/user-guide/prettier/) for APEX which is available through this [plugin](https://github.com/dangmai/prettier-plugin-apex). Most developers are auto-formatting their code upon save through the use of a code formatting engine. If multiple developers are working on the same file it is very beneficial that they use the same code formatter. For this reason, the use of this plugin is encouraged but not required.

From a code review perspective, the following standards will be enforced.

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

**Local** variables should be declared close to the point in the code where they are first used. Do not declare all **local** variables at the top of their containing block.

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

The maximum line length should generally not exceed 120 characters. Once the limit has been reached the line should be wrapped onto another line. There aren't strict rules for how a line should be wrapped in all situations as there are many valid approaches.

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

## Clean Code Principles

Clean code is simple, direct, elegant, and efficient. Another developer should be able to easily read your code and maintain it as well as extend it with additional functionality.

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

### Class & Method Conventions

#### Classes & Methods Should Be Small

Classes and methods should both be small. **Ideally, each class should have one responsibility and every method within that class should do one thing and do it well.** It is better to have a few small organized classes rather than one large class.

#### Class Cohesion

Classes should have a few instance variables at the top of the class that all of the methods within the class leverage and manipulate. When all of a class's methods are working together on the same instance variables the class has good cohesion. When a class ends up with a lot of instance variables that only a subset of methods are using it is a clear sign that it is time to break the class up into multiple classes.

#### Method Arguments

Methods should have the least number of arguments possible. When methods start to accumulate more than three arguments it is a sign that those arguments should be promoted to instance class variables.

#### Methods Shouldn't Have Hidden Consequences

Again, each method should do the one thing it says it is going to do without any additional hidden functionality. For example, if a method's name is getTaxRate() you would expect the method to simply get the tax rate, not get the tax rate if available `else` calculate an estimated tax rate and populate it on a record.

#### Code Shouldn't Be Repeated

If the same logic or code is being copied and pasted in several places it should instead be converted into one or more methods. If the logic needs to be updated it is much simpler to update the one method instead of having to keep separate instances of the logic up to date.

## Test Classes

The foremost goal of a test class is to ensure the Apex code it covers is working as designed and expected.

##### 100% Code Coverage is the Goal, 90% is Acceptable

We should always aim for 100% code coverage. However, there are many situations where it isn't possible or practical to hit 100% code coverage, thus 90% is acceptable. There are very few scenarios where 90% coverage is not obtainable.

##### Test Exception Handling and Negative Test Cases

Test methods should be written to ensure the code handles unexpected results, invalid data, and exceptions well. Adding tests to trigger exceptions and negative test cases not only ensures the code is handling those scenarios appropriately but it also will greatly assist in achieving proper code coverage as we will be covering the catch blocks of our try/catches.

Sometimes it can be difficult to trigger exceptions in the code via a test class. The following pattern can be used to allow test methods to trigger exceptions when needed.

```Apex
public class WarehouseInventory {
  /*  Create a testVisible variable that we will use to
   *  conditionally trigger an exception
   */
  @testVisible
  private String testThrowException;

  public static void checkInventory(){
      try {

          // Method logic goes here...

          /* If the variable is set in the test class
           * we will throw the exception. We use a string
           * variable instead of a boolean so that we can
           * trigger different exceptions throughout the code
           * by passing in different strings.
           */
          if(testThrowException == 'Throw DML Exception'){
              throw new DmlException();
          }

      } catch(Exception e){
        new ErrorHandler('Warehouse Inventory Class','Error Checking Inventory',e);
        throw e;
      }
  }
}
```

```Apex
@isTest
public class WarehouseInventory_Test {

    @isTest
    static void testDMLException(){
        WarehouseInventory inventory = new WarehouseInventory();
        inventory.testThrowException = 'Throw DML Exception';
        try {
            inventory.checkInventory();
        } catch(Exception e){
            system.assert(
                e.getTypeName() == ‘System.DmlException’,
                ‘The error caught should be a DML exception’
            );
            /* Here we would also assert our error logger
             * created a record as expected.
             */
        }
    }

```

##### System Asserts

System asserts should be used to verify the code is working as expected. It is not enough to simply achieve proper code coverage, we must also confirm the code is working as designed.

```Apex
@isTest
public class AccountTrigger_Test {

    @isTest
    static void testAccountInsertion(){
        List<Account> accountsToInsert = new List<Account>();
        for(Integer i = 0; i<10; i++){
            Account newAccount = new Account();
            newAccount.name='test'+i;
            newAccount.Division__c = 'Commercial';
            accountsToInsert.add(newAccount);
        }
        insert accountsToInsert;
        //Query the records that were just created
        List<Account> createdAccounts = [SELECT Id, Division__c, Contract__c From Account];

        //Ensure 10 accounts were inserted
        System.assert(createdAccounts.size() == 10,'10 account records should be returned');

        /*
         * Loop through the accounts and verify a contract
         * was created and associated with each account.
         * In the Account trigger a default contract record
         * should be created for every commercial account.
         * Here we are verifying the code is working as designed.
         */
        for(Account acc : createdAccounts){
            System.assert(acc.Contract__c != null,'A contract should be populated for each commercial account');
        }
    }
}

```

##### Don't Use SeeAllData = True

You should create all of the data that your test class relies on directly within the test class. Each test class should work independently and be isolated from the rest of the data in the environment.

##### Test Setup Methods

Test setup methods can and should be used to create test data that all test methods within the test class can rely on. Creating all or most of the data for the test class within one setup method will help keep the test methods clean.

```Apex
@isTest
public class AccountTrigger_Test {
    @testSetup static void setup() {
        /*
         * Create common test data that all test methods
         * within this class can use.
         */
        List<Account> testAccounts = new List<Account>();
        for(Integer i=0;i<2;i++) {
            testAccounts.add(new Account(Name = 'TestAcct '+i));
        }
        insert testAccounts;
    }
    @isTest
    static void testAccountUpdate(){

        Account a = [SELECT Id FROM Account WHERE Name='TestAcct 0' LIMIT 1];
        a.Division__c = 'Commercial';
        update a;

        // Get the updated Account
        Account updatedAccount = [SELECT Id, Division__c, Contract__c From Account WHERE Id = :a.Id LIMIT 1];
        System.assert(updatedAccount.Contract__c != null,'A contract should be populated for each account');
    }
}

```

##### Test Data Factory

Another great way to generate test data is through the use of a test data factory. A test data factory is a separate class that is responsible for generating test data. All test classes written can then rely on the test data factory class to generate the test data needed. If validation rules or new required fields are added to an object a developer just needs to update the test data factory class instead of having to update the object creation in every test class.

We should always ask to see if the client already has a test data factory that they would like for us to use. If they don't have one we should consider creating one if we will need to create several test classes for them.

## Error Handling

#### Know When to Use a Try/Catch

Knowing when to use a `try/catch` is paramount to understanding proper error handling in Apex. Review the following sections and consider what happens when an exception is caught vs uncaught.

##### Unhandled Exception Steps

1. Code execution stops
2. All DML operations processed before the exception are rolled back thus not committed to the database.
3. Exceptions are logged in debug logs
4. Salesforce sends an unhandled exception email to the developer who last modified and any additional emails entered in the Apex Exception Email list.
5. An error message is returned to the end user

##### Handled Exceptions Steps with Try/Catch

1. The logic placed in the `catch` block runs
2. The logic placed in the `finally` block runs. Note, code in the finally block will always run regardless of an exception.
3. The code after the `try/catch/finally` will continue to run unless the `catch/finally` returns out.

##### When to use a try/catch

When an unhandled exception occurs several steps are processed by Salesforce. Try/catch blocks can be used to successfully recover from exceptions as well as process a custom set of error handling steps. However, it is important to consider what you are giving up when you choose to use a `try/catch`. In most cases when you are catching exceptions you still want to process most if not all of the steps that occur when an exception is unhandled albeit you may process them in a more refined way. **Whatever you do don't simply catch an exception and log it to the developer log**.

###### Incorrect Example

```apex
try {
  // logic here...
} catch (Exception e){
  System.Debug('An Error Occurred '+e.getMessage());
  // Do you need to rollback any DML operations?
  // Should you email/notify the development team or store the log in an error log object?
  // Should you return a message to the user?
}
// Do you still want this logic to run after the exception is caught?

```

#### Error Handler Framework

As with the Trigger framework if the client doesn't have an existing error logging/handler framework you should consider using the Rosetree Error Handler framework (after reviewing with the client). The Rosetree framework utilizes platform events to save error log records to the database even if the DML operations are rolled back.

##### Rosetree Framework Steps

- Parses several data points out of the exception including the Apex class and method name from the stack trace.
- Logs detailed error information to the developer logs
- Publishes a platform event to insert the error details into a Error Log object

#### Try/Catch Example

```apex
Savepoint sp = Database.setSavepoint();
try {
  // logic here...
} catch (Exception e){
  // Rollback to the savepoint
  Database.rollback(sp);
  /* Use the Error Handler Framework to log the error
   * and store it in an error log object
   */
  ErrorHandler log = new ErrorHandler('Title','Description',e);
  // Consider if you need to return a message to the user
}

```

## ApexDocs

[ApexDocs](https://github.com/SalesforceFoundation/ApexDoc) is a Java app that can be used to document Salesforce Apex Classes. You simply point the app at the location of the Apex classes/project folder and it will automatically generate static HTML pages that fully document the class including its properties and methods.

At Rosetree we use the ApexDocs standard to document the classes we write. Even if the client doesn't choose to generate the static HTML pages that the Java app provides the documentation within the class is helpful for future developers. **At a minimum ApexDoc should be written for every class**, but it would also be beneficial to add to most methods.

#### Class Documentation

| token          | description                                                                 |
| -------------- | --------------------------------------------------------------------------- |
| @author        | the author of the class                                                     |
| @date          | the date the class was first implemented                                    |
| @group         | a group to display this class under, in the menu hierarchy                  |
| @group-content | a relative path to a static html file that provides content about the group |
| @description   | one or more lines that provide an overview of the class                     |

###### Example

```Apex
    /**
    * @author Rosetree Solutions
    * @website https://rosetreesolutions.com/
    * @email info@RosetreeSolutions.com
    * @phone (214) 731 - 7314
    * @date the date the class was first implemented
    * @description one or more lines that provide an overview of the class
    */
```

#### Method Documentation

| token               | description                                                                    |
| ------------------- | ------------------------------------------------------------------------------ |
| @description        | one or more lines that provide an overview of the method                       |
| @param _param name_ | a description of what the parameter does                                       |
| @return             | a description of the return value from the method                              |
| @example            | Example code usage. This will be wrapped in <code> tags to preserve whitespace |

###### Example

```Apex
    /*
    * @description Returns field describe data
    * @param objectName the name of the object to look up
    * @param fieldName the name of the field to look up
    * @return the describe field result for the given field
    */
    public static Schema.DescribeFieldResult getFieldDescribe(String objectName, String fieldName) {
     // Logic goes here...
    }

```
