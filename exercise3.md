# Exercise 3: Browsing the Repository with Postman

### 1. Use Postman to load the REST API landing page
[{{site.restService}}]({{site.restService}})

You will see some HTML text.  Postman will not render the HAL Browser.

## 2. Communities

### 2a. Use Postman to list all communities

`api/core/communities`

Note that the "Pretty" view will allow you to expand and collapse the JSON data that is returned.

At the bottom of the return object, you will see a count of the number of objects that was returned.  Collapse items in the return object to help locate this field.

### 2b. Use Postman to list all communities

`api/core/communities/search/top`

## 3. Items

### 3a. Use Postman to list all items

`api/core/items`

- How many items exist?
- How many items were returned?

### 3b. Add a __size__ parameter in postman to change the page size

Note the change in results.

### 3c. Add a __page__ and __size__ parameter in postman to navigate to a different page of results.

Note the change in results.


{% include credentials.html %}
{% include nav.html %}
