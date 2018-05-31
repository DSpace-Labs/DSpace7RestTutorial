# Exercise 2: Authenticating with the HAL Browser

## 1. Authentication status

### 1a. Using the authn endpoint, view your authentication status

`api/authn/status`

In the response body, note the status

    authenticated: false

### 1b. From the authn endpoint, POST to the login method
In order to perform a POST operation, you must use the NON-GET button "!".

`api/auth/login`

Supply the following login information in the popup box.

    {
      email: 'test@test.edu',
      password: 'admin'
    }

View the return status.  If it is successful, you should see a response header named "Authorization".

### 1c. Supply the authorization header and view your authentication status.
Copy and paste the Authentication token into the "Custom Request Headers" section.
Request the authentication status again.

`api/authn/status`

The response body should now include the following status.

    authenticated: true

Note: this authentication token will eventually expire.  Repeat step 1b if you need a new token.

For the rest of this exercise, continue to provide the authorization header.

## 2. View non-public content

- TODO: Based on the server we will be using, add notes to query for restricted content

## 3. Next steps

The HAL browser does not yet pass authentication status to requests that modify data, so this exercise will stop here.

Future modifications will enable more functionality through the browser.


{% include nav.html %}
