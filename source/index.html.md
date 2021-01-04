---
title: GoBoost API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='http://support.goboost.io' target="_blank">Contact us for Oauth credentials</a>

includes:
  - errors

search: false

code_clipboard: true
---

# Introduction

Welcome to the GoBoost API! You can use this API to get access to reporting data about your GoBoost account.

# Terminology
**Hierarchy -** The GoBoost app is set up with a family type hierarchy. There are Organizations, Companies, Users, and User Roles.

**Organization -** An Organization represents a partner business that refers small businesses to use GoBoost. Organizations can be setup in a hierarchy of their own, meaning an Organization can be a child of another organization. Examples of popular hierarchies: Manufacturer -> Distributor -> Small Business or Manufacturer -> Small Business. Users who are tied to an Organization are able to see all "descendents" of that Organization. If you have a role at an Organization that has Organizations below it and Companies below that, you'd be able to see everything about all Companies tied to all Organizations below you in the hierarchy.

**Company -** A Company represents a business that is using GoBoost products. Companies must belong to at least one Organization above it in the hierarchy.

**Users -** Users represent the actual people that are using the GoBoost system. User's are tied to Organizations and/or Companies and the role is what controls their access level.

**User Roles -** User Roles are used to tie a user to either an Organization or a Company or both. The role that's chosen when you authenticate into the API controls the level at which you'll be seeing data.

# Authentication

## Flow
GoBoost uses the Oauth 2.0 protocol for Authentication.<br /><br />
Oauth Flow:<br />
1. POST to get a grant code and the list of User Roles you have access to using your Base64 encoded login credentials and the Oauth app Client ID.<br />
2. POST the grant code you received from the previous step to the token route and you'll get a access token, refresh token, expires in seconds, and a status back.<br />
3. Use the access token as a bearer token in the Authorization header of each request you make to the GoBoost API.


## Grant Code

The first step in the authentication process is to get a Grant Code. To do this you will pass your Base64 encoded email and password login credentials and the auth route.

This will return the Grant Code, along with a list of the User Roles that this user has access to.

You'll need to select the role you want to authenticate and use the Grant Code you received to get an access token.

```shell
# With shell, you can just pass the correct header with each request
curl -X POST "https://lets.goboost.io/api/core/oauth/auth" \
-H "Content-Type: application/json" \
-d '{ "credentials": "email:password", "client_id": "xxx" }'
```

> Make sure to replace `email:password` with your Base64 Encoded email:password and replace `client_id` with your Oauth Application's Client ID.

> Successful Response:

```json
  {
    "code": "xxx",
    "status": 200,
    "user_roles": [
      {
        "id": 1,
        "role_description": "Has all abilities to adequately manage GoBoost Application",
        "role_id": 1,
        "role_name": "Application Editor",
        "role_type": "Organization",
        "role_type_id": 1,
        "role_type_name": "Demo",
        "user_email": "demo@goboost.io",
        "user_first_name": "John",
        "user_id": 1,
        "user_last_name": "Doe"
      }
    ]
  }
```


<aside class="notice">
  You must replace <code>email:password</code> with your Base64 encoded login credentials and <code>ClientID</code> from the Oauth Application.
</aside>

## Access Token

Once you have selected your role and received the grant code, you'll need to use that grant code to trade in for an access token.

In order to do that, you'll send a POST request to the token route along with your Oauth Application's Client Secret.

This will return an access token, refresh token, and the expires at date/time

```shell
curl -X POST "https://lets.goboost.io/api/core/oauth/token" \
-H "Content-Type: application/json" \
-d '{ "code": "xxx", "client_secret": "xxx", "grant_type": "authorization_code" }'
```

> You must replace `code` with the grant code you received from the previous step and `client_secret` with your Oauth Application's Client Secret.

> Successful Response:

```json
  {
    "access_token": "xxx",
    "expires_in_seconds": 3600,
    "refresh_token": "xxx",
    "status": 200
  }
```

<aside class="notice">
  You must replace <code>code</code> with the grant code you received from the previous step and replace <code>client_secret</code> from the Oauth Application.
</aside>

## Refresh Token
The access token you receive will expire. When the token expires, you'll need to use the refresh token you received in the access token step to get a new access token

```shell
# With shell, you can just pass the correct header with each request
curl -X POST "https://lets.goboost.io/api/core/oauth/token" \
-H "Content-Type: application/json" \
-d '{ "refresh_token": "xxx", "client_secret": "xxx", "grant_type": "refresh_token" }'
```

> You must replace `refresh_token` with the refresh you received from the access token step and `client_secret` with your Oauth Application's Client Secret.

> Successful Response:

```json
  {
    "access_token": "xxx",
    "expires_in_seconds": 3600,
    "status": 200
  }
```

<aside class="notice">
  You must replace <code>refresh_token</code> with the refresh you received from the access token step and <code>client_secret</code> with your Oauth Application's Client Secret.
</aside>

## Making Requests
Now that you have an access token, in order to make requests to GoBoost routes you'll need to add that token as an authorization header to each request. You'll also need to add GoBoost specific headers for the User Role that you have selected. You can choose the User Role from the list of User Roles provided in the grant code request in the first step of the Oauth process.

```shell
  curl "https://lets.goboost.io/api/(route)" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1",
```

> Replace `xxx` with the access token

<aside class="notice">
  Replace <code>xxx</code> with the access token.
</aside>

# Roles

## Roles
```shell
  curl "https://lets.goboost.io/api/core/roles"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "roles": [
    {
      "abilities": "ABILITY,ABILITY,ABILITY"
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "created_by_user_id": 1,
      "description": "Role Description",
      "id": 1,
      "name": "Role Name",
      "role_type": "Company or Organization",
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    },
    {
      "abilities": "ABILITY,ABILITY,ABILITY"
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "created_by_user_id": 1,
      "description": "Role Description",
      "id": 1,
      "name": "Role Name",
      "role_type": "Company or Organization",
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    }
  ],
  "status": 200
}
```

This endpoint retrieves data about the roles that are available.

### HTTP Request

`GET https://lets.goboost.io/api/core/roles`

# Users

## User Search
```shell
  curl "https://lets.goboost.io/api/core/users/search"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

Parameter | Description
--------- | -----------
email | Search for the user in GoBoost by email.
first_name | Search for the user in GoBoost by first name.
last_name | Search for the user in GoBoost by last name.

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "user": {
    "created_at": "Thu, 21 May 2020 16:04:31 GMT",
    "email": "john@doe.com",
    "first_name": "John",
    "last_name": "Doe",
    "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
  }
}
```

This endpoint retrieves data about the roles that are available.

### HTTP Request

`GET https://lets.goboost.io/api/core/users/search`


## Create User

```shell
  curl -X POST "https://lets.goboost.io/api/core/users"
    -d '{ "email": "john@doe.com", "first_name": "John", "last_name": "Doe" }' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`POST https://lets.goboost.io/api/core/users`

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "user": {
    "created_at": "Thu, 21 May 2020 16:04:31 GMT",
    "email": "john@doe.com",
    "first_name": "John",
    "last_name": "Doe",
    "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
  }
}
```

## User's User Roles
```shell
  curl "https://lets.goboost.io/api/core/users/:id/user_roles"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "user_roles": [
    {
      "id": 1,
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "user_id": 1,
      "role_id": 1,
      "role_type": "Company",
      "role_type_id": 1,
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    },
    {
      "id": 2,
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "user_id": 1,
      "role_id": 1,
      "role_type": "Company",
      "role_type_id": 2,
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    }
  ]
}
```

This endpoint retrieves data about the roles that are available to the specified user.

### HTTP Request

`GET https://lets.goboost.io/api/core/users/:id/user_roles`


## Current User's User Roles
```shell
  curl "https://lets.goboost.io/api/core/user_roles"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "user_roles": [
    {
      "id": 1,
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "user_id": 1,
      "role_id": 1,
      "role_type": "Company",
      "role_type_id": 1,
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    },
    {
      "id": 2,
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "user_id": 1,
      "role_id": 1,
      "role_type": "Company",
      "role_type_id": 2,
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    }
  ]
}
```

This endpoint retrieves data about the roles that are available to the current user.

### HTTP Request

`GET https://lets.goboost.io/api/core/users/:id/user_roles`

## Create User Role

```shell
  curl -X POST "https://lets.goboost.io/api/core/user_roles"
    -d '{ "role_id": 1, "role_type": "Company", user_id: 1, role_type_id: 1 }' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`POST https://lets.goboost.io/api/core/user_roles`

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "user_role": {
    "id": 1,
    "created_at": "Thu, 21 May 2020 16:04:31 GMT",
    "user_id": 1,
    "role_id": 1,
    "role_type": "Company",
    "role_type_id": 1,
    "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
  }
}
```

# Organizations

## Specific Organization

```shell
  curl "https://lets.goboost.io/api/core/organizations/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "default_sign_up_code": {
    "code": "xxx",
    "created_at": "Jan 1, 1970",
    "id": 1,
    "owner_organization_id": 1,
    "updated_at": "Jan 1, 1970"
  },
  "organization": {
   "created_at": "Jan 1, 1970",
   "default_company_sign_up_code_id": 1,
   "host_theme_id": 1,
   "id": 1,
   "name": "Demo Organization",
   "updated_at": "Jan 1, 1970"
  },
  "status": 200
}
```

This endpoint retrieves a specific organization

### HTTP Request

`GET https://lets.goboost.io/api/core/organizations/:id`

## Organization's Companies

```shell
  curl "https://lets.goboost.io/api/core/organizations/:id/companies"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "companies": [
    {
      "address_line_one": "123 st.",
      "address_line_two": "Suite 1",
      "city": "City",
      "country": "US",
      "coupon_code": "xxx",
      "created_at": "Jan 1, 1970",
      "google_place_id": "xxx",
      "id": 1,
      "latitude": "0.000",
      "longitude": "0.000",
      "name": "Demo Company",
      "phone_number": "(555) 555-5555",
      "postal_code": "12345",
      "primary_website": "goboost.io",
      "secondary_website": "servicecompany.com",
      "service_area": {
        "included_geographies": [
          "xxx",
          "xxx"
        ]
      },
      "sign_up_code_id": 1,
      "sign_up_organization_id": 1,
      "state": "AB",
      "time_zone_id": "America/New_York",
      "updated_at": "Jan 1, 1970"
    }
  ],
  "company_organizations": [
    {
      "company_id": 1,
      "name": "Demo Organization",
      "organization_id": 1
    }
  ],
  "count": 1,
  "status": 200
}
```

This endpoint retrieves all companies that are below an Organization in the hierarchy.

### HTTP Request

`GET https://lets.goboost.io/api/core/organizations/:id/companies`

## Organization's Users & User Roles

```shell
  curl "https://lets.goboost.io/api/core/organizations/:id/user_roles"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "count": 1,
  "status": 200,
  "user_roles": [
    {
      "id": 1,
      "role_abilities": "ABILITY,ABILITY",
      "role_description": "role description",
      "role_id": 1,
      "role_name": "Application Editor",
      "role_type": "Organization",
      "role_type_id": 1,
      "role_type_name": "Demo Organization",
      "user_email": "john@doe.com",
      "user_first_name": "John",
      "user_id": 1,
      "user_last_name": "Doe"
    },
    {
      "id": 2,
      "role_abilities": "ABILITY,ABILITY",
      "role_description": "role description",
      "role_id": 1,
      "role_name": "Application Editor",
      "role_type": "Organization",
      "role_type_id": 1,
      "role_type_name": "Demo Organization",
      "user_email": "john@doe.com",
      "user_first_name": "John",
      "user_id": 1,
      "user_last_name": "Doe"
    }
  ],
  "users": [
    {
      "email": "john@doe.com",
      "first_name": "John",
      "last_name": "Doe",
      "id": 1
    },
    {
      "email": "jane@doe.com",
      "first_name": "Jane",
      "last_name": "Doe",
      "id": 1
    }
  ]
}
```

This endpoint retrieves all users that are below an Organization in the hierarchy.

### HTTP Request

`GET https://lets.goboost.io/api/core/organizations/:id/user_roles`


## External Data

```shell
  curl "https://lets.goboost.io/api/core/organization_external_data"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```
### Query Parameters

Parameter | Description
--------- | -----------
owner_organization_id | The GoBoost ID for the organization that owns the data. Typically this is tied to the "brand" level organization. This is required.
external_id | The ID you use to identify the organization in your database. This allows you to query this route using your ID in order to get the GoBoost ID.
role_type | This is the type of entity you are looking for. Either a Company or Organization.
role_type_id | This is the GoBoost ID of the Company or Organization

### HTTP Request

`GET https://lets.goboost.io/api/core/organization_external_data`

> The above command returns JSON structured like this:

```json
{
  "organization_external_data": [
    {
      "created_at": "05 Oct 2020",
      "external_data": {},
      "external_id": "xxx-xx1",
      "id": 3,
      "owner_organization_id": 2,
      "role_type": "Organization",
      "role_type_id": 1,
      "updated_at": "05 Oct 2020",
    },
    {
      "created_at": "05 Oct 2020",
      "external_data": {},
      "external_id": "xxx-xx2",
      "id": 4,
      "owner_organization_id": 2,
      "role_type": "Organization",
      "role_type_id": 1,
      "updated_at": "05 Oct 2020",
    }
  ],
  "status": 200
}
```

## Organization External IDs

```shell
  curl "https://lets.goboost.io/api/core/organization_external_data/organizations/:organization_id/external_ids?external_ids[]=1,2,3"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### Query Parameters

Parameter | Description
--------- | -----------
external_ids | These are the ID's from your database that we store in our system so you can easily match up your data to ours. You can request up to 1,000 external ID's at a time. This is required.

### HTTP Request

`GET https://lets.goboost.io/api/core/organization_external_data/organizations/:organization_id/external_ids`

> The above command returns JSON structured like this:

```json
{
  "organization_external_data": [
    {
      "owner_organization_id": 1,
      "role_type": "Organization or Company",
      "role_type_id": 1,
      "external_id": "xxx",
      "external_data": {
        "key": "value"
      },
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    },
    {
      "owner_organization_id": 1,
      "role_type": "Organization or Company",
      "role_type_id": 1,
      "external_id": "xxx",
      "external_data": {
        "key": "value"
      },
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    }
  ],
  "status": 200
}
```

## Add External Data

```shell
  curl -X POST "https://lets.goboost.io/api/core/organization_external_data"
    -d '{ "owner_organization_id": 1, "role_type": "Company", "role_type_id": 1, "external_id": "xxx", external_data: { "key": "value" } }' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`POST https://lets.goboost.io/api/core/organization_external_data`

> The above command returns JSON structured like this:

```json
{
  "organization_external_data": {
    "owner_organization_id": 1,
    "role_type": "Organization or Company",
    "role_type_id": 1,
    "external_id": "xxx",
    "external_data": {
      "key": "value"
    },
    "created_at": "Thu, 21 May 2020 16:04:31 GMT",
    "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
  },
  "status": 200
}
```

## Organization Child Types

```shell
  curl "https://lets.goboost.io/api/core/organization_child_types"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```
### HTTP Request

`GET https://lets.goboost.io/api/core/organization_child_types`

> The above command returns JSON structured like this:

```json
  "organization_child_types": [
    {
      "id": 1,
      "name": "Type",
      "owner_organization_id": 1
    },
    {
      "id": 2,
      "name": "Type",
      "owner_organization_id": 1
    }
  ]
```
## Associate Company to Organization

```shell
  curl -X PUT "https://lets.goboost.io/api/core/organization_child/organizations/:id"
    -d '{ "organization_id": 1, "organization_child_type_id": 1 }' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`PUT https://lets.goboost.io/api/core/organization_child/organizations/:id`

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "organization_child": {
    "id": 1,
    "parent_organization_child_id": 1,
    "child_organization": 1,
    "child_organization_type_id: 1
  }
}
```

## Sign Up Codes
```shell
  curl "https://lets.goboost.io/api/core/sign_up_codes/organizations/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "sign_up_codes": [
    {
      "code": "SIGNUPCODE",
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "id": 1,
      "owner_organization_id": 1,
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    },
    {
      "code": "SIGNUPCODE2",
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "id": 2,
      "owner_organization_id": 1,
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    },
  ]
}
```

This endpoint retrieves gives the sign up codes that are available to the specified organization.

### HTTP Request

`GET https://lets.goboost.io/api/core/sign_up_codes/organizations/:id`


## Company Organizations

```shell
  curl "https://lets.goboost.io/api/core/company_organizations/:company_id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`GET https://lets.goboost.io/api/core/company_organizations/:company_id`

> The above command returns JSON structured like this:

```json
{
  "organizations": [
    {
      "company_id": 1,
      "name": "Organization Name",
      "organization_id": 4
    },
    {
      "company_id": 1,
      "name": "Organization Name",
      "organization_id": 4
    }
  ],
  "status": 200
}
```

## Create Company Organization Association

```shell
  curl -X POST "https://lets.goboost.io/api/core/company_organizations/:company_id"
    -d '{ "organization_id": 1 }' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`POST https://lets.goboost.io/api/core/company_organizations/:company_id`

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "company_organization": {
    "company_id": 1,
    "organization_id": 1,
    "created_at": "Thu, 21 May 2020 16:04:31 GMT",
    "updated_at": "Thu, 21 May 2020 16:04:31 GMT",
  }
}
```
## Delete Company Organization Association

```shell
  curl -X DELETE "https://lets.goboost.io/api/core/company_organizations/:company_id"
    -d '{ "organization_id": 1 }' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`DELETE https://lets.goboost.io/api/core/company_organizations/:company_id`

> The above command returns JSON structured like this:

```json
{
  "status": 200
}
```

## Organization Host Theme

```shell
  curl "https://lets.goboost.io/api/core/host_themes/organizations/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`GET https://lets.goboost.io/api/core/host_themes/organizations/:id`

> The above command returns JSON structured like this:

```json
{
  "host_theme": {
    "created_at": "Thu, 21 May 2020 16:04:31 GMT",
    "hostname": "example.goboost.io",
    "id": 1,
    "owner_organization_id": 1,
    "theme_id": 1,
    "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
  },
  "status": 200
}
```

# Companies
## Company
```shell
  curl "https://lets.goboost.io/api/core/companies/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "company": {
    "address_line_one": "123 st.",
    "address_line_two": "",
    "business_hours": "",
    "city": "City",
    "country": "United States",
    "coupon_code": "",
    "created_at": "27 Jul 2020",
    "google_place_id": "xxx",
    "id": 1,
    "latitude": 0.000,
    "longitude": -0.000,
    "name": "Demo Company",
    "phone_number": "5555555555",
    "postal_code": "12345",
    "primary_website": "goboost.io",
    "secondary_website": "",
    "service_area": {
      "excluded_geographies": ["xxx", "xxx"],
      "included_geographies": ["xxx", "xxx"]
    },
    "sign_up_code_id": 1,
    "sign_up_organization_id": 1,
    "state": "State",
    "stripe_error": "",
    "time_zone_id": "America/Boise",
    "updated_at": "05 Oct 2020"
  },
  "status": 200
}
```

This endpoint retrieves a specific company.

### HTTP Request

`GET https://lets.goboost.io/api/core/companies/:id`

## Company Subscriptions
```shell
  curl "https://lets.goboost.io/api/core/companies/:id/subscription"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "subscription": {
    "canceled_at": null,
    "company_id": 1,
    "created_at": "05 Oct 2020",
    "id": 1,
    "is_canceled": null,
    "updated_at": "05 Oct 2020"
  },
  "subscription_items": [
    {
      "created_at": "05 Oct 2020",
      "id": 1,
      "plan_id": 1,
      "subscription_id": 1,
      "updated_at": "05 Oct 2020"
    }
  ]
}
```

This endpoint retrieves all active subscriptions for a Company.

### HTTP Request

`GET https://lets.goboost.io/api/core/companies/:id/subscription`

## Company Plans
```shell
  curl "https://lets.goboost.io/api/core/companies/:id/plans"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "plans": [
    {
      "ads_management_fee_expires_after_days": null,
      "ads_management_fee_percent": 0,
      "base_site_domain": null,
      "created_at": "18 Mar 2019",
      "description_html": "",
      "id": 1,
      "is_active": true,
      "is_ads_included": true,
      "is_base_site": null,
      "is_review_trackers_included": false,
      "is_reviews_included": false,
      "is_sites_included": false,
      "is_social_included": null,
      "name": "Demo Plan",
      "owner_organization_id": 1,
      "plan_amount_cents": 0,
      "setup_fee_cents": 0,
      "stripe_plan_id": "plan_xxx",
      "trial_period_days": null,
      "updated_at": "18 Mar 2019"
    }

  ],
  "status": 200
}
```

This endpoint retrieves all available plans for a Company.

### HTTP Request

`GET https://lets.goboost.io/api/core/companies/:id/plans`

## Company Sites Profile
```shell
  curl "https://lets.goboost.io/api/sites/company_sites_profiles/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "company_sites_profile": {
    "business_hours": {},
    "byo_domain": "",
    "call_tracking_disabled": false,
    "company_id": 1,
    "company_site_design_id": null,
    "created_at": "27 Jul 2020",
    "id": 1,
    "is_setup_complete": false,
    "is_ssl_setup": false,
    "logo_url": null,
    "matomo_reports_last_enqueued_at": null,
    "matomo_reports_last_updated_at": null,
    "matomo_site_id": null,
    "new_domain": null,
    "updated_at": "27 Jul 2020"
  },
  "status": 200
}
```

This endpoint retrieves sites specific data for a specific company

### HTTP Request

`GET https://lets.goboost.io/api/sites/company_sites_profiles/:id`

## Company Reviews Profile
```shell
  curl "https://lets.goboost.io/api/reviews/company_reviews_profiles/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
  {
    "profile": {
      "company_id": 1,
      "created_at": "05 Oct 2020",
      "facebook_importer_last_completed_at": "05 Oct 2020",
      "google_importer_last_completed_at": "05 Oct 2020",
      "id": 1,
      "is_first_import_notification_sent": 1,
      "is_setup_complete": 1,
      "review_request_default_message": "",
      "review_trackers_importer_last_completed_at": "05 Oct 2020",
      "updated_at": "05 Oct 2020"
    },
    "status": 200
  }
```

This endpoint retrieves reviews specific data for a specific company

### HTTP Request

`GET https://lets.goboost.io/api/reviews/company_reviews_profile/:id`

<aside class="notice">
  The User Role you are using for this request MUST be a User Role for the Company you are looking for
</aside>

## Company Ads Profile
```shell
  curl "https://lets.goboost.io/api/ads/company_ads_profile/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "profile": {
    "active_boosted_services_count": 1,
    "adwords_account_id": "xxx",
    "adwords_account_negatives_shared_set_id": "xxx",
    "company_id": 1,
    "company_short_name": "Demo Company",
    "company_url_slug": "/slug",
    "created_at": "05 Oct 20",
    "facebook_page_url": "facebook.com",
    "id": 1,
    "is_setup_complete": 1,
    "opportunity_count": 1,
    "updated_at": "05 Oct 20"
  },
  "status": 200
}
```

This endpoint retrieves ads specific data for a specific Company.

### HTTP Request

`GET https://lets.goboost.io/api/ads/company_ads_profile/:id`

<aside class="notice">
  The User Role you are using for this request MUST be a User Role for the Company you are looking for
</aside>


## External Data

```shell
  curl "https://lets.goboost.io/api/core/organization_external_data"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```
### Query Parameters

Parameter | Description
--------- | -----------
owner_organization_id | The GoBoost ID for the organization that owns the data. Typically this is tied to the "brand" level organization. This is required.
external_id | The ID you use to identify the company in your database. This allows you to query this route using your ID in order to get the GoBoost ID.
role_type | This is the type of entity you are looking for. Either a Company or Organization.
role_type_id | This is the GoBoost ID of the Company or Organization

### HTTP Request

`GET https://lets.goboost.io/api/core/organization_external_data`

> The above command returns JSON structured like this:

```json
{
  "organization_external_data": [
    {
      "created_at": "05 Oct 2020",
      "external_data": {},
      "external_id": "xxx-xx1",
      "id": 3,
      "owner_organization_id": 2,
      "role_type": "Organization",
      "role_type_id": 1,
      "updated_at": "05 Oct 2020",
    },
    {
      "created_at": "05 Oct 2020",
      "external_data": {},
      "external_id": "xxx-xx2",
      "id": 4,
      "owner_organization_id": 2,
      "role_type": "Organization",
      "role_type_id": 1,
      "updated_at": "05 Oct 2020",
    }
  ],
  "count": 2,
  "status": 200
}
```

## Company Roles

```shell
  curl "https://lets.goboost.io/api/core/companies/:id/roles"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`GET https://lets.goboost.io/api/core/companies/:id/roles`

> The above command returns JSON structured like this:

```json
{
  "roles": [
    {
      "abilities": "ABILITY,ABILITY",
      "created_at": "05 Oct 2020",
      "created_by_user_id": 1,
      "description": "Short description of the role",
      "id": 1,
      "name": "Role Name",
      "role_type": "Organization",
      "updated_at": "05 Oct 2020"
    },
    {
      "abilities": "ABILITY,ABILITY",
      "created_at": "05 Oct 2020",
      "created_by_user_id": 1,
      "description": "Short description of the role",
      "id": 2,
      "name": "Role Name",
      "role_type": "Company",
      "updated_at": "05 Oct 2020"
    },
  ],
  "status": 200
}
```

## Company User Roles

```shell
  curl "https://lets.goboost.io/api/core/companies/:id/user_roles"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`GET https://lets.goboost.io/api/core/companies/:id/user_roles`

> The above command returns JSON structured like this:

```json
{
  "count": 2,
  "status": 200,
  "user_roles": [
    {
      "id": 1,
      "role_abilities": "ABILITY,ABILITY",
      "role_description": "Short description of the role",
      "role_id": 4,
      "role_name": "Role Name",
      "role_type": "Organization",
      "role_type_id": 5,
      "role_type_name": "Organization's Name",
      "user_email": "john@doe.com",
      "user_first_name": "John",
      "user_id": 6,
      "user_last_name": "Doe"
    },
    {
      "id": 2,
      "role_abilities": "ABILITY,ABILITY",
      "role_description": "Short description of the role",
      "role_id": 4,
      "role_name": "Role Name",
      "role_type": "Company",
      "role_type_id": 5,
      "role_type_name": "Company's Name",
      "user_email": "john@doe.com",
      "user_first_name": "John",
      "user_id": 6,
      "user_last_name": "Doe"
    }
  ],
  "status": 200
}
```

## Sign A Company Up For GoBoost

```shell
  curl -X POST "https://lets.goboost.io/api/core/sign_up"
    -d '{ "email": "john@doe.com", "google_place_id": "xxx", "code": "SIGNUPCODE", "hostname": "url.goboost.io", "token": "invite_token" } ' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`POST https://lets.goboost.io/api/core/sign_up`

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "message": "Welcome!",
  "credentials": {
    "auth_token": "xxx",
    "user_id": 1
  },
  "company": {
    "id": 1,
    "coupon_code": "xxx",
    "google_place_id": "xxxx",
    "sign_up_code_id": 1,
    "sign_up_organization_id": 1,
    "name": "Company Name",
    "address_line_one": "123 st.",
    "address_line_two": "Suite 123",
    "city": "City",
    "state": "State",
    "postal_code": "12345",
    "country": "US",
    "phone_number": "5555555555",
    "primary_website": "www.domain.com",
    "secondary_website": "",
    "latitude": 1.0000,
    "longitude": -1.0000,
    "time_zone_id": "",
    "service_area": {},
    "business_hours": {}
  },
  "user": {
    "id": 1,
    "email": "john@doe.com",
    "first_name": "John",
    "last_name": "Doe",
    "created_at": "Thu, 21 May 2020 16:04:31 GMT",
    "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
  }
}

```

# Opportunities

## Organization's Opportunities
```shell
  curl "https://lets.goboost.io/api/core/opportunities/organizations/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "opportunities": [
    {
      "adwords_search_query_id": ,
      "business_number": ,
      "call_recording_url": ,
      "call_status": ,
      "called_at": ,
      "caller_name": ,
      "caller_number": ,
      "channel_id": 1,
      "company_id": 1,
      "company_name": "Demo Company",
      "cost_cents": 0,
      "country": "US",
      "created_at": "05 Oct 2020",
      "ctm_city": "City",
      "ctm_id": "xxx",
      "ctm_latitude": 0.00,
      "ctm_longitude": -0.00,
      "ctm_postal_code": "12345",
      "ctm_state": "AB",
      "ctm_street": "123 Street.",
      "duration": 20,
      "email": "demo@demo.com",
      "form_data": {},
      "id": 1,
      "ip": "000.00.00.000",
      "ipstack_data": ,
      "is_bad_call": 0,
      "is_new_caller": 0,
      "is_spam": 0,
      "message": "form message",
      "name": "Jane Doe",
      "opportunity_source": 1,
      "opportunity_source_id": 1,
      "opportunity_type": 1,
      "phone_number": "+1234567890",
      "reviewed_at": "05 Oct 2020",
      "reviewed_by_user_id": 1,
      "reviewer_address_line_one": "123 st.",
      "reviewer_address_line_two": "",
      "reviewer_city": "City",
      "reviewer_latitude": 0.000,
      "reviewer_longitude": -0.000,
      "reviewer_notes": "",
      "reviewer_postal_code": "12345",
      "reviewer_rating": 5,
      "reviewer_state": "AB",
      "ring_time": 20,
      "sid": "xxx",
      "source": "Google Search",
      "talk_time": 85,
      "tracking_number_id": "xxx",
      "updated_at": "05 Oct 2020",
      "url": "xxx.com",
      "user_agent": "",
      "version": null
    },
    {
      "adwords_search_query_id": ,
      "business_number": ,
      "call_recording_url": ,
      "call_status": ,
      "called_at": ,
      "caller_name": ,
      "caller_number": ,
      "channel_id": 1,
      "company_id": 1,
      "company_name": "Demo Company",
      "cost_cents": 0,
      "country": "US",
      "created_at": "05 Oct 2020",
      "ctm_city": "City",
      "ctm_id": "xxx",
      "ctm_latitude": 0.00,
      "ctm_longitude": -0.00,
      "ctm_postal_code": "12345",
      "ctm_state": "AB",
      "ctm_street": "123 Street.",
      "duration": 20,
      "email": "demo@demo.com",
      "form_data": {},
      "id": 1,
      "ip": "000.00.00.000",
      "ipstack_data": ,
      "is_bad_call": 0,
      "is_new_caller": 0,
      "is_spam": 0,
      "message": "form message",
      "name": "Jane Doe",
      "opportunity_source": 1,
      "opportunity_source_id": 1,
      "opportunity_type": 1,
      "phone_number": "+1234567890",
      "reviewed_at": "05 Oct 2020",
      "reviewed_by_user_id": 1,
      "reviewer_address_line_one": "123 st.",
      "reviewer_address_line_two": "",
      "reviewer_city": "City",
      "reviewer_latitude": 0.000,
      "reviewer_longitude": -0.000,
      "reviewer_notes": "",
      "reviewer_postal_code": "12345",
      "reviewer_rating": 5,
      "reviewer_state": "AB",
      "ring_time": 20,
      "sid": "xxx",
      "source": "Google Search",
      "talk_time": 85,
      "tracking_number_id": "xxx",
      "updated_at": "05 Oct 2020",
      "url": "xxx.com",
      "user_agent": "",
      "version": null
    }
  ],
  "status": 200,
  "total": 1
}
```

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
channel_ids | [] | If channel IDs are present, it will only return opportunities belonging to those channels. Channels are: Google Ads, Facebook Ads, Site
start_date | null | If a date is given, it will return all opportunities from the start date on (inclusive). Format: ''
company_ids | [] | If Company IDs are present, it will only return opportunities belonging to those companies.
opportunity_types | [] | If opportunity types are present, it will filter for only opportunities of that type. Types: call or form.
limit | null | If limit is set, it will limit the number of returned results. Default is to return all.
page | null | If a page is set, in combination with limit, it will return paginated results.

This endpoint retrieves opportunities that belong to the organization's companies.

### HTTP Request

`GET https://lets.goboost.io/api/core/opportunities/organizations/:id`

## Company's Opportunities
```shell
  curl "https://lets.goboost.io/api/core/opportunities/companies/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "opportunities": [
    {
      "adwords_search_query_id": ,
      "business_number": ,
      "call_recording_url": ,
      "call_status": ,
      "called_at": ,
      "caller_name": ,
      "caller_number": ,
      "channel_id": 1,
      "company_id": 1,
      "company_name": "Demo Company",
      "cost_cents": 0,
      "country": "US",
      "created_at": "05 Oct 2020",
      "ctm_city": "City",
      "ctm_id": "xxx",
      "ctm_latitude": 0.00,
      "ctm_longitude": -0.00,
      "ctm_postal_code": "12345",
      "ctm_state": "AB",
      "ctm_street": "123 Street.",
      "duration": 20,
      "email": "demo@demo.com",
      "form_data": {},
      "id": 1,
      "ip": "000.00.00.000",
      "ipstack_data": ,
      "is_bad_call": 0,
      "is_new_caller": 0,
      "is_spam": 0,
      "message": "form message",
      "name": "Jane Doe",
      "opportunity_source": 1,
      "opportunity_source_id": 1,
      "opportunity_type": 1,
      "phone_number": "+1234567890",
      "reviewed_at": "05 Oct 2020",
      "reviewed_by_user_id": 1,
      "reviewer_address_line_one": "123 st.",
      "reviewer_address_line_two": "",
      "reviewer_city": "City",
      "reviewer_latitude": 0.000,
      "reviewer_longitude": -0.000,
      "reviewer_notes": "",
      "reviewer_postal_code": "12345",
      "reviewer_rating": 5,
      "reviewer_state": "AB",
      "ring_time": 20,
      "sid": "xxx",
      "source": "Google Search",
      "talk_time": 85,
      "tracking_number_id": "xxx",
      "updated_at": "05 Oct 2020",
      "url": "xxx.com",
      "user_agent": "",
      "version": null
    },
    {
      "adwords_search_query_id": ,
      "business_number": ,
      "call_recording_url": ,
      "call_status": ,
      "called_at": ,
      "caller_name": ,
      "caller_number": ,
      "channel_id": 1,
      "company_id": 1,
      "company_name": "Demo Company",
      "cost_cents": 0,
      "country": "US",
      "created_at": "05 Oct 2020",
      "ctm_city": "City",
      "ctm_id": "xxx",
      "ctm_latitude": 0.00,
      "ctm_longitude": -0.00,
      "ctm_postal_code": "12345",
      "ctm_state": "AB",
      "ctm_street": "123 Street.",
      "duration": 20,
      "email": "demo@demo.com",
      "form_data": {},
      "id": 1,
      "ip": "000.00.00.000",
      "ipstack_data": ,
      "is_bad_call": 0,
      "is_new_caller": 0,
      "is_spam": 0,
      "message": "form message",
      "name": "Jane Doe",
      "opportunity_source": 1,
      "opportunity_source_id": 1,
      "opportunity_type": 1,
      "phone_number": "+1234567890",
      "reviewed_at": "05 Oct 2020",
      "reviewed_by_user_id": 1,
      "reviewer_address_line_one": "123 st.",
      "reviewer_address_line_two": "",
      "reviewer_city": "City",
      "reviewer_latitude": 0.000,
      "reviewer_longitude": -0.000,
      "reviewer_notes": "",
      "reviewer_postal_code": "12345",
      "reviewer_rating": 5,
      "reviewer_state": "AB",
      "ring_time": 20,
      "sid": "xxx",
      "source": "Google Search",
      "talk_time": 85,
      "tracking_number_id": "xxx",
      "updated_at": "05 Oct 2020",
      "url": "xxx.com",
      "user_agent": "",
      "version": null
    }
  ],
  "status": 200,
  "total": 1
}
```
This endpoint retrieves opportunities that belong to the comapny

Parameter | Default | Description
--------- | ------- | -----------
channel_ids | [] | If channel IDs are present, it will only return opportunities belonging to those channels. Channels are: Google Ads, Facebook Ads, Site
start_date | null | If a date is given, it will return all opportunities from the start date on (inclusive). Format: ''
opportunity_types | [] | If opportunity types are present, it will filter for only opportunities of that type. Types: call or form.
limit | null | If limit is set, it will limit the number of returned results. Default is to return all.
page | null | If a page is set, in combination with limit, it will return paginated results.
### HTTP Request

`GET https://lets.goboost.io/api/core/opportunities/companies/:id`

# Reviews

## Review Sources
```shell
  curl "https://lets.goboost.io/api/reviews/review_sources"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "review_sources": [
    {
      "id": 1,
      "name": "Google",
      "short_name": "",
      "api_source_id": "",
      "code": "",
      "icon": "https://storage.googleapis.com/link/to/icon.png",
      "image": "https://storage.googleapis.com/link/to/image.png",
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    },
    {
      "id": 2,
      "name": "Facebook",
      "short_name": "",
      "api_source_id": "",
      "code": "",
      "icon": "https://storage.googleapis.com/link/to/icon.png",
      "image": "https://storage.googleapis.com/link/to/image.png",
      "created_at": "Thu, 21 May 2020 16:04:31 GMT",
      "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
    }
  ]
}
```
### HTTP Request

`GET https://lets.goboost.io/api/reviews/review_sources`



## Organization's Reviews
```shell
  curl "https://lets.goboost.io/api/reviews/organizations/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "reviews": [
    {
      "author": "Jane Doe",
      "company_id": 5,
      "company_name": "Demo Company",
      "content": "Demo Company is the best Company I've ever worked with",
      "created_at": "02 Oct 2020",
      "id": 1,
      "importer_api_response": {},
      "importer_review_id": "xxx",
      "importer_type": "GoogleAccountLocation",
      "importer_type_id": 1,
      "is_responded_to": 0,
      "name": "Jane Doe",
      "neg_keywords": null,
      "pos_keywords": null,
      "published_at": "01 Oct 2020",
      "rating": 5.0,
      "review_source_id": 1,
      "sentiment": null,
      "status": null,
      "summary": null,
      "total_reviews": 2,
      "updated_at": "06 Oct 2020",
      "url": null
    },
    {
      "author": "Jane Doe",
      "company_id": 5,
      "company_name": "Demo Company",
      "content": "Demo Company is the best Company I've ever worked with",
      "created_at": "02 Oct 2020",
      "id": 1,
      "importer_api_response": {},
      "importer_review_id": "xxx",
      "importer_type": "GoogleAccountLocation",
      "importer_type_id": 1,
      "is_responded_to": 0,
      "name": "Jane Doe",
      "neg_keywords": null,
      "pos_keywords": null,
      "published_at": "01 Oct 2020",
      "rating": 5.0,
      "review_source_id": 1,
      "sentiment": null,
      "status": null,
      "summary": null,
      "total_reviews": 2,
      "updated_at": "06 Oct 2020",
      "url": null
    }
  ],
  "status": 200
}
```

Parameter | Default | Description
--------- | ------- | -----------
review_source_id | [] | If a review_source_id is set, it will only return reviews belonging to that review source. Review sources are all the review sites we retrieve reviews from.
start_date | null | If a date is given, it will return all opportunities from the start date on (inclusive). Format: ''
company_ids | [] | If Company IDs are present, it will only return opportunities belonging to those companies.
limit | null | If limit is set, it will limit the number of returned results. Default is to return all.
page | null | If a page is set, in combination with limit, it will return paginated results.

### HTTP Request

`GET https://lets.goboost.io/api/reviews/organizations/:id`

## Organization Companies' Reviews
```shell
  curl "https://lets.goboost.io/api/reviews/organizations/:id/company_review_source_distribution"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "company_review_source_distribution": [
    {
      "company_id": 1,
      "count": 5,
      "is_responded_to_sum": 1,
      "latitude": 0.0000,
      "longitude": -0.000,
      "name": "GoBoost",
      "r1": 0,
      "r2": 0,
      "r3": 0,
      "r4": 0,
      "r5": 5,
      "rating_average": 5.0,
      "review_site_url": "https://reviewsite.com/page",
      "review_source_id": 1,
      "review_source_name": "Google"
    },
    {
      "company_id": 2,
      "count": 5,
      "is_responded_to_sum": 1,
      "latitude": 0.0000,
      "longitude": -0.000,
      "name": "GoBoost",
      "r1": 0,
      "r2": 0,
      "r3": 0,
      "r4": 0,
      "r5": 5,
      "rating_average": 5.0,
      "review_site_url": "https://reviewsite.com/page",
      "review_source_id": 2,
      "review_source_name": "Facebook"
    }
  ],
  "count": 1,
  "status": 200
}
```

Parameter | Default | Description
--------- | ------- | -----------
company_ids | [] | If company ids are passed only data for those company ids will be returned.
start_date | null | If a date is given, it will return all reviews from the start date on (inclusive). Format: ''
review_source_id | 'all' | If a review source ID is sent with the request only reviews belonging to those review sources are returned.
limit | null | If limit is set, it will limit the number of returned results. Default is 100 and max is 1,000.
offset | null | If an offset is set it starts at that number.

### HTTP Request

`GET https://lets.goboost.io/api/reviews/companies/:id`

## Company's Reviews
```shell
  curl "https://lets.goboost.io/api/reviews/companies/:id"
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "reviews": [
    {
      "author": "Jane Doe",
      "company_id": 5,
      "company_name": "Demo Company",
      "content": "Demo Company is the best Company I've ever worked with",
      "created_at": "02 Oct 2020",
      "id": 1,
      "importer_api_response": {},
      "importer_review_id": "xxx",
      "importer_type": "GoogleAccountLocation",
      "importer_type_id": 1,
      "is_responded_to": 0,
      "name": "Jane Doe",
      "neg_keywords": null,
      "pos_keywords": null,
      "published_at": "01 Oct 2020",
      "rating": 5.0,
      "review_source_id": 1,
      "sentiment": null,
      "status": null,
      "summary": null,
      "total_reviews": 2,
      "updated_at": "06 Oct 2020",
      "url": null
    },
    {
      "author": "Jane Doe",
      "company_id": 5,
      "company_name": "Demo Company",
      "content": "Demo Company is the best Company I've ever worked with",
      "created_at": "02 Oct 2020",
      "id": 1,
      "importer_api_response": {},
      "importer_review_id": "xxx",
      "importer_type": "GoogleAccountLocation",
      "importer_type_id": 1,
      "is_responded_to": 0,
      "name": "Jane Doe",
      "neg_keywords": null,
      "pos_keywords": null,
      "published_at": "01 Oct 2020",
      "rating": 5.0,
      "review_source_id": 1,
      "sentiment": null,
      "status": null,
      "summary": null,
      "total_reviews": 2,
      "updated_at": "06 Oct 2020",
      "url": null
    }
  ],
  "status": 200
}
```

Parameter | Default | Description
--------- | ------- | -----------
review_source_id | [] | If a review_source_id is set, it will only return reviews belonging to that review source. Review sources are all the review sites we retrieve reviews from.
start_date | null | If a date is given, it will return all opportunities from the start date on (inclusive). Format: ''
limit | null | If limit is set, it will limit the number of returned results. Default is to return all.
page | null | If a page is set, in combination with limit, it will return paginated results.

### HTTP Request

`GET https://lets.goboost.io/api/reviews/companies/:id`

## Create Company Review Site

```shell
  curl -X POST "https://lets.goboost.io/api/reviews/company_review_sites"
    -d '{ "company_id": 1, "url": "https://review_site.com/page", "review_source_id": 1 }' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: 1" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`POST https://lets.goboost.io/api/reviews/company_review_sites`

> The above command returns JSON structured like this:

```json
{
  "status": 200,
  "company_review_site": {
    "id": 1,
    "created_at": "Thu, 21 May 2020 16:04:31 GMT",
    "company_id": 1,
    "review_source_id": 1,
    "url": "https://review_site.com/page",
    "review_trackers_url_id": "xxx",
    "is_customer_shown": 1,
    "updated_at": "Thu, 21 May 2020 16:04:31 GMT"
  }
}
```

## Create Review Request

```shell
  curl -X POST "https://lets.goboost.io/api/reviews/review_requests"
    -d '{ "email": "jane@doe.com", "phone": "5555555555", "full_name": "Jane Doe" }' \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer xxx" \
    -H "current-user-role-id: (company_user_role_id)" \
    -H "true-user-role-id: 1"
```

### HTTP Request

`POST https://lets.goboost.io/api/reviews/review_requests`

> The above command returns JSON structured like this:

```json
{
  "review_request": {
    "id": 1,
    "company_id": 1,
    "current_user_id": 1,
    "full_name": 1,
    "email": "jane@doe.com",
    "phone": "5555555555",
    "message": "message to consumer",
    "is_clicked": false,
    "clicked_at": "",
    "created_at": "",
    "updated_at": "",
    "reference_id": "xxx",
    "true_user_role_id": 1,
    "current_user_role_id": 1
  },
  "status": 200
}
```