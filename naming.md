[🏠](README.md) › Naming Conventions

- [Repositories](#repositories)
- [Issue Names](#issuenames)
- [Routes](#routes)
- [Database](#database)

# Repositories
The below table outlines the prefix used for naming repos

Type | Repo Prefix | Description
---- | ----------- | -----------
Landing Page | landing- | Public landing page for application
Web App | web- | Frontend client side web application
Mobile App | mobile- | Frontend mobile application
Edge Service | edge- | Public facing service
Service | service- | Backend Service that has API
Worker | worker- | Backend worker service with no API
Plugin | plugin- | Hapi JS plugin
Module | module- | Node JS module


# Issue Names
`Topic: <issue_name>`

Topics | Details
------ | -------
Endpoint: | Creating an endpoint
DB: | Change to database structure
Feature: | New feature
Fix: | 

# Routes
We are going with singular route/path names for APIs and UIs. e.g. `/datastore` & `/datastore/123`

# Database

## Tables
- Table names are singular. e.g. `todo`
- The only exception is `users` where **user** is a reserved word.
- Separate map table names with `_`

Type | Example | Desc
---- | ------ | ----
Standard | todo | Standard table
User | users | Only exception to singluar name
Map | todo_users | Separate

## Columns
- All column names are singular
- Columns with multiple words are NOT separated by `-` or `_`. e.g. `statustime` or `tablename`
- Relational IDs separate table name with `_`. e.g. **todo** table has `users_id`
