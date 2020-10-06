---
title: GoBoost API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='https://support.goboost.io' target="_blank">Contact us for Oauth credentials</a>

includes:
  - errors

search: true

code_clipboard: true
---

# Introduction

Welcome to the GoBoost API! You can use this API to get access reporting data about your GoBoost account.

# Terminology
Heirarchy - The GoBoost app is setup with a family type heirarchy. There are Organizations, Companies, Users, and User Roles.

Organization - An organization represents a partner business that refers small businesses to use GoBoost. Organizations can be setup in a heirarchy of their own, meaning an organization can be a child of another organization. Examples of popular heirarchies: Manufacturer -> Distributor -> Small Business or Manufacturer -> Small Business. Users who are tied to an organization are able to see all "descendents" of that organization. So if you have a role at an organization that has organizations below it and companies below that, you'd be able to see everything about all companies tied to all organizations below you in the heirarchy.

Company - A company represents a business that GoBoost is using our products. Companies must belong to at least one organization above it in the heirarchy.

Users - Users represent the actual people that are using the GoBoost system. User's are tied to organizations and/or companies and the role is what controls their access level.

User Roles - User roles are used to tie a user to either an organization or a company or both. The role that's chosen when you Authenticate into the API controls the level at which you'll be seeing data.

# Authentication

## Flow
GoBoost uses the oauth 2.0 protocol for Authentication.<br /><br />
Oauth Flow:<br />
1. POST to get a grant code and the list of user roles you have access to using your Base64 encoded login credentials and the Oauth App Client ID.<br />
2. POST the grant code you received from the previous step to the token route and you'll get a access token, refresh token, expires in seconds, and a status back.<br />
3. Use the access token as a bearer token in the Authorization header of each request you make to the GoBoost API.


## Grant Code

The first step in the authentication process is to get a Grant code. To do this you will pass your Base64 encoded email and password login credentials and the auth route.

This will return the grant code, along with a list of the user roles that this user has access to.

You'll need to select the role you want to authenticate and use the grant code you received to get an access token.

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

Once you have selected your role and received the grant code, you'll need to use that grant code to trade in for an Access token.

In order to do that you'll send a POST request to the token route along with your Oauth Application's Client Secret.

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
  You must replace <code>code</code> with the grant code you received fro mthe previous step and replace <code>client_secret</code> from the Oauth Application.
</aside>

## Refresh Token
The access token you receive will expire. When the token expires you'll need to use the refresh token you received in the Access Token step to get a new Access token

```shell
# With shell, you can just pass the correct header with each request
curl -X POST "https://lets.goboost.io/api/core/oauth/auth" \
-H "Content-Type: application/json" \
-d '{ "refresh_token": "xxx", "client_secret": "xxx", "grant_type": "refresh_token" }'
```

> You must replace `refresh_token` with the refresh you received from the Access Token step and `client_secret` with your Oauth Application's Client Secret.

> Successful Response:

```json
  {
    "access_token": "xxx",
    "expires_in_seconds": 3600,
    "status": 200
  }
```

<aside class="notice">
  You must replace <code>refresh_token</code> with the refresh you received from the Access Token step and <code>client_secret</code> with your Oauth Application's Client Secret.
</aside>

## Making Requests
Now that you have an Access Token in order to make requests to GoBoost routes you'll need to add that token as an authorization header to each request. You'll also need to add GoBoost specific headers for the user role that you have selected. You can choose the user role from the list of user roles provided in the grant code request in the first step of the oauth process.

```shell
  curl "https://lets.goboost.io/api/(route)" \
  -H "Content-Type: application/json" \
  -H "Authorization: 'Bearer xxx" \
  -H "current-user-role-id: 1" \
  -H "true-user-role-id: 1",
```

> Replace `xxx` with the Access Token

<aside class="notice">
  Replace <code>xxx</code> with the Access Token.
</aside>

# Organizations

## Specific Organization

```shell
curl "https://lets.goboost.io/api/core/organizations/:id"
  -H "Content-Type: application/json" \
  -H "Authorization: xxx"
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
  -H "Authorization: xxx"
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

This endpoint retrieves all companies that are below an organization in the heirarchy.

### HTTP Request

`GET https://lets.goboost.io/api/core/organizations/:id/companies`

## Organization's users

```shell
curl "https://lets.goboost.io/api/core/organizations/:id/users"
  -H "Content-Type: application/json" \
  -H "Authorization: xxx"
  -H "current-user-role-id: 1" \
  -H "true-user-role-id: 1"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves all users that are below an organization in the heirarchy.

### HTTP Request

`GET https://lets.goboost.io/api/core/organizations/:id/users`


# Companies
## Company
```shell
curl "https://lets.goboost.io/api/core/companies/:id"
  -H "Content-Type: application/json" \
  -H "Authorization: xxx"
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
  -H "Authorization: xxx"
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

This endpoint retrieves all active subscriptions for a company.

### HTTP Request

`GET https://lets.goboost.io/api/core/companies/:id/subscription`

## Company Plans
```shell
curl "https://lets.goboost.io/api/core/companies/:id/plans"
  -H "Content-Type: application/json" \
  -H "Authorization: xxx"
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

This endpoint retrieves all available plans for a company.

### HTTP Request

`GET https://lets.goboost.io/api/core/companies/:id/plans`

## Company Sites Profile
```shell
curl "https://lets.goboost.io/api/sites/company_sites_profiles/:id"
  -H "Content-Type: application/json" \
  -H "Authorization: xxx"
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
  -H "Authorization: xxx"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves reviews specific data for a specific company

### HTTP Request

`GET https://lets.goboost.io/api/reviews/company_reviews_profile/:id`

## Company Ads Profile
```shell
curl "https://lets.goboost.io/api/ads/company_ads_profile/:id"
  -H "Authorization: xxx"
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves ads specific data for a specific company

### HTTP Request

`GET https://lets.goboost.io/api/ads/company_ads_profile/:id`

# Opportunities

## Organization's Opportunities
```shell
curl "https://lets.goboost.io/api/core/opportunities/organizations/:id"
  -H "Authorization: xxx"
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
channel_ids | [] | If channel IDs are present it will only return opportunities belonging to those channels. Channels are: Google Ads, Facebook Ads, Site
start_date | null | If a date is given it will return all opportunities from the start date on (inclusive). Format: ''
company_ids | [] | If company IDs are present it will only return opportunities belonging to those companies.
opportunity_types | [] | If opportunity types are present it will filter for only opportunities of that type. Types: call or form.
limit | null | If limit is set it will limit the number of returned results. Default is to return all.
page | null | If a page is set, in combination with limit, it will return paginated results.

This endpoint retrieves opportunities that belong to the organization's companies.

### HTTP Request

`GET https://lets.goboost.io/api/core/opportunities/organizations/:id`

## Company's Opportunities
```shell
curl "https://lets.goboost.io/api/core/opportunities/companies/:id"
  -H "Authorization: xxx"
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
channel_ids | [] | If channel IDs are present it will only return opportunities belonging to those channels. Channels are: Google Ads, Facebook Ads, Site
start_date | null | If a date is given it will return all opportunities from the start date on (inclusive). Format: ''
opportunity_types | [] | If opportunity types are present it will filter for only opportunities of that type. Types: call or form.
limit | null | If limit is set it will limit the number of returned results. Default is to return all.
page | null | If a page is set, in combination with limit, it will return paginated results.
### HTTP Request

`GET https://lets.goboost.io/api/core/opportunities/companies/:id`

# Reviews
## Organization's Reviews
```shell
curl "https://lets.goboost.io/api/reviews/organizations/:id"
  -H "Content-Type: application/json" \
  -H "Authorization: xxx"
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
      "content": "Demo Company is the best company I've ever worked with",
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
      "content": "Demo Company is the best company I've ever worked with",
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
review_source_id | [] | If a review_source_id is returned it will only return reviews belonging to that review source. Review sources are all the review sites we retrieve reviews from.
start_date | null | If a date is given it will return all opportunities from the start date on (inclusive). Format: ''
company_ids | [] | If company IDs are present it will only return opportunities belonging to those companies.
limit | null | If limit is set it will limit the number of returned results. Default is to return all.
page | null | If a page is set, in combination with limit, it will return paginated results.

### HTTP Request

`GET https://lets.goboost.io/api/reviews/organizations/:id`

## Company's Reviews
```shell
curl "https://lets.goboost.io/api/reviews/companies/:id"
  -H "Content-Type: application/json" \
  -H "Authorization: xxx"
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
      "content": "Demo Company is the best company I've ever worked with",
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
      "content": "Demo Company is the best company I've ever worked with",
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
review_source_id | [] | If a review_source_id is returned it will only return reviews belonging to that review source. Review sources are all the review sites we retrieve reviews from.
start_date | null | If a date is given it will return all opportunities from the start date on (inclusive). Format: ''
limit | null | If limit is set it will limit the number of returned results. Default is to return all.
page | null | If a page is set, in combination with limit, it will return paginated results.

### HTTP Request

`GET https://lets.goboost.io/api/reviews/companies/:id`