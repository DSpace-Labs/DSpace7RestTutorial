{% include navmenu.html %}
# GET /api/core/communities

## Get Communities Controller
---
### Locate the Controller for this request
The __[Spring MVC Framework](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.htm)__ looks for RestController annotations (org.springframework.web.bind.annotation.RestController).

The best matched controller for this request will be defined as `/api/{apiCategory}/{model}`

---
### Class org.dspace.app.rest.RestResourceController [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L85-L88")

The following annotaion indicates that this class is a RestController
```
@RestController
```
This annotation registers a URL path pattern with this class.

Note, any controller methods defined within this class will reference a path relative to the path assigned to the class.
```
@RequestMapping("/api/{apiCategory}/{model}")
```
- Values in curly braces are wildcard Values
- After the match is made, portions of the URL path will be assigned to the following variables
  - apiCategory
  - model
```
@SuppressWarnings("rawtypes")
public class RestResourceController implements InitializingBean {
```

---

### Find the appropriate method to call

The class has already matched to /api/core/communities.  We need to locate the request mapping that expects no additional path information.

---
### Method org.dspace.app.rest.RestResourceController.findAll() [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/RestResourceController.java#L769-L787")

Register this method as the handler for a URL Path relative to the controller path defined at the class level.  

Note that no additional path has been specified for this method.  As we look at other methods, you will see that they operation on qualified paths.
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
Parameter names confirm to Spring framework defaults for pagination.
```
    Pageable page,
    PagedResourcesAssembler assembler,
```
This annotation maps a URL parameter to a method variable.
When required is true, the method will fail if a param is not present.

@RequestParam is a DSpace annotation.  This annotation will ensure consistent treatment
of required parameters throughout the code base.  It should also provide a clear
indication of parameter behavior for developers.
```
    @RequestParam(required = false) String projection,
```
```
    HttpServletResponse response) {
```
Find the REST Repository Object[See below&rarr;](#rep) that will retrieve objects from DSpace.
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
Query for all items and load only a single page of items repository.findAll [See below&rarr;](#repfind).

repository::wrapResource[See below&rarr;](#wrap) is a Java 8 lambda function that will operate on each object
as it is added to the resources collection.

```
  Page<DSpaceResource<T>> resources;
  try {
      resources = repository.findAll(page).map(repository::wrapResource);
```
linkService::addLinks[See below&rarr;](#addlink) is a Java 8 lambda function that will operate on each object
in the resources collection.
```
  resources.forEach(linkService::addLinks);
```
Error handling

If incorrect pagination parameters are provided, an empty list will be returned along with a total count of available resources.
```
  } catch (PaginationException pe) {
      resources = new PageImpl<DSpaceResource<T>>(new ArrayList<DSpaceResource<T>>(), page, pe.getTotal());
```
Other errors will be handled with an exception.

The Spring MVC Framework allows exceptions to be captured in a single place.  Different types of exceptions can trigger different
return status values.  See **Class org.dspace.app.rest.exception.DSpaceApiExceptionControllerAdvice** [Code&rarr;]( https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/exception/DSpaceApiExceptionControllerAdvice.java#L33-L94)
```
  } catch (RepositoryMethodNotImplementedException mne) {
      throw mne;
  }
```
Assemble the page of resources into "PagedResources" a HATEOAS compliant manner.
- "Creates a new PagedResources by converting the given Page into a PagedResources.PageMetadata instance and wrapping the contained elements into Resource instances. Will add pagination links based on the given the self link."
  - from [Spring Framework JavaDoc](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/web/PagedResourcesAssembler.html#toResource-org.springframework.data.domain.Page-org.springframework.hateoas.Link-)

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
## Communities Repository
---

### <a name="rep"></a>Class org.dspace.app.rest.repository.CommunitiyRestRepository  [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L37)
```
@Component(CommunityRest.CATEGORY + "." + CommunityRest.NAME)
public class CommunityRestRepository extends DSpaceRestRepository<CommunityRest, UUID> {

```
Spring uses java reflection to link the DSpace API's Community Service into this class.
```
      @Autowired
      CommunityService cs;

```
Spring uses java reflection to link the [CommunityConverter&rarr;](#convert) class into the code.
This class will convert DSpace API Community objects into a representation within the REST API.
```
      @Autowired
      CommunityConverter converter;
```
---
### <a name="repfind"></a>Method org.dspace.app.rest.repository.CommunitiyRestRepository.findAll()  [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L63-L79)
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
Convert each result in the page set into its REST representation using the CommunityConverter.
```     
     Page<CommunityRest> page = new PageImpl<Community>(communities, pageable, total).map(converter);
     return page;
}
```
---
### <a name="wrap"></a>Lambda: org.dspace.app.rest.repository.CommunitiyRestRepository.wrapResource()  [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/CommunityRestRepository.java#L121-L124)
This method will wrap a CommunityRest object into a HAL compliant resource container.
```
@Override
public CommunityResource wrapResource(CommunityRest community, String... rels) {
    return new CommunityResource(community, utils, rels);
}
```  
---
### <a name="addlink"></a>Lambda: org.dspace.app.rest.link.HalLinkService.addLinks(HalResource)  [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/link/HalLinkService.java#L98-L105)
```
public HALResource addLinks(HALResource halResource) {
    try {
        addLinks(halResource, null);
    } catch (Exception ex) {
        log.warn("Unable to add links to HAL resource " + halResource, ex);
    }
    return halResource;
}
```

This method will create the standard components displayed in the HAL Broswer (Properties, Links, Embedded Resources).

#### Method org.dspace.app.rest.link.HalLinkService.addLinks(HalResource, Pageable) [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/link/HalLinkService.java#L42-L77)

```
public void addLinks(HALResource halResource, Pageable pageable) throws Exception {
    LinkedList<Link> links = new LinkedList<>();

    List<HalLinkFactory> supportedFactories = getSupportedFactories(halResource);
    for (HalLinkFactory halLinkFactory : supportedFactories) {
        links.addAll(halLinkFactory.getLinksFor(halResource, pageable));
    }

    links.sort((Link l1, Link l2) -> ObjectUtils.compare(l1.getRel(), l2.getRel()));

    halResource.add(links);

    for (Object obj : halResource.getEmbeddedResources().values()) {
        if (obj instanceof Collection) {
            for (Object subObj : (Collection) obj) {
                if (subObj instanceof HALResource) {
                    addLinks((HALResource) subObj);
                }
            }
        } else if (obj instanceof Map) {
            for (Object subObj : ((Map) obj).values()) {
                if (subObj instanceof HALResource) {
                    addLinks((HALResource) subObj);
                }
            }
        } else if (obj instanceof EmbeddedPage) {
            for (Object subObj : ((EmbeddedPage) obj).getPageContent()) {
                if (subObj instanceof HALResource) {
                    addLinks((HALResource) subObj);
                }
            }
        } else if (obj instanceof HALResource) {
            addLinks((HALResource) obj);
        }
    }
}
```
---
## Community Converter
---
### <a name="convert"></a>Class org.dspace.app.rest.converter.CommunityConverter  [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/converter/CommunityConverter.java#L27-L29)
This class will convert a DSpace API Community object into a REST representation of a Community object.
```
@Component
public class CommunityConverter
    extends DSpaceObjectConverter<org.dspace.content.Community, org.dspace.app.rest.model.CommunityRest> {
```
---
## Community REST Model
---
### Class org.dspace.app.rest.model.CommunityRest [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/model/CommunityRest.java#L19-L68)

This class helps to construct a JSON representation of the properties of a DSpace Object.
```
public class CommunityRest extends DSpaceObjectRest {
```
Define the REST API path elements for this object
```
    public static final String NAME = "community";
    public static final String CATEGORY = RestAddressableModel.CORE;
```
Linked objects will be accessible through links and will not be described in the JSON properties.
```
    @JsonIgnore
    private BitstreamRest logo;

    private List<CollectionRest> collections;

    @LinkRest(linkClass = CollectionRest.class)
    @JsonIgnore
    public List<CollectionRest> getCollections() {
        return collections;
    }

    public void setCollections(List<CollectionRest> collections) {
        this.collections = collections;
    }

    private List<CommunityRest> subcommunities;

    @LinkRest(linkClass = CommunityRest.class)
    @JsonIgnore
    public List<CommunityRest> getSubcommunities() {
        return subcommunities;
    }

    public void setSubCommunities(List<CommunityRest> subcommunities) {
        this.subcommunities = subcommunities;
    }

    public BitstreamRest getLogo() {
        return logo;
    }

    public void setLogo(BitstreamRest logo) {
        this.logo = logo;
    }
```
The following attributes will be included in the JSON representation
```
    @Override
    public String getCategory() {
        return CATEGORY;
    }

    @Override
    public String getType() {
        return NAME;
    }

}
```
Other Standard DSpace attributes are defined in the base class.

### Class org.dspace.app.rest.model.DSpaceObjectRest [Code&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-spring-rest/src/main/java/org/dspace/app/rest/model/DSpaceObjectRest.java#L19-L68)
```
public abstract class DSpaceObjectRest extends BaseObjectRest<String> {
    private String uuid;

    private String name;
    private String handle;

    List<MetadataEntryRest> metadata;

    @Override
    public String getId() {
        return uuid;
    }

    public String getUuid() {
        return uuid;
    }

    public void setUuid(String uuid) {
        this.uuid = uuid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getHandle() {
        return handle;
    }

    public void setHandle(String handle) {
        this.handle = handle;
    }

    public List<MetadataEntryRest> getMetadata() {
        return metadata;
    }

    public void setMetadata(List<MetadataEntryRest> metadata) {
        this.metadata = metadata;
    }

    @Override
    public Class getController() {
        return RestResourceController.class;
    }
}
```

{% include nav.html %}
