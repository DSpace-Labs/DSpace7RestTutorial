# Exercise 2: Authenticating with the HAL Browser

## 1. Searching for objects in an unauthenticated session
In this step, you will run a couple of DSpace discovery searches before authenticating and note the result counts.

### 1a. Search for all objects and note the result count

`api/discover/search/objects`

### 1b. Search for "research" and note the result count

`api/discover/search/objects?query=research`

After authenticating, you will see how the results change.

## 2. Authentication status

### 2a. Using the authn endpoint, view your authentication status

`api/authn/status`

In the response body, note the status

    authenticated: false

### 2b. From the authn endpoint, POST to the login method
In order to perform a POST operation, you must use the NON-GET button "!".

`api/auth/login`

Supply the following login information in the popup box.

    {
      email: 'test@test.edu',
      password: 'admin'
    }

View the return status.  If it is successful, you should see a response header named "Authorization".

### 2c. Supply the authorization header and view your authentication status.
Copy and paste the Authentication token into the "Custom Request Headers" section.
Request the authentication status again.

`api/authn/status`

The response body should now include the following status.

    authenticated: true

Note: this authentication token will eventually expire.  Repeat step 1b if you need a new token.

For the rest of this exercise, continue to provide the authorization header.

## 3. View non-public content
Note how these counts differ from your prior results.

### 3a. Search for all objects and note the result count

`api/discover/search/objects`

### 3b. Search for "research" and note the result count

`api/discover/search/objects?query=research`

## 4. Next steps

The HAL browser does not yet pass authentication status to requests that modify data, so this exercise will stop here.

Future modifications will enable more functionality through the browser.


{% include nav.html %}
