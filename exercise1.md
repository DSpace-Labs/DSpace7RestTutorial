{% include navmenu.html %}

# Exercise 1: Exploring Endpoints with HAL Browser

### 1. Open the HAL Browser landing page

[{{site.restService}}]({{site.restService}})

- Note the list of endpoints that are available

### 2. Communities and Collections
Use the HAL Browser to explore the community and collection endpoints.

#### 2a. List all Communities
Use the HAL Browser to list all communities.

`api/core/communities`

- Note that the results are paginated.
- Navigate through the list using the first, last, and next links.
- How many items are in the repository?
- How many pages of items are available?
  - Note: The DSpace 7 REST API will use the same approach for paginating all lists of objects.

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
- Does the community contain collections?
- If not, navigate the subcommunity hierarchy until you are able to retrieve a specific collection.

#### 2e. Request Collections within a subcommunity
Use the HAL Browser to navigate to a subcommunity that contains collections.  List those collections.
`api/core/communities/[uuid]/collections`

#### 2f. Expand an embedded collection resource
Expand a collection in the embedded resources section of the HAL Browser to look at its data.
- What is its name?

### 3. Browse the Items
An anonymous user can only browse public items. Access to the  

`api/core/items`

endpoint is restricted to administrator. Try! you will get a Forbidden 403 response

#### 3a. List all the browses
Use the HAL Browser to list all the available browses index in the repository.
- How many browses are defined?
- Explore the embedded definition and identify the browse by title and the browse by authors, do you see any difference in the exposed links?

#### 3a. Browse items by title
Use the HAL Browser to browse the public items in the repository by title.
- Note that the results are paginated.
- How many items are in the repository?
- How many pages of items are available?

`api/discover/browses/title/items`

#### 3b. Browse the authors
Use the HAL Browser to browse the repository author index.
- Note that the results are paginated.
- How many authors are in the repository?
- How many items has the first author?

`api/discover/browses/author/entries`

#### 3c. Browse the items by author
Use the HAL Browser to browse the items for a specific author.
- Note that the results are paginated.

`api/discover/browses/author/entries/[value]/items`

### 4. View the details of an item

#### 4a. View a specific item
Use the HAL Browser to view the details of a specific item, use the *self* link to access directly the item.
Note how the URL is changed

`api/core/items/[uuid]`

#### 4b. View the owning collection for an item
Use the HAL Browser to view the owning collection for a specific item.

`api/core/items/[uuid]/owningCollection`

#### 4c. View the bitstreams for an item
Use the HAL Browser to view the bitstreams for a specific item.

`api/core/items/[uuid]/bitstreams`


{% include credentials.html %}
{% include nav.html %}
