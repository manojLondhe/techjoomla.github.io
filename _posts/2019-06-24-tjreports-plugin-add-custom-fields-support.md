---
date: 2019-06-24
title: Custom fields (com_fields) support in TJReports plugins
categories:
  - TJ Reports
tags:
  - Joomla
  - tjreports
  - com_fields
type: Document
showSidebar: true
published: true
nav_ordering: 5
pageTitle: "Custom fields (com_fields) support in TJReports plugins"
permalink: tj-reports/tjreports-plugin-add-custom-fields-support.html
---

**Content**

- [A. Background](#a-background)
  - [Core TJReports plugins to index data created using com_fields](#core-tjreports-plugins-to-index-data-created-using-com_fields)
- [B. Which all com_fields's field types are support in TJReports?](#b-which-all-com_fieldss-field-types-are-support-in-tjreports)
  - [Database table used for this (indexed data)](#database-table-used-for-this-indexed-data)
- [C. How to add user fields to any report?](#c-how-to-add-user-fields-to-any-report)
  - [Steps](#steps)
    - [Step 1: Setup custom fields columns](#step-1-setup-custom-fields-columns)
    - [Step 2: Setup custom fields filters](#step-2-setup-custom-fields-filters)


### A. Background

TJReport has plugins that let you index the custom fields data created using Joomla's `com_fields` component.

#### Core TJReports plugins to index data created using com_fields

| Plugin group  | Plugin name | What does this plugin do |
| ------------- | ------------- | ------------- |
| content | tjreportsfields | This plugin adds / updates / deletse columns for *`#__tjreports_context` tables |
| user | tjreportsindexer | This plugin adds / updates / deletes user-data for `#__tjreports_com_users_user` table|

*In case of if you are using com_fields for com_users context table name will be as
`#__tjreports_context` = `#__tjreports_com_users_user`

### B. Which all com_fields's field types are support in TJReports?

| com_field field type  | tjreport indexer table column type | how / what indexer stores values for this | what type of filter in tjreports used for this |
| ------------- | ------------- | ------------- | ------------- |
| calendar | datetime  | save user entered  | daterange filter  |
| checkboxes | varchar (255)  | save TEXT and not VALUE  | select / equal match  |
| color | varchar (7)  | save user entered  | -  |
| editor | text  | save user entered  | text / equal match  |
| integer | int (11)  | save user entered  | select / equal match  |
| list | varchar (255)  | save TEXT and not VALUE  | select / equal match  |
| imagelist | _not supported_  | NA  | NA  |
| media | _not supported_   | NA  | NA  |
| radio | varchar (255)  | save TEXT and not VALUE  | select / equal match  |
| repeatable | _not supported_   | NA  | NA  |
| sql | text | save TEXT and not VALUE  | text / equal match  |
| text | text  | save user entered  | text / equal match  |
| textarea | text  | save user entered  | text / equal match  |
| url | varchar (250) [based on joomla weblinks table]  | save user entered  | text / equal match  |
| user | varchar (400) [based on joomla user table] | store user's name  | select / equal match  |
| usergrouplist | varchar (100) [based on joomla usergroups table]  | store usergorup title  | select / equal match  |

#### Database table used for this (indexed data)

* Every `tjreport` plugin will rely on a certain table where custom fields data is stored in rows (1 record per row)
  * eg. If `context` is `com_users.user` -> table name should be `#__tjreports_com_users_user`
* Column which is needed here in this example is
  * `record_id` *int (11)*

### C. How to add user fields to any report?

Below are steps for adding those user fields data columns into an existing report.
Eg: For a plugin named `mytjreportplugin`, open plugin entry file `mytjreportplugin.php`

#### Steps
1. [Setup custom fields columns](#step-1-setup-custom-fields-columns)
1. [Setup custom fields filters](#step-2-setup-custom-fields-filters)

#### Step 1: Setup custom fields columns

Change constructor from

```php
public function __construct($config = array())
{
	$this->columns = array (
		// Columns setup here
	);

	parent::__construct($config);
}
```

to below

```php
public function __construct($config = array())
{
	// Joomla fields integration
	// Define custom fields table, alias, and table.column to join on
	$this->customFieldsTable       = '#__tjreports_user_fields';
	$this->customFieldsTableAlias  = 'tuf';
	$this->customFieldsQueryJoinOn = 'lt.user_id';

	$this->columns = array (
		// Columns setup here
	);

	parent::__construct($config);
}
```

In step 1, we define
- `customFieldsTable`       - Custom field's indexed table in which we store duplicated data
- `customFieldsTableAlias`  - Alias for above table
- `customFieldsQueryJoinOn` - Table name and column name on which TJReport will do database query join on (TJReports' model will do query join on the custom fields table's **`record_id`** column and with the other DB table which plugin mainly runs the report on.)


#### Step 2: Setup custom fields filters

Change code from

```php
public function displayFilters()
{
	// Set filters code here
	$dispFilters = array(
		array(
			// Set filters code here
		),
		array(
			// Set filters code here
		)
	);

	return $dispFilters;
}
```

to

```php
public function displayFilters()
{
	// Set filters code here
	$dispFilters = array(
		array(
			// Set filters code here
		),
		array(
			// Set filters code here
		)
	);

	// Joomla fields integration
	// Call parent function to set filters for custom fields
	$this->setCustomFieldsDisplayFilters($dispFilters);

	return $dispFilters;
}
```

In step 2, we call the parent method `setCustomFieldsDisplayFilters()`, which sets up filters for columns from custom fields to be displayed on the report.

**Conclusion**

That's it, with these 2 simple steps, you will be able to easily add columns, filters, sorting on the columns form the Joomla user's custom fields.
