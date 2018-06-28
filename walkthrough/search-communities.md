{% include navmenu.html %}

## Search Communities
This request will match the following controller:

### Controller org.dspace.app.rest.RestResourceController
```
@RestController
@RequestMapping("/api/{apiCategory}/{model}")
@SuppressWarnings("rawtypes")
public class RestResourceController implements InitializingBean {
```

Based on the path specified, the following method will be invoked.


### Method org.dspace.app.rest.RestResourceController.executeSearchMethods() [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L838-L862)
Parameters
- apiCategory = core
- model = communities
- searchMethodName = top

```
@RequestMapping(method = RequestMethod.GET, value = "/search/{searchMethodName}")
@SuppressWarnings("unchecked")
public <T extends RestAddressableModel> ResourceSupport executeSearchMethods(@PathVariable String apiCategory,
                                                                             @PathVariable String model,
                                                                             @PathVariable String searchMethodName,
                                                                             Pageable pageable, Sort sort,
                                                                             PagedResourcesAssembler assembler,
                                                                             @RequestParam MultiValueMap<String,
                                                                                 Object> parameters)
    throws IllegalAccessException, IllegalArgumentException, InvocationTargetException {

    Link link = linkTo(this.getClass(), apiCategory, model).slash("search").slash(searchMethodName).withSelfRel();
    DSpaceRestRepository repository = utils.getResourceRepository(apiCategory, model);
    boolean returnPage = false;
    Object searchResult = null;
```
The following method will locate the appropriate search method by calling org.dspace.rest.utils.Utils.getSearchMethod()
```
    Method searchMethod = repositoryUtils.getSearchMethod(searchMethodName, repository);

    if (searchMethod == null) {
        if (repositoryUtils.haveSearchMethods(repository)) {
            throw new RepositorySearchMethodNotFoundException(model, searchMethodName);
        } else {
            throw new RepositorySearchNotFoundException(model);
        }
    }
```
Once the appropriate search method has been found, **org.dspace.app.rest.utils.Utils.executeQueryMethod()** will be invoked.
```
    searchResult = repositoryUtils
        .executeQueryMethod(repository, parameters, searchMethod, pageable, sort, assembler);

    returnPage = searchMethod.getReturnType().isAssignableFrom(Page.class);
    ResourceSupport result = null;
    if (returnPage) {
        Page<DSpaceResource<T>> resources = ((Page<T>) searchResult).map(repository::wrapResource);
        resources.forEach(linkService::addLinks);
        result = assembler.toResource(resources, link);
    } else {
        DSpaceResource<T> dsResource = repository.wrapResource((T) searchResult);
        linkService.addLinks(dsResource);
        result = dsResource;
    }
    return result;
}
```

### Method org.dspace.rest.utils.RestRepositoryUtils.getSearchMethod() [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/utils/RestRepositoryUtils.java#L97-L113)
```
    public Method getSearchMethod(String searchMethodName, DSpaceRestRepository repository) {
        Method searchMethod = null;
        for (Method method : repository.getClass().getMethods()) {
```
This method will search the code for the Annotation **org.dspace.app.rest.SearchRestMethod** and then compare the name of the method with the searchMethodName.
```
            SearchRestMethod ann = method.getAnnotation(SearchRestMethod.class);
            if (ann != null) {
                String name = ann.name();
                if (name.isEmpty()) {
                    name = method.getName();
                }
                if (StringUtils.equals(name, searchMethodName)) {
                    searchMethod = method;
                    break;
                }
            }
        }
        return searchMethod;
    }
```

The following method contains the matching **@SearchRestMethod** annotation.

### Method org.dspace.app.rest.repository.CommunityRepository.findAllTop() [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L81-L94)
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

## Unit Test for Find All Top Communities

### Test Method org.dspace.app.rest.CommunityRestRepositoryIT.fandAllSearchTop() [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/test/java/org/dspace/app/rest/CommunityRestRepositoryIT.java#L210-L256)
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
