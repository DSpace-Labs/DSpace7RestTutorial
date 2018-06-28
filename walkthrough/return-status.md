{% include navmenu.html %}

# Scenarios Returning a Special Return Status

---
## Unsupported Operation: GET /core/does-not-exist

This request will match the following controller:

### Controller org.dspace.app.rest.RestResourceController
```
@RestController
@RequestMapping("/api/{apiCategory}/{model}")
@SuppressWarnings("rawtypes")
public class RestResourceController implements InitializingBean {
```

Based on the path specified, the following method will be invoked.

### Method org.dspace.app.rest.RestResourceController.findAll()
```
@RequestMapping(method = RequestMethod.GET)
@SuppressWarnings("unchecked")
public <T extends RestAddressableModel> PagedResources<DSpaceResource<T>> findAll(
    Pageable page,
    PagedResourcesAssembler assembler,
    @RequestParam(required = false) String projection,
    HttpServletResponse response) {
  DSpaceRestRepository<T, ?> repository = utils.getResourceRepository(apiCategory, model);
```

This will call the following method with the following parameters
- apiCategory = core
- modelPlural = does-not-exist

### Method org.dspace.app.rest.utils.Utils.getResourceRepository()
```
public DSpaceRestRepository getResourceRepository(String apiCategory, String modelPlural) {
    String model = makeSingular(modelPlural);
    try {
        return applicationContext.getBean(apiCategory + "." + model, DSpaceRestRepository.class);
    } catch (NoSuchBeanDefinitionException e) {
        throw new RepositoryNotFoundException(apiCategory, model);
    }
}

```

This method will throw a **RepositoryNotFoundException**.

### Class org.dspace.app.exception.RepositoryNotFoundException
Note that this class extends RuntimeException
```
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "This endpoint is not found in the system")
public class RepositoryNotFoundException extends RuntimeException {
```
---

The following class handles exceptions for the Spring MVC Framework because of the presence of
- The @ControllerAdvice annotation on the class AND
- The @ExceptionHandle annotations on methods

### Class org.dspace.app.exception.DSpaceApiExceptionControllerAdvice
```
@ControllerAdvice
public class DSpaceApiExceptionControllerAdvice extends ResponseEntityExceptionHandler {

```

Since there is no handle in place for RepositoryNotFoundException or RuntimeException,
default exception handling within Spring is invoked and a **404** Status Code (**SC_NOT_FOUND**) is returned.

<div class="todo">
TODO: confirm that there is no other DSpace code triggering the 404.
</div>
---

## Missing Required Parameter: GET /core/communities/search/subCommunities

This request will match the following controller:

### Controller org.dspace.app.rest.RestResourceController
```
@RestController
@RequestMapping("/api/{apiCategory}/{model}")
@SuppressWarnings("rawtypes")
public class RestResourceController implements InitializingBean {
```

Based on the path specified, the following method will be invoked.

---
### Method org.dspace.app.rest.RestResourceController.executeSearchMethods()

Parameters
- apiCategory = core
- model = communities
- searchMethodName = subCommunities

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
---
### Method org.dspace.rest.utils.Utils.getSearchMethod()
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
---
The following method contains the matching **@SearchRestMethod** annotation.

### Method org.dspace.app.rest.repository.CommunityRepository.findSubCommunities()

```
// TODO: add method in dspace api to support direct query for subcommunities
// with pagination and authorization check
@SearchRestMethod(name = "subCommunities")
```
Note that the **parentCommunity** Parameter to this function contains the **@org.dspace.app.rest.Parameter** annotation.
Note that the annotation parameter named **required** has been set to **true**.
```
public Page<CommunityRest> findSubCommunities(@Parameter(value = "parent", required = true) UUID parentCommunity,
        Pageable pageable) {
    Context context = obtainContext();
    List<Community> subCommunities = new ArrayList<Community>();
    try {
        Community community = cs.find(context, parentCommunity);
        if (community == null) {
            throw new ResourceNotFoundException(
                CommunityRest.CATEGORY + "." + CommunityRest.NAME + " with id: " + parentCommunity + " not found");
        }
        subCommunities = community.getSubcommunities();
    } catch (SQLException e) {
        throw new RuntimeException(e.getMessage(), e);
    }
    Page<CommunityRest> page = utils.getPage(subCommunities, pageable).map(converter);
    return page;
}
```
---
### Annotation Interface org.dspace.app.rest.Parameter
```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
public @interface Parameter {
    String value() default "";
    boolean required() default false;
}
```

### Method org.dspace.app.rest.utils.Utils.executeQueryMethod()
```
public Object executeQueryMethod(DSpaceRestRepository repository, MultiValueMap<String, Object> parameters,
                                 Method method, Pageable pageable, Sort sort, PagedResourcesAssembler assembler) {

    MultiValueMap<String, Object> result = new LinkedMultiValueMap<String, Object>(parameters);
    MethodParameters methodParameters = new MethodParameters(method, PARAM_ANNOTATION);

    for (MethodParameter parameter : methodParameters.getParametersWith(Parameter.class)) {
```
The Spring Framework is able to convert each param with @Parameter Annotation into an **org.dspace.app.rest.Parameter** instance.
```
        final Parameter parameterAnnotation = parameter.getParameterAnnotation(Parameter.class);
        final String paramName = parameter.getParameterName();
        List<Object> value = parameters.get(paramName);
```
If the value associated with this Parameter is null (not found), then check to see if the annotation had marked this parameter as required.
```
        if (value == null) {
            if (parameterAnnotation.required()) {
```
In our example, the parameter named **parent** is required.  Since this parameter was not passed, an **org.dspace.app.rest.exeception.MissingParameterException** is thrown.
```
                throw new MissingParameterException(
                        String.format("Required Parameter[%s] Missing",
                                parameter.getParameterName()));
            }
            continue;
        }

        result.put(paramName, value);
    }

    return invokeQueryMethod(repository, method, result, pageable, sort);
}
```
---
In the **DSpaceApiExceptionControllerAdvice** class will be scanned for an appropriate exception handler for a MissingParameterException.

### Method org.dspace.app.exception.DSpaceApiExceptionControllerAdvice.MissingParameterException()

Note that this method will return an **HttpStatus.UNPROCESSABLE_ENTITY** or **422** status.
```
@ExceptionHandler(MissingParameterException.class)
protected void MissingParameterException(HttpServletRequest request, HttpServletResponse response, Exception ex)
    throws IOException {

    //422 is not defined in HttpServletResponse.  Its meaning is "Unprocessable Entity".
    //Using the value from HttpStatus.
    //Since this is a handled exception case, the stack trace will not be returned.
    sendErrorResponse(request, response, null,
                      ex.getMessage(),
                      HttpStatus.UNPROCESSABLE_ENTITY.value());
}

```
---
- Unauthorized action
- Empty response object

{% include nav.html %}
