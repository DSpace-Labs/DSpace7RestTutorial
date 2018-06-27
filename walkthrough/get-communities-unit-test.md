{% include navmenu.html %}

## Get Communities Unit Test
---
### Test Class org.dspace.app.rest.CommunityRestRepositoryIT [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/test/java/org/dspace/app/rest/CommunityRestRepositoryIT.java#L27)

```
package org.dspace.app.rest;

```
Note the "import static" declarations.  This will allow static methods defined in the test framework to be referenced staticly.

You will see these methods referenced below.  The method invocations may appear to be class methods, but they are referencing these external static methods.
```
import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
```
The following imports are more conventional.
```
import java.util.UUID;

import org.dspace.app.rest.builder.CollectionBuilder;
import org.dspace.app.rest.builder.CommunityBuilder;
import org.dspace.app.rest.matcher.CommunityMatcher;
import org.dspace.app.rest.test.AbstractControllerIntegrationTest;
import org.dspace.content.Collection;
import org.dspace.content.Community;
import org.hamcrest.Matchers;
import org.junit.Test;

```
The following test class will run as an integration test using Spring framework test conventions.

Several convenience methods are located in the base class.
```
public class CommunityRestRepositoryIT extends AbstractControllerIntegrationTest {
```
The @Test annotation indicates that a specific method should be executed during the testing phase.

The method has been assigned a meaningful name.
```
    @Test
    public void findAllTest() throws Exception {
```
The DSpace authorization system is bypassed in order to allow test objects to be loaded into the h2 database.
```
        //We turn off the authorization system in order to create the structure as defined below
        context.turnOffAuthorisationSystem();
```
Construct the following Hierarchy
- Parent Community
  - Sub Community
    - Collection 1

```
        //** GIVEN **
        //1. A community-collection structure with one parent community with sub-community and one collection.
        parentCommunity = CommunityBuilder.createCommunity(context)
                                          .withName("Parent Community")
                                          .build();
        Community child1 = CommunityBuilder.createSubCommunity(context, parentCommunity)
                                           .withName("Sub Community")
                                           .build();

        Collection col1 = CollectionBuilder.createCollection(context, child1)
                                           .withName("Collection 1")
                                           .withLogo("Test Logo")
                                           .build();
```
Spring MockMvc is used to simulate calls to a web service.

The **getClient()** method retrieves a MockMvc client.  

MockMvc allows a user to construct very readable test logic.  The underlying classes involved in the construction of this logic may look a little different than typical POJO (plain old java object) code.

The **getClient()** method is defined in the base integration test class. [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/test/java/org/dspace/app/rest/test/AbstractControllerIntegrationTest.java#L87-L89)

Note the use of the static **get()** method that builds a GET request.

MockMvc.perform() executes the get method that has been constructed.
```
        getClient().perform(get("/api/core/communities"))
```
The perform methed returns a [ResultActions](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultActions.html) object.

This object can be used to inspect expected response values or to return specific response values.
- the status code can be inspected
- the content type of the return data can be inspected
- if json is returned, the json object can be inspected

```
                   .andExpect(status().isOk())
                   .andExpect(content().contentType(contentType))
```
The [Hamcrest Java library](https://code.google.com/archive/p/hamcrest/wikis/Tutorial.wiki) is used to match json content.

The DSpace REST API has implemented a set of matcher classes for DSpace objects. See [Package org.dspace.app.rest.matcher&rarr;](https://github.com/DSpace/DSpace/tree/rest-tutorial/dspace-spring-rest/src/test/java/org/dspace/app/rest/matcher)

The [CommunityMatcher](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/test/java/org/dspace/app/rest/matcher/CommunityMatcher.java) class examines particular Community object properties.
```
                   .andExpect(jsonPath("$._embedded.communities", Matchers.containsInAnyOrder(
                       CommunityMatcher.matchCommunityEntry(parentCommunity.getName(), parentCommunity.getID(),
                                                            parentCommunity.getHandle()),
                       CommunityMatcher
                           .matchCommunityWithCollectionEntry(child1.getName(), child1.getID(), child1.getHandle(),
                                                              col1)
                   )))
```
Additional JSON value tests
```
                   .andExpect(jsonPath("$._links.self.href", Matchers.containsString("/api/core/communities")))
                   .andExpect(jsonPath("$.page.size", is(20)))
                   .andExpect(jsonPath("$.page.totalElements", is(2)))
        ;
    }

```
An additional test method that examines the pagination properties returned from the /core/communities endpoint.
```
    @Test
    public void findAllPaginationTest() throws Exception {
        //We turn off the authorization system in order to create the structure as defined below
        context.turnOffAuthorisationSystem();

        //** GIVEN **
        //1. A community-collection structure with one parent community with sub-community and one collection.
        parentCommunity = CommunityBuilder.createCommunity(context)
                                          .withName("Parent Community")
                                          .build();
        Community child1 = CommunityBuilder.createSubCommunity(context, parentCommunity)
                                           .withName("Sub Community")
                                           .build();
        Collection col1 = CollectionBuilder.createCollection(context, child1).withName("Collection 1").build();
```
Note that the **get()** method returns a request builder that can be enhanced.

In this instance, the request is being modified with the addition of a url parameter that controls pagination.
```
        getClient().perform(get("/api/core/communities")
                                .param("size", "1"))
                   .andExpect(status().isOk())
                   .andExpect(content().contentType(contentType))
                   .andExpect(jsonPath("$._embedded.communities", Matchers.contains(
                       CommunityMatcher.matchCommunityEntry(parentCommunity.getName(), parentCommunity.getID(),
                                                            parentCommunity.getHandle())
                   )))
                   .andExpect(jsonPath("$._embedded.communities", Matchers.not(
                       Matchers.contains(
                           CommunityMatcher.matchCommunityEntry(child1.getName(), child1.getID(), child1.getHandle())
                       )
                   )))
                   .andExpect(jsonPath("$._links.self.href", Matchers.containsString("/api/core/communities")))
        ;

```        
In this instance, the request is being modified with the addition of 2 url parameters that controls pagination.

Since the page size has been set to 1, the /core/communites request will return 2 pages.  
- Parent Community
- Sub Community

```
        getClient().perform(get("/api/core/communities")
                                .param("size", "1")
                                .param("page", "1"))
                   .andExpect(status().isOk())
                   .andExpect(content().contentType(contentType))
                   .andExpect(jsonPath("$._embedded.communities", Matchers.contains(
                       CommunityMatcher.matchCommunityEntry(child1.getName(), child1.getID(), child1.getHandle())
                   )))
                   .andExpect(jsonPath("$._embedded.communities", Matchers.not(
                       Matchers.contains(
                           CommunityMatcher.matchCommunityEntry(parentCommunity.getName(), parentCommunity.getID(),
                                                                parentCommunity.getHandle())
                       )
                   )))
                   .andExpect(jsonPath("$._links.self.href", Matchers.containsString("/api/core/communities")))
```
Note the inspection of the pagination properies that are returned.
```
                   .andExpect(jsonPath("$.page.size", is(1)))
                   .andExpect(jsonPath("$.page.totalElements", is(2)))
        ;
    }
```
Note, that this test class contains other methods that test specific functions within the core/communities endpoint.

{% include nav.html %}
