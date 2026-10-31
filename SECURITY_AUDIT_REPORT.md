# Security Audit Report

**Repository:** amazon-ecs-fargate-cdk-v2-cicd  
**Audit Date:** 2025-07-18  
**Audited Components:** CDK v2 (TypeScript/Node.js), Flask Docker App (Python)

---

## Executive Summary

This audit examined both the CDK infrastructure project (`cdk-v2/`) and the Flask web application (`flask-docker-app/`) for dependency vulnerabilities and code-level security issues.

**Key Findings:**

| Category | Critical | High | Moderate | Low | Info |
|---|---|---|---|---|---|
| npm dependencies (CDK) | 2 | 11 | 5 | 13 | 0 |
| pip dependencies (Flask) | 0 | 4 | 7 | 0 | 0 |
| Code security issues | 2 | 5 | 4 | 2 | 0 |

The most urgent issues are:

1. **Severely outdated Python dependencies** with 29 known vulnerabilities across Flask, Jinja2, and Werkzeug. These packages are 4-6 major versions behind current releases.
2. **31 npm vulnerabilities** in the CDK project, including 2 critical issues (arbitrary code execution via `@babel/traverse`, unsafe randomness in `form-data`).
3. **Dockerfile security concerns** including running as root, downloading pip installer over plain HTTP, and unpinned package versions.
4. **IAM policy with wildcard resources** in the CDK stack, granting overly broad permissions.

---

## 1. CDK / Node.js Dependency Audit

### 1.1 Outdated Packages

All packages in `cdk-v2/package.json` are significantly outdated. The project was last updated with CDK v2.37.1 (August 2022).

| Package | Current | Wanted | Latest | Status |
|---|---|---|---|---|
| `aws-cdk-lib` | 2.37.1 | 2.37.1 | 2.275.0 | **238 versions behind** |
| `aws-cdk` (dev) | 2.37.1 | 2.37.1 | 2.1147.0 | **Extremely outdated** |
| `typescript` | 3.9.10 | 3.9.10 | 7.0.2 | **4 major versions behind** |
| `jest` | 27.5.1 | 27.5.1 | 30.5.2 | **3 major versions behind** |
| `ts-jest` | 27.1.5 | 27.1.5 | 29.4.15 | **2 major versions behind** |
| `@types/jest` | 27.5.2 | 27.5.2 | 30.0.0 | **3 major versions behind** |
| `@types/node` | 10.17.27 | 10.17.27 | 26.6.6 | **16 major versions behind** |
| `@types/prettier` | 2.6.0 | 2.6.0 | 2.7.3 | Minor update available |
| `constructs` | 10.1.77 | 10.8.1 | 10.8.1 | Patch update available |
| `ts-node` | 10.9.1 | 10.9.2 | 10.9.2 | Patch update available |

### 1.2 npm Audit Vulnerabilities (31 total)

#### Critical Severity (2)

| Package | Advisory | Description |
|---|---|---|
| `@babel/traverse` < 7.23.2 | [GHSA-67hx-6x53-jw92](https://github.com/advisories/GHSA-67hx-6x53-jw92) | Arbitrary code execution when compiling specially crafted malicious code |
| `form-data` 3.0.0 - 3.0.4 | [GHSA-fjxv-7rqg-78g4](https://github.com/advisories/GHSA-fjxv-7rqg-78g4), [GHSA-hmw2-7cc7-3qxx](https://github.com/advisories/GHSA-hmw2-7cc7-3qxx) | Unsafe random function for boundary selection; CRLF injection in multipart field names |

#### High Severity (11)

| Package | Advisory | Description |
|---|---|---|
| `aws-cdk-lib` <= 2.259.0 | [GHSA-rx28-r23p-2qc3](https://github.com/advisories/GHSA-rx28-r23p-2qc3) | EKS overly permissive trust policies |
| `aws-cdk-lib` <= 2.259.0 | [GHSA-v4mq-x674-ff73](https://github.com/advisories/GHSA-v4mq-x674-ff73) | IAM OIDC custom resource connects to unauthorized provider |
| `aws-cdk-lib` <= 2.259.0 | [GHSA-qq4x-c6h6-rfxh](https://github.com/advisories/GHSA-qq4x-c6h6-rfxh) | Sensitive info logged when using Cognito UserPoolClient |
| `aws-cdk-lib` <= 2.259.0 | [GHSA-5pq3-h73f-66hr](https://github.com/advisories/GHSA-5pq3-h73f-66hr) | CodePipeline trusted entities are too broad |
| `aws-cdk-lib` <= 2.259.0 | [GHSA-999r-qq7v-r334](https://github.com/advisories/GHSA-999r-qq7v-r334) | OS Command Injection in NodejsFunction Bundling |
| `aws-cdk-lib` <= 2.259.0 | [GHSA-vcrf-j523-4mrf](https://github.com/advisories/GHSA-vcrf-j523-4mrf) | OS Command Injection in NodejsFunction Docker Bundling |
| `aws-cdk-lib` <= 2.259.0 | [GHSA-464c-974j-9xm6](https://github.com/advisories/GHSA-464c-974j-9xm6) | CodeBuild S3 Log Encryption Boolean Inversion |
| `brace-expansion` <= 1.1.20 | [GHSA-v6h2-p8h4-qcjw](https://github.com/advisories/GHSA-v6h2-p8h4-qcjw) and 7 others | Multiple ReDoS and DoS vulnerabilities |
| `braces` < 3.0.3 | [GHSA-grv7-fg5c-xmjg](https://github.com/advisories/GHSA-grv7-fg5c-xmjg) | Uncontrolled resource consumption |
| `browserslist` <= 4.28.6 | [GHSA-c83g-rgw3-j3cx](https://github.com/advisories/GHSA-c83g-rgw3-j3cx), [GHSA-73wf-gq98-2v4g](https://github.com/advisories/GHSA-73wf-gq98-2v4g) | Unbounded memory growth; prototype write via custom stats |
| `cross-spawn` 7.0.0 - 7.0.4 | [GHSA-3xgq-45jj-v275](https://github.com/advisories/GHSA-3xgq-45jj-v275) | Regular Expression Denial of Service |
| `js-yaml` <= 3.15.1 | [GHSA-mh29-5h37-fv8m](https://github.com/advisories/GHSA-mh29-5h37-fv8m) and 4 others | Prototype pollution in merge; multiple DoS vulnerabilities |
| `lodash` <= 4.17.23 | [GHSA-r5fr-rjxr-66jc](https://github.com/advisories/GHSA-r5fr-rjxr-66jc) and 2 others | Code Injection via `_.template`; Prototype Pollution via `_.unset`/`_.omit` |
| `minimatch` <= 11.0.0 | [GHSA-3ppc-4f35-3m26](https://github.com/advisories/GHSA-3ppc-4f35-3m26) and 3 others | Multiple ReDoS vulnerabilities |
| `picomatch` <= 2.3.1 | [GHSA-3v7f-55p6-f55p](https://github.com/advisories/GHSA-3v7f-55p6-f55p), [GHSA-c2c7-rcm5-vvqj](https://github.com/advisories/GHSA-c2c7-rcm5-vvqj) | Method Injection; ReDoS via extglob quantifiers |
| `semver` 6.0.0 - 7.5.1 | [GHSA-c2qf-rxjj-qqgw](https://github.com/advisories/GHSA-c2qf-rxjj-qqgw) | Regular Expression Denial of Service |
| `ws` 7.0.0 - 7.5.10 | [GHSA-3h5v-q93c-6h6q](https://github.com/advisories/GHSA-3h5v-q93c-6h6q), [GHSA-96hv-2xvq-fx4p](https://github.com/advisories/GHSA-96hv-2xvq-fx4p) | DoS via many HTTP headers; memory exhaustion from tiny fragments |

#### Moderate Severity (5)

| Package | Advisory | Description |
|---|---|---|
| `@babel/helpers` < 7.26.10 | [GHSA-968p-4wvh-cqc8](https://github.com/advisories/GHSA-968p-4wvh-cqc8) | Inefficient RegExp complexity in generated code |
| `micromatch` < 4.0.8 | [GHSA-952p-6rrq-rcjv](https://github.com/advisories/GHSA-952p-6rrq-rcjv) | Regular Expression Denial of Service |
| `tough-cookie` < 4.1.3 | [GHSA-72xf-g2v4-qvf3](https://github.com/advisories/GHSA-72xf-g2v4-qvf3) | Prototype Pollution |
| `word-wrap` < 1.2.4 | [GHSA-j8xg-fqg3-53r7](https://github.com/advisories/GHSA-j8xg-fqg3-53r7) | Regular Expression Denial of Service |
| `yaml` 1.0.0 - 1.10.2 | [GHSA-48c2-rrv3-qjmp](https://github.com/advisories/GHSA-48c2-rrv3-qjmp) | Stack Overflow via deeply nested YAML |

#### Low Severity (13)

| Package | Advisory | Description |
|---|---|---|
| `@babel/core` <= 7.29.0 | [GHSA-4x5r-pxfx-6jf8](https://github.com/advisories/GHSA-4x5r-pxfx-6jf8) | Arbitrary File Read via sourceMappingURL Comment |
| `@tootallnate/once` < 2.0.1 | [GHSA-vpq2-c234-7xj6](https://github.com/advisories/GHSA-vpq2-c234-7xj6) | Incorrect Control Flow Scoping |
| `deepmerge` >= 4.2.0 | [GHSA-gcqx-w48m-w2h4](https://github.com/advisories/GHSA-gcqx-w48m-w2h4) | Prototype poisoning of returned objects |
| `diff` 4.0.0 - 4.0.3 | [GHSA-73rr-hh4g-fpgx](https://github.com/advisories/GHSA-73rr-hh4g-fpgx) | DoS in parsePatch and applyPatch |

*Note: Some low-severity items are transitive dependencies of jest and other dev-only packages.*

---

## 2. Flask / Python Dependency Audit

### 2.1 Outdated Packages

All Python packages in `flask-docker-app/requirements.txt` are severely outdated (dating from 2019).

| Package | Current Version | Latest Version | Versions Behind |
|---|---|---|---|
| `Click` | 7.0 | 8.5.0 | **1 major version** |
| `Flask` | 1.1.1 | 3.1.3 | **2 major versions** |
| `itsdangerous` | 1.1.0 | 2.2.0 | **1 major version** |
| `Jinja2` | 2.11.3 | 3.1.6 | **1 major version** |
| `MarkupSafe` | 1.1.1 | 3.0.4 | **2 major versions** |
| `Werkzeug` | 0.15.6 | 3.1.9 | **3 major versions** |

### 2.2 pip-audit Vulnerabilities (29 findings across 3 packages)

Results from `pip-audit -r flask-docker-app/requirements.txt`:

#### Flask 1.1.1 (3 unique vulnerabilities)

| ID | Fix Version | Severity | Description |
|---|---|---|---|
| PYSEC-2023-62 | 2.2.5 / 2.3.2 | Moderate | Session response caching issue. A response containing data intended for one client may be cached and sent to others by a proxy when `session.permanent = True` is set and the session is not accessed or modified during the request. |
| PYSEC-2026-2151 | 3.1.3 | Moderate | Missing `Vary: Cookie` header when session is accessed via the `in` operator. Can cause proxy caches to serve incorrect session data to other users. |

#### Jinja2 2.11.3 (4 unique vulnerabilities)

| ID | Fix Version | Severity | Description |
|---|---|---|---|
| PYSEC-2026-1473 | 3.1.3 | Moderate | `xmlattr` filter accepts keys containing spaces, allowing XSS if user-controlled keys are rendered. |
| PYSEC-2026-1474 | 3.1.4 | Moderate | `xmlattr` filter accepts keys with non-attribute characters (`/`, `>`, `=`), enabling attribute injection and XSS. |
| PYSEC-2026-1475 | 3.1.5 | High | Sandbox escape via stored reference to `str.format` method passed through filters, enabling arbitrary code execution. |
| PYSEC-2026-1471 | 3.1.6 | High | Sandbox escape via `|attr` filter bypassing format method restrictions, enabling arbitrary code execution. |

#### Werkzeug 0.15.6 (10 unique vulnerabilities)

| ID | Fix Version | Severity | Description |
|---|---|---|---|
| PYSEC-2022-203 | 2.1.1 | Moderate | HTTP Request Smuggling via crafted requests with multiple bodies (disputed, development mode only). |
| PYSEC-2023-57 | 2.2.3 | Moderate | Nameless cookie parsing allows adjacent subdomain to set crafted cookies. |
| PYSEC-2023-58 | 2.2.3 | High | Multipart form parser DoS - unlimited parts cause CPU and memory exhaustion. |
| PYSEC-2023-221 | 2.3.8 / 3.0.1 | High | Multipart form DoS - files starting with CR/LF cause growing buffer and expensive boundary lookups. |
| PYSEC-2026-2043 | 3.0.3 | High | Debugger can allow remote code execution on developer's machine via domain/subdomain control. |
| PYSEC-2026-2045 | 3.0.6 | Moderate | `safe_join()` does not catch UNC paths on Python < 3.11 on Windows, allowing unintended data access. |
| PYSEC-2026-2046 | 3.1.4 | Moderate | `safe_join` allows Windows device names causing file read hang. |
| PYSEC-2026-2044 | 3.1.5 | Moderate | `safe_join` allows Windows device names with extensions/trailing spaces causing hang. |
| PYSEC-2026-2320 | 3.1.6 | Moderate | `safe_join` allows Windows device names when preceded by other path segments. |
| PYSEC-2026-4312 | 3.1.9 | Moderate | `safe_join` allows Windows device names with empty ADS marker on NTFS. |

---

## 3. Code Security Review

### 3.1 CDK Stack (`cdk-v2/lib/ecs_cdk-stack.ts`)

#### 3.1.1 IAM Policy with Wildcard Resources - **HIGH**

```typescript
const executionRolePolicy = new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  resources: ['*'],
  actions: [
    "ecr:getauthorizationtoken",
    "ecr:batchchecklayeravailability",
    "ecr:getdownloadurlforlayer",
    "ecr:batchgetimage",
    "logs:createlogstream",
    "logs:putlogevents"
  ]
});
```

**Issue:** The execution role policy uses `resources: ['*']`, granting these permissions across all ECR repositories and log groups in the account. While `ecr:GetAuthorizationToken` requires a wildcard resource, the other actions should be scoped to the specific ECR repository and log group ARNs.

**Recommendation:** Scope ECR actions to `ecrRepo.repositoryArn` and CloudWatch Logs actions to the specific log group ARN. Keep `ecr:GetAuthorizationToken` on `'*'` as required by the API, but split it into a separate statement.

#### 3.1.2 Deprecated CodeBuild Image - **MODERATE**

```typescript
environment: {
  buildImage: codebuild.LinuxBuildImage.AMAZON_LINUX_2_2,
  privileged: true
}
```

**Issue:** `AMAZON_LINUX_2_2` is a deprecated build image. Amazon Linux 2 reached standard support end-of-life on June 30, 2025, and this older image variant receives no security updates.

**Recommendation:** Upgrade to `codebuild.LinuxBuildImage.AMAZON_LINUX_2_ARM_3` or `codebuild.LinuxBuildImage.STANDARD_7_0` for current security patches.

#### 3.1.3 Privileged CodeBuild Container - **MODERATE**

```typescript
environment: {
  buildImage: codebuild.LinuxBuildImage.AMAZON_LINUX_2_2,
  privileged: true
}
```

**Issue:** `privileged: true` grants the build container elevated Docker permissions. This is required for Docker-in-Docker builds but increases the blast radius if the build is compromised.

**Recommendation:** This is currently necessary since the build runs `docker build`. Document the justification. Consider using AWS CodeBuild's native image building capabilities or Kaniko for rootless image builds.

#### 3.1.4 Public Load Balancer - **MODERATE**

```typescript
const fargateService = new ecs_patterns.ApplicationLoadBalancedFargateService(this, "ecs-service", {
  cluster: cluster,
  taskDefinition: taskDef,
  publicLoadBalancer: true,
  desiredCount: 1,
  listenerPort: 80
});
```

**Issues:**
- `publicLoadBalancer: true` exposes the service to the public internet.
- `listenerPort: 80` uses plain HTTP without HTTPS/TLS termination.
- No WAF (Web Application Firewall) is configured.
- No security group restrictions are applied beyond defaults.

**Recommendation:** Add HTTPS listener with an ACM certificate, redirect HTTP to HTTPS, and consider adding AWS WAF for web application protection.

#### 3.1.5 Cluster Admin Role Uses Account Root Principal - **LOW**

```typescript
const clusteradmin = new iam.Role(this, 'adminrole', {
  assumedBy: new iam.AccountRootPrincipal()
});
```

**Issue:** The `clusteradmin` role is assumable by the account root principal. While this role is defined, it does not appear to be used elsewhere in the stack, making it dead code that still creates an IAM resource.

**Recommendation:** Remove unused IAM roles. If needed, scope the trust policy to specific principals rather than the account root.

#### 3.1.6 Hardcoded `latest` Docker Tag - **LOW**

```typescript
commands: [
  'env',
  'export tag=latest'
]
```

**Issue:** The build spec hardcodes `tag=latest` instead of using the commit SHA (`CODEBUILD_RESOLVED_SOURCE_VERSION`). The original commit-SHA-based tagging is commented out. This means every deployment overwrites the `latest` tag, making rollbacks difficult and creating ambiguity about which image version is running.

**Recommendation:** Use the build source version as the image tag for traceability and rollback capability.

#### 3.1.7 Overly Aggressive Auto-Scaling Configuration - **INFO**

```typescript
scaling.scaleOnCpuUtilization('cpuscaling', {
  targetUtilizationPercent: 10,
  scaleInCooldown: cdk.Duration.seconds(60),
  scaleOutCooldown: cdk.Duration.seconds(60)
});
```

**Issue:** A 10% CPU target is extremely low and will likely keep the service at maximum capacity (6 tasks) at all times, increasing costs unnecessarily.

---

### 3.2 Flask Application (`flask-docker-app/app.py`)

#### 3.2.1 Debug Mode Controlled by Environment Variable - **HIGH**

```python
if __name__ == '__main__':
    app.run(
        debug=os.getenv('FLASK_DEBUG', False),
        host='0.0.0.0',
        port=5000
    )
```

**Issue:** Debug mode is controlled by the `FLASK_DEBUG` environment variable. If this variable is set to any truthy string value in production, the Werkzeug interactive debugger is exposed, which allows arbitrary code execution on the server. Note that `os.getenv('FLASK_DEBUG', False)` returns a string when the variable is set, and any non-empty string is truthy in Python, so even `FLASK_DEBUG=0` would enable debug mode.

**Recommendation:** Never enable debug mode in production. If environment-variable-based configuration is needed, explicitly compare against known safe values (e.g., `os.getenv('FLASK_DEBUG', '').lower() == 'true'`). Better yet, remove the debug flag entirely and use Flask's standard `FLASK_DEBUG` environment variable mechanism with `flask run`.

#### 3.2.2 Binding to All Network Interfaces - **MODERATE**

```python
app.run(host='0.0.0.0', port=5000)
```

**Issue:** Binding to `0.0.0.0` makes the application accessible on all network interfaces. In a container environment this is generally expected and necessary, but it should be coupled with proper network-level security controls.

**Recommendation:** This is acceptable for containerized deployments. Ensure the ECS task security groups restrict inbound traffic to the ALB only.

#### 3.2.3 Potential XSS via User Input - **MODERATE**

```python
name = flask.request.args.get("name", "Flask-demo")
return flask.render_template('hello.html', ..., name=name)
```

In `hello.html`:
```html
<h1>{{ name }}</h1>
```

**Issue:** The `name` parameter is taken directly from the query string and rendered in the template. Jinja2's autoescaping is enabled by default for `.html` templates, which mitigates basic XSS. However, relying solely on autoescaping without input validation is a defense-in-depth concern. If autoescaping were ever disabled or the value used in an unsafe context (e.g., `{{ name | safe }}`), XSS would result.

**Recommendation:** Add input validation and length limits on the `name` parameter. Consider sanitizing or rejecting unexpected characters.

#### 3.2.4 No CSRF Protection - **MODERATE**

**Issue:** The Flask application does not use Flask-WTF or any other CSRF protection mechanism. While the current app only has a GET endpoint, if POST endpoints are added in the future they would be vulnerable to cross-site request forgery.

**Recommendation:** Install and configure Flask-WTF with CSRF protection before adding any state-changing endpoints.

---

### 3.3 Dockerfile (`flask-docker-app/Dockerfile`)

#### 3.3.1 Running as Root - **HIGH**

```dockerfile
FROM public.ecr.aws/amazonlinux/amazonlinux:2022
RUN yum install python3 && \
...
CMD ["python3", "app.py"]
```

**Issue:** The container runs as root. There is no `USER` directive to switch to a non-privileged user. If the application is compromised, the attacker has root access inside the container, which increases the potential for container escape or lateral movement.

**Recommendation:** Add a non-root user and switch to it before running the application:
```dockerfile
RUN useradd -r -s /bin/false appuser
USER appuser
```

#### 3.3.2 Downloading pip Installer over HTTP - **HIGH**

```dockerfile
curl -O https://bootstrap.pypa.io/get-pip.py && \
python3 get-pip.py
```

**Issue:** While the URL uses HTTPS, the `curl` command lacks the `--fail` flag, meaning a failed download (e.g., HTTP error or redirect to an error page) would still be passed to `python3` for execution. Additionally, downloading and executing an arbitrary script during build time is a supply chain risk. There is no checksum verification.

**Recommendation:** Use the system package manager to install pip (`yum install python3-pip`) or verify the checksum of the downloaded script. Always use `curl --fail` to detect download errors.

#### 3.3.3 Unpinned Flask Installation - **MODERATE**

```dockerfile
pip install flask
```

**Issue:** Flask is installed without a version pin. The Dockerfile installs whatever the latest Flask version is at build time, making builds non-reproducible and potentially introducing breaking changes or vulnerabilities. Meanwhile, `requirements.txt` pins Flask at 1.1.1 but is not used in the `pip install` command in the Dockerfile.

**Recommendation:** Use the requirements.txt file in the Dockerfile: `pip install -r requirements.txt`. Pin all dependencies to specific versions for reproducibility. Better yet, update the requirements to current secure versions first.

#### 3.3.4 No HEALTHCHECK - **LOW**

**Issue:** The Dockerfile does not define a `HEALTHCHECK` instruction. While ECS uses its own health checks via the ALB target group, a Docker-level HEALTHCHECK provides an additional layer of container health monitoring.

**Recommendation:** Add a HEALTHCHECK:
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:5000/ || exit 1
```

#### 3.3.5 No .dockerignore File - **LOW**

**Issue:** There is no `.dockerignore` file. The `COPY . /app/` command copies everything including `pyvenv.cfg` and potentially other development artifacts into the container image.

**Recommendation:** Create a `.dockerignore` file to exclude unnecessary files (`.git`, `pyvenv.cfg`, `__pycache__`, `*.pyc`, etc.).

---

### 3.4 HTML Template (`flask-docker-app/templates/hello.html`)

#### 3.4.1 Reference to Defunct HTTP Script - **MODERATE**

```html
<!--[if lt IE 9]><script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script><![endif]-->
```

**Issues:**
1. **Plain HTTP:** The script is loaded over `http://` instead of `https://`, making it vulnerable to man-in-the-middle attacks.
2. **Defunct domain:** `googlecode.com` has been shut down. The domain could be re-registered by a malicious actor for domain takeover, which would allow them to serve arbitrary JavaScript to any browser that loads this script.
3. **IE conditional comment:** This targets IE versions below 9, which are long past end-of-life.

**Recommendation:** Remove this script reference entirely. It targets an obsolete browser and loads from a defunct service. If HTML5 compatibility is genuinely needed, use a modern polyfill served from your own domain.

---

## 4. Recommendations (Prioritized)

### Priority 1 - Critical (Address Immediately)

| # | Action | Component | Rationale |
|---|---|---|---|
| 1 | **Update Python dependencies** to current versions: Flask >= 3.1.3, Werkzeug >= 3.1.9, Jinja2 >= 3.1.6, MarkupSafe >= 3.0.4, itsdangerous >= 2.2.0, Click >= 8.5.0 | Flask App | 29 known vulnerabilities including DoS, request smuggling, remote code execution (debugger), and sandbox escapes |
| 2 | **Update `aws-cdk-lib`** to >= 2.275.0 | CDK | 7 known vulnerabilities including OS command injection, overly permissive trust policies, and sensitive info logging |
| 3 | **Fix Dockerfile** to not run as root, pin package versions, and use `requirements.txt` | Dockerfile | Container escape risk, non-reproducible builds, supply chain risk |
| 4 | **Remove debug mode** or fix the environment variable parsing in `app.py` | Flask App | Enables interactive debugger with code execution in production |

### Priority 2 - High (Address Soon)

| # | Action | Component | Rationale |
|---|---|---|---|
| 5 | **Scope IAM wildcard resources** to specific ARNs in execution role policy | CDK Stack | Overly permissive IAM violates least-privilege principle |
| 6 | **Add HTTPS** to the ALB with an ACM certificate, redirect HTTP to HTTPS | CDK Stack | Currently serving traffic over unencrypted HTTP |
| 7 | **Upgrade CodeBuild image** from deprecated `AMAZON_LINUX_2_2` | CDK Stack | Deprecated image no longer receives security updates |
| 8 | **Remove defunct `html5shim.googlecode.com` script reference** | HTML Template | Potential domain takeover leading to JavaScript injection |
| 9 | **Update all npm dev dependencies** (jest, typescript, ts-jest, @types/*) | CDK | 31 npm vulnerabilities, many in dev dependency chain |

### Priority 3 - Moderate (Address in Normal Development Cycle)

| # | Action | Component | Rationale |
|---|---|---|---|
| 10 | **Add input validation** on the `name` query parameter | Flask App | Defense in depth against XSS |
| 11 | **Add CSRF protection** via Flask-WTF before adding POST endpoints | Flask App | Prevents cross-site request forgery |
| 12 | **Use commit SHA as Docker image tag** instead of `latest` | CDK Stack | Enables proper deployment traceability and rollbacks |
| 13 | **Add a `.dockerignore` file** | Dockerfile | Reduces image size, prevents leaking dev artifacts |
| 14 | **Remove unused `clusteradmin` IAM role** | CDK Stack | Dead code that creates unnecessary IAM resources |
| 15 | **Add HEALTHCHECK to Dockerfile** | Dockerfile | Additional health monitoring layer |
| 16 | **Fix auto-scaling target** from 10% to a more reasonable value (e.g., 50-70%) | CDK Stack | Current setting wastes resources by over-provisioning |

---

## Appendix: Raw Tool Output

### A. npm audit summary

```
31 vulnerabilities (13 low, 5 moderate, 11 high, 2 critical)
```

### B. pip-audit summary

```
Found 29 known vulnerabilities in 3 packages
- Flask 1.1.1: 3 unique vulnerabilities (PYSEC-2023-62, PYSEC-2026-2151)
- Jinja2 2.11.3: 4 unique vulnerabilities (PYSEC-2026-1471, PYSEC-2026-1473, PYSEC-2026-1474, PYSEC-2026-1475)
- Werkzeug 0.15.6: 10 unique vulnerabilities (PYSEC-2022-203, PYSEC-2023-57, PYSEC-2023-58, PYSEC-2023-221, PYSEC-2026-2043, PYSEC-2026-2045, PYSEC-2026-2046, PYSEC-2026-2044, PYSEC-2026-2320, PYSEC-2026-4312)
```

### C. Tools Used

- `npm audit` (npm 11.4.2)
- `npm outdated`
- `pip-audit` 2.10.1
- `pip index versions`
- Manual code review
