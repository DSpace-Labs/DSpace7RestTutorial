###### Understanding the DSpace 7 REST API
- Before DSpace 7, most DSpace code used "plain old java objects"(POJO's)
- This presentation will illustrate how to understand the flow of the DSpace 7 code.

+++
###### A note about DSpace 6
- DSpace 6 introduced Hibernate for database interactions
  - This code relies on Java Annotations to build associate class files with Hibernate behavior.
  - See TBD for more information
+++
###### DSpace 7 relies heavily on the following conventions
- [Spring Framework](TBD)
- Java annotations
- Java 8 Lamdas
- ??? Test Framework

+++
###### Goals
- Understand how the REST Controllers are invoked
- Understand how annotations are used to interpret URL parameters and paths
- Understand how DSpace 7 implements the [HAL](TBD) and [HATEOAS](TBD) conventions
- Understand how JSON return objects are constructed
- Understand how code exceptions are converted to HTTP return status

---
## REST Request
    /api/core/communities

---
###### Locate the Controller for this request
- The __[Spring MVC Framework](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.htm)__ looks for RestController annotations (org.springframework.web.bind.annotation.RestController).
- The controller for this request will match `/api/{apiCategory}/{model}`
  - Values in curly braces are wildcard Values
  - After the match is made, portions of the URL path will be assigned to the following variables
    - apiCategory
    - model
+++
###### [org.dspace.app.rest.RestResourceController](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L84-L87")

```
@RestController
@RequestMapping("/api/{apiCategory}/{model}")
@SuppressWarnings("rawtypes")
public class RestResourceController implements InitializingBean {
```
@[1](Annotaion indicates that this class is a RestController)
@[2](Annotation registers a URL path pattern with this class)
@[3](TBD)
@[4](TBD)
+++
##### Find the appropriate method to call

The class has already matched to /api/core/communities.  We need to locate the request mapping that expects no additional path information.
+++
###### [org.dspace.app.rest.RestResourceController.findAll()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L769-L787")

```
@RequestMapping(method = RequestMethod.GET)
@SuppressWarnings("unchecked")
public <T extends RestAddressableModel> PagedResources<DSpaceResource<T>> findAll(
        @PathVariable String apiCategory,
        @PathVariable String model,
        Pageable page,
        PagedResourcesAssembler assembler,
        @RequestParam(required = false) String projection,
        HttpServletResponse response) {
  DSpaceRestRepository<T, ?> repository = utils.getResourceRepository(apiCategory, model);
  Link link = linkTo(methodOn(this.getClass(), apiCategory, model).findAll(apiCategory, model,
      page, assembler, projection, response))
      .withSelfRel();

  Page<DSpaceResource<T>> resources;
  try {
      resources = repository.findAll(page).map(repository::wrapResource);
      resources.forEach(linkService::addLinks);
  } catch (PaginationException pe) {
      resources = new PageImpl<DSpaceResource<T>>(new ArrayList<DSpaceResource<T>>(), page, pe.getTotal());
  } catch (RepositoryMethodNotImplementedException mne) {
      throw mne;
  }
  PagedResources<DSpaceResource<T>> result = assembler.toResource(resources, link);
  if (repositoryUtils.haveSearchMethods(repository)) {
      result.add(linkTo(this.getClass(), apiCategory, model).slash("search").withRel("search"));
  }
  return result;
}
```
@[1](Register this method as the handler for a URL Path relative to the class URL path)
@[1](Note - no additional path is provided)
@[3](This method will return a page of DSpaceResource objects)
@[3](The REST api controls the maximum number of return objects through pagination)
@[4-5](Remember that the following variables were extracted from the URL path at the class level)
@[6-7](Pagination and sort parameters are extracted from the URL parameters)
@[6-7](Parameter names confirm to Spring defaults)
@[8](This annotation maps a URL parameter to a method variable)
@[8](When required is true, the method will fail if a param is not present)
@[10](Find the "REST repository" object that will retrieve objects from DSpace)
@[10](NOTE: &rarr; We will examine the Repository object in a later slide)
@[11-13](Create a "self" link to this method)
@[17](Query for all items and load only a single page of items)
@[17](NOTE: &rarr; We will trace the findAll method in a later slide)
@[17](repository::wrapResource is a Java 8 lambda function)
@[17](NOTE: &rarr; We will trace repository::wrapResource in a later slide)
@[18](linkService::addLinks is a Java 8 lambda function)
@[18](NOTE: &rarr; We will trace linkService::addLinks in a later slide)
+++
#### [org.dspace.app.rest.repository.CommunitiyRestRepository](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L37)
```
@Component(CommunityRest.CATEGORY + "." + CommunityRest.NAME)
public class CommunityRestRepository extends DSpaceRestRepository<CommunityRest, UUID> {
```
---
###### [org.dspace.app.rest.repository.CommunitiyRestRepository.findAll()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L63-L79)
```
@Override
public Page<CommunityRest> findAll(Context context, Pageable pageable) {
     List<Community> it = null;
     List<Community> communities = new ArrayList<Community>();
     int total = 0;
     try {
         total = cs.countTotal(context);
         it = cs.findAll(context, pageable.getPageSize(), pageable.getOffset());
         for (Community c : it) {
             communities.add(c);
         }
     } catch (SQLException e) {
         throw new RuntimeException(e.getMessage(), e);
     }
     Page<CommunityRest> page = new PageImpl<Community>(communities, pageable, total).map(converter);
     return page;
}
```
---
###### [Lamda: org.dspace.app.rest.repository.CommunitiyRestRepository.wrapResource()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L121-L124)
```
```  
