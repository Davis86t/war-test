# war-test
Grails → Tomcat WAR Deployment Sandbox

This repository is a minimal, reproducible test harness for building, packaging, and deploying a Grails application as a .war file into a standalone Tomcat instance.

It exists for one purpose:

To isolate Grails + Gradle + Micronaut + Spring + Tomcat behavior outside of Banner, PageBuilder, or any enterprise stack.

If it breaks here, it is a framework problem.
If it works here but not in production, it is an environment problem.

--------------------------------------------------

WHY THIS EXISTS

University stacks (Banner, PageBuilder, Ellucian, etc.) introduce:
- Custom classloaders
- Shaded JARs
- Non-standard Tomcat configs
- Legacy Spring contexts
- Micronaut bridges
- Grails overlays

When WAR deployment fails inside those systems, it is impossible to know which layer is responsible.

This repo removes all of that.

It gives you:
- A clean Grails app
- A clean Gradle build
- A clean Tomcat WAR deploy
With nothing else in the way.

--------------------------------------------------

WHAT THIS REPO IS

This repo contains:
- A Grails app configured for WAR packaging
- A Gradle build configured for reproducible WAR output
- A Tomcat 9 compatible deployment target
- Dependency resolution tooling
- A place to experiment with Micronaut vs Spring, classloaders, plugins, and WAR behavior

This is not an application.
It is a diagnostic rig.

--------------------------------------------------

WHAT THIS REPO IS NOT

This repo does not contain:
- Ellucian code
- Banner
- PageBuilder
- University business logic
- Credentials
- Proprietary libraries

Everything here is framework-level and safe to publish.

--------------------------------------------------

TYPICAL WORKFLOW

Clean build:
./gradlew --stop
rm -rf build
./gradlew clean war

Inspect dependencies:
./gradlew -q dependencyInsight --dependency micronaut-core --configuration runtimeClasspath
./gradlew -q dependencyInsight --dependency spring-web --configuration runtimeClasspath

Deploy:
cp build/libs/*.war $TOMCAT_HOME/webapps/war-test.war
catalina run

--------------------------------------------------

HOW TO READ FAILURES

If it fails here, it is a Grails or framework problem.
If it works here but not in Banner or PageBuilder, it is stack pollution.

This repo is the control group.

--------------------------------------------------

OWNERSHIP

This repository is developer-owned tooling for exposing WAR behavior without enterprise interference.
