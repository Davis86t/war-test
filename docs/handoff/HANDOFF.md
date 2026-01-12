# war-test — Handoff Summary

This repository is used to validate WAR → Tomcat 9 deployment on macOS Apple Silicon using Java 17.

It exists to prove that a Grails-built WAR can be packaged, deployed, and served by a standalone Tomcat without any enterprise stack in the way.

## What is proven
- Tomcat 9 installed via Homebrew (tomcat@9) runs on port 8080.
- Deploying by copying a WAR into webapps/war-test.war works.
- The application responds at http://localhost:8080/war-test/.
- index.html returns 404 unless a real static file exists inside the WAR.

## Key paths
Tomcat home:
/opt/homebrew/opt/tomcat@9/libexec

Webapps:
/opt/homebrew/opt/tomcat@9/libexec/webapps/

Logs:
/opt/homebrew/opt/tomcat@9/libexec/logs/

## Canonical workflow
See DEPLOY.md.

## Failure handling
See TROUBLESHOOTING.md.

## Notes on Micronaut and Spring
Earlier failures were caused by framework components attempting to bootstrap inside external Tomcat with missing classes and beans.

The correct debugging sequence is:
1. Prove the WAR deployment pipeline using a minimal static, JSP, or GSP response.
2. Reintroduce Grails, Spring, and Micronaut incrementally with strict dependency control.

## One-command redeploy
See redeploy_wartest() in DEPLOY.md.
