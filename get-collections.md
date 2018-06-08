## /api/core/communities

### Locate the Controller for this request

The controller for this request will match `/api/*/*`

The __[Spring MVC Framework](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.htm)__ looks for controller annotations (org.springframework.web.bind.annotation.RestController).

The paths after /api will be stored in the variables apiCaterory and model

#### [org.dspace.app.rest.RestResourceController](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L84-L87")

```
@RestController
@RequestMapping("/api/{apiCategory}/{model}")
@SuppressWarnings("rawtypes")
public class RestResourceController implements InitializingBean {
```

### Find the appropriate method to call

The class has already matched to /api/core/communities.  We need to locate the request mapping that expects no additional path information.

#### [org.dspace.app.rest.RestResourceController.findAll()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L769-L787")

```
@RequestMapping(method = RequestMethod.GET)
@SuppressWarnings("unchecked")
public <T extends RestAddressableModel> PagedResources<DSpaceResource<T>> findAll(

        # Remember that the following variables were extracted from the URL path at the class level.
        @PathVariable String apiCategory,
        @PathVariable String model,

        # Paging and sorting parameters are passed in as URL parameters.
        # Paging parameters follow Spring naming conventions
        # The interface import org.springframework.data.domain.Pageable will provide access
        Pageable page,

        PagedResourcesAssembler assembler,
        @RequestParam(required = false) String projection,
        HttpServletResponse response) {

  # Remember that apiCategory and model were set from the components of the URL path.
  # Locate the REST repository object that will retrieve communities.
  #
  # This call will be described below
  DSpaceRestRepository<T, ?> repository = utils.getResourceRepository(apiCategory, model);

  # Build the self link that will appear in the links section.  For now, just trust that this builds a link in the return object.
  Link link = linkTo(methodOn(this.getClass(), apiCategory, model).findAll(apiCategory, model,
      page, assembler, projection, response))
      .withSelfRel();

  # Query for all items and load only a single page of items.
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
#### [org.dspace.app.rest.repository.CommunitiyRestRepository](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L37)
```
@Component(CommunityRest.CATEGORY + "." + CommunityRest.NAME)
public class CommunityRestRepository extends DSpaceRestRepository<CommunityRest, UUID> {
```

#### [org.dspace.app.rest.repository.CommunitiyRestRepository.findAll()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L63-L79)
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
#### [Lamda: org.dspace.app.rest.repository.CommunitiyRestRepository.wrapResource()](https://github.com/DSpace/DSpace/blob/master/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L121-L124)
```
```  
