# REST API with JWT authentication using Jersey and Spring Security

[![Build Status](https://travis-ci.org/cassiomolin/jersey-jwt-springsecurity.svg?branch=master)](https://travis-ci.org/cassiomolin/jersey-jwt-springsecurity)
[![MIT Licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/cassiomolin/jersey-jwt-springsecurity/master/LICENSE.txt)

This sample application demonstrates how to perform token-based authentication using:

- **Spring Boot:** Framework for creating standalone Java applications.
- **Spring Security:** Framework for authentication and authorization.
- **Spring Data JPA:** Framework for implementing JPA repositories.
- **H2:** In memory relational database.
- **Jersey:** JAX-RS reference implementation for creating RESTful web services in Java.
- **Jackson:** JSON parser for Java.
- **JJWT:** Library for creating and parsing JSON Web Tokens (JWTs) in Java.

> **Note:** For a CDI and JAX-RS approach _without_ Spring (Boot, Data and Security), have a look at the [`jersey-jwt`][jaxrs-cdi-example] project.

## How token-based authentication works?

In a token-based authentication, the client exchanges _hard credentials_ (such as username and password) for a piece of data called _token_. Instead of sending the hard credentials in every request, the client will send the token to the server to perform authentication and authorisation.

In a few words, an authentication scheme based on tokens follow these steps:

1. The client sends their credentials (username and password) to the server.
1. The server authenticates the credentials and issues a token.
1. The server can store the previously generated token in some storage along with the user identifier.
1. The server sends the generated token in the response.
1. In each request, the client sends the token to the server.
1. The server, in each request, extracts the token from the incoming request. With the token, the server looks up the user details to perform authentication and authorisation.
    1. If the token is valid, the server accepts the request.
    1. If the token is invalid, the server refuses the request.
1. The server can provide an endpoint to refresh tokens.

## What tokens can be like?

A token can be _opaque_ which reveals no details other than the value itself (like a random string) or can be _self-contained_ (like JWT, which is used in this example).

JWT stands for JSON Web Token. It's a standard method for representing claims securely between two parties, defined in the [RFC 7519][]. JWT is a self-contained token and enables you to store a user identifier, an expiration date and whatever you want (but don't store passwords) in a payload, which is a JSON encoded as Base64. The payload can be read by the client and the integrity of the token can be easily checked by verifying its signature on the server.

To find some great resources to work with JWT, have a look at [http://jwt.io][jwt.io].

JWT allows you to perform stateless authentication, that is, you won't need to persist JWT tokens if you don't need to track them. Although, by persisting the tokens, you will have the possibility of invalidating and revoking the access of them. To keep the track of JWT tokens, instead of persisting the whole token, you could persist the token identifier (the [`jti`][jti claim] claim) and some metadata (the user you issued the token for, the expiration date, etc) if you need.

Your application can provide some functionality to revoke the tokens, but always consider revoking the tokens when the users change their password. When persisting tokens, consider removing the old ones in order to prevent your database from growing indefinitely.

## Building and running this application

To build and run this application, follow these steps:

1. Open a command line window or terminal.
1. Navigate to the root directory of the project, where the `pom.xml` resides.
1. Compile the project: `mvn clean compile`.
1. Package the application: `mvn package`.
1. Change into the `target` directory: `cd target`
1. You should see a file with the following or a similar name: `jersey-jwt-springsecurity-1.0.jar`.
1. Execute the JAR: `java -jar jersey-jwt-springsecurity-1.0.jar`.
1. The application should be available at `http://localhost:8080/api`.

When the application starts up, the database will be populated with the following users:

ID | Username | Password | Active | Roles
---|--------- |----------|--------|------------
1  | admin    | password | true   | ADMIN, USER
2  | user     | password | true   | USER
3  | disabled | password | false  | USER

## Application overview

Find below a quick description of the most relevant classes of this application:

- [`WebSecurityConfig`](src/main/java/com/cassiomolin/example/security/config/WebSecurityConfig.java): Spring Security configuration class.

- [`AuthenticationResource`](src/main/java/com/cassiomolin/example/security/api/resource/AuthenticationResource.java): REST endpoint for exchanging hard credentials for a JWT token.

- [`JwtAuthenticationToken`](src/main/java/com/cassiomolin/example/security/api/jwt/JwtAuthenticationToken.java): [`Authentication`](https://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/core/Authentication.html) implementation designed for presentation of a JWT token.

- [`JwtAuthenticationTokenFilter`](src/main/java/com/cassiomolin/example/security/api/jwt/JwtAuthenticationTokenFilter.java): [`OncePerRequestFilter`](http://docs.spring.io/autorepo/docs/spring/current/javadoc-api/org/springframework/web/filter/OncePerRequestFilter.html) for extracting the authentication token from the `Authorization` header of the HTTP request.

- [`JwtAuthenticationProvider`](src/main/java/com/cassiomolin/example/security/api/jwt/JwtAuthenticationProvider.java): [`AuthenticationProvider`](https://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/authentication/AuthenticationProvider.html) used to authenticate the JWT token.

- [`JwtAuthenticationEntryPoint`](src/main/java/com/cassiomolin/example/security/api/jwt/JwtAuthenticationEntryPoint.java): [`AuthenticationEntryPoint`](https://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/AuthenticationEntryPoint.html) implementation that simply returns error details related to authentication failures.

- [`DefaultUserDetailsService`](src/main/java/com/cassiomolin/example/security/service/impl/DefaultUserDetailsService.java): [`UserDetailsService`](https://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/core/userdetails/UserDetailsService.html) implementation.

## REST API overview

See the [curl][] scripts below with the REST API supported operations:

### Exchange hard credentials for an authentication token

Valid credentials must be sent in the request payload to be exchanged for a token.

```bash
curl -X POST \
  'http://localhost:8080/api/auth' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "username": "<username>",
  "password": "<password>"
}'
```

### Get a public greeting

No authentication is required to perform this operation.

```bash
curl -X GET \
  'http://localhost:8080/api/greetings/public' \
  -H 'Accept: text/plain'
```

### Get a greeting for the user

Authentication and `USER` role are required to perform this operation.

```bash
curl -X GET \
  'http://localhost:8080/api/greetings/protected' \
  -H 'Accept: text/plain' \
  -H 'Authorization: Bearer <authentication-token>'
```

### Get all users

Authentication and `ADMIN` role are required to perform this operation.

```bash
curl -X GET \
  'http://localhost:8080/api/users' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <authentication-token>'
```

### Get a user by id

Authentication and `ADMIN` role are required to perform this operation.

```bash
curl -X GET \
  'http://localhost:8080/api/users/<user-id>' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <authentication-token>'
```

### Get the current user

No authentication is required to perform this operation. However, if the request is performed with a valid token, the server will return details for the current user.

```bash
curl -X GET \
  'http://localhost:8080/api/users/me' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <authentication-token>'
```

## Targeting the REST API with Postman

Alternatively to [curl], you can use [Postman][] to target the REST API. The Postman collection files are available in the [`src/main/postman`](src/main/postman) directory.


[RFC 7519]: https://tools.ietf.org/html/rfc7519
[jwt.io]: http://jwt.io/
[jti claim]: https://tools.ietf.org/html/rfc7519#section-4.1.7
[Postman]: https://www.getpostman.com/
[jaxrs-cdi-example]: https://github.com/cassiomolin/jersey-jwt
[curl]: https://curl.haxx.se/
