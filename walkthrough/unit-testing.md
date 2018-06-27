{% include navmenu.html %}

## Unit Testing in DSpace
All code within the DSpace REST API will have test coverage.

Unit and integration tests can be invoked when running a maven build on your code base.

`mvn clean install license:check -Dmaven.test.skip=false -DskipITs=false -P "!assembly" -B -V -Dsurefire.rerunFailingTestsCount=2`

A single integration test can be run in the following manner.

`mvn test -Dtest=org.dspace.app.rest.AuthenticationRestControllerIT -Dmaven.test.skip=false -DskipITs=false`

Unit and integration tests that require database functionality take place using a test h2 database. The h2 database is an in memory SQL database used for testing SQL functionality.  The database and asset store used by the automated testing framework do not persist after tests are completed.

There is a [Test Framework dspace.cfg file&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-api/src/test/data/dspaceFolder/config/local.cfg) that supports automated DSpace testing.

```
# For Unit Testing we use the H2 (in memory) database
db.driver = org.h2.Driver
db.dialect=org.hibernate.dialect.H2Dialect
# Use a 10 second database lock timeout to avoid occasional JDBC lock timeout errors
db.url = jdbc:h2:mem:test;LOCK_TIMEOUT=10000;
db.username = sa
db.password =
# H2's default schema is PUBLIC
db.schema = PUBLIC
```

When a pull request is submitted to DSpace, the automated test suite is executed by the Travis continuous integration service.

If any test fails, the pull request will be flagged in error.

#### Question: Do we have a wiki page that instructs developers on running the unit test framework?

{% include nav.html %}
