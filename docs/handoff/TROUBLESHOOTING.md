# war-test — Troubleshooting (Tomcat 9 on Homebrew, macOS)

This is the “when things break” page for your exact setup.

---

## 1) `/war-test/` is 200 but `/war-test/index.html` is 404

**Why**
- `/war-test/` requests the webapp root. Your app can respond via a controller/servlet route or a welcome mapping.
- `/war-test/index.html` is a *literal static file request*. If the WAR doesn’t contain `/index.html` at web root, Tomcat returns 404.

**Fix**
Add `src/main/webapp/index.html`, rebuild, redeploy:
```bash
mkdir -p src/main/webapp
printf '%s
' '<!doctype html><html><body><h1>OK</h1></body></html>' > src/main/webapp/index.html
./gradlew clean war
```
Then redeploy per `DEPLOY.md`.

---

## 2) “Where is Tomcat actually deploying?” (Cellar vs opt paths)

You deploy to:
- `/opt/homebrew/opt/tomcat@9/libexec/webapps/`

Logs may show:
- `/opt/homebrew/Cellar/tomcat@9/9.0.113/libexec/webapps/`

**Why**
- `/opt/homebrew/opt/tomcat@9` is a symlink to the currently installed Cellar version.

**Rule**
- Use `/opt/homebrew/opt/...` in your commands/scripts so upgrades don’t break paths.

---

## 3) Which logs matter and how to tail them

Logs folder:
```bash
ls -1 /opt/homebrew/opt/tomcat@9/libexec/logs/
```

Most useful:
```bash
tail -n 200 -f /opt/homebrew/opt/tomcat@9/libexec/logs/catalina.out
```

If you need rotated daily logs:
```bash
ls -1 /opt/homebrew/opt/tomcat@9/libexec/logs/catalina.*.log
tail -n 200 /opt/homebrew/opt/tomcat@9/libexec/logs/catalina.*.log
```

What you’re looking for:
- deployment messages mentioning `war-test.war`
- `SEVERE` lines and stack traces

---

## 4) `zsh: command not found: catalina`

**Why**
- Homebrew doesn’t add Tomcat’s `bin/` scripts (like `catalina`) to your PATH automatically.

**Use these instead**
```bash
brew services start tomcat@9
brew services stop tomcat@9
brew services restart tomcat@9
```

**If you want foreground debug mode**
```bash
/opt/homebrew/opt/tomcat@9/libexec/bin/catalina.sh run
```
(Do not run foreground mode while the brew service is also running.)

---

## 5) “Address already in use” (8080 already bound)

Find the process:
```bash
lsof -nP -iTCP:8080 -sTCP:LISTEN
```

If it’s Tomcat stuck:
```bash
brew services restart tomcat@9
```

If it’s another process:
```bash
kill <PID>
# if stubborn:
kill -9 <PID>
```

Verify:
```bash
lsof -nP -iTCP:8080 -sTCP:LISTEN
curl -i http://localhost:8080/ | head -n 20
```

---

## 6) Micronaut / Spring missing-class errors in a WAR-on-Tomcat setup

Examples you saw:
- missing `MicronautBeanFactoryConfiguration`
- earlier `NoSuchBeanException`

**What this usually means**
- Your WAR *deploys*, but the runtime classpath inside external Tomcat does not match your dev/runtime expectations.
- Typical reasons:
  - dependency scope mismatch (`compileOnly` / `provided` / exclusions)
  - incompatible versions pulled transitively
  - code expecting embedded-server behavior

**Fast checks**
Confirm what got packaged inside the WAR:
```bash
jar tf build/libs/*.war | head
jar tf build/libs/*.war | grep -E 'WEB-INF/lib|micronaut|spring' | head -n 80
```

See who pulled the dependency:
```bash
./gradlew -q dependencyInsight --dependency micronaut-spring --configuration runtimeClasspath
./gradlew -q dependencyInsight --dependency micronaut-core  --configuration runtimeClasspath
```

**Best practice for sanity**
Keep a tiny static WAR test (index.html) so you can quickly prove:
- Tomcat works
- deploy works
- the pipeline is clean
before reintroducing framework complexity.

---

## 7) Deploy keeps acting “cached” / old output keeps showing

Do a clean redeploy:
```bash
brew services stop tomcat@9

WEBAPPS="/opt/homebrew/opt/tomcat@9/libexec/webapps"
rm -f  "$WEBAPPS/war-test.war"
rm -rf "$WEBAPPS/war-test"

brew services start tomcat@9
```

Then copy the new WAR and restart again (see `DEPLOY.md`).

---

## 8) Quick “what’s deployed right now?” commands

```bash
WEBAPPS="/opt/homebrew/opt/tomcat@9/libexec/webapps"
ls -la "$WEBAPPS" | grep war-test || true
```

If exploded:
```bash
ls -la "$WEBAPPS/war-test" | head
```
