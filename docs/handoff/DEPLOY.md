# war-test — Deploy to Homebrew Tomcat 9 (macOS Apple Silicon)

Assumptions (matches Tanner’s setup):
- Tomcat installed via Homebrew: `tomcat@9`
- Tomcat managed via `brew services`
- Java 17 via SDKMAN is already configured in your shell
- Tomcat home (stable symlink): `/opt/homebrew/opt/tomcat@9/libexec`
- Logs: `/opt/homebrew/opt/tomcat@9/libexec/logs/`
- Deployment folder: `/opt/homebrew/opt/tomcat@9/libexec/webapps/`
- This project builds a WAR with: `./gradlew clean war`

> Note on paths: logs may show the **Cellar** path (e.g. `/opt/homebrew/Cellar/tomcat@9/9.0.113/...`). That’s normal — `/opt/homebrew/opt/...` is a symlink to the current Cellar version.

---

## 0) Quick health checks

### Tomcat running?
```bash
brew services list | grep tomcat
lsof -nP -iTCP:8080 -sTCP:LISTEN
curl -i http://localhost:8080/ | head -n 20
```

---

## 1) Minimal “clean redeploy” workflow (do this every time)

### A) Build the WAR
```bash
cd ~/war-test
./gradlew --stop
./gradlew clean war
ls -lh build/libs
```

### B) Stop Tomcat
```bash
brew services stop tomcat@9
```

### C) Remove old app (WAR + exploded directory)
```bash
WEBAPPS="/opt/homebrew/opt/tomcat@9/libexec/webapps"

rm -f  "$WEBAPPS/war-test.war"
rm -rf "$WEBAPPS/war-test"
```

### D) Copy new WAR into webapps
```bash
cp build/libs/*.war "$WEBAPPS/war-test.war"
ls -lh "$WEBAPPS/war-test.war"
```

### E) Start Tomcat
```bash
brew services start tomcat@9
```

### F) Verify via logs + curl
```bash
tail -n 120 /opt/homebrew/opt/tomcat@9/libexec/logs/catalina.out

curl -i http://localhost:8080/war-test/ | head -n 40
curl -i http://localhost:8080/war-test/index.html | head -n 40
```

---

## 2) Sanity test: make `/war-test/index.html` return 200

If `/war-test/` is 200 but `/war-test/index.html` is 404, you simply don’t have a real static file at that path.

Add a tiny static file at web root:

```bash
cd ~/war-test
mkdir -p src/main/webapp

cat > src/main/webapp/index.html <<'EOF'
<!doctype html>
<html>
  <body><h1>war-test index.html OK</h1></body>
</html>
EOF
```

Then rebuild + redeploy using the workflow above.

Verify:
```bash
curl -i http://localhost:8080/war-test/index.html | head -n 40
```

---

## 3) One-command redeploy helper (optional)

Paste into your shell (`~/.zshrc`) or run directly in a terminal session:

```bash
redeploy_wartest() {
  set -e
  cd ~/war-test

  ./gradlew --stop >/dev/null 2>&1 || true
  ./gradlew clean war

  brew services stop tomcat@9

  WEBAPPS="/opt/homebrew/opt/tomcat@9/libexec/webapps"
  rm -f  "$WEBAPPS/war-test.war"
  rm -rf "$WEBAPPS/war-test"

  cp build/libs/*.war "$WEBAPPS/war-test.war"

  brew services start tomcat@9

  echo "== catalina.out (tail) =="
  tail -n 120 /opt/homebrew/opt/tomcat@9/libexec/logs/catalina.out

  echo "== curl checks =="
  curl -i http://localhost:8080/war-test/ | head -n 30
  curl -i http://localhost:8080/war-test/index.html | head -n 30
}
```

Run:
```bash
redeploy_wartest
```
