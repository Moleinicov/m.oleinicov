# Peytz Mail API version 1


## Introduction

This document describes the version 1 API of Peytz Mail newsletter system.
The Peytz Mail API is a REST API supporting XML and JSON data formats.
To make the examples work you need to replace `{CUSTOMER_NAME}` and `{API_KEY}`
with the information we provided you.

__API Endpoint__
{CUSTOMER_NAME}.peytzmail.com

### Authentication
Authentication to the API is done with [http basic authentication](http://en.wikipedia.org/wiki/Basic_access_authentication).
Use your `API_KEY` as username and an __empty__ password. All request to the API must be done through __https__.

#### Examples
__Curl:__

GET request:

```bash
curl https://{CUSTOMER_NAME}.peytzmail.com/api/v1/mailinglists.json -u {API_KEY}:
```
POST request with body from file `post.xml`

```bash
curl -d @post.xml -H "Content-Type: application/xml" https://{CUSTOMER_NAME}.peytzmail.com/api/v1/mailinglists/{MAILINGLIST}/newsletters/create_and_send.xml -u {API_KEY}:
```
__Ruby:__

```ruby
require 'net/http'
uri = URI('https://{CUSTOMER_NAME}.peytzmail.com/api/v1/mailinglists.json')
request = Net::HTTP::Get.new(uri.request_uri)
request.basic_auth '{API_KEY}', ''
response = Net::HTTP.start(uri.hostname, uri.port) {|http|
  http.request(request)
}
p response.body
```

__PHP:__

```php
<?php
function peytzMailApiCall($url, $path, $method, $data) {

  $jsonData = json_encode($data);

  $headers = array( 'Accept: application/json', 'Content-Type: application/json' );
  $apiKey = '{API_KEY}';

  $handle = curl_init();
  curl_setopt($handle, CURLOPT_URL, $url . $path);
  curl_setopt($handle, CURLOPT_HTTPHEADER, $headers);
  curl_setopt($handle, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($handle, CURLOPT_SSL_VERIFYHOST, false);
  curl_setopt($handle, CURLOPT_SSL_VERIFYPEER, false);
  curl_setopt($handle, CURLOPT_USERPWD, $apiKey.':');

  switch($method) {
  case 'PUT':
    curl_setopt($handle, CURLOPT_CUSTOMREQUEST, 'PUT');
    curl_setopt($handle, CURLOPT_POSTFIELDS, $jsonData);
    break;
  case 'DELETE':
    curl_setopt($handle, CURLOPT_CUSTOMREQUEST, 'DELETE');
    curl_setopt($handle, CURLOPT_POSTFIELDS, $jsonData);
    break;
  case 'POST':
    curl_setopt($handle, CURLOPT_POST, true);
    curl_setopt($handle, CURLOPT_POSTFIELDS, $jsonData);
    break;
  default: // GET is default
    curl_setopt($handle, CURLOPT_HTTPGET, true);
    break;
  }


  $response = curl_exec($handle);
  $code = curl_getinfo($handle, CURLINFO_HTTP_CODE);

  return array('code' => $code, 'response' => $response);
}

$url = 'https://{CUSTOMER_NAME}.peytzmail.com';
$path = '/api/v1/mailinglists.json';
print_r(peytzMailApiCall($url, $path, 'GET', null));
```

#### HTTP Responses
All HTTP responses related to failing authentication are listed below.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>401 Unauthorized</td>
      <td>You're not authorized to make that API request!</td>
    </tr>
    <tr>
      <td>403 Forbidden</td>
      <td>You're not allowed to call that API method</td>
    </tr>
    <tr>
      <td>420</td>
      <td>The API has been disabled. Maybe you were a little too trigger happy?</td>
    </tr>
    <tr>
      <td>500 Internal Server Error</td>
      <td>We're sorry! An unhandled exception occured when handling your API request. We have been notified and will take a look at the incident as soon as possible.</td>
    </tr>
  </tbody>
</table>

### Responses in general
The API will respond with http status codes to indicate the status of the
request and a JSON string as response body.

__HTTP Status Code Summary__

* 200 OK - Everything worked as expected.
* 201 Created - The request was accepted and resource created.
* 202 Accepted - The request was accepted for further processing.
* 400 Bad Request - Missing a required parameter.
* 401 Unauthorized - No valid API key provided.
* 403 Forbidden - The user requesting the API did not have permission to access
the resource or sub-resources (e.g. the user did not have permissions
to a particular mailinglist and newsletters in the mailinglist).
* 404 Not Found - The requested item doesn't exist.
* 409 Conflict - Some rule/setting has been violated (e.g. minimum publish interval
has been violated).
* 412 Precondition Failed - Some required initial setup could not be performed.
* 413 Request Entity Too Large - Request is creating/updating too many resources at once
(e.g. subscribing more subscribers than allowed at once).
* 420 API Disabled - The API has been disabled (may be due to calling the API too
frequently).
* 422 Unprocessable Entity - The request was well-formed but the systen was
unable to carry out the requested action.
* 500 Internal Server Error - something went wrong on Peytz Mail's end.
* 501 Not Implemented - The feature is not implemented yet.

### Pagination
Some result sets are limited and paginated to avoid excessive data transfer.
To get to the following page you must follow the link in the bottom of the
result set.

## Subscriber Methods

### Get subscribers

_Gets all subscribers_

* URL: `/api/v1/subscribers.json`
* Method: GET
* Optional parameters: `page`
* Notes
  * This method makes use of pagination, follow the next page url for more data.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/subscribers.json`

Response:

```json
{
  "current_page": 1,
  "total_records": 1,
  "total_pages": 1,
  "next_page": null,
  "subscribers": [
  {
    "id": "john-smith-at-example-com",
      "email": "john.smith@example.com",
      "first_name": "John",
      "last_name": "Smith",
      "foreign_id": "000001",
      "mailinglist_ids": [
        "daily-newsletter",
        "weekly-newsletter"
      ],
      "pending_mailinglist_ids": [
        "hourly-newsletter"
      ],
      "extra_fields": {
        "city": "New York",
        "gender": "Male"
      }
  }
  ]
}
```

```xml
<subscribers>
  <current-page type="integer">1</current-page>
  <total-records type="integer">1</total-records>
  <total-pages type="integer">1</total-pages>
  <next-page nil="true"></next-page>
  <subscribers type="array">
    <subscriber>
      <id>john-smith-at-example-com</id>
      <email>john.smith@example.com</email>
      <first-name>John</first-name>
      <last-name>Smith</last-name>
      <foreign-id>000001</foreign-id>
      <mailinglist-ids type="array">
        <mailinglist-id>daily-newsletter</mailinglist-id>
        <mailinglist-id>weekly-newsletter</mailinglist-id>
      </mailinglist-ids>
      <pending-mailinglist-ids type="array">
        <pending-mailinglist-id>daily-newsletter</pending-mailinglist-id>
      </pending-mailinglist-ids>
      <extra-fields>
        <city>New York</city>
        <gender>Male</gender>
      </extra-fields>
    </subscriber>
  </subscribers>
</subscribers>
```

### Find subscribers

_Finds subscribers based on search criterias_

* URL: `/api/v1/subscribers/search.json`
* Method: GET
* Required parameters: `criteria[]`
* Notes
  * The search value is case-sensitive
  * You add search parameters by passing a criteria hash
  * This method makes use of pagination, follow the next page url for more data

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing search criteria</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/subscribers/search.json?criteria[email]=john.smith@example.com`

Response:

```json
{
  "current_page": 1,
  "total_records": 1,
  "total_pages": 1,
  "next_page": null,
  "subscribers": [
  {
    "id": "john-smith-at-example-com",
      "email": "john.smith@example.com",
      "first_name": "John",
      "last_name": "Smith",
      "foreign_id": "000001",
      "mailinglist_ids": [
        "daily-newsletter",
        "weekly-newsletter"
      ],
      "pending_mailinglist_ids": [
        "hourly-newsletter"
      ],
      "extra_fields": {
        "city": "New York",
        "gender": "Male"
      }
  }
  ]
}
```
```xml
<subscribers>
  <current-page type="integer">1</current-page>
  <total-records type="integer">1</total-records>
  <total-pages type="integer">1</total-pages>
  <next-page nil="true"></next-page>
  <subscribers type="array">
    <subscriber>
      <id>john-smith-at-example-com</id>
      <email>john.smith@example.com</email>
      <first-name>John</first-name>
      <last-name>Smith</last-name>
      <foreign-id>000001</foreign-id>
      <mailinglist-ids type="array">
        <mailinglist-id>daily-newsletter</mailinglist-id>
        <mailinglist-id>weekly-newsletter</mailinglist-id>
      </mailinglist-ids>
      <pending-mailinglist-ids type="array">
        <pending-mailinglist-id>daily-newsletter</pending-mailinglist-id>
      </pending-mailinglist-ids>
      <extra-fields>
        <city>New York</city>
        <gender>Male</gender>
      </extra-fields>
    </subscriber>
  </subscribers>
</subscribers>
```

### Get subscriber

_Gets a subscriber_

* URL: `/api/v1/subscribers/<subscriber-id>.json`
* Method: GET
* Required parameters: `<subscriber-id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/subscribers/john-smith-at-example-com.json`

Response:

```json
{
  "subscriber": {
    "id": "john-smith-at-example-com",
    "email": "john.smith@example.com",
    "first_name": "John",
    "last_name": "Smith",
    "mailinglist_ids": [
      "daily-newsletter",
      "weekly-newsletter"
    ],
    "pending_mailinglist_ids": [
      "hourly-newsletter"
    ],
    "extra_fields": {
      "city": "New York",
      "gender": "Male"
    }
  }
}
```
```xml
<subscriber>
  <id>john-smith-at-example-com</id>
  <email>john.smith@example.com</email>
  <first-name>John</first-name>
  <last-name>Smith</last-name>
  <mailinglist-ids type="array">
    <mailinglist-id>daily-newsletter</mailinglist-id>
    <mailinglist-id>weekly-newsletter</mailinglist-id>
  </mailinglist-ids>
  <pending-mailinglist-ids type="array">
    <pending-mailinglist-id>daily-newsletter</pending-mailinglist-id>
  </pending-mailinglist-ids>
  <extra-fields>
    <city>New York</city>
    <gender>Male</gender>
  </extra-fields>
</subscriber>
```

### Update subscriber

_Updates a subscriber by Peytz Mail subscriber id without changing any mailinglists subscriptions_

* URL: `/api/v1/subscribers/<subscriber-id>.json`
* Method: PUT
* Required parameters: `subscriber-id`
* Optional parameters: `email`, `foreign_id`, `first_name`, `last_name`, `gender`, `city`...
* Notes
  * You can add more parameters for storing extra information on the subscriber
  * You can use this method to update an email address on an already existing subscriber
  * Setting the optional unsafe query parameter will prevent overwriting
  existing subscriber fields
  * Find the subscriber id by using the subscriber find api method to look up by email first

<table border="0" cellspacing="0" cellpadding="0">
  <tr>
    <th>HTTP response</th>
    <th>Reason</th>
  </tr>
  <tr>
    <td>200 OK</td>
    <td>Subscriber data has been updated.</td>
  </tr>
  <tr>
    <td>422 Unprocessable Entity</td>
    <td>Subscriber could not be updated</td>
  </tr>
  <tr>
    <td>422 Unprocessable Entity</td>
    <td>Malformed data</td>
  </tr>
</table>

#### Example
Request:

`/api/v1/subscribers/john-smith-at-example-com.json`

Post data:

```json
{
  "subscriber": {
    "email": "joe.bloggs@example.com",
    "first_name": "Joe",
    "last_name": "Bloggs",
    "gender": "Male",
    "city": "London"
  }
}
```
```xml
  <subscriber>
    <email>joe.bloggs@example.com</email>
    <first-name>Joe</first-name>
    <last-name>Bloggs</last-name>
    <gender>Male</gender>
    <city>London</city>
  </subscriber>
```

Response:

```json
{
  "subscriber": {
    "id": "john-smith-at-example-com",
    "email": "joe.bloggs@example.com",
    "first_name": "Joe",
    "last_name": "Bloggs",
    "mailinglist_ids": [
      "daily-newsletter",
      "weekly-newsletter"
    ],
    "pending_mailinglist_ids": [
      "hourly-newsletter"
    ],
    "extra_fields": {
      "city": "London",
      "gender": "Male"
    }
  }
}
```

#### Example with unsafe set

_This example shows an update of an existing subscriber with the first_name John and no last_name._

Request:
`/api/v1/subscribers/john-smith-at-example-com.json?unsafe`

```json
{
  "subscriber": {
    "email": "joe.bloggs@example.com",
    "first_name": "Joe",
    "last_name": "Bloggs",
    "city": "London"
  }
}
```
```xml
  <subscriber>
    <email>joe.bloggs@example.com</email>
    <first-name>Joe</first-name>
    <last-name>Bloggs</last-name>
    <city>London</city>
  </subscriber>
```

Response:

_Because of the unsafe query parameter first_name, email and city wasn't changed but last_name was since it wasn't set at first._

```json
{
  "subscriber": {
    "id": "john-smith-at-example-com",
    "email": "john.smith@example.com",
    "first_name": "John",
    "last_name": "Bloggs",
    "mailinglist_ids": [
      "daily-newsletter",
      "weekly-newsletter"
    ],
    "pending_mailinglist_ids": [
      "hourly-newsletter"
    ],
    "extra_fields": {
      "city": "New York",
      "gender": "Male"
    }
  }
}
```

### Create or update subscriber

_Adds or updates a subscriber without signing up to any mailinglists_

* URL: `/api/v1/subscribers.json`
* Method: POST
* Required parameters: `email`
* Optional parameters: `first_name`, `last_name`, `gender`, `city`...
* Notes
  * You can add more parameters for storing extra information on the subscriber
  * If you post an already existing subscriber the subscriber will be updated
  * Setting the optional unsafe query parameter will prevent overwriting
  existing subscriber fields

It is possible to add a single subscriber or multiple subscribers at the same
time. Adding multiple subscribers is subject to a bulk limit.

#### Single subscriber response
<table border="0" cellspacing="0" cellpadding="0">
  <tr>
    <th>HTTP response</th>
    <th>Reason</th>
  </tr>
  <tr>
    <td>200 OK</td>
    <td>Subscriber was already created. Subscriber data has been updated.</td>
  </tr>
  <tr>
    <td>201 Created</td>
    <td>New subscriber has been created</td>
  </tr>
  <tr>
    <td>422 Unprocessable Entity</td>
    <td>Subscriber could not be created or updated</td>
  </tr>
  <tr>
    <td>422 Unprocessable Entity</td>
    <td>Malformed data</td>
  </tr>
</table>

#### Multiple subscribers response
<table border="0" cellspacing="0" cellpadding="0">
  <tr>
    <th>HTTP response</th>
    <th>Reason</th>
  </tr>
  <tr>
    <td>200 OK</td>
    <td>You are trying to subscribe multiple subscribers - check response body for which have been created/updated/failed.</td>
  </tr>
  <tr>
    <td>413 Request Entity Too Large</td>
    <td>You are sending too many subscribers per request</td>
  </tr>
  <tr>
    <td>422 Unprocessable Entity</td>
    <td>Malformed data</td>
  </tr>
</table>

#### Example with a single subscriber
Request:
`/api/v1/subscribers.json`

Post data:

```json
{
  "subscriber": {
    "email": "john.smith@example.com",
    "first_name": "John",
    "last_name": "Smith",
    "gender": "Male",
    "city": "New York"
  }
}
```
```xml
  <subscriber>
    <email>john.smith@example.com</email>
    <first-name>John</first-name>
    <last-name>Smith</last-name>
    <gender>Male</gender>
    <city>New York</city>
  </subscriber>
```

Response:

```json
{
  "subscriber": {
    "id": "john-smith-at-example-com",
    "email": "john.smith@example.com",
    "first_name": "John",
    "last_name": "Smith",
    "mailinglist_ids": [
      "daily-newsletter",
      "weekly-newsletter"
    ],
    "pending_mailinglist_ids": [
      "hourly-newsletter"
    ],
    "extra_fields": {
      "city": "New York",
      "gender": "Male"
    }
  }
}
```

#### Example with a single existing subscriber and with unsafe set

_This example shows an update of an existing subscriber with the first_name John and no last_name._

Request:
`/api/v1/subscribers.json?unsafe`

Post data:

```json
{
  "subscriber": {
    "email": "john.smith@example.com",
    "first_name": "Joe",
    "last_name": "Bloggs",
    "gender": "Male",
    "city": "New York"
  }
}
```
```xml
  <subscriber>
    <email>john.smith@example.com</email>
    <first-name>Joe</first-name>
    <last-name>Bloggs</last-name>
    <gender>Male</gender>
    <city>New York</city>
  </subscriber>
```

Response:

_Because of the unsafe query parameter first_name wasn't changed but last_name was since it wasn't set at first._

```json
{
  "subscriber": {
    "id": "john-smith-at-example-com",
    "email": "john.smith@example.com",
    "first_name": "John",
    "last_name": "Bloggs",
    "mailinglist_ids": [
      "daily-newsletter",
      "weekly-newsletter"
    ],
    "pending_mailinglist_ids": [
      "hourly-newsletter"
    ],
    "extra_fields": {
      "city": "New York",
      "gender": "Male"
    }
  }
}
```

#### Example with multiple subscribers
Request:
`/api/v1/subscribers.json`

Post data:

```json
{
  "subscribers": [
    {
      "email": "john.smith@example.com",
      "first_name": "John",
      "last_name": "Smith",
      "gender": "Male",
      "city": "New York"
    },
    {
      "email": "no-such-email",
      "first_name": "John",
      "last_name": "Smith",
      "gender": "Male",
      "city": "New York"
    }
  ]
}
```
```xml
 <subscribers type="array">
  <subscriber>
    <email>john.smith@example.com</email>
    <first_name>John</first_name>
    <last_name>Smith</last_name>
    <gender>Male</gender>
    <city>New York</city>
  </subscriber>
  <subscriber>
    <email>no-such-email</email>
    <first_name>John</first_name>
    <last_name>Smith</last_name>
    <gender>Male</gender>
    <city>New York</city>
  </subscriber>
</subscribers>
```

Response:

```json
{
  "subscribers": [
    {
      "id": "john-smith-at-example-com",
      "email": "john.smith@example.com",
      "foreign_id": "",
      "state": "ok"
    },
    {
      "email": "no-such-email",
      "state": "failed",
      "message": "Failed creating or updating subscriber: Email is not a valid email address (no-such-email)"
    }
  ]
}
```

### Get subscriber events

_Gets a paginated list of subscriber events for a given subscriber based on id_

* URL: `/api/v1/subscribers/<subscriber-id>/events.json`
* Method: GET
* Required parameters: `<subscriber-id>`
* Optional parameters:
  * `period_start`
  * `period_end`
  * `from_id`
  * `sort_order`
  * `per_page`

`from_id` can be a `event_id` value. `event_id`s are strictly monotonically increasing and setting `from_id` will only return tracks with an id strictly greater than. `from_id` will override `period_start` and `period_end` if set.

`sort_order` can be `desc` or `asc` (default). Be aware of pagination issues when sorting descending.

`per_page` can be an integer up to a global max limit.

<table border="0" cellspacing="0" cellpadding="0">
	<thead>
		<tr>
			<th>HTTP response</th>
			<th>Reason</th>
		</tr>
	</thead>
	<tbody>
		<tr><td>200 OK</td><td>-</td></tr>
		<tr><td>404 Not Found</td><td>Subscriber does not exist</td></tr>
		<tr><td>400 Bad Request</td><td>Invalid <tt>from_id</tt></td></tr>
		<tr><td>400 Bad Request</td><td>Unable to parse <tt>period_start</tt> or <tt>period_end</tt></td></tr>
	</tbody>
</table>

#### Example
Request:
`/api/v1/subscribers/john-smith-at-example-com/events.json`

Response:

```json
{
  "current_page": 1,
  "total_records": 2,
  "total_pages": 1,
  "next_page": null,
  "events": [
    {
      "event_id": "544f8dbd890495076f00001a",
      "created": "2014-10-28T13:36:13+01:00",
      "change": "subscribe",
      "source": "backend",
      "subscriber_token": "phfmhx",
      "extra_fields":
      {
        "email": "john.smith@example.com"
      },
      "mailinglist_id": "weekly-newsletter",
      "referer": "Backend subscribe by KH, skip_confirm, skip_welcome"
    },
    {
      "event_id": "544f8e31890495dae1000078",
      "created": "2014-10-28T13:38:09+01:00",
      "change": "mail_sent",
      "source": "newsletter",
      "subscriber_token": "phfmhx",
      "mailinglist_id": "weekly-newsletter",
      "referer": "john-smith-at-example-com",
      "newsletter_id": "newsletter-week-2"
    }
  ]
}
```

## Mailinglist Methods

### Create mailinglist
_Creates a new mailinglist (only available for administrators)_

* URL: `/api/v1/mailinglists.json`
* Method: POST
* Required parameters: `title`, `default_template`, `send_confirmation_mail`
and `send_welcome_mail`
* Optional parameters: `description`, `from_name`, `from_email`, `reply_to`,
`domain_key`, `send_confirmation_mail`, `send_welcome_mail`,
`confirmation_mail_category`, `welcome_mail_category`,
`litmus_settings_override`, `litmus_tracking_code`,
`auto_tracking`, `auto_counting`, `text_template`,
`cacheable`, `data_feeds_attributes` (only API level 0 users) and `inline_css`.

The list of optional parameters is not complete. Contact us for additional
information.

* Notes
  * By default `send_confirmation_mail` and `send_welcome_mail` are set to true
  so remember to set these to false if you're not providing other settings for
  these features!

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>201 Created</td>
      <td>Mailinglist was created</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist parameters given</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Validation errors</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>Mailinglist with that id already exists</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists.json`

Post data:

```json
{
  "mailinglist": {
    "title": "Weekly newsletter",
    "from_name": "The example company",
    "from_email": "newsletters@example-company.com",
    "reply_to": "admin@example-company.com",
    "default_template": "weekly_newsletter",
    "default_theme_id": "company",
    "default_layout_id": "weekly",
    ...
    "description": "Weekly newsletter sent to all subscribers",
    "domain_key": "jan12-dkim-example-company-com",
    "send_confirmation_mail": false,
    "send_welcome_mail": false,
    "data_feeds_attributes": [
      {
        "run": true,
        "run_on_preview": true,
        "feed_id": 111,
        "no_data": "continue",
        "failed": "skip"
      }
    ]
  }
}
```
```xml
<mailinglist>
  <title>Weekly newsletter</title>
  <from-name>The example company</from-name>
  <from-email>newsletters@example-company.com</from-email>
  <reply-to>admin@example-company.com</reply-to>
  <default-template>weekly_newsletter</default-template>
  <default-theme-id>company</default-theme-id>
  <default-layout-id>weekly</default-layout-id>
  ...
  <description>Weekly newsletter sent to all subscribers</description>
  <domain-key-id>jan12-dkim-example-company-com</domain-key-id>
  <send-confirmation-mail type="boolean">false</send-confirmation-mail>
  <send-welcome-mail type="boolean">false</send-welcome-mail>
  <data-feeds-attributes type="array"> <!-- Only API level 0 users can set this -->
    <data-feeds-attributes>
      <run>true</run>
      <run-on-preview>true</run-on-preview>
      <feed-id>111</feed-id>
      <no-data>continue</no-data>
      <failed>skip</failed>
    </data-feeds-attributes>
  </data-feeds-attributes>
</mailinglist>
```

### Update mailinglist
_Updates mailinglist (only available for administrators)_

* URL: `/api/v1/mailinglists/1.json`
* Method: PUT
* Required parameters: `id`
* Optional parameters: `description`, `from_name`, `from_email`, `reply_to`,
`domain_key`, `send_confirmation_mail`, `send_welcome_mail`,
`confirmation_mail_category`, `welcome_mail_category`,
`litmus_settings_override`, `litmus_tracking_code`,
`auto_tracking`, `auto_counting`, `text_template`,
`cacheable`, `data_feeds_attributes` (only API level 0 users) and `inline_css`.

The list of optional parameters is not complete. Contact us for additional
information.

* Notes
  * By default `send_confirmation_mail` and `send_welcome_mail` are set to true
  so remember to set these to false if you're not providing other settings for
  these features!

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>Mailinglist was updated</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist parameters given</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Validation errors</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>Mailinglist with that id already exists</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/1.json`

Post data:

```json
{
  "id": "weekly-newsletter",
  "mailinglist": {
    "title": "Weekly newsletter",
    "from_name": "The example company",
    "from_email": "newsletters@example-company.com",
    "reply_to": "admin@example-company.com",
    "default_template": "weekly_newsletter",
    "default_theme_id": "company",
    "default_layout_id": "weekly",
    ...
    "description": "Weekly newsletter sent to all subscribers",
    "domain_key": "jan12-dkim-example-company-com",
    "send_confirmation_mail": false,
    "send_welcome_mail": false,
    "data_feeds_attributes": [
      {
        "run": true,
        "run_on_preview": true,
        "feed_id": 111,
        "no_data": "continue",
        "failed": "skip"
      }
    ]
  }
}
```
```xml
<mailinglist>
  <title>Weekly newsletter</title>
  <from-name>The example company</from-name>
  <from-email>newsletters@example-company.com</from-email>
  <reply-to>admin@example-company.com</reply-to>
  <default-template>weekly_newsletter</default-template>
  <default-theme-id>company</default-theme-id>
  <default-layout-id>weekly</default-layout-id>
  ...
  <description>Weekly newsletter sent to all subscribers</description>
  <domain-key-id>jan12-dkim-example-company-com</domain-key-id>
  <send-confirmation-mail type="boolean">false</send-confirmation-mail>
  <send-welcome-mail type="boolean">false</send-welcome-mail>
  <data-feeds-attributes type="array"> <!-- Only API level 0 users can set this -->
    <data-feeds-attributes>
      <run>true</run>
      <run-on-preview>true</run-on-preview>
      <feed-id>111</feed-id>
      <no-data>continue</no-data>
      <failed>skip</failed>
    </data-feeds-attributes>
  </data-feeds-attributes>
</mailinglist>
```

### Get mailinglists
_Gets a list of mailinglists_

* URL: `/api/v1/mailinglists.json`
* Method: GET

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Response:

```json
{
  "mailinglists": [
    {
      "id": "weekly-newsletter",
      "title": "Weekly newsletter",
      "from_name": "The example company",
      "from_email": "newsletters@example-company.com",
      "reply_to": "admin@example-company.com",
      "default_template": "weekly_newsletter",
      "default_theme_id": "company",
      "default_layout_id": "weekly",
      "description": "Weekly newsletter sent to all subscribers",
      "domain_key": "jan12-dkim-example-company-com"
    },
    ...
  ]
}
```
```xml
<mailinglists type="array">
  <mailinglist>
    <id>weekly-newsletter</id>
    <title>Weekly newsletter</title>
    <from-name>The example company</from-name>
    <from-email>newsletters@example-company.com</from-email>
    <reply-to>admin@example-company.com</reply-to>
    <default-template>weekly_newsletter</default-template>
    <default-theme-id>company</default-theme-id>
    <default-layout-id>weekly</default-layout-id>
    <description>Weekly newsletter sent to all subscribers</description>
    <domain-key-id>jan12-dkim-example-company-com</domain-key-id>
  </mailinglist>
  ...
</mailinglists>
```

### Get mailinglist
_Get information about a mailinglist_

* URL: `/api/v1/mailinglists/<mailinglist-id>.json`
* Method: GET
* Required parameters: `<mailinglist-id>`
* Deprecated properties will always return null, and can't be set.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist parameters given</td>
    </tr>
    <tr>
      <td>404 Not found</td>
      <td>Mailinglist does not exist</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter.json`

Response:

```json
{
  "mailinglist": {
    "id": "weekly-newsletter",
    "title": "Weekly newsletter",
    "from_name": "The example company",
    "from_email": "newsletters@example-company.com",
    "reply_to": "admin@example-company.com",
    "default_template": "weekly_newsletter",
    "default_theme_id": "company",
    "default_layout_id": "weekly",
    "description": "Weekly newsletter sent to all subscribers",
    "domain_key": "jan12-dkim-example-company-com",
    "default-frontend-layout": null,
    "frontend-settings-override": null,
  }
}
```
```xml
<mailinglist>
  <id>weekly-newsletter</id>
  <title>Weekly newsletter</title>
  <from-name>The example company</from-name>
  <from-email>newsletters@example-company.com</from-email>
  <reply-to>admin@example-company.com</reply-to>
  <default-template>weekly_newsletter</default-template>
  <default-theme-id>company</default-theme-id>
  <default-layout-id>weekly</default-layout-id>
  <description>Weekly newsletter sent to all subscribers</description>
  <domain-key-id>jan12-dkim-example-company-com</domain-key-id>
  <default-frontend-layout nil="true"/>
  <frontend-settings-override nil="true"/>
</mailinglist>
```

### Get mailinglist archive
_Get archive url's for newsletters sent on a mailinglist_

* URL: `/api/v1/mailinglists/<mailinglist-id>/archive.json`
* Method: GET
* Required parameters: `<mailinglist-id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist parameters given</td>
    </tr>
    <tr>
      <td>404 Not found</td>
      <td>Mailinglist does not exist</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/archive.json`

Response:

```json
{
  "current_page": 1,
  "total_records": 1,
  "total_pages": 1,
  "next_page": null,
  "newsletters": [
  {
    "id": "weekly-newsletter-weeknumber-20",
    "title": "Weekly newsletter weeknumber 20",
    "token": "iqNwOeFv",
    "subject": "This weeks newsletter",
    "from_email": "newsletters@example-company.com",
    "from_name": "The example company",
    "reply_to": "reply_to@example-company.com",
    "template": "weekly_newsletter",
    "state": "archived",
    "general_state": "closed",
    "sent_at": "2014-06-24 14:07:51 +0200",
    "created_at": "2014-06-24 13:03:14 +0200",
    "archive_urls": [
        {
            "id": "john-smith-at-example-com",
            "email": "john.smith@example.com",
            "url": "http://example-company.peytzmail.com/v/hoswzd/bgmtus/7968434624"
        },
        {
            "id": "jane-doe-at-example-com",
            "email": "jane.doe@example.com",
            "url": "http://example-company.peytzmail.com/v/ftkpgw/qcqebm/4234898672"
        }
    ]
  },
  ...
  ]
}
```
```xml
<newsletters>
  <current-page type="integer">1</current-page>
  <total-records type="integer">1</total-records>
  <total-pages type="integer">1</total-pages>
  <next-page nil="true"/>
  <newsletters type="array">
    <newsletter>
      <id>weekly-newsletter-weeknumber-20</id>
      <title>Weekly newsletter weeknumber 20</title>
      <token>iqNwOeFv</token>
      <subject>This weeks newsletter</subject>
      <from-email>newsletters@example-company.com</from-email>
      <from-name>The example company</from-name>
      <reply-to>reply-to@example.com</reply-to>
      <template>weekly_newsletter</template>
      <state>archived</state>
      <general-state>closed</general-state>
      <sent-at type="datetime">2014-06-24T14:07:51+02:00</sent-at>
      <created-at type="datetime">2014-06-24T13:03:14+02:00</created-at>
      <archive-urls type="array">
        <archive-url>
          <id>john-smith-at-example-com</id>
          <email>john.smith@example.com</email>
          <url>http://example-company.peytzmail.com/v/hoswzd/bgmtus/7968434624</url>
        </archive-url>
        <archive-url>
          <id>jane-doe-at-example-com</id>
          <email>jane.doe@example.com</email>
          <url>http://example-company.peytzmail.com/v/ftkpgw/qcqebm/4234898672</url>
        </archive-url>
      </archive-urls>
    </newsletter>
    ...
  </newsletters>
</newsletters>
```

### Get mailinglist statistics
_Get statistics on a mailinglist_

* URL: `/api/v1/mailinglists/<mailinglist-id>/statistics.json`
* Method: GET
* Required parameters: `<mailinglist-id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist parameters given</td>
    </tr>
    <tr>
      <td>404 Not found</td>
      <td>Mailinglist does not exist</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/statistics.json`

Response:

```json
{
  "current_page": 1,
  "total_records": 1,
  "total_pages": 1,
  "next_page": null,
  "newsletters": [
    {
      "id": "weekly-newsletter",
      "title": "Weekly newsletter",
      "sent_at": "2014-06-24 14:07:51 +0200",
      "sent": "1688",
      "errors": "0",
      "failed": "0",
      "bounced": "25",
      "complaints": "0",
      "opened": "530",
      "clickers": "334",
      "clicks": "385",
      "net_fail_rate": "0",
      "net_bounce_rate": "0.015",
      "net_open_rate": "0.314",
      "net_click_rate": "0.228",
      "net_clickers_rate": "0.198"
    }
  ]
}
```
```xml
<newsletters>
  <current-page type="integer">1</current-page>
  <total-records type="integer">1</total-records>
  <total-pages type="integer">1</total-pages>
  <next-page nil="true"/>
  <newsletters type="array">
    <newsletter>
      <id>weekly-newsletter</id>
      <title>Weekly newsletter</title>
      <sent-at type="datetime">2014-06-24T14:07:51+02:00</sent-at>
      <sent type="integer">1688</sent>
      <errors type="integer">0</errors>
      <failed type="integer">0</failed>
      <bounced type="integer">25</bounced>
      <complaints type="integer">0</complaints>
      <opened type="integer">530</opened>
      <clickers type="integer">334</clickers>
      <clicks type="integer">385</clicks>
      <net-fail-rate type="float">0.0</net-fail-rate>
      <net-bounce-rate type="float">0.015</net-bounce-rate>
      <net-open-rate type="float">0.314</net-open-rate>
      <net-click-rate type="float">0.228</net-click-rate>
      <net-clickers-rate type="float">0.198</net-clickers-rate>
    </newsletter>
  </newsletters>
</newsletters>
```

### Get subscribers in mailinglist
_Get subscribers in a mailinglist_

* URL: `/api/v1/mailinglists/<mailinglist-id>/subscribers.json`
* Method: GET
* Required parameters: `<mailinglist-id>`
* Notes
  * This method makes use of pagination, follow the next page url for more data.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist parameters given</td>
    </tr>
    <tr>
      <td>404 Not found</td>
      <td>Mailinglist does not exist</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/subscribers.json`

Response:

```json
{
  "current_page": 1,
  "total_records": 1,
  "total_pages": 1,
  "next_page": null,
  "subscribers": [
    {
      "subscriber": {
        "id": "john-smith-at-example-com",
        "email": "john.smith@example.com",
        "first_name": "John",
        "last_name": "Smith",
        "foreign_id": "000001",
        "mailinglist_ids": [
          "daily-newsletter",
          "weekly-newsletter"
        ],
        "pending_mailinglist_ids": [
          "hourly-newsletter"
        ],
        "extra_fields": {
          "city": "New York",
          "gender": "Male"
        }
      }
    }
  ]
}
```
```xml
<subscribers>
  <current-page type="integer">1</current-page>
  <total-records type="integer">1</total-records>
  <total-pages type="integer">1</total-pages>
  <next-page nil="true"></next-page>
  <subscribers type="array">
    <subscriber>
      <subscriber>
        <id>john-smith-at-example-com</id>
        <email>john.smith@example.com</email>
        <first-name>John</first-name>
        <last-name>Smith</last-name>
        <foreign-id>000001</foreign-id>
        <mailinglist-ids type="array">
          <mailinglist-id>daily-newsletter</mailinglist-id>
          <mailinglist-id>weekly-newsletter</mailinglist-id>
        </mailinglist-ids>
        <pending-mailinglist-ids type="array">
          <pending-mailinglist-id>daily-newsletter</pending-mailinglist-id>
        </pending-mailinglist-ids>
        <extra-fields>
          <city>New York</city>
          <gender>Male</gender>
        </extra-fields>
      </subscriber>
    </subscriber>
  </subscribers>
</subscribers>
```

### Subscribe to mailinglist(s)
_Add a subscriber to one or more mailinglists_

* URL: `/api/v1/mailinglists/subscribe.json`
* Method: POST
* Required parameters: `email` and `mailinglist_ids` or `mailinglists`
* Optional parameters: `merge_fields`, `skip_confirm`, `skip_welcome`, `first_name`,
`last_name`, `gender`, `city`...
* Notes
  * You can skip confirmation and welcome mails by setting the `skip_confirm`
  and `skip_welcome` options
  * You can add more parameters for storing extra information on the subscriber.
  * You can add tags to your call for tracking subscribes under subscription sources.
  * You can add merge_fields to your call for confirmation and welcome mails.
    * See more about merge_fields under trigger mail calls.
  * Adding a permission will automatically create a subscriber, if one doesn't
  already exist on the specified email address.
  * Setting the optional unsafe query parameter will prevent overwriting
  existing subscriber fields
  * Only specify one of mailinglists or mailinglist_ids

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>Permissions already exist</td>
    </tr>
    <tr>
      <td>200 OK</td>
      <td>No permissions created</td>
    </tr>
    <tr>
      <td>201 Created</td>
      <td>Permissions created</td>
    </tr>
    <tr>
      <td>202 Accepted</td>
      <td>Permissions pending confirmation</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing required parameters</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Permissions failed</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Mailing lists not found</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Failed creating or updating subscriber</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Mail address is blacklisted</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/subscribe.json?unsafe`

_Setting the optional query parameter unsafe is only for cases where you
want to make sure existing subscriber fields aren't overwritten. Only use this
for calls from public available forms, where unauthenticated users are allowed
to post information that can change properties on existing subscribers._

Post data:

```json
{
  "subscribe": {
    "mailinglist_ids": ["daily-newsletter", "weekly-newsletter"],
    "subscriber": {
      "email": "john.smith@example.com",
      "first_name": "John",
      "last_name": "Smith",
      "gender": "Male",
      "city": "New York"
    },
    "merge_fields": {
      "campaign": "Spring 2015",
      "products": ["Daily updates", "Weekly updates"]
    },
    "tags": ["webshop", "campaign-spring-2015"],
    "skip_confirm": true,
    "skip_welcome": false
  }
}
```
```xml
<subscribe>
  <mailinglist-ids type="array">
    <mailinglist-id>daily-newsletter</mailinglist-id>
    <mailinglist-id>weekly-newsletter</mailinglist-id>
  </mailinglist-ids>
  <subscriber>
    <email>john.smith@example.com</email>
    <first-name>John</first-name>
    <last-name>Smith</last-name>
    <gender>Male</gender>
    <city>New York</city>
  </subscriber>
  <merge-fields>
    <campaign>Spring 2015</campaign>
    <products type="Array">
      <product>Daily updates</product>
      <product>Weekly updates</product>
    </products>
  </merge-fields>
  <tags type="array">
    <tag>webshop</tag>
    <tag>campaign-spring-2015</tag>
  </tags>
  <skip-confirm type="boolean">true</skip-confirm>
  <skip-welcome type="boolean">false</skip-welcome>
</subscribe>
```

Response:

```json
{
  "result": "created",
  "message": "Permissions created",
  "subscribes": {
    "existing": [],
    "pending": [],
    "created": [
      {
        "mailinglist_id": "daily-newsletter",
        "permission_id": "507f191e810c19729de860ea"
      },
      {
        "mailinglist_id": "weekly-newsletter",
        "permission_id": "507f1f77bcf86cd799439011"
      }
    ],
    "failed": [],
    "blacklisted": [],
    "not_found": []
  },
  "subscriber": {
    "id": "john-smith-at-example-com",
    "email": "john.smith@example.com",
    "first_name": "John",
    "last_name": "Smith",
    "mailinglist_ids": [
      "daily-newsletter",
      "weekly-newsletter"
    ],
    "pending_mailinglist_ids": [
      "hourly-newsletter"
    ],
    "extra_fields": {
      "city": "New York",
      "gender": "Male"
    }
  }
  "permissions": {
    "daily-newsletter": {
      "_id": "507f191e810c19729de860ea",
      "confirmed_at": "2013-02-01T15:30:00+01:00",
      "confirmed_by": "force",
      "created_at": "2013-02-01T15:30:00+01:00",
      "mailinglist_id": "daily-newsletter",
      "pending_at": null,
      "skip_welcome": null,
      "subscriber_id": "john-smith-at-example-com",
      "updated_at": "2013-02-01T15:30:00+01:00"
    },
    "weekly-newsletter": {
      "_id": "507f1f77bcf86cd799439011",
      "confirmed_at": "2013-02-01T15:30:00+01:00",
      "confirmed_by": "force",
      "created_at": "2013-02-01T15:30:00+01:00",
      "mailinglist_id": "weekly-newsletter",
      "pending_at": null,
      "skip_welcome": null,
      "subscriber_id": "john-smith-at-example-com",
      "updated_at": "2013-02-01T15:30:00+01:00"
    }
  }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<hash>
  <result type="symbol">ok</result>
  <message>Permissions already exists</message>
  <subscribes>
    <existing type="array">
      <existing>
        <mailinglist-id>daily-newsletter</mailinglist-id>
        <permission-id>507f191e810c19729de860ea</permission-id>
      </existing>
      <existing>
        <mailinglist-id>weekly-newsletter</mailinglist-id>
        <permission-id>507f1f77bcf86cd799439011</permission-id>
      </existing>
    </existing>
    <pending type="array"/>
    <created type="array"/>
    <failed type="array"/>
    <blacklisted type="array"/>
    <not-found type="array"/>
  </subscribes>
  <subscriber>
    <id>john-smith-at-example-com</id>
    <email>john.smith@example.com</email>
    <first-name>John</first-name>
    <last-name>Smith</last-name>
    <mailinglist-ids type="array">
      <mailinglist-id>daily-newsletter</mailinglist-id>
      <mailinglist-id>weekly-newsletter</mailinglist-id>
    </mailinglist-ids>
    <pending-mailinglist-ids type="array">
      <pending-mailinglist-id>daily-newsletter</pending-mailinglist-id>
    </pending-mailinglist-ids>
    <extra-fields>
      <city>New York</city>
      <gender>Male</gender>
    </extra-fields>
  </subscriber>
  <permissions>
    <daily-newsletter>
      <id>507f191e810c19729de860ea</id>
      <subscriber-id>john-smith-at-example-com</subscriber-id>
      <mailinglist-id>daily-newsletter</mailinglist-id>
      <pending-at nil="true"/>
      <skip-welcome nil="true"/>
      <confirmed-at type="datetime">2013-02-01T15:30:00+01:00</confirmed-at>
      <confirmed-by type="symbol">force</confirmed-by>
      <created-at type="datetime">2013-02-01T15:30:00+01:00</created-at>
      <updated-at type="datetime">2013-02-01T15:30:00+01:00</updated-at>
    </daily-newsletter>
    <weekly-newsletter>
      <id>507f1f77bcf86cd799439011</id>
      <subscriber-id>john-smith-at-example-com</subscriber-id>
      <mailinglist-id>weekly-newsletter</mailinglist-id>
      <skip-welcome nil="true"/>
      <pending-at nil="true"/>
      <confirmed-at type="datetime">2013-02-01T15:30:00+01:00</confirmed-at>
      <confirmed-by type="symbol">force</confirmed-by>
      <created-at type="datetime">2013-02-01T15:30:00+01:00</created-at>
      <updated-at type="datetime">2013-02-01T15:30:00+01:00</updated-at>
    </weekly-newsletter>
  </permissions>
</hash>
```

#### Example
Request:
`/api/v1/mailinglists/subscribe.json`

_Subscribe to mailinglists with topics._

Post data:

```json
{
  "subscribe": {
    "mailinglists": [
     {
        "id": "daily-newsletter",
        "limit": "1",
        "topic": "sku:1234567_DK",
        "until": ""
      }
    ],
    "subscriber": {
      "email": "john.smith@example.com",
      "first_name": "John",
      "last_name": "Smith",
      "gender": "Male",
      "city": "New York"
    },
    "merge_fields": {
      "campaign": "Spring 2015",
      "products": ["Daily updates", "Weekly updates"]
    },
    "tags": ["webshop", "campaign-spring-2015"],
    "skip_confirm": true,
    "skip_welcome": false
  }
}
```
```xml
<subscribe>
  <mailinglists type="array">
    <mailinglist>
      <id>daily-newsletter</id>
      <limit>1</limit>
      <topic>sku:1234567_DK</topic>
      <until></until>
    </mailinglist>
  </mailinglists>
  <subscriber>
    <email>john.smith@example.com</email>
    <first-name>John</first-name>
    <last-name>Smith</last-name>
    <gender>Male</gender>
    <city>New York</city>
  </subscriber>
  <merge-fields>
    <campaign>Spring 2015</campaign>
    <products type="array">
      <product>Daily updates</product>
      <product>Weekly updates</product>
    </products>
  </merge-fields>
  <tags type="array">
    <tag>webshop</tag>
    <tag>campaign-spring-2015</tag>
  </tags>
  <skip-confirm type="boolean">true</skip-confirm>
  <skip-welcome type="boolean">false</skip-welcome>
</subscribe>
```

Response:

```json
{
  "result": "created",
  "message": "Permissions created",
  "subscribes": {
    "existing": [],
    "pending": [],
    "created": [
      {
        "mailinglist_id": "daily-newsletter",
        "permission_id": "58380c756226c97fed000005",
        "topic": "sku:1234567_DK"
      }
    ],
    "failed": [],
    "blacklisted": [],
    "not_found": []
  },
  "subscriber": {
    "id": "john-smith-at-example-com",
    "email": "john.smith@example.com",
    "first_name": "John",
    "last_name": "Smith",
    "mailinglist_ids": [
      "daily-newsletter",
      "weekly-newsletter"
    ],
    "pending_mailinglist_ids": [
      "hourly-newsletter"
    ],
    "extra_fields": {
      "city": "New York",
      "gender": "Male"
    }
  },
  "permissions": {
    "daily-newsletter": {
      "_id": "507f191e810c19729de860ea",
      "confirmed_at": "2013-02-01T15:30:00+01:00",
      "confirmed_by": "force",
      "created_at": "2013-02-01T15:30:00+01:00",
      "mailinglist_id": "daily-newsletter",
      "pending_at": null,
      "skip_welcome": null,
      "subscriber_id": "john-smith-at-example-com",
      "updated_at": "2013-02-01T15:30:00+01:00"
    }
  }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<hash>
  <result type="symbol">ok</result>
  <message>Permissions created</message>
  <subscribes>
    <created type="array">
      <created type="array">
        <created>
          <mailinglist-id>asdas</mailinglist-id>
          <permission-id>5838110e6226c90eb3000017</permission-id>
          <topic>sku:1234567_DK</topic>
        </created>
      <created>
    <pending type="array"/>
    <created type="array"/>
    <failed type="array"/>
    <blacklisted type="array"/>
    <not-found type="array"/>
  </subscribes>
  <subscriber>
    <id>john-smith-at-example-com</id>
    <email>john.smith@example.com</email>
    <first-name>John</first-name>
    <last-name>Smith</last-name>
    <mailinglist-ids type="array">
      <mailinglist-id>daily-newsletter</mailinglist-id>
      <mailinglist-id>weekly-newsletter</mailinglist-id>
    </mailinglist-ids>
    <pending-mailinglist-ids type="array">
      <pending-mailinglist-id>daily-newsletter</pending-mailinglist-id>
    </pending-mailinglist-ids>
    <extra-fields>
      <city>New York</city>
      <gender>Male</gender>
    </extra-fields>
  </subscriber>
  <permissions>
    <daily-newsletter>
      <id>507f191e810c19729de860ea</id>
      <subscriber-id>john-smith-at-example-com</subscriber-id>
      <mailinglist-id>daily-newsletter</mailinglist-id>
      <pending-at nil="true"/>
      <skip-welcome nil="true"/>
      <confirmed-at type="datetime">2013-02-01T15:30:00+01:00</confirmed-at>
      <confirmed-by type="symbol">force</confirmed-by>
      <created-at type="datetime">2013-02-01T15:30:00+01:00</created-at>
      <updated-at type="datetime">2013-02-01T15:30:00+01:00</updated-at>
    </daily-newsletter>
    <weekly-newsletter>
      <id>507f1f77bcf86cd799439011</id>
      <subscriber-id>john-smith-at-example-com</subscriber-id>
      <mailinglist-id>weekly-newsletter</mailinglist-id>
      <skip-welcome nil="true"/>
      <pending-at nil="true"/>
      <confirmed-at type="datetime">2013-02-01T15:30:00+01:00</confirmed-at>
      <confirmed-by type="symbol">force</confirmed-by>
      <created-at type="datetime">2013-02-01T15:30:00+01:00</created-at>
      <updated-at type="datetime">2013-02-01T15:30:00+01:00</updated-at>
    </weekly-newsletter>
  </permissions>
</hash>
```

### Unsubscribe from mailinglist(s)
_Remove a subscriber from one or more mailinglists_

* URL: `/api/v1/mailinglists/unsubscribe.json`
* Method: POST
* Required parameters: `email` and `mailinglist_ids`
* Optional parameter: `newsletter_token`
* Notes
  * The optional `newsletter_token` parameter is used to track reference to
  which newsletter let to the unsubscribe

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>Deleted permissions</td>
    </tr>
    <tr>
      <td>200 OK</td>
      <td>No permissions deleted</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing required parameters</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Subscriber not found</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Deleting permissions failed</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/unsubscribe.json`

Post data:

```json
{
  "unsubscribe": {
    "mailinglist_ids": ["daily-newsletter", "weekly-newsletter"],
    "email": "john.smith@example.com"
  }
}
```
```xml
<unsubscribe>
  <mailinglist-ids type="array">
    <mailinglist-id>daily-newsletter</mailinglist-id>
    <mailinglist-id>weekly-newsletter</mailinglist-id>
  </mailinglist-ids>
  <email>john.smith@example.com</email>
</unsubscribe>
```

Response:

```json
{
  "result": "ok",
  "message": "Deleted permissions",
  "unsubscribes": {
    "deleted": [
      {
        "mailinglist_id": "daily-newsletter",
      },
      {
        "mailinglist_id": "weekly-newsletter",
      }
    ],
    "failed": []
  },
  "subscriber": {
    "id": "john-smith-at-example-com",
    "email": "john.smith@example.com",
    "first_name": "John",
    "last_name": "Smith",
    "mailinglist_ids": [],
    "pending_mailinglist_ids": [
      "hourly-newsletter"
    ],
    "extra_fields": {
      "city": "New York",
      "gender": "Male"
    }
  }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<hash>
  <result type="symbol">ok</result>
  <message>Deleted permissions</message>
  <unsubscribes>
    <deleted type="array">
      <deleted>
        <mailinglist-id>daily-newsletter</mailinglist-id>
      </deleted>
      <deleted>
        <mailinglist-id>weekly-newsletter</mailinglist-id>
      </deleted>
    </deleted>
    <failed type="array"/>
  </unsubscribes>
  <subscriber>
    <id>john-smith-at-example-com</id>
    <email>john.smith@example.com</email>
    <first-name>John</first-name>
    <last-name>Smith</last-name>
    <mailinglist-ids type="array"/>
    <pending-mailinglist-ids type="array">
      <pending-mailinglist-id>hourly-newsletter</pending-mailinglist-id>
    </pending-mailinglist-ids>
    <extra-fields>
      <city>New York</city>
      <gender>Male</gender>
    </extra-fields>
  </subscriber>
</hash>
```

### Unsubscribe from a mailinglist
_Remove a subscriber from a single mailinglist by the subscriber's id_

* URL: `/api/v1/mailinglists/<mailinglist-id>/subscribers/<subscriber-id>.json`
* Method: DELETE
* Required parameters: `<mailinglist_id>` and `<subscriber_id>`
* Optional parameters: `newsletter_token`
* Notes
  * The optional `newsletter_token` parameter is used to track reference to
  which newsletter let to the unsubscribe

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>Deleted permissions</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Mailinglist or subscriber could not be found</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/subscribers/john-smith-at-example-com.json`
`/api/v1/mailinglists/weekly-newsletter/subscribers/john-smith-at-example-com.json?newsletter_token=aabbcc`

## Newsletter Methods

### Create newsletter
_Creates a new newsletter_

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters.json`
* Method: POST
* Required parameters: `<mailinglist_id>` and `title`
* Optional parameters: `preheader`, `subject`, `from_name`, `from_email`, `template`,
`theme_id`, `layout_id`, `delayed_sending_at`, `group_id`, `topic`,
`skip_duplicate_check`, `merge_fields` and `feeds`
* Notes
  * The `skip_duplicate_check` only has an effect on newsletter templates
  supporting the duplicate check feature
  * You can push data in the same call as you create the newsletter

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>201 Created</td>
      <td>Newsletter was created</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist_id parameters given.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Required newsletter parameter missing or not a hash</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist.</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>Minimum publish interval has been violated.</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Delayed sendout violated minimum takeoff delay. Sendout refused!</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Could not understand the time you set for delayed sending at. Sendout refused!</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>No feeds found for pushed data</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>'name' and either 'data' or 'source' params are required for pushed feed data</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Data are missing</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Data can not be pushed to newsletters that has been sent</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/newsletters.json`

Post data:

```json
{
  "newsletter": {
    "title": "Weekly newsletter weeknumber 20",
    "topic": "sku:12345678_DK",
    "from_name": "The example company",
    "from_email": "newsletters@example-company.com",
    "template": "weekly_newsletter",
    "theme_id": "company-white",
    "layout_id": "2-col-white",
    "delayed_sending_at": "2012-02-15 14:00:00 CET",
    "feeds": [
      {
        "name": "config_feed",
        "data": {
          "send_today": true
        }
      },
      {
        "name": "content_feed",
        "data": {
          "header": "Weekly news - number 20",
          "body": "News of the week"
        }
      }
    ]
  }
}
```
```xml
<newsletter>
  <title>Weekly newsletter weeknumber 20</title>
  <topic>sku:12345678_DK</topic>
  <from-name>The example company</from-name>
  <from-email>newsletters@example-company.com</from-email>
  <template>weekly_newsletter</template>
  <theme-id>company-white</theme-id>
  <layout-id>2-col-white</layout-id>
  <delayed-sending-at>2012-02-15 14:00:00 CET</delayed-sending-at>
  <feeds type="array">
    <feed>
      <name>config_feed</name>
      <data>
        <send-today type="boolean">true</send-today>
      </data>
    </feed>
    <feed>
      <name>content_feed</name>
      <data>
        <header>Weekly news - number 20</header>
        <body>News of the week</body>
      </data>
    </feed>
  </feeds>
</newsletter>
```

### Update newsletter
_Updates an existing newsletter_

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters/<newsletter-id>.json`
* Method: PUT
* Required parameters: `<mailinglist_id>` and `<newsletter-id>`
* _Look under create newsletter for parameters and responses_

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20.json`

Put data:

```json
{
  "newsletter": {
    "delayed_sending_at": "2014-05-15 14:00:00 CET",
    "merge_fields": {
      "main_offer_title": "Free island trip",
      "main_offer_price": "$0"
    }
  }
}
```
```xml
<newsletter>
  <delayed-sending-at>2014-05-15 14:00:00 CET</delayed-sending-at>
  <merge-fields>
    <main-offer-title>Free island trip</main-offer-title>
    <main-offer-price>$0</main-offer-price>
  </merge-fields>
</newsletter>
```

### Push data to newsletter
_Push data to an __existing__ newsletter which is __not__ sent_

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters/<newsletter-id>/push_data.json`
* Method: POST
* Required parameters: `<mailinglist_id>`, `<newsletter_id>`, `feeds[][name]` and either `feeds[][data]`
or `feeds[][source]`
* Optional parameters: `feeds[][data]`, `feeds[][source]`
* Notes
  * The name in `feeds[][name]` should be the feed node tags name.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>Data pushed to newsletter</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist_id parameters given.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No newsletter_id parameter given.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Newsletter does not exist.</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>No feeds found for pushed data</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>'name' and either 'data' or 'source' params are required for pushed feed data</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Data are missing</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Data can not be pushed to newsletters that has been sent</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20/push_data.json`

Post data:

```json
{
  "newsletter": {
    "feeds": [
      {
        "name": "config_feed",
        "data": {
          "send_today": true
        }
      },
      {
        "name": "content_feed",
        "data": {
          "header": "Weekly news - number 20",
          "body": "News of the week"
        }
      }
    ]
  }
}
```
```xml
<newsletter>
  <feeds type="array">
    <feed>
      <name>config_feed</name>
      <data>
        <send-today type="boolean">true</send-today>
      </data>
    </feed>
    <feed>
      <name>content_feed</name>
      <data>
        <header>Weekly news - number 20</header>
        <body>News of the week</body>
      </data>
    </feed>
  </feeds>
</newsletter>
```

### Preview newsletter
_Previews html and/or text parts of a newsletter to the given email address_

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters/<newsletter-id>/preview.<format>`
* Method: GET
* Required parameters: `<mailinglist_id>`, `<newsletter_id>` and `<format>`
* Optional parameters: `email` and `clear_cache`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist_id parameters given.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No newsletter_id parameter given.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Newsletter does not exist.</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Could not preview newsletter</td>
    </tr>
  </tbody>
</table>

#### Examples
* Request returning html and text parts wrapped in a JSON response
  * Using a custom email address and clearing the cache
  * `/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20/preview.json?email=customer@example.com&clear_cache`
* Request returning html and text parts wrapped in an XML response
  * Using a custom email address without clearing cache
  * `/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20/preview.xml?email=customer@example.com`
* Request returning unwrapped html part only
  * Using the mail address of the calling api user without clearing cache
  * `/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20/preview.html`
* Request returning unwrapped text part only
  * Using a custom email address and clearing the cache
  * `/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20/preview.text?email=customer@example.com&clear_cache`

### Test newsletter
_Sends a test newsletter to the given email address_

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters/<newsletter-id>/test.json`
* Method: GET
* Required parameters: `<mailinglist_id>`, `<newsletter_id>` and `email`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>Newsletter queued for testing</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist_id parameters given.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No newsletter_id parameter given.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Email is required</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Newsletter does not exist.</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>ERROR: Test mail could not be sent!</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20/test.json?email=admin@example.com`

### Send newsletter
_Sends a __tested__ newsletter __once__ to all subscribers on the mailinglist_

Before sending the test newsletter, it must be opened and pictures viewed.

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters/<newsletter-id>/send.json`
* Method: GET
* Required parameters: `<mailinglist_id>` and `<newsletter_id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>Newsletter queued for sending</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist_id parameters given.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No newsletter_id parameter given.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Newsletter does not exist.</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>Minimum publish interval has been violated.</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>ERROR: Newsletter could not be queued for sendout! It's been created but not sent.</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20/send.json`

### Create and send newsletter
_Creates and sends a newsletter __without__ testing it_

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters/create_and_send.json`
* Method: POST
* Required parameters: `<mailinglist_id>` and `title`
* Optional parameters: `preheader`, `subject`, `from_name`, `from_email`, `template`,
`theme_id`, `layout_id`, `delayed_sending_at`, `topic`,
`group_id`, `skip_duplicate_check`, `merge_fields` and `feeds[]`
* Notes
  * The `skip_duplicate_check` has only an effect on newsletter templates
  supporting the duplicate check feature
  * You can push data in the same call as you create and send the newsletter

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>201 Created</td>
      <td>Created and queued newsletter for sending</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist_id parameters given.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Required newsletter parameter missing or not a hash</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist.</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>Minimum publish interval has been violated.</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Delayed sendout violated minimum takeoff delay. Sendout refused!</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Could not understand the time you set for delayed sending at. Sendout refused!</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>No feeds found for pushed data</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>'name' and either 'data' or 'source' params are required for pushed feed data</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Data are missing</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Data can not be pushed to newsletters that has been sent</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>ERROR: Newsletter could not be queued for sendout! It's been created but not sent.</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/newsletters/create_and_send.json`

Post data:

```json
{
  "newsletter": {
    "title": "Weekly newsletter weeknumber 20",
    "topic": "sku:12345678_DK",
    "preheader": "News from week 20",
    "subject": "This weeks newsletter",
    "from_name": "The example company",
    "from_email": "newsletters@example-company.com",
    "template": "weekly_newsletter",
    "theme_id": "company-white",
    "layout_id": "2-col-white",
    "delayed_sending_at": "2012-02-15 14:00:00 CET",
    "feeds": [
      {
        "name": "config_feed",
        "data": {
          "send_today": true
        }
      },
      {
        "name": "content_feed",
        "data": {
          "header": "Weekly news - number 20",
          "body": "News of the week"
        }
      }
    ]
  }
}
```
```xml
<newsletter>
  <title>Weekly newsletter weeknumber 20</title>
  <topic>sku:12345678_DK</topic>
  <preheader>News from week 20</preheader>
  <subject>This weeks newsletter</subject>
  <from-name>The example company</from-name>
  <from-email>newsletters@example-company.com</from-email>
  <template>weekly_newsletter</template>
  <theme-id>company-white</theme-id>
  <layout-id>2-col-white</layout-id>
  <delayed-sending-at>2012-02-15 14:00:00 CET</delayed-sending-at>
  <feeds type="array">
    <feed>
      <name>config_feed</name>
      <data>
        <send-today type="boolean">true</send-today>
      </data>
    </feed>
    <feed>
      <name>content_feed</name>
      <data>
        <header>Weekly news - number 20</header>
        <body>News of the week</body>
      </data>
    </feed>
  </feeds>
</newsletter>
```

### Get newsletter
_Finds a newsletter and shows its informations and status_

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters/<newsletter-id>.json`
* Method: GET
* Required parameters: `<mailinglist_id>` and `<newsletter_id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist_id parameters given.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No newsletter_id parameter given.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Newsletter does not exist.</td>
    </tr>
  </tbody>
</table>

#### Response
* Possible values for general state and state:

```
drafts: 'draft', 'testing', 'tested', 'test_sent', 'test_failed'
queued: 'cleared_for_takeoff', 'preparing_data', 'queueing', 'waiting_for_workers', 'paused'
active: 'sending'
failed: 'failed', 'skipped', 'aborted'
closed: 'archived'
```

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/newsletters/weekly-newsletter-weeknumber-20.json`

Response:

```json
{
  "newsletter": {
    "id": "weekly-newsletter-weeknumber-20",
    "title": "Weekly newsletter weeknumber 20",
    "token": "iqNwOeFv",
    "subject": "This weeks newsletter",
    "from_email": "newsletters@example-company.com",
    "from_name": "The example company",
    "reply_to": "reply_to@example-company.com",
    "template": "weekly_newsletter",
    "state": "archived",
    "general_state": "closed",
    "sent": 100,
    "failed": 0,
    "complaint": 0,
    "bounced": 0,
    "opened": 70,
    "clicks": 50,
    "clickers": 35,
    "links": [
      {
        "title": "Get our weekend offer",
        "url": "http://example-company.com/weekend-offer",
        "tags": {},
        "clicks": 50,
        "clickers": 35
      }
    ]
  }
}
```
```xml
<newsletter>
  <id>weekly-newsletter-weeknumber-20</id>
  <title>Weekly newsletter weeknumber 20</title>
  <token>iqNwOeFv</token>
  <subject>This weeks newsletter</subject>
  <from-email>newsletters@example-company.com</from-email>
  <from-name>The example company</from-name>
  <reply-to>reply-to@example.com</reply-to>
  <template>weekly_newsletter</template>
  <state>archived</state>
  <general-state>closed</general-state>
  <sent type="integer">100</sent>
  <failed type="integer">0</failed>
  <complaint type="integer">0</complaint>
  <bounced type="integer">0</bounced>
  <opened type="integer">70</opened>
  <clicks type="integer">50</clicks>
  <clickers type="integer">35</clickers>
  <links type="array">
    <link>
      <title>Get our weekend offer</title>
      <url>http://example-company.com/weekend-offer</url>
      <tags></tags>
      <clicks type="integer">50</clicks>
      <clickers type="integer">35</clickers>
    </link>
  </links>
</newsletter>
```

### Get newsletters
_Gets a paginated list of newsletters_

* URL: `/api/v1/mailinglists/<mailinglist-id>/newsletters.json`
* Method: GET
* Required parameters: `<mailinglist_id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No mailinglist_id parameters given.</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Mailinglist does not exist.</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/mailinglists/weekly-newsletter/newsletters.json`

Response:

```json
{
  "current_page": 1,
  "total_records": 1,
  "total_pages": 1,
  "next_page": null,
  "newsletters": [
  {
    "id": "weekly-newsletter-weeknumber-20",
    "title": "Weekly newsletter weeknumber 20",
    "token": "iqNwOeFv",
    "subject": "This weeks newsletter",
    "from_email": "newsletters@example-company.com",
    "from_name": "The example company",
    "reply_to": "reply_to@example-company.com",
    "template": "weekly_newsletter",
    "state": "archived",
    "general_state": "closed"
  },
  ...
  ]
}
```
```xml
<newsletters>
  <current-page type="integer">1</current-page>
  <total-records type="integer">1</total-records>
  <total-pages type="integer">1</total-pages>
  <next-page nil="true"/>
  <newsletters type="array">
    <newsletter>
      <id>weekly-newsletter-weeknumber-20</id>
      <title>Weekly newsletter weeknumber 20</title>
      <token>iqNwOeFv</token>
      <subject>This weeks newsletter</subject>
      <from-email>newsletters@example-company.com</from-email>
      <from-name>The example company</from-name>
      <reply-to>reply-to@example.com</reply-to>
      <template>weekly_newsletter</template>
      <state>archived</state>
      <general-state>closed</general-state>
    </newsletter>
    ...
  </newsletters>
</newsletters>
```

### Get templates
_Gets a list of templates_

* URL: `/api/v1/templates.json`
* Method: GET
* Required parameters: none

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/templates.json`

Response:

```json
{
"templates": [
  "weekly-newsletter",
  "daily-newsletter",
  ...
]
}
```
```xml
<templates>
  <template>weekly-newsletter</template>
  <template>daily-newsletter</template>
  ...
</templates>
```
## Trigger mail Methods
Triggers mails are individual emails that are sent as soon as they are created
(unless immediate trigger mail processing has been disabled by settings).

### Get trigger mail
_Gets informations about a trigger mail_

* URL: `/api/v1/trigger_mails/<trigger-mail-id>.json`
* Method: GET
* Required parameters: `<trigger-mail-id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No trigger mail parameters given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Trigger mail does not exist</td>
    </tr>
  </tbody>
</table>

#### Example

Response:

```json
{
  "trigger_mail": {
    "id": "502a28e90dc71e8b22000040",
    "created_at": "2012-08-14T12:31:05+02:00",
    "reply_to": "no-reply@exampe-company.com",
    "from_email": "support@exampe-company.com",
    "from_name": "The example company",
    "to_email": "new_user@example.com",
    "to_name": "New User",
    "subject": "There must be a subject!",
    "state": "queued",
    "general_state": "queued",
    "tag": "welcome",
    "tags": ["test"],
    "trigger_mail_category_id": "welcome-mails"
  }
}
```
```xml
<trigger-mail>
  <id>502a28e90dc71e8b22000040</id>
  <created-at>2012-08-14T12:31:05+02:00</created-at>
  <reply-to>no-reply@exampe-company.com</reply-to>
  <from-email>support@exampe-company.com</from-email>
  <from-name>The example company</from-name>
  <to-email>new_user@example.com</to-email>
  <to-name>New User</to-name>
  <blind-copies></blind-copies>
  <subject>There must be a subject!</subject>
  <state>sent</state>
  <general-state>sent</general-state>
  <tag>welcome</tag>
  <tags type="array">
    <tag>test</tag>
  </tags>
  <trigger-mail-category-id>welcome-mails</trigger-mail-category-id>
</trigger-mail>
```

### Get trigger mails by tags
_Gets informations about a trigger mails selected by their tags_

* URL: `/api/v1/trigger_mails/tagged.json?tags[]=test&tags[]=something`
* Method: GET
* Required parameters: `tags`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No trigger mail tags given</td>
    </tr>
  </tbody>
</table>

#### Example

Response:

```json
{
  "current_page": 1,
  "total_records": 4,
  "total_pages": 2,
  "next_page": "http://localhost:3000/api/v1/trigger_mails/tagged.json?page=2&tags%5B%5D=test&tags%5B%5D=something",
  "trigger_mails": [
    {
      "id": "56dd2b20995b77573f000005",
      "created_at": "2016-03-07T08:17:52+01:00",
      "reply_to": "no-reply@exampe-company.com",
      "from_email": "support@exampe-company.com",
      "from_name": "The example company",
      "to_email": "new_user@example.com",
      "to_name": "New User",
      "subject": "Trigger mails med tags",
      "state": "opened",
      "general_state": "closed",
      "is_delayed_sending": false,
      "delayed_sending_at": null,
      "tag": "test",
      "tags": [
        "test",
        "blabla",
        "something"
      ],
      "trigger_mail_category_id": "test"
    },
    {
      "id": "56dd2bb5995b772e1f000009",
      "created_at": "2016-03-07T08:20:21+01:00",
      "reply_to": "no-reply@exampe-company.com",
      "from_email": "support@exampe-company.com",
      "from_name": "The example company",
      "to_email": "new_user@example.com",
      "to_name": "New User",
      "subject": "Trigger mails med tags",
      "state": "opened",
      "general_state": "closed",
      "is_delayed_sending": false,
      "delayed_sending_at": null,
      "tag": "test",
      "tags": [
        "test",
        "blabla",
        "something",
        "else"
      ],
      "trigger_mail_category_id": "test"
    }
  ]
}
```
```xml
<trigger-mails>
  <current-page type="integer">1</current-page>
  <total-records type="integer">4</total-records>
  <total-pages type="integer">2</total-pages>
  <next-page>http://localhost:3000/api/v1/trigger_mails/tagged.json?page=2&tags%5B%5D=test&tags%5B%5D=something</next-page>
  <trigger-mails type="array">
    <trigger-mail>
      <id>56dd2b20995b77573f000005</id>
      <created-at>2016-03-07T08:17:52+01:00</created-at>
      <reply-to>no-reply@exampe-company.com</reply-to>
      <from-email>support@exampe-company.com</from-email>
      <from-name>The example company</from-name>
      <to-email>new_user@example.com</to-email>
      <to-name>New User</to-name>
      <blind-copies></blind-copies>
      <subject>There must be a subject!</subject>
      <state>sent</state>
      <general-state>closed</general-state>
      <tag>welcome</tag>
      <tags type="array">
        <tag>test</tag>
        <tag>blabla</tag>
        <tag>something</tag>
      </tags>
      <trigger-mail-category-id>welcome-mails</trigger-mail-category-id>
    </trigger-mail>
    <trigger-mail>
      <id>56dd2bb5995b772e1f000009</id>
      <created-at>2016-03-07T08:20:21+01:00</created-at>
      <reply-to>no-reply@exampe-company.com</reply-to>
      <from-email>support@exampe-company.com</from-email>
      <from-name>The example company</from-name>
      <to-email>new_user@example.com</to-email>
      <to-name>New User</to-name>
      <blind-copies></blind-copies>
      <subject>There must be a subject!</subject>
      <state>sent</state>
      <general-state>closed</general-state>
      <tag>welcome</tag>
      <tags type="array">
        <tag>test</tag>
        <tag>blabla</tag>
        <tag>something</tag>
        <tag>else</tag>
      </tags>
      <trigger-mail-category-id>welcome-mails</trigger-mail-category-id>
    </trigger-mail>
  </trigger-mails>
</trigger-mails>
```

### Create and send trigger mails
_Creates one or more trigger mails and schedules them for immediate or delayed
delivery_

* URL: `/api/v1/trigger_mails.json`
* Method: POST
* Required parameters: `recipients[]` (depends on the template in use)
* Optional parameters: `preheader`, `subject`, `from_name`, `from_email`, `reply_to`,
`trigger_mail_category_id`, `tag`,
`template`, `theme_id`, `layout_id`, `delayed_sending_at`, `feeds` and `merge_fields`
* Notes
  * Setting the optional unsafe query parameter will prevent overwriting
  existing subscriber fields

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>202 Accepted</td>
      <td>
        The trigger mail request was successfully processed. Mails for valid
        recipients has been enqueued for later processing, but you should still
        check our response for invalid recipients and failed trigger mails.
        You will get this response even if every recipient in your request
        failed!
      </td>
    </tr>
    <tr>
      <td>412 Precondition Failed</td>
      <td>Could not find or create the trigger mail category</td>
    </tr>
    <tr>
      <td>413 Request Entity Too Large</td>
      <td>
        You are sending too many trigger mails per request. Please call us to
        have a talk about what are trigger mails and what are bulk emails.
      </td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>No trigger mail category matches the tag given</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>No trigger mail category matches the id given</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>No trigger mail category for these trigger mails</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Failed creating the trigger mail feed object</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Trigger mail recipients rejected</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>No recipients to send trigger mails to</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Required fields are missing</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>
        Could not understand the time you set for delayed sending at. Trigger
        mail refused!
      </td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Could not create trigger mails. No mails will be sent.</td>
    </tr>
    <tr>
      <td>500 Internal Server Error</td>
      <td>No trigger mails ready for enqueueing</td>
    </tr>
  </tbody>
</table>

#### Why do I get 202 accepted even when every recipient failed and no trigger mails were sent?

If you ask us to send 2 trigger mails in one request, and the first trigger
mail succeeds, but the second trigger mail doesn't, then the expected response
would be an overall success - even though 50% of the trigger mails requested
failed. This means you have to check the contents of our response to see if any
trigger mails failed anyway, and that's why we have chosen not to confuse our
consumers with different responses for requests that fail 100% and those that
fails between 0% and 100%. Check the contents of the create and send trigger
mails response, and you'll be certain to get the right states for the trigger
mails you're sending.

#### Required fields and custom templates

The trigger mail template in use defines the parameters passed with this call.
The example shown below was made for the default trigger mail template. For this
template you should tell the subject of the mail to send, sender details
(`from_name`, `from_email` and `reply_to`) (if you don't want to use the fields
set in the trigger mail category), a trigger mail category tag or id (if you
don't want to use the default trigger mail category set in Peytz Mail settings),
and a list of recipients that should receive the trigger mail. Other custom
developed templates may hold the logic of who to receive the trigger mail (which
means the recipients array would be empty), the subject for the trigger mail,
sender details etc. Refer to the documentation for your trigger mail template
implementations for details of the specific usages.

Merge fields are a way to submit recipients specific variables for merge into
the trigger mails. Merge fields defined under each recipient are recipient
specific, and merge fields defined under the trigger mail are common for all
recipients of that specific trigger mail call. Merge field values are not
written to the subscribers in Peytz Mail. Only fields defined as extra fields
are stored on the subscriber.

#### Notes about `recipients[]`
If `recipients[]` is required and in use by the template, additional fields may
be required on each element.

`to_email` will overwrite any value used for the `email` field. If `to_name` is
set, its value will be used for the `full_name` field, which in turn sets
`first_name` and `last_name`. Defining a value for `to_name` and any of `first_name`
or `last_name` may yield unexpected results.

#### Limits on bulk trigger mail sending

There's a limit of how many trigger mails you can send with one trigger mail
create call. The default setting is 50 recipients per call and there can be a
limit for how many trigger mails you can send per hour or per day. We have these
limits to ensure a fair use of the trigger mail feature. If your implementation
begins to mimick mass mailing, then we recommend you use our mailinglist and
newsletter features instead.

#### Example
_This example uses the default trigger mail template. For further customization
please consult Peytz & Co_

Request:
`/api/v1/trigger_mails.json`

Post data:

```json
{
  "trigger_mail": {
    "preheader": "Here are your order details...",
    "subject": "Thank you for signing up",
    "from_name": "The example company",
    "from_email": "no-reply@example-company.com",
    "tag": "welcome_mail",
    "template": "default",
    "theme_id": "company-default",
    "layout_id": "trigger-default",
    "delayed_sending_at": "2012-02-15 14:00:00 CET",
    "recipients": [
      {
        "foreign_id": "000001",
        "to_name": "New User 1",
        "to_email": "new_user_1@example.com",
        "merge_fields": {
          "order_number": "123456",
          "sale": true,
          "products": [
            "Boat",
            "Oars",
            "Anchor",
            "Life vest"
          ]
        }
      },
      {
        "foreign_id": "000002",
        "to_name": "New User 2",
        "to_email": "new_user_2@example.com",
        "merge_fields": {
          "order_number": "123457",
          "sale": true,
          "products": [
            "Kayak",
            "Paddle",
            "Stabilizer kit",
            "Life vest"
          ]
        }
      },
      {
        "foreign_id": "000003",
        "to_name": "New User 3",
        "to_email": "new_user_3@example.com",
        "merge_fields": {
          "order_number": "123458",
          "sale": false,
          "products": [
            "Canoe",
            "Paddle",
            "Life vest"
          ]
        }
      }
    ],
    "feeds": [
      {
        "name": "content",
        "data": {
          "html": "
            <!DOCTYPE html>
            <html lang=\"en\">
            <head>
              <meta charset=\"UTF-8\" />
            </head>
            <body>
              <h1>Dear {{subscriber.full_name}}</h1>
              <p>Thank you for your order number {{merge_fields.order_number}}.</p>
              <p>Here's a list of the products you bought:
                <ul>
                  {{#merge_fields.products}}
                    <li>{{.}}</li>
                  {{/merge_fields.products}}
                </ul>
              </p>
              {{#merge_fields.sale}}
                <p>Remember our big sale in the period {{merge_fields.sale_start}} to {{merge_fields.sale_end}}!</p>
              {{/merge_fields.sale}}
            </body>
            </html>",
          "text":
            "Dear {{subscriber.full_name}}
            Thank you for your order number {{merge_fields.order_number}}.
            Here's a list of the products you bought:
            {{#merge_fields.products}}
              - {{.}}
            {{/merge_fields.products}}
            {{#merge_fields.sale}}Remember our big sale in the period {{merge_fields.sale_start}} to {{merge_fields.sale_end}}!{{/merge_fields.sale}}"
        }
      }
    ],
    "merge_fields": {
      "sale_start": "2013-12-01",
      "sale_end": "2013-12-20"
    }
  }
}
```
```xml
<trigger-mail>
  <preheader>Here are your order details...</preheader>
  <subject>Thank you for signing up</subject>
  <from-name>The example company</from-name>
  <from-email>no-reply@example-company.com</from-email>
  <tag>welcome_mail</tag>
  <template>default</template>
  <theme-id>company-default</theme-id>
  <layout-id>trigger-default</layout-id>
  <delayed-sending-at>2012-02-15 14:00:00 CET</delayed-sending-at>
  <recipients type="array">
    <recipient>
      <foreign-id>000001</foreign-id>
      <to-name>New User 1</to-name>
      <to-email>new_user_1@example.com</to-email>
      <merge-fields>
        <order-number>123456</order-number>
        <sale type="boolean">true</sale>
        <products type="array">
          <product>Boat</product>
          <product>Oars</product>
          <product>Anchor</product>
          <product>Life vest</product>
        </products>
      </merge-fields>
    </recipient>
    <recipient>
      <foreign-id>000002</foreign-id>
      <to-name>New User 2</to-name>
      <to-email>new_user_2@example.com</to-email>
      <merge-fields>
        <order-number>123457</order-number>
        <sale type="boolean">true</sale>
        <products type="array">
          <product>Kayak</product>
          <product>Paddle</product>
          <product>Stabilizer kit</product>
          <product>Life vest</product>
        </products>
      </merge-fields>
    </recipient>
    <recipient>
      <foreign-id>000003</foreign-id>
      <to-name>New User 3</to-name>
      <to-email>new_user_3@example.com</to-email>
      <merge-fields>
        <order-number>123458</order-number>
        <sale type="boolean">false</sale>
        <products type="array">
          <product>Canoe</product>
          <product>Paddle</product>
          <product>Life vest</product>
        </products>
      </merge-fields>
    </recipient>
  </recipients>
  <feeds type="array">
    <feed>
      <name>content</name>
      <data>
        <html>
          <![CDATA[
            <!DOCTYPE html>
            <html lang="en">
            <head>
              <meta charset="UTF-8" />
            </head>
            <body>
              <h1>Dear {{subscriber.full_name}}</h1>
              <p>Thank you for your order number {{merge_fields.order_number}}.</p>
              <p>Here's a list of the products you bought:
                <ul>
                  {{#merge_fields.products}}
                    <li>{{.}}</li>
                  {{/merge_fields.products}}
                </ul>
              </p>
              {{#merge_fields.sale}}
                <p>Remember our big sale in the period {{merge_fields.sale_start}} to {{merge_fields.sale_end}}!</p>
              {{/merge_fields.sale}}
            </body>
            </html>
          ]]>
        </html>
        <text>
          Dear {{subscriber.full_name}}
          Thank you for your order number {{merge_fields.order_number}}.
          Here's a list of the products you bought:
          {{#merge_fields.products}}
            - {{.}}
          {{/merge_fields.products}}
          {{#merge_fields.sale}}Remember our big sale in the period {{merge_fields.sale_start}} to {{merge_fields.sale_end}}!{{/merge_fields.sale}}
        </text>
      </data>
    </feed>
  </feeds>
  <merge-fields>
    <sale-start>2013-12-01</sale-start>
    <sale-end>2013-12-20</sale-end>
  </merge-fields>
</trigger-mail>
```

Response:

```json
{
  "trigger_mails": [
    {
      "id": "502a28e90dc71e8b22000041",
      "to_email": "new_user_1@example.com",
      "state": "queued"
    },
    {
      "id": "502a28e90dc71e8b22000042",
      "to_email": "new_user_2@example.com",
      "state": "queued"
    },
    {
      "id": null,
      "to_email": "new_user_3@example.c",
      "state": "failed",
      "message": "Failed creating or updating subscriber: Email is not a valid email address (new_user_3@example.c)"
    }
  ]
}
```
```xml
<trigger-mails type="array">
  <trigger-mail>
    <id>502a28e90dc71e8b22000041</id>
    <to-email>new_user_1@example.com</to-email>
    <state>queued</state>
  </trigger-mail>
  <trigger-mail>
    <id>502a28e90dc71e8b22000042</id>
    <to-email>new_user_2@example.com</to-email>
    <state>queued</state>
  </trigger-mail>
  <trigger-mail>
    <id nil="true"/>
    <to-email>new_user_3@example.c</to-email>
    <message>Failed creating or updating subscriber: Email is not a valid email address (new_user_3@example.c)</message>
    <state>failed</state>
  </trigger-mail>
</trigger-mails>
```

#### Simple example
_This is a simple example with fewer fields using the default category and its
trigger mail template and layout. Also we're showing the use of the optional
unsafe query parameter here. Setting unsafe will prevent your calls
from overwriting already set subscriber fields on the subscriber you're
sending trigger mails to - and thus updating in the database._

Request:
`/api/v1/trigger_mails.json?unsafe`

Post data:

```json
{
  "trigger_mail": {
    "subject": "Thank you for signing up",
    "recipients": [
      {
        "to_email": "new_user@example.com",
        "full_name": "Joe Bloggs"
      }
    ]
  }
}
```
```xml
<trigger-mail>
  <subject>Thank you for signing up</subject>
  <recipients type="array">
    <recipient>
      <to-email>new_user@example.com</to-email>
      <full-name>Joe Bloggs</full-name>
    </recipient>
  </recipients>
</trigger-mail>
```

Response:

```json
{
  "trigger_mails": [
    {
      "id": "502a28e90dc71e8b22000040",
      "to_email": "new_user@example.com",
      "state": "queued"
    }
  ]
}
```
```xml
<trigger-mails type="array">
  <trigger-mail>
    <id>502a28e90dc71e8b22000040</id>
    <to-email>new_user@example.com</to-email>
    <state>queued</state>
  </trigger-mail>
</trigger-mails>
```

### Get categories
_Gets a list of categories to be used when sending trigger mails_

* URL: `/api/v1/trigger_mails/categories.json`
* Method: GET

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

## User Methods
### List users
_Lists users with access to Peytz Mail_

* URL: `/api/v1/users.json`
* Method: GET

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/users.json`

Response:

```json
{
  "users": [{
    "id": "4e8b4b716b1e8d6442000001",
    "first_name": "John",
    "last_name": "Doe",
    "initials": "JD",
    "cell_phone": "4512345678",
    "organization": "The example company",
    "email": "jd@example-company.com",
  },
  ...
  ]
}
```
```xml
<users type="array">
  <user>
    <id>4e8b4b716b1e8d6442000001</id>
    <first-name>John</first-name>
    <last-name>Doe</last-name>
    <initials>JD</initials>
    <cell-phone>4512345678</cell-phone>
    <organization>The example company</organization>
    <email>jd@example-company.com</email>
  </user>
  ...
</users>
```

### Show user
_Gets informations about a user_

* URL: `/api/v1/users/<user-id>.json`
* Method: GET
* Required parameters: `<user-id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No user_id parameters given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>User does not exist</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/users/4e8b4b716b1e8d6442000001.json`

Response:

```json
{
  "user": {
    "id": "4e8b4b716b1e8d6442000001",
    "first_name": "John",
    "last_name": "Doe",
    "initials": "JD",
    "cell_phone": "4512345678",
    "organization": "The example company",
    "email": "jd@example-company.com"
  }
}
```
```xml
<user>
  <id>4e8b4b716b1e8d6442000001</id>
  <first-name>John</first-name>
  <last-name>Doe</last-name>
  <initials>JD</initials>
  <cell-phone>4512345678</cell-phone>
  <organization>The example company</organization>
  <email>jd@example-company.com</email>
</user>
```

### Create user
_Creates a new user_

* URL: `/api/v1/user.json`
* Method: POST
* Required parameters: `email`, `initials` and `password`
* Optional parameters: `cell_phone`, `first_name`, `last_name` and
`organization`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>201 Created</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Validation errors</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Missing required parameters</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/user.json`

Post data:

```json
{
  "user": {
    "first_name": "John",
    "last_name": "Doe",
    "initials": "JD",
    "cell_phone": "4512345678",
    "organization": "The example company",
    "email": "jd@example-company.com",
    "password": "Big secret"
  }
}
```
```xml
<user>
  <first-name>John</first-name>
  <last-name>Doe</last-name>
  <initials>JD</initials>
  <cell-phone>4512345678</cell-phone>
  <organization>The example company</organization>
  <email>jd@example-company.com</email>
  <password>Big secret</password>
</user>
```

### Update user
_Updates an existing user_

* URL: `/api/v1/users/<user-id>.json`
* Method: PUT
* Required parameters: `<user-id>`
* Optional parameters: `cell_phone`, `first_name`, `last_name` and `organization`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No user_id parameters given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>User does not exist</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/users/4e8b4b716b1e8d6442000001.json`

Put data:

```json
{
  "user": {
    "cell_phone": "4511111111",
    "organization": "The new example company"
  }
}
```
```xml
<user>
  <cell-phone>4511111111</cell-phone>
  <organization>The new example company</organization>
</user>
```

### Delete user
_Deletes an existing user_

* URL: `/api/v1/users/<user-id>.json`
* Method: DELETE
* Required parameters: `<user-id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No user_id parameters given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>User does not exist</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/users/4e8b4b716b1e8d6442000001.json`

## User Groups Methods
### List user groups
_Lists user groups defined in Peytz Mail_

* URL: `/api/v1/user_groups.json`
* Method: GET

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/user_groups.json`

Response:

```json
{
  "user_groups": [{
    "id": "test-user-group",
    "title": "Test User Group",
    "description": "This is a test user group",
    "user_ids": ["4e8b4b716b1e8d6442000001"],
    "mailinglist_ids": ["test-mailinglist"]
  },
  ...
  ]
}
```
```xml
<user-groups type="array">
  <user-group>
    <id>test-user-group</id>
    <title>Test User Group</title>
    <description>This is a test user group</description>
    <user-ids type="array">
      <user-id>4e8b4b716b1e8d6442000001</user-id>
    </user-ids>
    <mailinglist-ids type="array">
      <mailinglist-id>test-mailinglist</mailinglist-id>
    </mailinglist-ids>
  </user-group>
  ...
</user-groups>
```

### Show user groups
_Gets informations about a user group_

* URL: `/api/v1/user_groups/<user-group-id>.json`
* Method: GET
* Required parameters: `<user_group_id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No user_group_id parameters given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>UserGroup does not exist</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/user_groups/test-user-group.json`

Response:

```json
{
  "user_group": {
    "id": "test-user-group",
    "title": "Test User Group",
    "description": "This is a test user group",
    "user_ids": ["4e8b4b716b1e8d6442000001"],
    "mailinglist_ids": ["test-mailinglist"]
  }
}
```
```xml
<user-group>
  <id>test-user-group</id>
  <title>Test User Group</title>
  <description>This is a test user group</description>
  <user-ids type="array">
    <user-id>4e8b4b716b1e8d6442000001</user-id>
  </user-ids>
  <mailinglist-ids type="array">
    <mailinglist-id>test-mailinglist</mailinglist-id>
  </mailinglist-ids>
</user-group>
```

### Create user group
_Creates a new user group_

* URL: `/api/v1/user_group.json`
* Method: POST
* Required parameters: `title`
* Optional parameters: `id`, `description`, `user_ids[]` and `mailinglist_ids[]`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>201 Created</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No user group parameters given</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Validation errors</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>User group with that id already exists</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/user.json`

Post data:

```json
{
  "user_group": {
    "id": "test-user-group",
    "title": "Test User Group",
    "description": "This is a test user group",
    "user_ids": ["4e8b4b716b1e8d6442000001"],
    "mailinglist_ids": ["test-mailinglist"]
  }
}
```
```xml
<user-group>
  <id>test-user-group</id>
  <title>Test User Group</title>
  <description>This is a test user group</description>
  <user-ids type="array">
    <user-id>4e8b4b716b1e8d6442000001</user-id>
  </user-ids>
  <mailinglist-ids type="array">
    <mailinglist-id>test-mailinglist</mailinglist-id>
  </mailinglist-ids>
</user-group>
```

### Update user group
_Updates an existing user group_

* URL: `/api/v1/user_groups/<user-group-id>.json`
* Method: PUT
* Required parameters: `<user_group_id>`
* Optional parameters: `title`, `description`, `user_ids[]`, `mailinglist_ids[]`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No user_group_id parameters given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>UserGroup does not exist</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/user_groups/test-user-group.json`

Put data:

```json
{
  "user_group": {
    "title": "Test User Group",
    "user_ids": ["4e8b4b716b1e8d6442000001"],
    "mailinglist_ids": ["test-mailinglist"]
  }
}
```
```xml
<user-group>
  <title>Test User Group</title>
  <user-ids type="array">
    <user-id>4e8b4b716b1e8d6442000001</user-id>
  </user-ids>
  <mailinglist-ids type="array">
    <mailinglist-id>test-mailinglist</mailinglist-id>
  </mailinglist-ids>
</user-group>
```

### Add mailinglists to user group
_Adds one or more existing mailinglists to a user group_

With this call you can ask Peytz Mail to create the user group for you, if the
user group doesn't already exist. The title and description fields will be set
automatically from the id field, if the group is created. The http status code
will indicate whether the user group was created or just updated.

* URL: `/api/v1/user_groups/<user-group-id>/add_mailinglists.json`
* Method: PUT
* Required parameters: `<user_group_id>` and `mailinglist_ids[]`
* Optional parameter: `create_if_not_exists`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>An existing user group was updated by adding the mailinglists to the list of mailinglists on the group.</td>
    </tr>
    <tr>
      <td>201 Created</td>
      <td>A new user group was created and the list of mailinglists on the group was set to the mailinglists passed in the call.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Mailinglist does not exist</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing user group or mailinglist parameters</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>UserGroup does not exist</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>A user group with the given id could not be found nor created. Please change your id or try again</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/user_groups/test-user-group/add_mailinglists.json`

Put data:

```json
{
  "user_group": {
    "create_if_not_exists": true,
    "mailinglist_ids": ["test-mailinglist-1", "test-mailinglist-2"]
  }
}
```
```xml
<user-group>
  <create-if-not-exists type="boolean">true</create-if-not-exists>
  <mailinglist-ids type="array">
    <mailinglist-id>test-mailinglist</mailinglist-id>
  </mailinglist-ids>
</user-group>
```

### Remove mailinglists from user group
_Removes one or more existing mailinglists to a user group_

With this call you remove mailinglists from user groups. It doesn't matter if
what you're trying to remove are actually mailinglists or if their present on
the user group, Peytz Mail will respond with an Ok as long as the user group
your trying to remove mailinglists from is a valid and existing user group.

* URL: `/api/v1/user_groups/<user-group-id>/remove_mailinglists.json`
* Method: PUT
* Required parameters: `<user_group_id>` and `mailinglist_ids[]`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>An existing user group was updated by removing the mailinglists from the list of mailinglists on the group.</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No user_group_id parameters given</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing user group or mailinglist parameters</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>UserGroup does not exist</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>A user group with the given id could not be found nor created. Please change your id or try again</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/user_groups/test-user-group/remove_mailinglists.json`

Put data:

```json
{
  "user_group": {
    "mailinglist_ids": ["test-mailinglist-1", "test-mailinglist-2"]
  }
}
```
```xml
<user-group>
  <mailinglist-ids type="array">
    <mailinglist-id>test-mailinglist</mailinglist-id>
  </mailinglist-ids>
</user-group>
```

### Delete user group
_Deletes an existing user group_

* URL: `/api/v1/user_groups/<user-group-id>.json`
* Method: DELETE
* Required parameters: `<user_group_id>`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No user_group_id parameters given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>UserGroup does not exist</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/users/test-user-group.json`

## Content Feeds

Contents for newsletters can be delivered to Peytz Mail automatically using a
content feed. The content feed is served by the customers setup, and pulled by
Peytz Mail immediately before a newsletter sending is commenced.

Content feeds can be delivered in JSON or XML formats. The expected format is
given by the URL we setup Peytz Mail to pull the content feed from.

Every newsletter implemented in the Peytz Mail newsletter system can have its
own custom structure and set of content elements. The example provided below
is purely a hypothetical example. Almost any construction of content elements
can be parsed by Peytz Mail newsletter templates, and the implementation of
your newsletter content feed will be negotiated between the technical staff on
both your side and ours.

For easy integration, choose a standard format in either JSON or XML and keep
the ``nodes > node`` structure and naming. The naming of the fields on each node
is up to you.

### Content Feed Example URL's:

* http://example.com/content/feed.xml (XML format content feed)
* http://example.com/content/feed.json (JSON format content feed)

### Standard Content Feed JSON Example:
```json
{
  "nodes": [
    {
      "node": {
        "title": "Titel på nyhed",
        "body": "Brødtekst",
        "image_src": "http://peytz.dk/sites/peytz.dk/themes/air/logo.png",
        "created": "2013-09-10T03:06Z"
      }
    },
    {
      "node": {
        "title": "Titel på nyhed 2",
        "body": "Brødtekst",
        "image_src": "http://peytz.dk/sites/peytz.dk/themes/air/logo.png",
        "created": "2013-09-10T03:06Z"
      }
    }
  ]
}
```

### Standard Content Feed XML Example:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<nodes>
  <node>
    <title>Titel på nyhed</title>
    <body>Brødtekst</body>
    <image-src>http://peytz.dk/sites/peytz.dk/themes/air/logo.png</image-src>
    <created>2013-09-10T03:06Z</created>
  </node>
  <node>
    <title>Titel på nyhed 2</title>
    <body>Brødtekst</body>
    <image-src>http://peytz.dk/sites/peytz.dk/themes/air/logo.png</image-src>
    <created>2013-09-10T03:06Z</created>
  </node>
</nodes>
```
### Custom Content Feed JSON Example:

```json
{
  "daily_deals": {
    "subject": "Daily Deals for November 21, 2011",
    "title": "Get the best deals for November 21 right here!",
    "sections": {
      "center": {
        "deals": [
          {
            "header": "Free Spa Night",
            "body": "Get your free spa night today!",
            "link": {
              "text": "Click here for your free spa night",
              "url": "http://example.com/free_spa_night.html"
            }
          },
          {
            "header": "Lorem ipsum dolor sit amet",
            "body": "...consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.",
            "link": {
              "text": "Click here for some lorem ipsum",
              "url": "http://example.com/lorem_ipsum.html"
            }
          },
          {
            "header": "Lorem ipsum BUT NO amet",
            "body": "...consectetur adipisicing elit... Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.",
            "link": {
              "text": "You get NO lorem ipsum by clicking here",
              "url": "http://example.com/no_lorem_ipsum.html"
            }
          }
        ]
      },
      "right": {
        "banners": [
          {
            "header": "Free Spa Night",
            "body": "Get your free spa night today!",
            "image": "http://example.com/free_spa_night_small.jpg",
            "link": {
              "text": "Click here for your free spa night",
              "url": "http://example.com/free_spa_night.html"
            }
          },
          {
            "header": "Refer your friends",
            "body": "Get better offers the more friends you refer to our daily deals!",
            "image": "http://example.com/friends.jpg",
            "link": {
              "text": "Click here to refer friends",
              "url": "http://example.com/referals.html"
            }
          }
        ]
      },
      "footer": {
        "text": "Look forward to getting more daily deals tomorrow",
        "link": {
          "text": "Sneak Peak at Tomorrows Deals HERE!",
          "link": "http://example.com/tomorrows_deals.html"
        }
      }
    }
  }
}
```

## Feeds

Feeds can hold any kind of data used for updating subscribers, mailing lists or
any other data kept in the newsletter system, served by remote systems the
newsletter system. Feeds can either be pushed or pulled:

* Pushed - a remote system initiates a feed push and delivers data to Peytz Mail
* Pulled - Peytz Mail initiates the retreival of remotely generated data

### Pushing feed files to Peytz Mail

When pushing large amounts of data to Peytz Mail, it can be necessary to push
a compressed archive containing the feed file, instead of just posting the data
as structured information as a part of the push request. This can be done by
doing an HTTP POST file upload to the restful api feeds push method. The feed
has to be defined in the Peytz Mail administration system beforehand, and the
feed id to be known by the remote system initiating the feed push.

### API endpoint URL
* Subscriber feeds: `https://{CUSTOMER_NAME}.peytzmail.com/api/v1/subscriber_feeds/{FEED_ID}.json`
* Content feeds: `https://{CUSTOMER_NAME}.peytzmail.com/api/v1/content_feeds/{FEED_ID}.json`

__Deprecation warning__: `https://{CUSTOMER_NAME}.peytzmail.com/api/v1/feeds/{FEED_ID}/push.json`
will be deprecated at some point in the future.

### Pushing feed file examples

__Curl:__

```bash
curl -X POST -F file=@feedfile.zip -u {API_KEY} https://{CUSTOMER_NAME}.peytzmail.com/api/v1/subscriber_feeds/{FEED_ID}.xml
```
__Windows Powershell:__

```powershell
$file = 'C:\Users\Peytz\feedfile.zip'
$api_key = '{API_KEY}'
$url = 'https://{CUSTOMER_NAME}.peytzmail.com/api/v1/feeds/{FEED_ID}/push.xml'
$webclient = new-object System.Net.WebClient
$credCache = new-object System.Net.CredentialCache
$creds = new-object System.Net.NetworkCredential($api_key, '')
$credCache.Add($url, "Basic", $creds)
$webclient.Credentials = $credCache
$webclient.UploadFile($url, $file)
```

### Combining multiple content feeds

The combined parsed output of multiple feeds can be retrieved at the following
URL:

```bash
https://{CUSTOMER_NAME}.peytzmail.com/api/v1/feeds/combine_feeds.json?feed_ids[]=feed-id-1&feed_ids[]=feed-id-2
```

The parameter ``feed_ids[]`` should contain the feeds IDs in the order of which
they should be combined.

The output is a standard feed in JSON or XML format. The keys for each node may
vary since they can be the product of different feed parsers.

The API endpoint URL can be used as the URI for a feed definition, but requires
authentication to be configured, e.g. API key as username in Basic Auth.

## Subscriptions

### Quick subscribe
Quick subscribe is a non-authenticated end point used for making subscriptions. The functionality is not enabled by default.

* URL: `/subscribe/<mailinglist-id>/quick`
* Method: GET and POST
* Required parameters: `email` and `mailinglist-id`
* Optional parameters: `first_name`, `last_name`, ...

The end point allows [Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) by setting the `Access-Control-Allow-Origin: *` header. This allows you to POST using an asynchronous Javascript call. Be aware of required browser support.

#### Redirects

It is possible to control the redirect flow after a form submission by adding the following parameters:

* `redirect[success]`: the URL which the user is redirected to on a successful subscription
* `redirect[error]`: the URL which the user is redirected to in the event of an error

#### Form example

```html
<form method="post" action="http://{CUSTOMER_NAME}.peytzmail.com/subscribe/weekly-newsletter/quick">
  <input type="hidden" name="redirect[success]" value="http://example.com/newsletter/success" />
  <input type="hidden" name="redirect[error]" value="http://example.com/newsletter/error" />

  Email: <input type="text" name="email" id="email" />

  <input name="button" type="submit" id="button" value="Subscribe" />
</form>
```

## Subscriber fields
### List subscriber fields
_Gets a list of subscriber fields_

* URL: `/api/v1/subscriber_fields.json`
* Method: GET
* Notes
  * The attribute mandatory_field is only included for mandatory fields (email, foreign_id, first_name, last_name, full_name).
  * The attributes min and max is only included when the field type is relevant for min max settings (string, password, text, integer, float, decimal, range, datetime, date, time).
  * The attribute selection_list is only included when the field type is relevant for selection_list settings (select, radio_buttons, check_boxes).

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
```
/api/v1/subscriber_fields.json
```

Response:

```
{
  "subscriber_fields": [
    {
      "id": "email",
      "label": "Email",
      "description": null,
      "hint": null,
      "placeholder": null,
      "field_type": "email",
      "max": null,
      "min": null,
      "position": 1,
      "tracked": true,
      "required": true,
      "default_value": null,
      "segmentable": true,
      "backend_editable": true,
      "frontend_editable": true,
      "frontend_viewable": true,
      "mandatory_field": true
    },
    {
      "id": "foreign_id",
      "label": "Foreign ID",
      "description": null,
      "hint": null,
      "placeholder": null,
      "field_type": "string",
      "max": null,
      "min": null,
      "position": 2,
      "tracked": true,
      "required": false,
      "default_value": null,
      "segmentable": false,
      "backend_editable": false,
      "frontend_editable": false,
      "frontend_viewable": false,
      "mandatory_field": true
    },
    {
      "id": "first_name",
      "label": "First name",
      "description": null,
      "hint": null,
      "placeholder": null,
      "field_type": "string",
      "max": null,
      "min": null,
      "position": 3,
      "tracked": true,
      "required": false,
      "default_value": null,
      "segmentable": false,
      "backend_editable": true,
      "frontend_editable": true,
      "frontend_viewable": true,
      "mandatory_field": true
    },
    {
      "id": "last_name",
      "label": "Last name",
      "description": null,
      "hint": null,
      "placeholder": null,
      "field_type": "string",
      "max": null,
      "min": null,
      "position": 4,
      "tracked": true,
      "required": false,
      "default_value": null,
      "segmentable": false,
      "backend_editable": true,
      "frontend_editable": true,
      "frontend_viewable": true,
      "mandatory_field": true
    },
    {
      "id": "interests",
      "label": "I'm interested in",
      "description": null,
      "hint": null,
      "placeholder": null,
      "field_type": "check_boxes",
      "position": 5,
      "tracked": true,
      "required": false,
      "default_value": null,
      "selection_list": [
        { "key": "al", "value": "Alligators" },
        { "key": "bi", "value": "Birds" },
        { "key": "ca", "value": "Cats" },
        { "key": "do", "value": "Dogs" },
        { "key": "el", "value": "Elephants" },
        { "key": "ho", "value": "Horses" },
        { "key": "mo", "value": "Monkeys" },
        { "key": "ze", "value": "Zebras" }
      ],
      "segmentable": true,
      "backend_editable": true,
      "frontend_editable": true,
      "frontend_viewable": true
    }
  ]
}
```

### Get subscriber field
_Gets a single subscriber field from it's id_

* URL: `/api/v1/subscriber_fields/<subscriber_field_id>.json`
* Method: GET
* Required parameters:
  * `subscriber_field_id`

`subscriber_field_id` can be thought of as the field name for the subscriber field.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No subscriber_field_id parameter given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>SubscriberField does not exist</td>
    </tr>
  </tbody>
</table>

#### Example 1
Request:
```
/api/v1/subscriber_fields/first_name.json
```

Response:

```
{
  "subscriber_field": {
    "id": "first_name",
    "label": "First name",
    "description": null,
    "hint": null,
    "placeholder": null,
    "field_type": "string",
    "max": null,
    "min": null,
    "position": 3,
    "tracked": true,
    "required": false,
    "default_value": null,
    "segmentable": false,
    "backend_editable": true,
    "frontend_editable": true,
    "frontend_viewable": true,
    "mandatory_field": true
  }
}
```

### Create subscriber field
_Creates a new subscriber field_

* URL: `/api/v1/subscriber_fields.json`
* Method: POST
* Required parameters:
  * `label`
  * `field_type`
* Optional parameters:
  * `id`
  * `description`
  * `hint`
  * `placeholder`
  * `max`
  * `min`
  * `required`
  * `default_value`
  * `selection_list`
  * `segmentable`
  * `backend_editable`
  * `frontend_editable`
  * `frontend_viewable`
  * `position`
  * `tracked`
* Possible values for `field_type`
  * boolean
  * string
  * email
  * url
  * tel
  * password
  * text
  * file
  * hidden
  * integer
  * float
  * decimal
  * range
  * datetime
  * date
  * time
  * select
  * radio_buttons
  * check_boxes
  * country
  * time_zone

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>201 Created</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No subscriber field parameters given</td>
    </tr>
    <tr>
      <td>409 Conflict</td>
      <td>Subscriber field with that id already exists</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/subscriber_fields.json`

Post data:

```json
{
  "subscriber_field": {
    "id": "interests",
    "label": "I'm interested in",
    "field_type": "check_boxes",
    "position": 10,
    "selection_list": [
      { "key": "al", "value": "Alligators" },
      { "key": "bi", "value": "Birds" },
      { "key": "ca", "value": "Cats" },
      { "key": "do", "value": "Dogs" },
      { "key": "el", "value": "Elephants" },
      { "key": "ho", "value": "Horses" },
      { "key": "mo", "value": "Monkeys" },
      { "key": "ze", "value": "Zebras" }
    ],
    "segmentable": true,
    "backend_editable": true,
    "frontend_viewable": true,
    "frontend_editable": true
  }
}
```
```xml
<subscriber-field>
  <id>interests</id>
  <label>I'm interested in</label>
  <field-type>check_boxes</field-type>
  <position>10</position>
  <selection-list type="array">
    <selection-list>
      <key>al</key>
      <value>Alligators</value>
    </selection-list>
    <selection-list>
      <key>bi</key>
      <value>Birds</value>
    </selection-list>
    <selection-list>
      <key>ca</key>
      <value>Cats</value>
    </selection-list>
    <selection-list>
      <key>do</key>
      <value>Dogs</value>
    </selection-list>
    <selection-list>
      <key>el</key>
      <value>Elephants</value>
    </selection-list>
    <selection-list>
      <key>ho</key>
      <value>Horses</value>
    </selection-list>
    <selection-list>
      <key>mo</key>
      <value>Monkeys</value>
    </selection-list>
    <selection-list>
      <key>ze</key>
      <value>Zebras</value>
    </selection-list>
  </selection-list>
  <segmentable type="boolean">true</segmentable>
  <backend-editable type="boolean">true</backend-editable>
  <frontend-viewable type="boolean">true</frontend-viewable>
  <frontend-editable type="boolean">true</backend-editable>
</subscriber-field>
```

### Create or update subscriber field
_Updates an existing subscriber field by searching on the given id, and if not found creates a newly persisted one._

* URL: `/api/v1/subscriber_fields/<subscriber-field-id>/upsert.json`
* Method: PUT
* Required parameters:
  * `<subscriber_field_id>`
* Other parameters:
  * _Look under create subscriber field_

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No subscriber field parameters given</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/subscriber_fields/interests/upsert.json`

Put data:
_Look under create subscriber field_

### Update subscriber field
_Updates an existing subscriber field_

* URL: `/api/v1/subscriber_fields/<subscriber-field-id>.json`
* Method: PUT
* Required parameters:
  * `<subscriber_field_id>`
* Other parameters:
  * _Look under create subscriber field_

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No subscriber_field_id parameter given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Subscriber field does not exist</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/subscriber_fields/interests.json`

Put data:

```json
{
  "subscriber_field": {
    "selection_list": [
      { "key": "al", "value": "Alligators" },
      { "key": "bi", "value": "Birds" },
      { "key": "ca", "value": "Cats" },
      { "key": "do", "value": "Dogs" },
      { "key": "el", "value": "Elephants" },
      { "key": "fe", "value": "Ferrets" },
      { "key": "gi", "value": "Girafs" },
      { "key": "ho", "value": "Horses" },
      { "key": "ig", "value": "Iguanas" },
      { "key": "ja", "value": "Jackals" },
      { "key": "ka", "value": "Kangaroos" },
      { "key": "la", "value": "Ladybirds" },
      { "key": "mo", "value": "Monkeys" },
      { "key": "nu", "value": "Numbats" },
      { "key": "ze", "value": "Zebras" }
    ],
  }
}
```
```xml
<subscriber-field>
  <selection-list type="array">
    <selection-list>
      <key>al</key>
      <value>Alligators</value>
    </selection-list>
    <selection-list>
      <key>bi</key>
      <value>Birds</value>
    </selection-list>
    <selection-list>
      <key>ca</key>
      <value>Cats</value>
    </selection-list>
    <selection-list>
      <key>do</key>
      <value>Dogs</value>
    </selection-list>
    <selection-list>
      <key>el</key>
      <value>Elephants</value>
    </selection-list>
    <selection-list>
      <key>fe</key>
      <value>Ferrets</value>
    </selection-list>
    <selection-list>
      <key>gi</key>
      <value>Girafs</value>
    </selection-list>
    <selection-list>
      <key>ho</key>
      <value>Horses</value>
    </selection-list>
    <selection-list>
      <key>ig</key>
      <value>Iguanas</value>
    </selection-list>
    <selection-list>
      <key>ja</key>
      <value>Jackals</value>
    </selection-list>
    <selection-list>
      <key>ka</key>
      <value>Kangaroos</value>
    </selection-list>
    <selection-list>
      <key>la</key>
      <value>Ladybirds</value>
    </selection-list>
    <selection-list>
      <key>mo</key>
      <value>Monkeys</value>
    </selection-list>
    <selection-list>
      <key>nu</key>
      <value>Numbats</value>
    </selection-list>
    <selection-list>
      <key>ze</key>
      <value>Zebras</value>
    </selection-list>
  </selection-list>
</subscriber-field>
```

## Subscriber events
### Get a list of events related to a mailing list
_Gets a paginated list of subscriber events_

* URL: `/api/v1/subscriber_events/mailinglists/<id>.json`
* Method: GET
* Required parameters: `<id>`
* Optional parameters:
  * `changes[]`
  * `period_start`
  * `period_end`
  * `from_id`
  * `sort_order`
  * `per_page`

`changes[]` can be any of the following values: `subscribe`, `unsubscribe`, `click` or `open`.

`from_id` can be a `event_id` value. `event_id`s are strictly monotonically increasing and setting `from_id` will only return tracks with an id strictly greater than. `from_id` will override `period_start` and `period_end` if set.

`sort_order` can be `desc` or `asc` (default). Be aware of pagination issues when sorting descending.

`per_page` can be an integer up to a global max limit.

The `extra_fields` output might contain different content based on `change`.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing id</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing/invalid arguments</td>
    </tr>
  </tbody>
</table>

#### Example 1
Request:
```
/api/v1/subscriber_events/mailinglists/daily-deals.json?changes[]=unsubscribe&changes[]=subscribe
```

Response:

```
{
  "events": [
    {
      "change": "unsubscribe",
      "created": "2013-10-22T14:12:47+02:00",
      "id": "52666bbf2ceb657174000001",
      "source": "backend",
      "subscriber_token": "qweghj",
      "extra_fields": {
        "foreign_id": "550e8400-e29b-41d4-a716-446655440000",
      }
    },
    {
      "change": "subscribe",
      "created": "2013-10-22T14:09:47+02:00",
      "id": "52666b0b2ceb65c453000001",
      "source": "frontend",
      "subscriber_token": "qweghj",
      "extra_fields": {
        "email": "john@example.org",
        "some_other_field": "123123",
        "zip": "2000"
      }
    }
  ],
  "current_page": 1,
  "next_page": null,
  "total_pages": 1,
  "total_records": 2
}
```

### Get a list of events related to subscribers
_Gets a paginated list of subscriber events_

* URL: `/api/v1/subscriber_events.json`
* Method: GET
* Optional parameters:
  * `changes[]`
  * `period_start`
  * `period_end`
  * `from_id`
  * `sort_order`
  * `per_page`

`changes[]` can be any of the following values: `update` or `create`.

`from_id` can be a `event_id` value. `event_id`s are strictly monotonically increasing and setting `from_id` will only return tracks with an id strictly greater than. `from_id` will override `period_start` and `period_end` if set.

`sort_order` can be `desc` or `asc` (default). Be aware of pagination issues when sorting descending.

`per_page` can be an integer up to a global max limit.

The `extra_fields` output might contain different content based on `change`.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing/invalid arguments</td>
    </tr>
  </tbody>
</table>

#### Example 1
Request:
```
/api/v1/subscriber_events.json?changes[]=update
```

Response:

```
{
  "events": [
    {
      "change": "update",
      "created": "2014-02-17T15:34:16+01:00",
      "event_id": "53021de8f59fc10390000001",
      "extra_fields": {
        "email": "john@example.org",
        "first_name": "Kristoffer"
      },
      "extra_fields_were": {
        "first_name": "Christoffer"
      },
      "source": "backend",
      "subscriber_token": "eaeihv"
    }
  ],
  "current_page": 1,
  "next_page": null,
  "total_pages": 1,
  "total_records": 1
}
```

## Tracking

### Get a list of events
_Gets a paginated list of tracking events_

* URL: `/api/v1/tracks/<event>.json`
* Method: GET
* Required parameters: `<event>` and `mailinglist_id`
* Optional parameters:
  * `types[]` if `unsubscribes` or `subscribes`
  * `newsletter_ids[]` if `clicks` or `opens`
  * `period_start`
  * `period_end`
  * `from_id`

`<event>` can be any of the following: `unsubscribes`, `subscribes`, `received`, `failed`, `clicks`, or `opens`.

The possible values for `types[]` depends on `<event>`:

* `unsubscribes`:
  * `backend`, `blacklist`, `complaint`, `frontend`, `soft`, `hard`, `profile`, `api`
* `subscribes`:
  * `backend`, `confirm`, `frontend`, `profile`, `api`

`from_id` can be a `track_id` value. `track_id`s are strictly monotonically increasing and setting `from_id` will only return tracks with an id strictly greater than. `from_id` will override `period_start` and `period_end` if set.

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing event parameter</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing mailinglist_id</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>Missing/invalid arguments</td>
    </tr>
  </tbody>
</table>

#### Example 1
Request:
```
/api/v1/tracks/unsubscribes.json?mailinglist_id=daily_deals&types[]=frontend&types[]=complaint
```

Response:

```
{
  "current_page": 1,
  "total_records": 2,
  "total_pages": 1,
  "next_page": null,
  "unsubscribes": [
    {
      "created": "2013-10-17T14:47:08+02:00",
      "email": "john.smith@example.com",
      "track_id": "525fdc4c2ceb65693c00008a",
      "foreign_id": "",
      "id": "john-smith-example-com",
      "type": "complaint"
    },
    {
      "created": "2013-10-17T13:41:58+02:00",
      "email": "jane.smith@example.com",
      "track_id": "525fdc4c2ceb65693c00008a",
      "foreign_id": "",
      "subscriber_id": "jane-smith-example-com",
      "type": "frontend"
    }
  ]
}
```

#### Example 2
Request:
```
/api/v1/tracks/subscribes.json?mailinglist_id=daily_deals&types[]=frontend&types[]=profile
```

Response:

```
{
  "current_page": 1,
  "total_records": 2,
  "total_pages": 1,
  "next_page": null,
  "subscribes": [
    {
      "created": "2013-10-17T14:47:08+02:00",
      "email": "john.smith@example.com",
      "track_id": "525fdc4c2ceb65693c00008a",
      "foreign_id": "",
      "subscriber_id": "john-smith-example-com",
      "type": "frontend"
    },
    {
      "created": "2013-10-17T13:41:58+02:00",
      "email": "jane.smith@example.com",
      "track_id": "525fdc4c2ceb65693c00008a",
      "foreign_id": "",
      "subscriber_id": "jane-smith-example-com",
      "type": "profile"
    }
  ]
}
```

#### Example 2
Request:
```
/api/v1/tracks/clicks.json?mailinglist_id=daily_deals&newsletter_ids[]=my-newsletter
```

Response:

```
{
  "current_page": 1,
  "total_records": 2,
  "total_pages": 1,
  "next_page": null,
  "clicks": [
    {
      "created": "2013-10-17T14:47:08+02:00",
      "email": "john.smith@example.com",
      "track_id": "525fdc4c2ceb65693c00008a",
      "foreign_id": "",
      "subscriber_id": "john-smith-example-com",
      "newsletter_id": "my-newsletter"
    },
    {
      "created": "2013-10-17T13:41:58+02:00",
      "email": "jane.smith@example.com",
      "track_id": "525fdc4c2ceb65693c00008a",
      "foreign_id": "",
      "subscriber_id": "jane-smith-example-com",
      "newsletter_id": "my-newsletter"
    }
  ]
}
```

## Settings

### Get domain keys
_Gets domain keys_

* URL: `/api/v1/domain_keys.json`
* Method: GET
* Required parameters: none

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
`/api/v1/domain_keys.json`

Response:

```json
{
  "domain_keys": [
    {
      "id": "test-domainkey-dk",
      "selector": "test",
      "domain": "peytz.dk",
      "description": "",
      "active": false,
      "dns_state": "inactive",
      "system_default": true
    }
  ]
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<domain-keys type="array">
  <domain-key>
    <id>test-domainkey-dk</id>
    <selector>test</selector>
    <domain>peytz.dk</domain>
    <description></description>
    <active type="boolean">false</active>
    <dns-state>inactive</dns-state>
    <system-default type="boolean">true</system-default>
  </domain-key>
</domain-keys>
```

## Unsubscribe callback
If configured, Peytz Mail will initiate an unsubscribe callback every time a subscriber is unsubscribed. Any unsubscription regardless of method will trigger a callback, except if configured to filter based on the reason parameter.

The callback is a HTTP request and supports BasicAuth and HTTPS.

### HTTP methods
When the callback is triggered, a HTTP request will be sent using one of the following methods:

* POST form
  - The parameters will be sent as `application/x-www-form-urlencoded`
* POST
  - The body will contain the parameters as a JSON or XML document
* GET
  - The parameters will be used as query parameters in the URL

### Parameters
The following parameters will be sent as part of the request:

* `action`, e.g. `"unsubscribe"`
* `email`
* `mailinglist_id`
* `reason`, e.g. `"api"`
* `subscriber_foreign_id`
* `subscriber_id`
* `time`, e.g. `"2013-09-20T08:02:19+00:00"` (in ISO8601)

and

* `cs` (checksum)

### Checksum
You can validate the callback by calculating a checksum and checking whether it corresponds to the parameters.
Upon configuration of the callback we will agree on a checksum algorithm and a shared secret.

Possible algorithms: SHA2, SHA1, MD5 and CRC32

The checksum is then calculated by concatenating the parameters (sorted by parameter name ascending) and the shared secret.

Ruby example:

```ruby
require "digest/sha2"
action = "unsubscribe"
email = "joe@example.org"
mailinglist_id = "my-mailinglist"
reason = "api"
subscriber_foreign_id = "1188"
subscriber_id = "joe-at-example-org"
time = "2013-09-20T08:02:19+00:00"

secret = "very secret and should be changed before deploy!"

cstr = action + email + mailinglist_id + reason + subscriber_foreign_id + subscriber_id + time + secret
Digest::SHA2.hexdigest(cstr)
=> "e2723f04fc13cbd231a7ea77b0253c14c4d97ca1e7f0e77c5efca09c50e7bfd5"
```

### Reason
The unsubscribe reason can be one of the following codes:

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>Code</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>api</td>
      <td>Via our API</td>
    </tr>
    <tr>
      <td>backend</td>
      <td>Via our backend interface</td>
    </tr>
    <tr>
      <td>blacklist</td>
      <td>The user has been blacklisted</td>
    </tr>
    <tr>
      <td>complaint</td>
      <td>Via a complaint feedback loop</td>
    </tr>
    <tr>
      <td>event_action</td>
      <td>Via internal Peytz Mail event notification</td>
    </tr>
    <tr>
      <td>feed</td>
      <td>Via subscriber update feed</td>
    </tr>
    <tr>
      <td>import</td>
      <td>Via import</td>
    </tr>
    <tr>
      <td>inactivity</td>
      <td>Due to inactivity</td>
    </tr>
    <tr>
      <td>soft</td>
      <td>Due to a bounce</td>
    </tr>
    <tr>
      <td>unsubscribe</td>
      <td>Via a Peytz Mail hosted unsubscribe page</td>
    </tr>
  </tbody>
</table>

## Attachments

### List attachments
_Gets a list of attachments_

* URL: `/api/v1/attachments.json`
* Method: GET

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
```
/api/v1/attachments.json
```

Response:

```
{
  "attachments": [
    {
      "hosted_url": "http://...",
      "public_url": "http://...",
      "title": "This is my title",
      "updated_at": "2015-10-08T12:18:43+02:00",
      "created_at": "2015-10-08T12:18:43+02:00",
      "display_name": "attachment.pdf",
      "id": "attachment-id-568",
      "tags": ["tag1", "tag2"]
    },
    {
      "hosted_url": "http://...",
      "public_url": "http://...",
      "title": "This is another title",
      "updated_at": "2015-10-08T12:29:13+02:00",
      "created_at": "2015-10-08T12:29:13+02:00",
      "display_name": "attachment.pdf",
      "id": "attachment-id-569",
      "tags": ["tag1"]
    }
  ]
}
```

### Get attachment
_Gets a single attachment by its id_

* URL: `/api/v1/attachments/<attachment_id>.json`
* Method: GET
* Required parameters:
  * `attachment_id`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200 OK</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No attachment_id parameter given</td>
    </tr>
    <tr>
      <td>404 Not Found</td>
      <td>Attachment does not exist</td>
    </tr>
  </tbody>
</table>

#### Example
Request:
```
/api/v1/attachments/1.json
```

Response:

```
{
  "attachment": {
    "hosted_url": "http://...",
    "public_url": "http://...",
    "title": "This is my title",
    "updated_at": "2015-10-08T12:18:43+02:00",
    "created_at": "2015-10-08T12:18:43+02:00",
    "display_name": "attachment.pdf",
    "id": "attachment-id-568",
    "tags": ["tag1", "tag2"]
  }
}
```

### Create attachment
_Creates a new attachment_

* URL: `/api/v1/attachments.json`
* Method: POST
* Required parameters:
  * A file, either by remote URL or by multipart body
* Optional parameters:
  * `title`
  * `display_name`
  * `tags`

<table border="0" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th>HTTP response</th>
      <th>Reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>201 Created</td>
      <td>-</td>
    </tr>
    <tr>
      <td>400 Bad Request</td>
      <td>No attachment parameters given</td>
    </tr>
    <tr>
      <td>422 Unprocessable Entity</td>
      <td>Validation errors</td>
    </tr>
  </tbody>
</table>

#### Example with a remote file
Request:
```
/api/v1/attachments.json
```

Post data:

```
{
  "attachment": {
    "title": "This is my title",
    "remote_file_url": "http://example.org/image.png",
    "tags": ["tag1", "tag2"]
  }
}
```

#### Example with a pushed file
Request:
```
/api/v1/attachments.json
```

Post data should be a multipart body where the file is named `file`.

__Curl:__

```bash
curl -X POST -F file=@attachment.pdf -u {API_KEY} https://{CUSTOMER_NAME}.peytzmail.com/api/v1/attachments.json
```
__Windows Powershell:__

```powershell
$file = 'C:\Users\Peytz\attachment.pdf'
$api_key = '{API_KEY}'
$url = 'https://{CUSTOMER_NAME}.peytzmail.com/api/v1/attachments.json'
$webclient = new-object System.Net.WebClient
$credCache = new-object System.Net.CredentialCache
$creds = new-object System.Net.NetworkCredential($api_key, '')
$credCache.Add($url, "Basic", $creds)
$webclient.Credentials = $credCache
$webclient.UploadFile($url, $file)
```

#### Example with a pushed file and params in the URL
Request:
```
/api/v1/attachments.json?title=this-is-my-title&tags[]=tag1&tags[]=tag2
```

Post data should be a multipart body where the file is named `file`.

## Appendix

### Alternative authentication

**Query parameters**

It is possible to authenticate by setting the query parameters `user_email` and `user_token` to the email and API key of the API user, respectively.

Example:

```bash
curl https://{CUSTOMER_NAME}.peytzmail.com/api/v1/mailinglists.json?user_email={API_USER_EMAIL}&user_token={API_KEY}
```

**Custom headers**

It is also possible to authenticate by setting the `X-User-Email` and `X-User-Token` headers to the email and API key of the API user, respectively.

Example:

```bash
curl -H 'X-User-Email: {API_USER_EMAIL}' -H 'X-User-Token: {API_KEY}' https://{CUSTOMER_NAME}.peytzmail.com/api/v1/mailinglists.json
```

## Changelog

#### 2016-07-05 version 3.30.0
* Deprecated mailinglist properties:
* - auto_text_part (default: false, deprecated: null)
* - frontend_settings_override (default: false, deprecated: null)
* - default_frontend_layout (default: "frontend", deprecated: null)
* New mailinglist properties:
* - text_template (default: false)
* - subscribe_page_settings_override (default: false)
* - unsubscribe_page_settings_override (default: false)
* - default_subscribe_form_view (default: "subscriptions/subscribe_form")
* - default_subscribe_view (default: "subscriptions/subscribe")
* - default_subscribe_confirmed_view (default: "subscriptions/confirmed")
* - default_unsubscribe_view (default: "subscriptions/unsubscribe")
* - unsubscribe_settings_type (default: "frontend_views")
* - unsubscribe_page_id (default: null)
* - subscribe_settings_type (default: "frontend_views")
* - subscription_page_id (default: null)

#### 2016-03-08 version 3.26.0
* Added get trigger mails by tags, and tags to trigger mails
* Added general state to trigger mails and newsletters

#### 2016-03-02 version 3.25.0
* Added received and failed events to the tracks API method

#### 2015-11-05 version 3.17.4
* Added description of tags on mailinglist subscribes

#### 2015-10-08 version 3.16.0
* Added description of attachments functionality

#### 2015-09-17 version 3.11.7
* Added description of reason in unsubscribe callback

#### 2015-08-04 version 3.10.3
* Added description of quick subscribe functionality

#### 2015-07-29 version 3.10.2
* Added events end point for subscribers

#### 2015-07-08 version 3.8.0
* Added example of using merge_fields for lists

#### 2015-01-12 version 3.5.24
* Added update method to subscribers
* Added merge_fields option for mailinglist subscribe
* Fix faulty XML examples for merge-fields (should not be type array)

#### 2014-08-22 version 2.10.15
* Added response examples for mailinglist subscribe and unsubscribe

#### 2014-05-14 version 2.7.0
* Added merge_fields to newsletters
* Added update method to newsletters
* Changed subscriber field selections lists to key-value pairs

#### 2014-04-22 version 2.6.0
* Added documentation for combining feeds

#### 2014-03-24 version 2.5.0
* Added documentation for subscriber fields
* Added optional fields litmus_settings_override, litmus_tracking_code and auto_text_part to the mailing list methods
* Changed id field for trigger mail responses from _id to id
* Added default_theme_id and theme_id properties to mailinglists, newsletters and trigger mails

#### 2014-03-14 version 2.4.7
* Added notes on trigger mail create and send response with failing recipients

#### 2013-12-17 version 1.28.0
* Added notes about allowed fields on mailinglist create

#### 2013-11-05 version 1.25.0
* Added functionality for retrieving a list of tracks

#### 2013-10-22 version 1.24.0
* Added documentation for subscriber events
* Added information about unsubscribe callbacks

#### 2013-10-01 version 1.23.0
* Added functionality for creating multiple subscribers

#### 2013-09-12 version 1.22.0
* Added standard feed examples and description
* Added curl example for POST request

#### 2013-08-12 version 1.16.0
* Added user group methods

#### 2013-07-17
* Added default_layout_id property to mailinglists

#### 2013-05-02
* Added preview method for newsletters

#### 2013-04-16
* Added description of pushing compressed feed files

#### 2012-08-14
* Added options for sending confirmation and welcome mails to the create mailing
lists method (defaults to being enabled!)
* Removed name as an option on mailing lists - use id instead
* Added the get trigger mail method
* Added id to the response from create and send trigger mails - telling the id
of the trigger mail the call created

#### 2012-08-14 release 2
* Fixed parameter for to_email for create and send trigger mails in this
document
* Added skip_confirm and skip_welcome parameters to the mailinglists subscribe
method
* Added mailinglist_ids and pending_mailinglist_ids to the output of get
subscriber and find subscriber
* Find subscribers now makes use of pagination - notice the XML output has
changed format!

#### 2012-08-20 version 0.8.0.rc.9
* Added optional property skip_duplicate_check to the create and
create_and_send newsletter

#### 2012-08-20 version 0.8.0.rc.10
* Removed template as a required field when creating newsletters
* Added default_template as a required field when creating a mailinglists

#### 2012-08-22 version 0.8.0.rc.11
* Added support for pushing data in the create newsletter call
* Fixed description and examples for the unsubscribe from a mailinglist method
* Added description of the possible state values in the get newsletters response
