{% include navmenu.html %}

## EPerson Functionality
<div class="todo">
**This page is under construction**
</div>
---
See code in [or2018-workshop commits](https://github.com/DSpace/DSpace/compare/master...4Science:or2018-workshop) and reference these changes in examples.

These code examples illustrate the addition of functionality for EPersons.  These changes are not yet merged.

### [Auth test](https://github.com/4Science/DSpace/commit/07b4926a762d050a3d5c8eaa2e612b820fd36bf1)
dspace-spring-rest/src/test/java/org/dspace/app/rest/EPersonRestRepositoryIT.java
```
public class EPersonRestRepositoryIT extends AbstractControllerIntegrationTest {

    @Test
    public void findAllTest() throws Exception {
        context.turnOffAuthorisationSystem();

        EPerson newUser = EPersonBuilder.createEPerson(context)
                                        .withNameInMetadata("John", "Doe")
                                        .withEmail("Johndoe@gmail.com")
                                        .build();

        String authToken = getAuthToken(admin.getEmail(), password);
        getClient(authToken).perform(get("/api/eperson/eperson"))
                   .andExpect(status().isOk())
                   .andExpect(content().contentType(contentType))
                   .andExpect(jsonPath("$._embedded.epersons", Matchers.containsInAnyOrder(
                       EPersonMatcher.matchEPersonEntry(newUser),
                       EPersonMatcher.matchDefaultTestEPerson(),
                       EPersonMatcher.matchDefaultTestEPerson()
                   )))
                   .andExpect(jsonPath("$.page.size", is(20)))
                   .andExpect(jsonPath("$.page.totalElements", is(3)))
        ;
    }

    @Test
    public void findAllUnauthorizedTest() throws Exception {
        context.turnOffAuthorisationSystem();

        EPerson newUser = EPersonBuilder.createEPerson(context)
                                        .withNameInMetadata("John", "Doe")
                                        .withEmail("Johndoe@gmail.com")
                                        .build();

        getClient().perform(get("/api/eperson/eperson"))
                   .andExpect(status().isUnauthorized());
    }

    @Test
    public void findAllForbiddenTest() throws Exception {
        context.turnOffAuthorisationSystem();

        EPerson newUser = EPersonBuilder.createEPerson(context)
                                        .withNameInMetadata("John", "Doe")
                                        .withEmail("Johndoe@gmail.com")
                                        .build();

        String authToken = getAuthToken(eperson.getEmail(), password);
        getClient(authToken).perform(get("/api/eperson/eperson"))
                            .andExpect(status().isForbidden());
    }

    @Test
    public void findAllPaginationTest() throws Exception {
        context.turnOffAuthorisationSystem();

        EPerson ePerson = EPersonBuilder.createEPerson(context)
                                        .withNameInMetadata("John", "Doe")
                                        .withEmail("Johndoe@gmail.com")
                                        .build();

        String authToken = getAuthToken(admin.getEmail(), password);
        // using size = 2 the first page will contains our test user and admin
        getClient(authToken).perform(get("/api/eperson/eperson")
                                .param("size", "2"))
                   .andExpect(status().isOk())
                   .andExpect(content().contentType(contentType))
                   .andExpect(jsonPath("$._embedded.epersons", Matchers.containsInAnyOrder(
                           EPersonMatcher.matchDefaultTestEPerson(),
                           EPersonMatcher.matchDefaultTestEPerson()
                   )))
                   .andExpect(jsonPath("$._embedded.epersons", Matchers.not(
                       Matchers.contains(
                           EPersonMatcher.matchEPersonEntry(ePerson)
                       )
                   )))
                   .andExpect(jsonPath("$.page.size", is(2)))
                   .andExpect(jsonPath("$.page.totalElements", is(3)))
        ;

        // using size = 2 the first page will contains our test user and admin
        getClient(authToken).perform(get("/api/eperson/eperson")
                                .param("size", "2")
                                .param("page", "1"))
                   .andExpect(status().isOk())
                   .andExpect(content().contentType(contentType))
                   .andExpect(jsonPath("$._embedded.epersons", Matchers.contains(
                       EPersonMatcher.matchEPersonEntry(ePerson)
                   )))
                   .andExpect(jsonPath("$.page.size", is(2)))
                   .andExpect(jsonPath("$.page.totalElements", is(3)))
        ;
    }


    @Test
    public void findOneTest() throws Exception {
        context.turnOffAuthorisationSystem();

        EPerson ePerson = EPersonBuilder.createEPerson(context)
                                        .withNameInMetadata("John", "Doe")
                                        .withEmail("Johndoe@gmail.com")
                                        .build();

        EPerson ePerson2 = EPersonBuilder.createEPerson(context)
                                         .withNameInMetadata("Jane", "Smith")
                                         .withEmail("janesmith@gmail.com")
                                         .build();

        String authToken = getAuthToken(admin.getEmail(), password);
        getClient(authToken).perform(get("/api/eperson/epersons/" + ePerson2.getID()))
                   .andExpect(status().isOk())
                   .andExpect(content().contentType(contentType))
                   .andExpect(jsonPath("$", is(
                       EPersonMatcher.matchEPersonEntry(ePerson2)
                   )))
                   .andExpect(jsonPath("$", Matchers.not(
                       is(
                           EPersonMatcher.matchEPersonEntry(ePerson)
                       )
                   )));

    }

    @Test
    public void findOneRelsTest() throws Exception {
        context.turnOffAuthorisationSystem();

        EPerson ePerson = EPersonBuilder.createEPerson(context)
                                        .withNameInMetadata("John", "Doe")
                                        .withEmail("Johndoe@gmail.com")
                                        .build();

        EPerson ePerson2 = EPersonBuilder.createEPerson(context)
                                         .withNameInMetadata("Jane", "Smith")
                                         .withEmail("janesmith@gmail.com")
                                         .build();

        String authToken = getAuthToken(admin.getEmail(), password);
        getClient(authToken).perform(get("/api/eperson/epersons/" + ePerson2.getID()))
                   .andExpect(status().isOk())
                   .andExpect(content().contentType(contentType))
                   .andExpect(jsonPath("$", is(
                       EPersonMatcher.matchEPersonEntry(ePerson2)
                   )))
                   .andExpect(jsonPath("$", Matchers.not(
                       is(
                           EPersonMatcher.matchEPersonEntry(ePerson)
                       )
                   )))
                   .andExpect(jsonPath("$._links.self.href",
                                       Matchers.containsString("/api/eperson/epersons/" + ePerson2.getID())));
    }


    @Test
    public void findOneTestWrongUUID() throws Exception {
        context.turnOffAuthorisationSystem();

        EPerson ePerson = EPersonBuilder.createEPerson(context)
                                        .withNameInMetadata("John", "Doe")
                                        .withEmail("Johndoe@gmail.com")
                                        .build();

        EPerson ePerson2 = EPersonBuilder.createEPerson(context)
                                         .withNameInMetadata("Jane", "Smith")
                                         .withEmail("janesmith@gmail.com")
                                         .build();

        String authToken = getAuthToken(admin.getEmail(), password);
        getClient(authToken).perform(get("/api/eperson/epersons/" + UUID.randomUUID()))
                   .andExpect(status().isNotFound());

    }
}
```
### [Auth fix](https://github.com/DSpace/DSpace/commit/1d7efbd89d6e18605101350362d1a8baa9bec9c6)
dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/EPersonRestRepository.java
```
@Autowired
AuthorizeService authorizeService;
```

```
@Override
public Page<EPersonRest> findAll(Context context, Pageable pageable) {
    List<EPerson> epersons = null;
    int total = 0;
    try {
        if (!authorizeService.isAdmin(context)) {
            throw new RESTAuthorizationException(
                    "The EPerson collection endpoint is reserved to system administrators");
        }
        total = es.countTotal(context);
        epersons = es.findAll(context, EPerson.ID, pageable.getPageSize(), pageable.getOffset());
    } catch (SQLException e) {
        throw new RuntimeException(e.getMessage(), e);
    }
    Page<EPersonRest> page = new PageImpl<EPerson>(epersons, pageable, total).map(converter);
    return page;
}

```
### [Search test](https://github.com/DSpace/DSpace/commit/3e7ab875d774eb9da4e819545dd21709663eeb37)
dspace-spring-rest/src/test/java/org/dspace/app/rest/EPersonRestRepositoryIT.java
```
@Test
 public void searchMethodsExist() throws Exception {
     String authToken = getAuthToken(admin.getEmail(), password);
     getClient(authToken).perform(get("/api/eperson/epersons"))
             .andExpect(jsonPath("$._links.search.href", Matchers.notNullValue()));

     getClient(authToken).perform(get("/api/eperson/epersons/search"))
     .andExpect(status().isOk())
     .andExpect(content().contentType(contentType))
     .andExpect(jsonPath("$._links.byEmail", Matchers.notNullValue()))
     .andExpect(jsonPath("$._links.byName", Matchers.notNullValue()));
 }

 @Test
 public void findByEmail() throws Exception {
     context.turnOffAuthorisationSystem();

     EPerson ePerson = EPersonBuilder.createEPerson(context)
                                     .withNameInMetadata("John", "Doe")
                                     .withEmail("Johndoe@fake-email.com")
                                     .build();

     // create a second eperson to put the previous one in a no special position (is not the first as we have default
     // epersons is not the latest created)
     EPerson ePerson2 = EPersonBuilder.createEPerson(context)
                                      .withNameInMetadata("Jane", "Smith")
                                      .withEmail("janesmith@fake-email.com")
                                      .build();

     String authToken = getAuthToken(admin.getEmail(), password);
     getClient(authToken).perform(get("/api/eperson/epersons/search/byEmail")
                 .param("email", ePerson.getEmail()))
                 .andExpect(status().isOk())
                 .andExpect(content().contentType(contentType))
                 .andExpect(jsonPath("$", is(
                     EPersonMatcher.matchEPersonEntry(ePerson)
                 )));

     // it must be case-insensitive
     getClient(authToken).perform(get("/api/eperson/epersons/search/byEmail")
             .param("email", ePerson.getEmail().toUpperCase()))
             .andExpect(status().isOk())
             .andExpect(content().contentType(contentType))
             .andExpect(jsonPath("$", is(
                 EPersonMatcher.matchEPersonEntry(ePerson)
             )));
 }

 @Test
 public void findByEmailUndefined() throws Exception {
     String authToken = getAuthToken(admin.getEmail(), password);
     getClient(authToken).perform(get("/api/eperson/epersons/search/byEmail")
             .param("email", "undefined@undefined.com"))
             .andExpect(status().isNotFound());
 }

 @Test
 public void findByEmailUnprocessable() throws Exception {
     String authToken = getAuthToken(admin.getEmail(), password);
     getClient(authToken).perform(get("/api/eperson/epersons/search/byEmail"))
             .andExpect(status().isUnprocessableEntity());
 }

 @Test
 public void findByName() throws Exception {
     context.turnOffAuthorisationSystem();
     EPerson ePerson = EPersonBuilder.createEPerson(context)
                                     .withNameInMetadata("John", "Doe")
                                     .withEmail("Johndoe@fake-email.com")
                                     .build();

     EPerson ePerson2 = EPersonBuilder.createEPerson(context)
                                      .withNameInMetadata("Jane", "Smith")
                                      .withEmail("janesmith@fake-email.com")
                                      .build();

     EPerson ePerson3 = EPersonBuilder.createEPerson(context)
             .withNameInMetadata("Tom", "Doe")
             .withEmail("tomdoe@fake-email.com")
             .build();

     EPerson ePerson4 = EPersonBuilder.createEPerson(context)
             .withNameInMetadata("Dirk", "Doe-Postfix")
             .withEmail("dirkdoepostfix@fake-email.com")
             .build();

     EPerson ePerson5 = EPersonBuilder.createEPerson(context)
             .withNameInMetadata("Harry", "Prefix-Doe")
             .withEmail("harrydoeprefix@fake-email.com")
             .build();

     String authToken = getAuthToken(admin.getEmail(), password);
     getClient(authToken).perform(get("/api/eperson/epersons/search/byName")
                 .param("q", ePerson.getLastName()))
                 .andExpect(status().isOk())
                 .andExpect(content().contentType(contentType))
                 .andExpect(jsonPath("$._embedded.epersons", Matchers.containsInAnyOrder(
                         EPersonMatcher.matchEPersonEntry(ePerson),
                         EPersonMatcher.matchEPersonEntry(ePerson3),
                         EPersonMatcher.matchEPersonEntry(ePerson4),
                         EPersonMatcher.matchEPersonEntry(ePerson5)
                 )))
                 .andExpect(jsonPath("$.page.totalElements", is(4)));

     // it must be case insensitive
     getClient(authToken).perform(get("/api/eperson/epersons/search/byName")
             .param("q", ePerson.getLastName().toLowerCase()))
             .andExpect(status().isOk())
             .andExpect(content().contentType(contentType))
             .andExpect(jsonPath("$._embedded.epersons", Matchers.containsInAnyOrder(
                     EPersonMatcher.matchEPersonEntry(ePerson),
                     EPersonMatcher.matchEPersonEntry(ePerson3),
                     EPersonMatcher.matchEPersonEntry(ePerson4),
                     EPersonMatcher.matchEPersonEntry(ePerson5)
             )))
             .andExpect(jsonPath("$.page.totalElements", is(4)));
 }

 @Test
 public void findByNameUndefined() throws Exception {
     String authToken = getAuthToken(admin.getEmail(), password);
     getClient(authToken).perform(get("/api/eperson/epersons/search/byName")
             .param("q", "Doe, John"))
     .andExpect(status().isOk())
     .andExpect(content().contentType(contentType))
     .andExpect(jsonPath("$.page.totalElements", is(0)));
 }

 @Test
 public void findByNameUnprocessable() throws Exception {
     String authToken = getAuthToken(admin.getEmail(), password);
     getClient(authToken).perform(get("/api/eperson/epersons/search/byName"))
             .andExpect(status().isUnprocessableEntity());
 }

}
```
### [Search stub](https://github.com/DSpace/DSpace/commit/84ce25070eb7078457bd4d15f5a76a7e306871b7)
dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/EPersonRestRepository.java
```
@SearchRestMethod(name = "byName")
public Page<EPersonRest> findByName(@Parameter(value = "q") String q,
       Pageable pageable) {
   return null;
}

@SearchRestMethod(name = "byEmail")
public EPersonRest findByEmail(@Parameter(value = "email") String email) {
  return null;
}
```
### [Search mand param](https://github.com/DSpace/DSpace/commit/ef4695903bbf60022aac4b2748ad1e949846b34b)
```
@SearchRestMethod(name = "byName")
public Page<EPersonRest> findByName(@Parameter(value = "q", required = true) String q,
        Pageable pageable) {
    return null;
}

@SearchRestMethod(name = "byEmail")
public EPersonRest findByEmail(@Parameter(value = "email", required = true) String email) {
    return null;
}

```
### [Search impl](https://github.com/DSpace/DSpace/commit/162cc7a01d13332aea6fd01a877ea91bb18bf15b)
 dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/EPersonRestRepository.java
```
@SearchRestMethod(name = "byName")
public Page<EPersonRest> findByName(@Parameter(value = "q", required = true) String q,
        Pageable pageable) {
    List<EPerson> epersons = null;
    int total = 0;
    try {
        Context context = obtainContext();
        epersons = es.search(context, q, pageable.getOffset(), pageable.getOffset() + pageable.getPageSize());
        total = es.searchResultCount(context, q);
    } catch (SQLException e) {
        throw new RuntimeException(e.getMessage(), e);
    }
    Page<EPersonRest> page = new PageImpl<EPerson>(epersons, pageable, total).map(converter);
    return page;
}

@SearchRestMethod(name = "byEmail")
public EPersonRest findByEmail(@Parameter(value = "email", required = true) String email) {
    EPerson eperson = null;
    try {
        Context context = obtainContext();
        eperson = es.findByEmail(context, email);
    } catch (SQLException e) {
        throw new RuntimeException(e.getMessage(), e);
    }
    if (eperson == null) {
        return null;
    }
    return converter.fromModel(eperson);
}
```
### [Delete IT](https://github.com/DSpace/DSpace/commit/d698d3df859b9d75e7fc43f302569f97ebe18e8a)
dspace-spring-rest/src/test/java/org/dspace/app/rest/EPersonRestRepositoryIT.java
```
@Test
public void deleteOne() throws Exception {
    context.turnOffAuthorisationSystem();
    EPerson ePerson = EPersonBuilder.createEPerson(context)
                                    .withNameInMetadata("John", "Doe")
                                    .withEmail("Johndoe@fake-email.com")
                                    .build();

    String token = getAuthToken(admin.getEmail(), password);

    // Delete
    getClient(token).perform(delete("/api/eperson/epersons/" + ePerson.getID()))
            .andExpect(status().is(204));

    // Verify 404 after delete
    getClient().perform(get("/api/eperson/epersons/" + ePerson.getID()))
            .andExpect(status().isNotFound());
}

@Test
public void deleteUnauthorized() throws Exception {
    context.turnOffAuthorisationSystem();
    EPerson ePerson = EPersonBuilder.createEPerson(context)
                                    .withNameInMetadata("John", "Doe")
                                    .withEmail("Johndoe@fake-email.com")
                                    .build();

    // login as a basic user
    String token = getAuthToken(eperson.getEmail(), password);

    // Delete using an unauthorized user
    getClient(token).perform(delete("/api/eperson/epersons/" + ePerson.getID()))
            .andExpect(status().isForbidden());

    // login as admin
    String adminToken = getAuthToken(admin.getEmail(), password);

    // Verify the eperson is still here
    getClient(adminToken).perform(get("/api/eperson/epersons/" + ePerson.getID()))
            .andExpect(status().isOk());
}

@Test
public void deleteForbidden() throws Exception {
    context.turnOffAuthorisationSystem();
    EPerson ePerson = EPersonBuilder.createEPerson(context)
                                    .withNameInMetadata("John", "Doe")
                                    .withEmail("Johndoe@fake-email.com")
                                    .build();

    // Delete as anonymous user
    getClient().perform(delete("/api/eperson/epersons/" + ePerson.getID()))
            .andExpect(status().isUnauthorized());

    // login as admin
    String adminToken = getAuthToken(admin.getEmail(), password);

    // Verify the eperson is still here
    getClient(adminToken).perform(get("/api/eperson/epersons/" + ePerson.getID()))
            .andExpect(status().isOk());
}

@Ignore
@Test
public void deleteViolatingConstraints() throws Exception {
    // We turn off the authorization system in order to create the structure as defined below
    context.turnOffAuthorisationSystem();

    EPerson ePerson = EPersonBuilder.createEPerson(context)
            .withNameInMetadata("Sample", "Submitter")
            .withEmail("submitter@fake-email.com")
            .build();

    // force the use of the new user for subsequent operation
    context.setCurrentUser(ePerson);

    // ** GIVEN **
    // 1. A community with a logo
    parentCommunity = CommunityBuilder.createCommunity(context).withName("Community").withLogo("logo_community")
                                      .build();

    // 2. A collection with a logo
    Collection col = CollectionBuilder.createCollection(context, parentCommunity).withName("Collection")
                                      .withLogo("logo_collection").build();


    // 3. Create an item that will prevent the deletation of the eperson account (it is the submitter)
    Item item = ItemBuilder.createItem(context, col).build();

    String token = getAuthToken(admin.getEmail(), password);

    // 422 error when trying to DELETE the eperson=submitter
    getClient(token).perform(delete("/api/eperson/epersons/" + ePerson.getID()))
               .andExpect(status().is(422));

    // Verify the eperson is still here
    getClient(token).perform(get("/api/eperson/epersons/" + ePerson.getID()))
            .andExpect(status().isOk());
}
```
### [Delete Impl](https://github.com/DSpace/DSpace/commit/d7aeff721dbadc604cad7a3be5d96986dfd3f5f7)
dspace-spring-rest/src/main/java/org/dspace/app/rest/repository/EPersonRestRepository.java
```
@Override
protected void delete(Context context, UUID id) throws AuthorizeException, RepositoryMethodNotImplementedException {
    EPerson eperson = null;
    try {
        eperson = es.find(context, id);
        List<String> constraints = es.getDeleteConstraints(context, eperson);
        if (constraints != null && constraints.size() > 0) {
            throw new UnprocessableEntityException(
                    "The eperson cannot be deleted due to the following constraints: "
                            + StringUtils.join(constraints, ", "));
        }
    } catch (SQLException e) {
        throw new RuntimeException(e.getMessage(), e);
    }
    try {
        es.delete(context, eperson);
    } catch (SQLException | IOException e) {
        throw new RuntimeException(e.getMessage(), e);
    }
}

```

{% include nav.html %}
