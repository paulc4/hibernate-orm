
<img src="http://static.jboss.org/hibernate/images/hibernate_logo_whitebkg_200px.png" />  <img align="right" height="50" src="https://www.nuodb.com/sites/all/themes/nuodb/logo.svg" /> 

# Hibernate and NuoDB

This is a fork of Hibernate ORM (http://github.com/hibernate/hibernate-orm) to allow testing of NuoDB's Hibernate dialect.  The tests of interest are the matrix tests (which allow testing against multiple databases). Unfortunately the section on Matrix testing (in the original README below) is yet to be written.

## Running Tests

To run the matrix tests for NuoDB:

1. Firstly make sure you have our Hibernate 5 dialect jar available:

   * clone https://github.com/nuodb/HibernateDialect5
   * Run `mvn install` - see [project README](https://github.com/nuodb/HibernateDialect5/blob/master/README.md)
   * Check the version in the POM - it will be of the form `20.x.x-hib5`
      * You will need to set `DIALECT_VERSION` to `20.x.x``to match - see below

1. This project's gradle build file assumes you have your maven repository in
   the default location (`.m2` in your home directory). If so, skip this step.

   Otherwise you must tell gradle where this dependency can be found. For example
   suppose you use `m2` instead of `.m2`:
   ```
   export ADDITIONAL_REPO=~/m2/repository/com/nuodb/hibernate/nuodb-hibernate/20.x.x-hib5               (Linux/MacOS)

   set ADDITIONAL_REPO=c:\Users\yourname\m2\repository\com\nuodb\hibernate\nuodb-hibernate/20.x.x-hib5  (Windows)
   ```

1. Set the Hibernate dialect - this must match the Hibernate 5 dialect you installed earlier.
   Note the value you set _does not_ have `-hib5` in the end:
   ```
   export DIALECT_VERSION=20.x.x      (Linux/MacOS)
   set DIALECT_VERSION=20.x.x         (Windows)
   ```
   Alternatively, non-Windows user may prepend it to any command: `DIALECT_VERSION=20.x.x ./gradlew ...`

1. Compile the code: `./gradlew clean compile`

1. Tell the tests about your NuoDB database:
    1. `cp databases/nuodb/resources/hibernate.properties hibernate-core/target/resources/test`.  
    2. Edit `hibernate-core/target/resources/test/hibernate.properties` and set the database URL, username and password
       as required.

1. Run tests:

   * Execute `./gradlew clean hibernate-core:matrix_nuodb`. On Windows run `./gradlew.bat`.
     To setup gradle, see below.  The expected output is:

   ```
   6935 tests completed, 114 failed, 822 skipped
   ```

   **Note:** If you run the tests without the `clean` option you may get a weird internal error in the compiler.

   **Note:** Not all tests clean up after themselves.  You may need to drop the DBO schema used by the tests by
       running "`DROP SCHEMA DBO CASCADE`".

1. Run individual tests

   Example commands:

   ```
   ./gradlew clean :hibernate-core:test --tests org.hibernate.jpa.test.packaging.PackagedEntityManagerTest
   ./gradlew clean :hibernate-core:test --tests *.PackagedEntityManagerTest
   ./gradlew clean :hibernate-core:test --tests org.hibernate.jpa.test.packaging.*
   ```

1. Pull Jar from Sonatype
   Once our jar is put up at Sonatype, its URL is something like https://oss.sonatype.org/content/repositories/comnuodb-NNNN/com/nuodb/hibernate/nuodb-hibernate/20.x.x-hib5/nuodb-hibernate-20.x.x-hib5.jar.
   Note the build number - NNNN (a 4 digit number such as 1050). To use this dependency run as follows:

   ```
   SONATYPE_VERSION=NNNN gradle clean ...   (Linux)

   set SONATYPE_VERSION=NNNN                (Windows)
   gradle clean ...
   ```

Please note that even if NuoDB is not available, 3603 tests complete, 1922 fail, and 801 are skipped. So 880 tests pass without using the database because the tests are intended for testing Hibernate not the underlying database.  We are just piggybacking on them for convenience.

## Upgrade Hibernate Dialect

If the Hibernate dialect has a new vewrsion number:

1. Update the environment variable: `SET DIALECT_VERSION=20.x.x`

This variable is used in three places and should get picked up by all of them:

* `build.gradle`
* `databases/nuodb/matrix.gradle`
* `hibernate-core/hibernate-core.gradle`

## Upgrade NuoDB JDBC Driver

This must be changed manually in two places:

1. `databases/nuodb/matrix.gradle`: `jdbcDependency "com.nuodb.jdbc:nuodb-jdbc:20.0.0"`
2. `hibernate-core/hibernate-core.gradle`:  `testRuntime( "com.nuodb.jdbc:nuodb-jdbc:20.0.0" )`

## Changes Made to Project

To use NuoDB

1. Added `databases/nuodb` to define dependencies and configuration required to use NuoDB.

1. Added references to the NuoDB dialect and/or NuoDB JDBC jars to:
     * `build.gradle`
     * `databases/nuodb/matrix.gradle`
     * `hibernate-core/hibernate-core.gradle`

To configure NuoDB

1. Set the versions of NuoDB's JDBC and Dialect Jars in  [`databases/nuodb/matrix.gradle`](databases/nuodb/matrix.gradle)
2. To configure the NuoDB data source modify [`databases/nuodb/resources/hibernate.properties`](databases/nuodb/resources/hibernate.properties)
3. Make same modifications to [`hibernate-core/src/test/resources/hibernate.properties`](hibernate-core/src/test/resources/hibernate.properties) - this is the one that actually gets used.

## To Run in IntelliJ

It is possible to run the tests in IntelliH (Eclipse's gradle support can't handle this project).

Open as a gradle project in IntelliJ in the usual way.

To force it to use NuoDB: `cp databases/nuodb/resources/hibernate.properties hibernate-core/out/test/resources/hibernate.properties`.


# Original README

Hibernate ORM is a library providing Object/Relational Mapping (ORM) support
to applications, libraries, and frameworks.

It also provides an implementation of the JPA specification, which is the standard Java specification for ORM.

This is the repository of its source code: see [Hibernate.org](http://hibernate.org/orm/) for additional information.

[![Build Status](http://ci.hibernate.org/job/hibernate-orm-main-h2-main/badge/icon)](http://ci.hibernate.org/job/hibernate-orm-main-h2-main/)
[![Language grade: Java](https://img.shields.io/lgtm/grade/java/g/hibernate/hibernate-orm.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/hibernate/hibernate-orm/context:java)

Building from sources
=========

The build requires a Java 8 JDK as JAVA_HOME.

You will need [Git](https://git-scm.com/) to obtain the [source](https://github.com/hibernate/hibernate-orm/).

Hibernate uses [Gradle](https://gradle.org) as its build tool.  See the _Gradle Primer_ section below if you are new to
Gradle.

Contributors should read the [Contributing Guide](CONTRIBUTING.md).

See the guides for setting up [IntelliJ](http://hibernate.org/community/contribute/intellij-idea/) or
[Eclipse](https://hibernate.org/community/contribute/eclipse-ide/) as your development environment.

Check out the _Getting Started_ section in CONTRIBUTING.md for getting started working on Hibernate source.


Continuous Integration
=========

Hibernate makes use of [Jenkins](http://jenkins-ci.org) for its CI needs.  The project is built continuous on each 
push to the upstream repository.   Overall there are a few different jobs, all of which can be seen at 
[http://ci.hibernate.org/view/ORM/](http://ci.hibernate.org/view/ORM/)


Gradle primer
=========

This section describes some of the basics developers and contributors new to Gradle might 
need to know to get productive quickly.  The Gradle documentation is very well done; 2 in 
particular that are indispensable:

* [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide_single.html) is a typical user guide in that
it follows a topical approach to describing all of the capabilities of Gradle.
* [Gradle DSL Guide](https://docs.gradle.org/current/dsl/index.html) is unique and excellent in quickly
getting up to speed on certain aspects of Gradle.


Using the Gradle Wrapper
------------------------

For contributors who do not otherwise use Gradle and do not want to install it, Gradle offers a very cool
feature called the wrapper.  It lets you run Gradle builds without a previously installed Gradle distro in 
a zero-conf manner.  Hibernate configures the Gradle wrapper for you.  If you would rather use the wrapper and 
not install Gradle (or to make sure you use the version of Gradle intended for older builds) you would just use
the command `gradlew` (or `gradlew.bat`) rather than `gradle` (or `gradle.bat`) in the following discussions.
Note that `gradlew` is only available in the project's root dir, so depending on your working directory you may
need to adjust the path to `gradlew` as well.

Examples use the `gradle` syntax, but just swap `gradlew` (properly relative) for `gradle` if you wish to use 
the wrapper.

Another reason to use `gradlew` is that it uses the exact version of Gradle that the build is defined to work with.


Executing Tasks
------------------------

Gradle uses the concept of build tasks (equivalent to Ant targets or Maven phases/goals). You can get a list of
available tasks via 

    gradle tasks

To execute a task across all modules, simply perform that task from the root directory.  Gradle will visit each
sub-project and execute that task if the sub-project defines it.  To execute a task in a specific module you can 
either:

1. `cd` into that module directory and execute the task
2. name the "task path".  For example, to run the tests for the _hibernate-core_ module from the root directory you could say `gradle hibernate-core:test`

Common Java related tasks
------------------------

* _build_ - Assembles (jars) and tests this project
* _buildDependents_ - Assembles and tests this project and all projects that depend on it.  So think of running this in hibernate-core, Gradle would assemble and test hibernate-core as well as hibernate-envers (because envers depends on core)
* _classes_ - Compiles the main classes
* _testClasses_ - Compiles the test classes
* _compile_ (Hibernate addition) - Performs all compilation tasks including staging resources from both main and test
* _jar_ - Generates a jar archive with all the compiled classes
* _test_ - Runs the tests
* _publish_ - Think Maven deploy
* _publishToMavenLocal_ - Installs the project jar to your local maven cache (aka ~/.m2/repository).  Note that Gradle 
never uses this, but it can be useful for testing your build with other local Maven-based builds.
* _eclipse_ - Generates an Eclipse project
* _idea_ - Generates an IntelliJ/IDEA project (although the preferred approach is to use IntelliJ's Gradle import).
* _clean_ - Cleans the build directory


Testing and databases
=====================

Testing against a specific database can be achieved in 2 different ways:


Using the "Matrix Testing Plugin" for Gradle.
---------------------------------------------

Coming soon...


Using "profiles"
------------------------

The Hibernate build defines several database testing "profiles" in `databases.gradle`.  These
profiles can be activated by name using the `db` build property which can be passed either as
a JVM system prop (`-D`) or as a Gradle project property (`-P`).  Examples below use the Gradle
project property approach.

    gradle clean build -Pdb=pgsql

To run a test from your IDE, you need to ensure the property expansions happen.
Use the following command:

    gradle clean compile -Pdb=pgsql

_*NOTE: If you are running tests against a JDBC driver that is not available via Maven central be sure to add these drivers to your local Maven repo cache (~/.m2/repository) or (better) add it to a personal Maven repo server*_

Running database-specific tests from the IDE using "profiles"
-------------------------------------------------------------

You can run any test on any particular database that is configured in a `databases.gradle` profile.

All you have to do is run the following command:

    gradlew setDataBase -Pdb=pgsql
    
or you can use the shortcut version:    

    gradlew sDB -Pdb=pgsql
    
You can do this from the module which you are interested in testing or from the `hibernate-orm` root folder.

Afterward, just pick any test from the IDE and run it as usual. Hibernate will pick the database configuration from the `hibernate.properties`
file that was set up by the `setDataBase` Gradle task.

Starting test databases locally as docker containers
-------------------------------------------------------------

You don't have to install all databases locally to be able to test against them in case you have docker available.
The script `docker_db.sh` allows you to start a pre-configured database which can be used for testing.

All you have to do is run the following command:

    ./docker_db.sh postgresql_9_5

omitting the argument will print a list of possible options.

When the database is properly started, you can run tests with special profiles that are suffixed with `_ci`
e.g. `pgsql_ci` for PostgreSQL. By using the system property `dbHost` you can configure the IP address of your docker host.

The command for running tests could look like the following:

    gradlew test -Pdb=pgsql_ci "-DdbHost=192.168.99.100"