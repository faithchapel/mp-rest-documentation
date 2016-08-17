# Ministry Platform REST Documentation
A list of realizations/gotchas when developing with ministry platforms REST API

## Reading Records

### Get a list of tables
This will return all tables that the user has access to.
``` HTML
tables/
```

### Get all records in a table
This returns all records on the given table.
``` HTML
tables/contacts/
```

### Get a specific record
This returns a single record on the given table with the specified primary key value.
``` HTML
tables/contacts/1
```

### Get a filtered list
This returns a all records that match the filter requirements
``` HTML
tables/contacts?$Filter=Last_Name='Administrator'
```

### Get only specific fields
This returns the specified fields on the requested record(s)
``` HTML
tables/contacts/1?$Select=First_Name, Last_Name
```

### Select and join another table
Returns all contact fields and joins the Household table and get the linked Household_Name
``` HTML
tables/contacts/1?$Select=Contacts.*, Household_ID_Table.Household_Name
```

### Get a specified number of records
This returns the top 5 contacts
``` HTML
tables/contacts?$Top=5
```
This returns the top 5 contacts after skipping the first 5
``` HTML
tables/contacts?$Top=5&$Skip=5
```

### Get a distinct record set
Returns a distinct list of all cities
``` HTML
tables/addresses?$Select=City&$Distinct=true
```

### Order record set
Orders the response by the specified column
``` HTML
tables/addresses?$OrderBy=City
```

### Get records and group by column with aggregate function
Selects all donations, sums the Donation_Amount and groups by Donor_ID
``` HTML
tables/donations?$Select=Donor_ID,SUM(Donation_Amount) AS Amount&$GroupBy=Donor_ID
```

## Creating Records

## Updating Records

## Deleting Records


## Nesting on tables with columns that have required relationships
If you want to create a contact record and a participant record in one call you must make the participant record the base and then link the contact. This is because the participant record has a required column of Contact_ID but the contact record's Participant_Record column is not required

```javascript
[{
  Participant_Type_ID: 1,
  Contact_ID: {
    First_Name: "Steve",
    ...
  }
}]
```

## Nesting with updating records
When using nesting and updating records you are required to supply a primary key for each nested object
```javascript
[{
  Contact_ID: 1234,
  Household_ID: {
    Household_ID: 1234, // This value is required
    Household_Name: "Steve's Household"
  }
}]
```

## Tables not in ministry platform
If you have added a table in SQL but not created the page for it in ministry platform, that table is not accessible by the REST API.


## Page's Filter_Clause apply to the API 
The Filter_Clause column in the Pages table also applies to the REST API. 

For example, if there was a Filter_Clause on the Contacts page record
``` SQL
Contact_ID <> 1234
```
The following GET call would return nothing
``` javascript
[{
  Contact_ID: 1234
}]
```
```