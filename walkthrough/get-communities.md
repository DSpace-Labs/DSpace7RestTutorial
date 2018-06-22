## /api/core/communities
{% include navmenu.html %}
#### Locate the Controller for this request
The __[Spring MVC Framework](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.htm)__ looks for RestController annotations (org.springframework.web.bind.annotation.RestController).

The controller for this request will match `/api/{apiCategory}/{model}`
- Values in curly braces are wildcard Values
- After the match is made, portions of the URL path will be assigned to the following variables
  - apiCategory
  - model

---
### org.dspace.app.rest.RestResourceController [Code&rarr;](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L84-L87")

The following annotaion indicates that this class is a RestController
```
@RestController
```
This annotation registers a URL path pattern with this class.

Note, any methods in this class that annotated with a path will be relative to this path.
```
@RequestMapping("/api/{apiCategory}/{model}")
```
```
@SuppressWarnings("rawtypes")
public class RestResourceController implements InitializingBean {
```

---

### Find the appropriate method to call

The class has already matched to /api/core/communities.  We need to locate the request mapping that expects no additional path information.

---
### org.dspace.app.rest.RestResourceController.findAll() [Code&rarr;](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L769-L787")

Register this method as the handler for a URL Path relative to the class URL path.  

Note that no additional path has been specified for this method.
```
@RequestMapping(method = RequestMethod.GET)
@SuppressWarnings("unchecked")
```
This method will return a page of DSpaceResource objects.
The REST api controls the maximum number of return objects through pagination.
```
public <T extends RestAddressableModel> PagedResources<DSpaceResource<T>> findAll(
```
Remember that the following variables were extracted from the URL path at the class level.
```
    @PathVariable String apiCategory,
    @PathVariable String model,
```
Pagination and sort parameters are extracted from the URL parameters.
Parameter names confirm to Spring framework defaults.
```
    Pageable page,
    PagedResourcesAssembler assembler,
```
This annotation maps a URL parameter to a method variable.
When required is true, the method will fail if a param is not present.
This annotation is a DSpace annotation.
```
    @RequestParam(required = false) String projection,
```
```
    HttpServletResponse response) {
```
Find the [REST repository object&rarr;](#rep) that will retrieve objects from DSpace.
```
  DSpaceRestRepository<T, ?> repository = utils.getResourceRepository(apiCategory, model);
```
Create a "self" link to this method.
We will explore link creation in another section.
```
  Link link = linkTo(methodOn(this.getClass(), apiCategory, model).findAll(apiCategory, model,
      page, assembler, projection, response))
      .withSelfRel();
```
Query for all items and load only a single page of items [repository.findAll&rarr;](#repfind).

[repository::wrapResource&rarr;](#wrap) is a Java 8 lambda function.

```
  Page<DSpaceResource<T>> resources;
  try {
      resources = repository.findAll(page).map(repository::wrapResource);
```
[linkService::addLinks&rarr;](#tbd) is a Java 8 lambda function.
```
  resources.forEach(linkService::addLinks);
```
Error handling
```
  } catch (PaginationException pe) {
      resources = new PageImpl<DSpaceResource<T>>(new ArrayList<DSpaceResource<T>>(), page, pe.getTotal());
  } catch (RepositoryMethodNotImplementedException mne) {
      throw mne;
  }
```
Assemble resources in a HATEOAS compliant manner.
```
  PagedResources<DSpaceResource<T>> result = assembler.toResource(resources, link);
```
If a search method exists for this repository, create a link for it.
```
  if (repositoryUtils.haveSearchMethods(repository)) {
      result.add(linkTo(this.getClass(), apiCategory, model).slash("search").withRel("search"));
  }
  return result;
}
```
---
### <a name="rep"></a>org.dspace.app.rest.repository.CommunitiyRestRepository [Code&rarr;](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L37)
```
@Component(CommunityRest.CATEGORY + "." + CommunityRest.NAME)
public class CommunityRestRepository extends DSpaceRestRepository<CommunityRest, UUID> {

```
Spring uses java reflection to link the DSpace API's Community Service into this class.
```
      @Autowired
      CommunityService cs;

```
Spring uses java reflection to link the [CommunityConverter&rarr;](#tbd) class into the code.
This class will convert DSpace API Community objects into a representation within the REST API.
```
      @Autowired
      CommunityConverter converter;
```
---
### <a name="repfind"></a>org.dspace.app.rest.repository.CommunitiyRestRepository.findAll() [Code&rarr;](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L63-L79)
```
@Override
public Page<CommunityRest> findAll(Context context, Pageable pageable) {
     List<Community> it = null;
     List<Community> communities = new ArrayList<Community>();
     int total = 0;
     try {
```
Call the DSpace API Community Service to get a page of results.
```      
         total = cs.countTotal(context);
         it = cs.findAll(context, pageable.getPageSize(), pageable.getOffset());
         for (Community c : it) {
             communities.add(c);
         }
     } catch (SQLException e) {
         throw new RuntimeException(e.getMessage(), e);
     }
```
Convert each result in the page set into its REST representation.
```     
     Page<CommunityRest> page = new PageImpl<Community>(communities, pageable, total).map(converter);
     return page;
}
```
---
### <a name="convert"></a>Lambda: org.dspace.app.rest.converter.CommunityConverter [Code&rarr;](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/converter/CommunityConverter.java#L27-L29)
This class will convert a DSpace API Community object into a REST representation of a Community object.
```
@Component
public class CommunityConverter
    extends DSpaceObjectConverter<org.dspace.content.Community, org.dspace.app.rest.model.CommunityRest> {
```
---
### <a name="wrap"></a>Lambda: org.dspace.app.rest.repository.CommunitiyRestRepository.wrapResource() [Code&rarr;](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L121-L124)
This method will wrap a CommunityRest object into a HATEOAS compliant resource container.
```
@Override
public CommunityResource wrapResource(CommunityRest community, String... rels) {
    return new CommunityResource(community, utils, rels);
}
```  
{% include nav.html %}
