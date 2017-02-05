# Ministry Platform REST API Documentation
Basic documentation on how to work with the Ministry Platform REST API. This is a general list of things that we have found that are not well explained through current documentation.

This is intended as a supplement to the Swagger documentation (https://my.yourdomainname/ministryplatformapi/swagger). The Swagger documentation is a great resource that documents all of the REST API endpoints, and expected input and output. This document is not intended as a replacement for Swagger, but simply as an extension to fill in some gaps that are not included in that documentation.

Some endpoints documented in Swagger are duplicated here for clarity and quick reference. For full details on those endpoints refer to Swagger. All examples given use the Tables resource, however the concepts should apply to most other resources available through the REST API. 

## Table of Contents
- [Ministry Platform REST Documentation](#ministry-platform-rest-documentation)
  * [Reading Records](#reading-records)
    + [Get a list of tables](#get-a-list-of-tables)
    + [Get all records in a table](#get-all-records-in-a-table)
    + [Get a specific record](#get-a-specific-record)
    + [Get a filtered list](#get-a-filtered-list)
    + [Get only specific fields](#get-only-specific-fields)
    + [Select and join another table](#select-and-join-another-table)
    + [Get a specified number of records](#get-a-specified-number-of-records)
    + [Get a distinct record set](#get-a-distinct-record-set)
    + [Order record set](#order-record-set)
    + [Get records and group by column with aggregate function](#get-records-and-group-by-column-with-aggregate-function)
  * [Creating Records](#creating-records)
    + [Create a single record](#create-a-single-record)
    + [Create a series of records](#create-a-series-of-records)
    + [Create a record as a specific user](#create-a-record-as-a-specific-user)
    + [Create a nested set of records](#create-a-nested-set-of-records)
    + [Create nested dependent records](#create-nested-dependent-records)
  * [Updating Records](#updating-records)
    + [Update a single record](#update-a-single-record)
    + [Update a series of records](#update-a-series-of-records)
    + [Update a record as a specific user](#update-a-record-as-a-specific-user)
    + [Update a nested set of records](#update-a-nested-set-of-records)
  * [Deleting Records](#deleting-records)
    + [Delete a single record](#delete-a-single-record)
  * [Ministry Platform Configuration](#ministry-platform-configuration)
    + [OAUTH Client Credentials](#oauth-client-credentials)
    + [REST API User](#rest-api-user)
    + [Security Roles](#security-roles)
    + [Stored Procedure Access](#stored-procedure-access)
    + [Table Accessibility](#table-accessibility)
    + [MP Page Filter_Clause applies to the API](#mp-page-filter-clause-applies-to-the-api)


## Reading Records

### Get a list of tables
This will return all tables that the user has access to.
```
GET tables/
```

### Get all records in a table
This returns all records on the given table.
```
GET tables/contacts/
```

### Get a specific record
This returns a single record on the given table with the specified primary key value.
```
GET tables/contacts/1
```

### Get a filtered list
This returns all records that match the filter requirements
```
GET tables/contacts?$Filter=Last_Name='Administrator'
```
Boolean logic is accepted in the filter
```
GET tables/contacts?$Filter=(Last_Name='Allan' AND Last_Name='Tavie') OR Last_Name='Administrator'
```

### Get only specific fields
This returns the specified fields on the requested record(s)
```
GET tables/contacts/1?$Select=First_Name, Last_Name
```

### Select and join another table
Returns all contact fields and joins the Household table and get the linked Household_Name
```
GET tables/contacts/1?$Select=Contacts.*, Household_ID_Table.Household_Name
```

### Get a specified number of records
This returns the top 5 contacts
```
GET tables/contacts?$Top=5
```
This returns the top 5 contacts after skipping the first 5
```
GET tables/contacts?$Top=5&$Skip=5
```

### Get a distinct record set
Returns a distinct list of all cities
```
GET tables/addresses?$Select=City&$Distinct=true
```

### Order record set
Orders the response by the specified column
```
GET tables/addresses?$OrderBy=City
```

### Get records and group by column with aggregate function
Selects all donations, sums the Donation_Amount and groups by Donor_ID
```
GET tables/donations?$Select=Donor_ID,SUM(Donation_Amount) AS Amount&$GroupBy=Donor_ID
```

## Creating Records

### Create a single record
To create a single record, POST an array containing a single JSON object with all fields that should be included for the new record. All fields that do not allow nulls and do not specify a default value are considered required. 
```
POST tables/households
```

```javascript
[
  {
  	Household_Name: "Household, Test",
	Home_Phone: "123-456-7890"
  }
]
```

### Create a series of records
To create a multiple records, POST an array containing JSON objects for each record you would like to create.

```
POST tables/households
```

```javascript
[
  {
  	Household_Name: "Household, First",
	Home_Phone: "123-456-7890"
  },
  {
  	Household_Name: "Household, Second",
	Home_Phone: "098-765-4321"
  }
]
```

### Create a record as a specific user
To create a record as a specific user, specify the user id using the $User keyword in the query string. 
```
POST tables/households?$User=96
```

```javascript
[
  {
  	Household_Name: "Household, Test",
	Home_Phone: "123-456-7890"
  }
]
```

### Create a nested set of records
To create a nested set of records, assign a JSON object to a field with a foreign key relationship to another table.

```
POST tables/households
```

```javascript
[
  {
  	Household_Name: "Household, Test",
  	Address_ID: {
    	Address_Line_1: "123 Test St.",
        City: "TestCity",
        "State/Region": "TX",
        Postal_Code: 73301
    }
  }
]
```
This will create both a new household and an address linked to the household.

### Create nested dependent records
To create a set of nested records where one or more records is dependant on another record, the order of creation is important. Consider the following. If I would like to create a contact, and participant I may try the following.

```
POST tables/contacts
```

```javascript
//This code will not work!
[
  {
  First_Name: "Test",
  Last_Name: "Contact",
  Display_Name: "Contact, Test",
  Company: false,
  Participant_Record: {
  		Participant_Type_ID: 4,
        Participant_Start_Date: "2016-03-15 09:23:51.000"
    }
  }
]
```
This will not work because the Contact_ID field on the participant record is a required field. A contact needs to exist to link the participant to and the REST API starts at the deepest level of the nesting structure and works backwards. To make this work you need to invert the nesting.

```
POST tables/participants
```

```javascript
[
  {
    Participant_Type_ID: 4,
    Participant_Start_Date: "2016-03-15 09:23:51.000",
    Contact_ID: {
      First_Name: "Test",
      Last_Name: "Contact",
      Display_Name: "Contact, Test",
      Company: false
	}
  }
]
```
This will work because a contact record is not required to have a participant. The Contact gets created first, and it's ID gets passed into the participant record when it is created.

## Updating Records

### Update a single record
To update a single record, PUT an array containing a single JSON object with the primary key value of the record you would like to change, and any fields that should update. 
```
PUT tables/households
```

```javascript
[
  {
  	Household_ID: 1234,
	Home_Phone: "123-456-7890"
  }
]
```

### Update a series of records
To update multiple records, PUT an array containing JSON objects for each record you would like to update.
```
PUT tables/households
```

```javascript
[
  {
  	Household_ID: 1234,
	Home_Phone: "123-456-7890"
  },
  {
  	Household_ID: 1235,
	Home_Phone: "098-765-4321"
  }
]
```

### Update a record as a specific user
To update a record as a specific user, specify the user id using the $User keyword in the query string. 
```
PUT tables/households?$User=96
```

```javascript
[
  {
  	Household_ID: 1234,
	Home_Phone: "123-456-7890"
  }
]
```

### Update a nested set of records
When using nesting and updating records you are required to supply a primary key for each nested object
```javascript
[
	{
		Contact_ID: 1234,
		Household_ID: {
			Household_ID: 1234, // This value is required
			Household_Name: "Steve's Household"
		}
	}
]
```

## Deleting Records
Deleting records is requires caution. There is nothing to stop you from doing serious harm to your database, if you have permission to delete something, you can delete it. Be careful, write your deletes in a sandbox and test extensively. 

### Delete a single record
To delete a single record.
```
DELETE tables/households/116022
```

## Ministry Platform Configuration

### OAuth Client Credentials
To use the Client Credentials grant flow for OAuth, credentials need to be specified in the MinistryPlatform Web.config file. They are placed in Platform.Security.Clients in the following format.
```xml
<clients>
	<client clientId="YourApplicationName" clientSecret="YourClientSecret" callbackAddress="" tokenLifetime="00:30:00"></client>
</clients>
```

### REST API User
While using the Client Credentials grant flow, unless a user is explicitly specified using the $User keyword in each POST, PUT, or DELETE call, records created through the REST API are created using the AnonymousUserId specified in the MinistryPlatformAPI Web.config file. Security roles applied to that user are used in authorizing requests regardless of what user is specified with the $User keyword.

### Security Roles
Access to tables is restricted through security roles applied to the user making the call. Giving users access to pages gives them access to the underlying tables.

### Stored Procedure Access
To use a stored procedure two things need to happen. First, there needs to be a record referencing the stored procedure in the System Setup/API Procedures page. Secondly, the user needs a security role applied to it with the API Procedure linked to the security role. This is done through the API Procedures sub page on the Security Roles page.

### Table Accessibility
To be accessible through the REST API tables must be associated with pages.
If you have added a table in SQL the table will not be accessible by the REST API until a page has been created for it in MP.

### MP Page Filter_Clause applies to the API 
The Filter_Clause column in the Pages table also applies to the REST API. Pages that filter out parts of the data set in the page will also filter out the same data from the API. If there is a filtered page, and an unfiltered page, and the user's security role allows access to both pages, the unfiltered page will be used. 

For example, if there was a Filter_Clause on the Contacts page record
``` SQL
Contact_ID <> 1234
```
The following GET call would return nothing
```
GET tables/contacts/1234 
```