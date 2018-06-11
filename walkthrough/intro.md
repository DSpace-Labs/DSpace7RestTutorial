## DSpace 7 REST Code Walkthrough
{% include nav.html %}
#### Understanding the DSpace 7 REST API
- Before DSpace 7, most DSpace code used "plain old java objects"(POJO's)
- This tutorial will illustrate how to understand the flow of the DSpace 7 code.

#### A note about DSpace 6
- DSpace 6 introduced Hibernate for database interactions
  - This code relies on Java Annotations to build associate class files with Hibernate behavior.
  - See TBD for more information

#### DSpace 7 relies heavily on the following conventions
- [Spring Framework](TBD)
- Java annotations
- Java 8 Lamdas
- ??? Test Framework

#### Goals
- Understand how the REST Controllers are invoked
- Understand how annotations are used to interpret URL parameters and paths
- Understand how DSpace 7 implements the [HAL](TBD) and [HATEOAS](TBD) conventions
- Understand how JSON return objects are constructed
- Understand how code exceptions are converted to HTTP return status

{% include nav.html %}
