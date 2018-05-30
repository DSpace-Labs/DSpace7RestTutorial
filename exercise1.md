# Exercise 1: Exploring HAL

### 1. Open the HAL Browser landing page

[{{site.restService}}]({{site.restService}})

- Note the list of endpoints that are available

### 2. Communities and Collections
Use the HAL Browser to explore the community and collection endpoints.

#### 2a. List all Communities
Use the HAL Browser to list all communities.

`api/core/communities`

Note the search operation that exists.

#### 2b. Search for all top level communities
Use the HAL Browser to list all top level communities.

`api/core/communities/search/top`

#### 2c. Expand the embedded community resources
In the "Embedded Resources" section, expand a specific top level community
- What is its name?
- What is its handle?
- What is its unique id?
- Note the links that are available in the embedded Resources.
- Click the "self" link to retrieve the object on its own.

#### 2d. Request a specific community
After retrieving a specific community...

`api/core/communities/[uuid]`

Explore the community
- Does the community have subcommunities?
  - (NOTE: Will the PR be in place that exposes subcommunities?)
- Does the community contain collections?
- Navigate the subcommunity hierarchy until you are able to retrieve a specific collection.

#### 2e. Request Collections within a subcommunity
Use the HAL Browser to navigate to a subcommunity that contains collections.  List those collections.
`api/core/communities/[uuid]/collections`

#### 2f. Expand an embedded collection resource
Expand a collection in the embedded resources section of the HAL Browser to look at its data.
- What is its title?

### 3. Items

#### 3a. List all items
Use the HAL Browser to list all of the items in the repository.
- Note that the results are paginated.
- Navigate through the list using the first, last, and next links.
- How many items are in the repository?
- How many pages of items are available?
  - Note: The DSpace 7 REST API will use the same approach for paginating all lists of objects.

`api/core/items`

#### 3b. View a specific item
Use the HAL Browser to view a specific item.
`api/core/items/[uuid]`

#### 3c. View the owning collection for an item
Use the HAL Browser to view the owning collection for a specific item.
`api/core/items/[uuid]/owningCollection`

#### 3d. View the bitstreams for an item
Use the HAL Browser to view the bitstreams for a specific item.
`api/core/items/[uuid]/bitstreams`

### 4. EPersons and Groups

#### 4a. View all EPersons
Use the HAL Browser to view all people known to the repository.
`api/eperson/epersons`

#### 4b. View a specific EPerson
Use the HAL Browser to view the data for a specific user.
`api/eperson/epersons/[uuid]`

#### 4c. View all Groups
Use the HAL Browser to view all user groups known to the repository.

`api/eperson/groups`

#### 4d. View a specific Group
Use the HAL Browser to view the data for a specific user group.
`api/eperson/groups/[uuid]`

{% include nav.html %}
