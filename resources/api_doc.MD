<div markdown="1" class="sidebar">
## Summary

* [Introduction](#introduction)
* [Authentication](#authentication)
  * [Client Setup](#client-setup)
  * [Password Grant](#password-grant)
  * [Authorization Code Grant](#authorization-code-grant)
* [RSQL Filtering](#rsql-filtering)
  * [Operators](#operators)
* [Endpoints](#endpoints)
</div>

<div markdown="1" class="main-content">
# GLPI API Documentation

## Introduction {#introduction}

The GLPI API is a RESTful API. This version of the API is referred to as the "High-level API".
The original API is now referred to as the "legacy API" but it is still available and in some cases, allows more functionality at the cost of user-friendliness.

For more information on the legacy API, see the [legacy API documentation](../apirest.md).

## Authentication {#authentication}

Authentication for the API is all done via OAuth2.
The following OAuth2 grant types are supported:

- Password: Authenticate with a client ID, secret, and username/password. This is the recommended method for automated scripts.
- Authorization Code: Interactive authentication. This is the recommended method for user-facing applications.

Grants that do not require specific user credentials are not currently supported as there is no virtual user and scope system implemented at this time.

### Client Setup {#client-setup}

Clients must be registered with the GLPI server before they can be used to authenticate.
Clients are used to identify the application/user that is using the API and may be limited to specific grant types.

To register a client, you can do so from Setup > OAuth Clients.
When creating a new client, you will be prompted for just a name.
When the client is created, there will be an automatically generated client ID and secret.
There will also be a field to specify the allowed grant types.

### Password Grant {#password-grant}

To authenticate with the password grant, you will need to make a POST request to the `/api.php/token` endpoint with the following parameters in the request body:

- `grant_type`: `password`
- `client_id`: The client ID of the client you are using
- `client_secret`: The client secret of the client you are using
- `username`: The username of the user you are authenticating as
- `password`: The password of the user you are authenticating as
- `scope`: The scope of the request. This must be supplied but is not currently used. It can be an empty string.

The endpoint will respond (if successful) with a JSON object containing the following fields:

- `token_type`: `Bearer`
- `expires_in`: The number of seconds until the token expires. By default, this is set to 1 hour, but it can be controlled by defining the `GLPI_OAUTH_ACCESS_TOKEN_EXPIRES` in your `config/local_define.php` file (See the [installation documentation](https://glpi-install.readthedocs.io/en/latest/install/index.html#files-and-directories-locations) for more information on configuring GLPI config constants).
- `access_token`: The access token that can be used to authenticate future requests.

For API requests, you should include the access token in the `Authorization` header of the request with the `Bearer` prefix in the format `Authorization: Bearer <access_token>`.

### Authorization Code Grant {#authorization-code-grant}

To authenticate with the authorization code grant, you will need to make a GET request to the `/api.php/authorize` endpoint with the following parameters in the query string:

- `response_type`: `code`
- `client_id`: The client ID of the client you are using
- `scope`: The scope of the request. This must be supplied but is not currently used. It can be an empty string.

The endpoint will respond with a redirect to the GLPI login page.
If successful, the user will be redirected back to the client with a `code` parameter in the query string.
This code can be used to get an access token by making a POST request to the `/api.php/token` endpoint with the following parameters in the request body:

- `grant_type`: `authorization_code`
- `client_id`: The client ID of the client you are using
- `client_secret`: The client secret of the client you are using
- `code`: The code returned by the `/api.php/authorize` endpoint

The endpoint will respond (if successful) with a JSON object containing the following fields:

- `token_type`: `Bearer`
- `expires_in`: The number of seconds until the token expires. By default, this is set to 1 hour, but it can be controlled by defining the `GLPI_OAUTH_ACCESS_TOKEN_EXPIRES` in your `config/local_define.php` file (See the [installation documentation](https://glpi-install.readthedocs.io/en/latest/install/index.html#files-and-directories-locations) for more information on configuring GLPI config constants).
- `access_token`: The access token that can be used to authenticate future requests.
- `refresh_token`: The refresh token that can be used to get a new access token when the current one expires.

## RSQL Filtering {#rsql-filtering}

The GLPI supports "REST Query Language" (RSQL) filtering for certain endpoints that return collections of items.
RSQL itself is a super-set of the "Feed Item Query Language" (FIQL).
It allows for an intuitive and easy to use syntax for filtering collections of items.

The basic syntax is as follows: `<field><operator><value>`. For some operators, there is no value, so the syntax is just `<field><operator>`.

### Operators {#operators}
| Operator      | Description              | Example                |
|---------------|--------------------------|------------------------|
| `==`          | Equal to                 | `name==John`           |
| `!=`          | Not equal to             | `name!=John`           |
| `=in=`        | In                       | `name=in=(John,Paul)`  |
| `=out=`       | Not in                   | `name=out=(John,Paul)` |
 | `=lt=`        | Less than                | `age=lt=18`            |
 | `=le=`        | Less than or equal to    | `age=le=18`            |
 | `=gt=`        | Greater than             | `age=gt=18`            |
 | `=ge=`        | Greater than or equal to | `age=ge=18`            |
 | `=like=`      | Like                     | `name=like=John`       |
| `=ilike=`     | Case insensitive like    | `name=ilike=John`      |
| `=isnull=`    | Is null                  | `name=isnull=`         |
| `=isnotnull=` | Is not null              | `name=isnotnull=`      |
| `=empty=`     | Is empty                 | `name=empty=`          |
| `=notempty=`  | Is not empty             | `name=notempty=`       |

The fields available for filtering are dependent on the endpoint. Everything that is returned by the endpoint can be filtered on in the exact same format that they are returned from the server.
For example, if an endpoint returns a collection of items and each item has an `emails` property with an array of email addresses which each contain an `email` property, you can filter using `emails.email` as the field name in the filter.

## Endpoints {#endpoints}

For specific documentation on each endpoint, see the Swagger documentation at `/api.php/doc`.
You may also make a request to `/api.php/doc` to get the raw Swagger JSON document if you set the `Content-Type` header to `application/json` or add `.json` to the end of the URL.
</div>
