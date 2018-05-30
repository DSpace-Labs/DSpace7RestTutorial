# Exercise 1: Exploring HAL

### 1. Open the landing page

- [{{site.restService}}]({{site.restService}})

### 2. Communities and Collections

#### List all Communities

`api/core/communities`

Note the search operation that exists.

#### Search for all top level communities

`api/core/communities/search/top`

#### Expanding Embedded Resources
In the "Embedded Resources" section, expand a specific top level community
- What is its name?
- What is its handle?
- What is its unique id?
- Note the links that are available in the embedded Resources.
- Click the "self" link to retrieve the object on its own.

#### Request a specific community
`api/core/communities/[uuid]`

Explore the community
- Does the community have subcommunities?
  - (NOTE: Will the PR be in place that exposes subcommunities?)
- Does the community contain collections?
- Navigate the subcommunity hierarchy until you are able to retrieve a specific collection.

#### Request Collections within a subcommunity
`api/core/communities/[uuid]/collections`

#### Expand an embedded community Resource
- What is its title?

### 3. Items
- Note that the results are paginated.
- Navigate through the list using the first, last, and next links.

{% include nav.html %}
