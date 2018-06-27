{% include navmenu.html %}

## Unit Testing in DSpace
All code within the DSpace REST API will have test coverage.

Unit and integration tests can be invoked when running a maven build on your code base.

`mvn clean install license:check -Dmaven.test.skip=false -DskipITs=false -P "!assembly" -B -V -Dsurefire.rerunFailingTestsCount=2`

A single integration test can be run in the following manner.

` mvn test -Dtest=org.dspace.app.rest.AuthenticationRestControllerIT -Dmaven.test.skip=false -DskipITs=false
`
Unit and integration tests that require database functionality take place using a test h2 database. The h2 database is an in memory SQL database used for testing SQL functionality.  The database and asset store used by the automated testing framework do not persist after tests are completed.

There is a [Test Framework dspace.cfg file&rarr;](https://github.com/DSpace/DSpace/blob/rest-tutorial/dspace-api/src/test/data/dspaceFolder/config/local.cfg) that supports automated DSpace testing.

#### Note: We probably need a DSpace Wiki page dedicated to unit testing
I was unable to find such a page

{% include nav.html %}
