# Ministry Platform REST Documentation
A list of realizations/gotchas when developing with ministry platforms REST API


## Nesting on tables with columns that have required relationships
If you want to create a contact record and a participant record in one call you must make the participant record the bass and then link the contact. This is because the participant record has a required column of Contact_ID but the contact record's Participant_Record column is not required

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
If you have added a table in SQL but not created the page for it in ministry platform, that table is not accesseble by the REST API.


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

## Joins
-- needed
