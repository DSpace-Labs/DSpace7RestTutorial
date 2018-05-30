# Exercise 1: Exploring HAL

### 1. Open the landing page

- [{{site.restService}}]({{site.restService}})

### 2. Communities and Collections

- List all Communities

`api/core/communities`

- Note the search operation that exists.
  - Search for all top level communities

`api/core/communities/search/top`

- In the "Embedded Resources" section, expand a specific top level community
  - What is its name?
  - What is its handle?
  - What is its unique id?
  - Note the links that are available in the embedded Resources.
  - Click the "self" link to retrieve the object on its own.

`api/core/communities/[uuid]`


  - Does the community have subcommunities?
  - Does the community contain collections?
  - Look at the list of collections within that community
- Look at a specific collection

{% include nav.html %}
