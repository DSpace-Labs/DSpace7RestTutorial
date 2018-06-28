{% include navmenu.html %}

## EPerson Functionality
---
See code in [or2018-workshop commits](https://github.com/DSpace/DSpace/compare/master...4Science:or2018-workshop) and reference these changes in examples.

Ideally, this code would exist in the code base.

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
### [Search stub](https://github.com/DSpace/DSpace/commit/84ce25070eb7078457bd4d15f5a76a7e306871b7)
### [Search mand param](https://github.com/DSpace/DSpace/commit/ef4695903bbf60022aac4b2748ad1e949846b34b)
### [Search impl](https://github.com/DSpace/DSpace/commit/162cc7a01d13332aea6fd01a877ea91bb18bf15b)
### [Delete IT](https://github.com/DSpace/DSpace/commit/d698d3df859b9d75e7fc43f302569f97ebe18e8a)
### [Delete Impl](https://github.com/DSpace/DSpace/commit/d7aeff721dbadc604cad7a3be5d96986dfd3f5f7)

{% include nav.html %}
