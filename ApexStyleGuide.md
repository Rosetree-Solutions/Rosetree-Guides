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

### Test Classes

The foremost goal of a test class is to ensure that Apex code is working as designed and expected.

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

###### Test Setup Methods

Test setup methods can and should be used to create test data that all test methods within the test class can rely in. Creating all or most of the data for the test class within one setup method will help keep the test methods clean.

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

###### Test Data Factory

Another great way to generate test data is through the use of a test data factory. A test data factory is a separate class that is responsible for generating test data. All test classes written can then rely on the test data factory class to generate the test data needed. If validation rules or new required fields are added to an object a developer just needs to update the test data factory class instead of having to update the object creation in every test class.

We should always ask to see if the client already has a test data factory that they would like for us to use. If they don't have one we should consider creating one if we will need to create several test classes for them.

### Error Handling

-- Example of what not to do i.e. catch an exception and system.debug it
-- Example of Try Catch
-- Example of Try Catch with Apex Logger
