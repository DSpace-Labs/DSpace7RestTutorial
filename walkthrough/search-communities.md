{% include navmenu.html %}

## Search Communities

https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L820-L837
```
@RequestMapping(method = RequestMethod.GET, value = "/search")
public ResourceSupport listSearchMethods(@PathVariable String apiCategory, @PathVariable String model) {
    ResourceSupport root = new ResourceSupport();
    DSpaceRestRepository repository = utils.getResourceRepository(apiCategory, model);

    List<String> searchMethods = repositoryUtils.listSearchMethods(repository);

    if (CollectionUtils.isEmpty(searchMethods)) {
        throw new RepositorySearchNotFoundException(model);
    }

    for (String name : searchMethods) {
        Link link = linkTo(this.getClass(), apiCategory, model).slash("search").slash(name).withRel(name);
        root.add(link);
    }
    return root;
}


```
https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L81-L94
```
// TODO: Add methods in dspace api to support pagination of top level
// communities
@SearchRestMethod(name = "top")
public Page<CommunityRest> findAllTop(Pageable pageable) {
    List<Community> topCommunities = null;
    try {
        topCommunities = cs.findAllTop(obtainContext());
    } catch (SQLException e) {
        throw new RuntimeException(e.getMessage(), e);
    }
    Page<CommunityRest> page = utils.getPage(topCommunities, pageable).map(converter);
    return page;
}

```


### Unit Test

https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/test/java/org/dspace/app/rest/CommunityRestRepositoryIT.java#L210-L256
```
@Test
 public void findAllSearchTop() throws Exception {

     //We turn off the authorization system in order to create the structure as defined below
     context.turnOffAuthorisationSystem();

     //** GIVEN **
     //1. A community-collection structure with one parent community with sub-community and one collection.
     parentCommunity = CommunityBuilder.createCommunity(context)
                                       .withName("Parent Community")
                                       .withLogo("ThisIsSomeDummyText")
                                       .build();

     Community parentCommunity2 = CommunityBuilder.createCommunity(context)
                                                  .withName("Parent Community 2")
                                                  .withLogo("SomeTest")
                                                  .build();

     Community child1 = CommunityBuilder.createSubCommunity(context, parentCommunity)
                                        .withName("Sub Community")
                                        .build();

     Community child12 = CommunityBuilder.createSubCommunity(context, child1)
                                         .withName("Sub Sub Community")
                                         .build();
     Collection col1 = CollectionBuilder.createCollection(context, child1).withName("Collection 1").build();


     getClient().perform(get("/api/core/communities/search/top"))
                .andExpect(status().isOk())
                .andExpect(content().contentType(contentType))
                .andExpect(jsonPath("$._embedded.communities", Matchers.containsInAnyOrder(
                    CommunityMatcher.matchCommunityEntry(parentCommunity.getName(), parentCommunity.getID(),
                                                         parentCommunity.getHandle()),
                    CommunityMatcher.matchCommunityEntry(parentCommunity2.getName(), parentCommunity2.getID(),
                                                         parentCommunity2.getHandle())
                )))
                .andExpect(jsonPath("$._embedded.communities", Matchers.not(Matchers.containsInAnyOrder(
                    CommunityMatcher.matchCommunityEntry(child1.getName(), child1.getID(), child1.getHandle()),
                    CommunityMatcher.matchCommunityEntry(child12.getName(), child12.getID(), child12.getHandle())
                ))))
                .andExpect(
                    jsonPath("$._links.self.href", Matchers.containsString("/api/core/communities/search/top")))
                .andExpect(jsonPath("$.page.size", is(20)))
                .andExpect(jsonPath("$.page.totalElements", is(2)))
     ;
 }

```
{% include nav.html %}
