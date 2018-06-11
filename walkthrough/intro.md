## DSpace 7 REST Code Walkthrough

{% include nav.html %}

Before DSpace 7, most DSpace code used "plain old java objects"(POJO's)
This tutorial will illustrate how to understand the flow of the DSpace 7 code.

DSpace 6 introduced [Hibernate](http://hibernate.org/orm/) for database interactions.  
Hibernate relies on Java Annotations to build associate class files database actions.

#### DSpace 7 relies heavily on the following non-POJO conventions
- [Spring MVC Framework](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html)
- [Java annotations](https://docs.oracle.com/javase/tutorial/java/annotations/basics.html)
- [Java 8 Lambdas](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- Test Frameworks
  - [Hamcrest Assertions](https://code.google.com/archive/p/hamcrest/wikis/Tutorial.wiki)
  - [Spring MVC Test](https://docs.spring.io/spring/docs/current/spring-framework-reference/testing.html#spring-mvc-test-framework)

#### Goals of this Walkthrough
- Understand how the REST Controllers are invoked
- Understand how annotations are used to interpret URL parameters and paths
- Understand how DSpace 7 implements the [HAL](TBD) and [HATEOAS](TBD) conventions
- Understand how JSON return objects are constructed
- Understand how code exceptions are converted to HTTP return status

{% include nav.html %}
