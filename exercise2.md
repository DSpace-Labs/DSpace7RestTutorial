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
(This is not currerntly working)

For convenience, the HAL browser also provides a login box at the top of the page.  
Click "Login" and supply the credentials at the bottom of this page.

Behind the scenes, the login box will perform the following operation.

`api/auth/login`

This action will return a response header named "Authorization" which contains an authorization token.  
The HAL browser will pass this token in subsequent requests.

### 2c. Verify your authentication status.

`api/authn/status`

The response body should now include the following status.

    authenticated: true

Note: this authentication token will eventually expire.  Repeat step 1b if you need a new token.

For the rest of this exercise, the steps will make use of this authentication token.

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
