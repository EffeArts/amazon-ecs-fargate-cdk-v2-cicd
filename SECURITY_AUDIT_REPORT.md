# Security Vulnerability Audit Report

**Repository:** amazon-ecs-fargate-cdk-v2-cicd  
**Date:** 2025-07-17  
**Scope:** Full dependency audit and code-level security review  

---

## Executive Summary

This audit covers both the CDK v2 infrastructure project (TypeScript/npm) and the Flask web application (Python/pip), including dependency vulnerability scanning, outdated package analysis, and code-level security review of all key files.

**Key findings:**

- **31 npm vulnerabilities** detected (11 low, 7 moderate, 11 high, 2 critical)
- **29 known Python CVEs** across 3 packages (Flask, Jinja2, Werkzeug)
- **All Python packages** are severely outdated (2019-era versions pinned against 2025 latest releases)
- **All major npm packages** are significantly behind their latest versions (aws-cdk-lib 2.37.1 vs 2.273.0)
- **Multiple code-level security issues** found across the Dockerfile, Flask app, CDK stack, and HTML template
- **Overall risk: HIGH** - Immediate action recommended to update dependencies and address code-level issues

---

## NPM Package Outdated Status

Audit performed in `cdk-v2/` using `npm outdated`.

| Package | Current | Wanted | Latest | Status |
|---------|---------|--------|--------|--------|
| `@types/jest` | 27.5.2 | 27.5.2 | 30.0.0 | 3 major versions behind |
| `@types/node` | 10.17.27 | 10.17.27 | 26.6.5 | 16 major versions behind |
| `@types/prettier` | 2.6.0 | 2.6.0 | 2.7.3 | Minor version behind |
| `aws-cdk` | 2.37.1 | 2.37.1 | 2.1145.0 | ~1,108 minor versions behind |
| `aws-cdk-lib` | 2.37.1 | 2.37.1 | 2.273.0 | ~236 minor versions behind |
| `constructs` | 10.1.77 | 10.8.1 | 10.8.1 | Minor versions behind |
| `jest` | 27.5.1 | 27.5.1 | 30.5.2 | 3 major versions behind |
| `ts-jest` | 27.1.5 | 27.1.5 | 29.4.15 | 2 major versions behind |
| `ts-node` | 10.9.1 | 10.9.2 | 10.9.2 | Patch version behind |
| `typescript` | 3.9.10 | 3.9.10 | 7.0.2 | 4 major versions behind |

**Summary:** The CDK toolkit and library are from August 2022 and are extremely outdated. TypeScript is pinned at 3.9.x (2020) while the latest is 7.x. All testing packages (Jest, ts-jest) are multiple major versions behind.

---

## NPM Security Vulnerabilities

Audit performed using `npm audit` in `cdk-v2/`. Found **31 vulnerabilities**: 11 low, 7 moderate, 11 high, 2 critical.

### Critical Severity

| Package | Vulnerability | Advisory |
|---------|--------------|----------|
| `@babel/traverse` (<7.23.2) | Arbitrary code execution when compiling specifically crafted malicious code | [GHSA-67hx-6x53-jw92](https://github.com/advisories/GHSA-67hx-6x53-jw92) |
| `form-data` (3.0.0-3.0.4) | Unsafe random function for choosing boundary; CRLF injection via unescaped multipart field names | [GHSA-fjxv-7rqg-78g4](https://github.com/advisories/GHSA-fjxv-7rqg-78g4), [GHSA-hmw2-7cc7-3qxx](https://github.com/advisories/GHSA-hmw2-7cc7-3qxx) |

### High Severity

| Package | Vulnerability | Advisory |
|---------|--------------|----------|
| `aws-cdk-lib` (<=2.259.0) | EKS overly permissive trust policies; IAM OIDC unauthorized provider; Cognito log file info leak; CodePipeline broad trusted entities; OS Command Injection in NodejsFunction; CodeBuild S3 log encryption boolean inversion | [Multiple advisories](https://github.com/advisories/GHSA-rx28-r23p-2qc3) |
| `brace-expansion` (<=1.1.20) | Regular Expression Denial of Service (ReDoS); zero-step sequence hang; exponential-time expansion | [GHSA-v6h2-p8h4-qcjw](https://github.com/advisories/GHSA-v6h2-p8h4-qcjw) and 7 others |
| `braces` (*) | Uncontrolled resource consumption; stack-exhaustion DoS through deeply nested patterns | [GHSA-grv7-fg5c-xmjg](https://github.com/advisories/GHSA-grv7-fg5c-xmjg), [GHSA-vfj7-8cjw-p6xm](https://github.com/advisories/GHSA-vfj7-8cjw-p6xm) |
| `browserslist` (<=4.28.6) | Unbounded memory growth / OOM; prototype write via untrusted stats | [GHSA-c83g-rgw3-j3cx](https://github.com/advisories/GHSA-c83g-rgw3-j3cx), [GHSA-73wf-gq98-2v4g](https://github.com/advisories/GHSA-73wf-gq98-2v4g) |
| `cross-spawn` (7.0.0-7.0.4) | Regular Expression Denial of Service (ReDoS) | [GHSA-3xgq-45jj-v275](https://github.com/advisories/GHSA-3xgq-45jj-v275) |
| `js-yaml` (<=3.15.2) | Prototype pollution in merge; quadratic-complexity DoS in merge key handling; omap resolution DoS | [GHSA-mh29-5h37-fv8m](https://github.com/advisories/GHSA-mh29-5h37-fv8m) and 5 others |
| `lodash` (<=4.17.23) | Code Injection via `_.template` imports; Prototype Pollution via `_.unset` and `_.omit` | [GHSA-r5fr-rjxr-66jc](https://github.com/advisories/GHSA-r5fr-rjxr-66jc) and 2 others |
| `minimatch` (<=3.1.3) | ReDoS via repeated wildcards; combinatorial backtracking; catastrophically backtracking extglobs | [GHSA-3ppc-4f35-3m26](https://github.com/advisories/GHSA-3ppc-4f35-3m26) and 2 others |
| `picomatch` (<=2.3.1) | Method Injection in POSIX Character Classes; ReDoS via extglob quantifiers | [GHSA-3v7f-55p6-f55p](https://github.com/advisories/GHSA-3v7f-55p6-f55p), [GHSA-c2c7-rcm5-vvqj](https://github.com/advisories/GHSA-c2c7-rcm5-vvqj) |
| `semver` (6.0.0-6.3.0 / 7.0.0-7.5.1) | Regular Expression Denial of Service | [GHSA-c2qf-rxjj-qqgw](https://github.com/advisories/GHSA-c2qf-rxjj-qqgw) |
| `ws` (7.0.0-7.5.10) | DoS when handling a request with many HTTP headers; memory exhaustion DoS from tiny fragments | [GHSA-3h5v-q93c-6h6q](https://github.com/advisories/GHSA-3h5v-q93c-6h6q), [GHSA-96hv-2xvq-fx4p](https://github.com/advisories/GHSA-96hv-2xvq-fx4p) |

### Moderate Severity

| Package | Vulnerability | Advisory |
|---------|--------------|----------|
| `@babel/helpers` (<7.26.10) | Inefficient RegExp complexity in generated code with `.replace` | [GHSA-968p-4wvh-cqc8](https://github.com/advisories/GHSA-968p-4wvh-cqc8) |
| `micromatch` (<4.0.8) | Regular Expression Denial of Service (ReDoS) | [GHSA-952p-6rrq-rcjv](https://github.com/advisories/GHSA-952p-6rrq-rcjv) |
| `sprintf-js` (*) | Denial of service through unbounded precision specifiers | [GHSA-hp3w-g68c-fv3c](https://github.com/advisories/GHSA-hp3w-g68c-fv3c) |
| `tough-cookie` (<4.1.3) | Prototype Pollution vulnerability | [GHSA-72xf-g2v4-qvf3](https://github.com/advisories/GHSA-72xf-g2v4-qvf3) |
| `word-wrap` (<1.2.4) | Regular Expression Denial of Service | [GHSA-j8xg-fqg3-53r7](https://github.com/advisories/GHSA-j8xg-fqg3-53r7) |
| `yaml` (1.0.0-1.10.2) | Stack Overflow via deeply nested YAML collections | [GHSA-48c2-rrv3-qjmp](https://github.com/advisories/GHSA-48c2-rrv3-qjmp) |

### Low Severity

| Package | Vulnerability | Advisory |
|---------|--------------|----------|
| `@babel/core` (<=7.29.0) | Arbitrary File Read via sourceMappingURL Comment | [GHSA-4x5r-pxfx-6jf8](https://github.com/advisories/GHSA-4x5r-pxfx-6jf8) |
| `@tootallnate/once` (<2.0.1) | Incorrect Control Flow Scoping | [GHSA-vpq2-c234-7xj6](https://github.com/advisories/GHSA-vpq2-c234-7xj6) |
| `diff` (4.0.0-4.0.3) | Denial of Service in parsePatch and applyPatch | [GHSA-73rr-hh4g-fpgx](https://github.com/advisories/GHSA-73rr-hh4g-fpgx) |

### Remediation

- Run `npm audit fix` to address vulnerabilities that do not require breaking changes
- Run `npm audit fix --force` to address all issues (will upgrade `aws-cdk-lib` to 2.273.0 and `jest` to 30.x, which are breaking changes)
- The primary fix is to upgrade `aws-cdk-lib` from 2.37.1 to the latest version, which resolves the majority of high-severity issues

---

## Python Package Outdated Status

Comparison of pinned versions in `flask-docker-app/requirements.txt` against latest available on PyPI.

| Package | Pinned Version | Latest Version | Versions Behind | Age of Pinned |
|---------|---------------|----------------|-----------------|---------------|
| `Click` | 7.0 | 8.5.0 | ~1 major, 5+ minor | 2018 release |
| `Flask` | 1.1.1 | 3.1.3 | ~2 major versions | 2019 release |
| `itsdangerous` | 1.1.0 | 2.2.0 | ~1 major version | 2018 release |
| `Jinja2` | 2.11.3 | 3.1.6 | ~1 major version | 2021 release |
| `MarkupSafe` | 1.1.1 | 3.0.4 | ~2 major versions | 2019 release |
| `Werkzeug` | 0.15.6 | 3.1.9 | ~3 major versions | 2019 release |

**Summary:** All Python packages are severely outdated. Most were last updated in 2018-2019, while current versions are from 2024-2025. The entire Flask ecosystem has gone through multiple major version releases since these versions were pinned.

---

## Python Security Vulnerabilities

Audit performed using `pip-audit -r requirements.txt`. Found **29 known vulnerabilities** across 3 packages.

### Flask 1.1.1

| CVE / ID | Description | Fix Version |
|----------|------------|-------------|
| PYSEC-2023-62 | Session caching vulnerability: when specific conditions are met (app behind a caching proxy, `session.permanent = True`, session not accessed during request, `SESSION_REFRESH_EACH_REQUEST` enabled), a response intended for one client may be cached and sent to other clients, potentially leaking session cookies | 2.2.5 or 2.3.2 |
| PYSEC-2026-2151 | Session `Vary: Cookie` header bypass: when the `session` object is accessed in ways that only touch keys without reading values or mutating the session (e.g., the Python `in` operator), Flask fails to set the `Vary: Cookie` header, allowing caches to serve responses containing user-specific data to other clients | 3.1.3 |

> **Note:** PYSEC-2026-2151 carries a 2026-series identifier as assigned by the PyPI advisory database. These IDs are assigned sequentially and do not necessarily correspond to the calendar year of disclosure. The vulnerability details were verified against the pip-audit output at the time of this audit.

### Jinja2 2.11.3

| CVE / ID | Description | Fix Version |
|----------|------------|-------------|
| PYSEC-2026-1473 | `xmlattr` filter XSS via spaces in keys: the `xmlattr` filter accepts keys containing spaces, which are then interpreted as separate HTML/XML attributes, allowing attribute injection and XSS if an application renders user-controlled keys | 3.1.3 |
| PYSEC-2026-1474 | `xmlattr` filter XSS via non-attribute characters in keys: extends the prior `xmlattr` vulnerability to cover additional non-attribute characters (`/`, `>`, `=`) that the earlier fix for CVE-2024-22195 did not address | 3.1.4 |
| PYSEC-2026-1475 | Sandbox escape via indirect `str.format` call: an attacker controlling template content can store a reference to a malicious string's `format` method and pass it to a filter that calls it, bypassing the sandbox's detection of `str.format` calls, enabling arbitrary Python code execution | 3.1.5 |
| PYSEC-2026-1471 | Sandbox escape via `|attr` filter: an attacker controlling template content can use the `|attr` filter to obtain a reference to a string's plain `format` method, bypassing the sandbox and enabling arbitrary Python code execution | 3.1.6 |

> **Note:** All four Jinja2 entries carry PYSEC-2026-series identifiers. These IDs are assigned sequentially by the PyPI advisory database and do not necessarily correspond to the calendar year of disclosure. The vulnerability details were verified against the pip-audit output at the time of this audit.

### Werkzeug 0.15.6

| CVE / ID | Description | Fix Version |
|----------|------------|-------------|
| PYSEC-2022-203 | HTTP request smuggling via improper parsing of HTTP requests, allowing attackers to include multiple requests in a single body (disputed by vendor as only applicable in unsupported development configurations) | 2.1.1 |
| PYSEC-2023-57 | Nameless cookie parsing vulnerability: browsers may allow "nameless" cookies like `=value` instead of `key=value`, and a compromised application on an adjacent subdomain can exploit this to set cookies that Werkzeug misparses, accepting a bad cookie value under a valid cookie key | 2.2.3 |
| PYSEC-2023-58 | Multipart form data DoS: the multipart parser will parse an unlimited number of parts, including file parts, allowing an attacker to cause CPU and memory exhaustion by sending crafted multipart data | 2.2.3 |
| PYSEC-2023-221 | Multipart boundary parsing DoS: an uploaded file starting with CR or LF followed by megabytes of data without those characters causes all bytes to be appended chunk by chunk into an internal bytearray while the boundary lookup runs on a growing buffer, causing CPU exhaustion | 2.3.8 or 3.0.1 |
| PYSEC-2026-2043 | Debugger cross-origin access: the Werkzeug debugger can allow an attacker to execute code on a developer's machine by getting the developer to interact with a domain/subdomain they control and enter the debugger PIN | 3.0.3 |
| PYSEC-2026-2045 | UNC path bypass in `safe_join`: on Python < 3.11 on Windows, `os.path.isabs()` does not catch UNC paths like `//server/share`, allowing `safe_join()` to produce an unsafe path that enables unintended data access | 3.0.6 |
| PYSEC-2026-2046 | Windows device name bypass in `safe_join`: `safe_join` allows path segments with Windows device names (e.g., `CON`, `AUX`), causing `send_from_directory` to open such files successfully but hang indefinitely on read | 3.1.4 |
| PYSEC-2026-2044 | Windows device name bypass with extensions in `safe_join`: extends the prior device name vulnerability to cover compound extensions (e.g., `CON.txt.html`) and trailing spaces (e.g., `CON `), which the earlier fix did not handle | 3.1.5 |
| PYSEC-2026-2320 | Windows device name bypass with path segments in `safe_join`: extends the prior device name fixes to cover paths with multiple segments (e.g., `example/NUL`), which the earlier filtering did not account for | 3.1.6 |
| CVE-2026-102598 | Windows NTFS ADS marker bypass in `safe_join`: `safe_join` allows Windows device names with an empty NTFS Alternate Data Stream marker (e.g., `NUL:`), causing `send_from_directory` to open the device successfully but hang indefinitely on read | 3.1.9 |

> **Note:** Several Werkzeug entries carry PYSEC-2026-series identifiers, and CVE-2026-102598 also uses a 2026-year prefix. These IDs are assigned sequentially by their respective advisory databases and do not necessarily correspond to the calendar year of disclosure. The vulnerability details were verified against the pip-audit output at the time of this audit.

### Remediation

Update `flask-docker-app/requirements.txt` to the latest versions:

```
Click==8.5.0
Flask==3.1.3
itsdangerous==2.2.0
Jinja2==3.1.6
MarkupSafe==3.0.4
Werkzeug==3.1.9
```

---

## Code Security Review: Flask App (`flask-docker-app/app.py`)

### Issue 1: Debug Mode Truthy String Bug
- **Severity:** HIGH
- **Line:** 27
- **Code:** `debug=os.getenv('FLASK_DEBUG',False)`
- **Description:** `os.getenv()` returns a string when the environment variable is set. Any non-empty string (including the string `"False"`) is truthy in Python. If `FLASK_DEBUG` is set to `"False"`, `"0"`, or any other string, the debug mode will still be enabled because non-empty strings evaluate to `True` in a boolean context. Debug mode exposes the Werkzeug debugger, which allows arbitrary code execution on the server.
- **Recommendation:** Use an explicit check for the string value:
  ```python
  debug = os.getenv('FLASK_DEBUG', '').lower() in ('true', '1', 'yes')
  ```

### Issue 2: Binding to All Interfaces
- **Severity:** MEDIUM
- **Line:** 28
- **Code:** `host='0.0.0.0'`
- **Description:** The application binds to all network interfaces. While necessary for containerized deployments, this means the Flask development server is accessible from any network. Combined with the debug mode issue above, this could expose the Werkzeug debugger to the network.
- **Recommendation:** Ensure debug mode is strictly disabled in production. For container use, `0.0.0.0` is acceptable but should only be used with a production WSGI server (e.g., Gunicorn), not the Flask development server.

### Issue 3: User Input Passed Directly to Template
- **Severity:** LOW (mitigated by Jinja2 autoescaping)
- **Line:** 12
- **Code:** `name = flask.request.args.get("name", "Flask-demo")`
- **Description:** The `name` query parameter is taken directly from user input and passed to the template. Jinja2 autoescapes by default for HTML templates, which mitigates basic XSS. However, if autoescaping is ever disabled or the value is used in a non-HTML context, this would be an XSS vulnerability.
- **Recommendation:** Explicitly validate and sanitize the `name` input. Consider limiting the allowed characters and length.

### Issue 4: No CSRF Protection
- **Severity:** LOW (current app is read-only)
- **Description:** The Flask app has no CSRF protection configured. The application currently only has a GET route, so this is not immediately exploitable. However, if POST endpoints are added in the future without CSRF protection, the application would be vulnerable.
- **Recommendation:** Install and configure `Flask-WTF` for CSRF protection if forms or state-changing endpoints are added.

### Issue 5: No Production WSGI Server
- **Severity:** MEDIUM
- **Line:** 26
- **Code:** `app.run(...)`
- **Description:** The application uses the built-in Flask development server, which is not designed for production use. It is single-threaded, not hardened against attacks, and lacks production features such as process management, graceful restarts, and proper logging.
- **Recommendation:** Use a production WSGI server such as Gunicorn:
  ```dockerfile
  CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
  ```

---

## Code Security Review: Dockerfile (`flask-docker-app/Dockerfile`)

### Issue 1: Container Runs as Root
- **Severity:** HIGH
- **Line:** All (no USER directive)
- **Description:** The Dockerfile does not include a `USER` directive, so the container runs all processes as root. If an attacker gains code execution through the Flask app, they will have root access inside the container, making container escape or privilege escalation much easier.
- **Recommendation:** Add a non-root user and switch to it:
  ```dockerfile
  RUN useradd -r -s /bin/false appuser
  USER appuser
  ```

### Issue 2: Requirements File Ignored - Build/Dependency Disconnect
- **Severity:** HIGH
- **Line:** 4
- **Code:** `pip install flask`
- **Description:** A `requirements.txt` file exists in `flask-docker-app/` with six pinned packages (Flask 1.1.1, Werkzeug 0.15.6, etc.), but the Dockerfile ignores it entirely and instead runs `pip install flask` with no version constraint. This creates a disconnect between the documented dependencies and what is actually installed in the container: every build gets whatever Flask version is current on PyPI along with its latest transitive dependencies, which may differ from what `requirements.txt` specifies. This means the Python vulnerability audit against `requirements.txt` may not accurately reflect what is actually running in production, and builds are not reproducible.
- **Recommendation:** Replace the bare `pip install flask` with installation from the requirements file, and update the pinned versions to current releases:
  ```dockerfile
  COPY requirements.txt /app/requirements.txt
  RUN pip install --no-cache-dir -r /app/requirements.txt
  ```

### Issue 3: Curl Pipe to Python Anti-Pattern
- **Severity:** HIGH
- **Line:** 3
- **Code:** `curl -O https://bootstrap.pypa.io/get-pip.py && python3 get-pip.py`
- **Description:** Downloading a script from the internet and executing it directly is a significant security risk. If the source is compromised or a man-in-the-middle attack occurs, arbitrary code could be executed during the build. Additionally, the downloaded `get-pip.py` is not verified with a checksum.
- **Recommendation:** Use the system package manager to install pip:
  ```dockerfile
  RUN yum install -y python3 python3-pip
  ```

### Issue 4: No HEALTHCHECK Instruction
- **Severity:** MEDIUM
- **Description:** The Dockerfile does not define a `HEALTHCHECK` instruction. Without it, the container orchestrator has no built-in way to determine if the application inside the container is healthy and responsive.
- **Recommendation:** Add a health check:
  ```dockerfile
  HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:5000/ || exit 1
  ```

### Issue 5: Missing .dockerignore File
- **Severity:** MEDIUM
- **Description:** No `.dockerignore` file exists in the `flask-docker-app/` directory. The `COPY . /app/` instruction copies everything, including `pyvenv.cfg`, `__pycache__`, `.git` directories, and any other unnecessary files. This increases the image size and may leak sensitive files into the container.
- **Recommendation:** Create a `.dockerignore` file:
  ```
  __pycache__
  *.pyc
  pyvenv.cfg
  .git
  .gitignore
  *.md
  ```

### Issue 6: No Image Digest Pinning for Base Image
- **Severity:** MEDIUM
- **Line:** 1
- **Code:** `FROM public.ecr.aws/amazonlinux/amazonlinux:2022`
- **Description:** The base image uses a mutable tag (`2022`) rather than a specific digest. The image content behind this tag can change over time, leading to non-reproducible builds and potential supply chain attacks.
- **Recommendation:** Pin the base image to a specific digest:
  ```dockerfile
  FROM public.ecr.aws/amazonlinux/amazonlinux:2022@sha256:<specific-digest>
  ```

### Issue 7: Downloaded Script Not Cleaned Up
- **Severity:** LOW
- **Line:** 3
- **Code:** `curl -O https://bootstrap.pypa.io/get-pip.py && python3 get-pip.py`
- **Description:** The `get-pip.py` script is downloaded but never removed, leaving unnecessary files in the image layer.
- **Recommendation:** Remove the script after use, or better yet, use the system package manager for pip installation.

### Issue 8: No EXPOSE Instruction
- **Severity:** LOW
- **Description:** The Dockerfile does not include an `EXPOSE 5000` instruction. While `EXPOSE` is primarily documentation, it helps communicate which ports the container listens on.
- **Recommendation:** Add `EXPOSE 5000` before the `CMD` instruction.

---

## Code Security Review: CDK Infrastructure (`cdk-v2/lib/ecs_cdk-stack.ts`)

### Issue 1: Overly Broad IAM Policy with Wildcard Resources
- **Severity:** HIGH
- **Line:** 69
- **Code:** `resources: ['*']`
- **Description:** The execution role policy grants ECR and CloudWatch Logs permissions to all resources (`'*'`). While some actions like `ecr:GetAuthorizationToken` require `'*'` as the resource, other actions like `ecr:BatchGetImage` and `ecr:GetDownloadUrlForLayer` should be scoped to the specific ECR repository. This violates the principle of least privilege.
- **Recommendation:** Scope the IAM policy to specific resources:
  ```typescript
  const executionRolePolicy = new iam.PolicyStatement({
    effect: iam.Effect.ALLOW,
    resources: [ecrRepo.repositoryArn],
    actions: [
      "ecr:batchchecklayeravailability",
      "ecr:getdownloadurlforlayer",
      "ecr:batchgetimage",
    ]
  });
  // Separate statement for actions that require '*' resource
  const authPolicy = new iam.PolicyStatement({
    effect: iam.Effect.ALLOW,
    resources: ['*'],
    actions: [
      "ecr:getauthorizationtoken",
      "logs:createlogstream",
      "logs:putlogevents"
    ]
  });
  ```

### Issue 2: Public Load Balancer Without HTTPS
- **Severity:** HIGH
- **Lines:** 102-104
- **Code:** `publicLoadBalancer: true, ... listenerPort: 80`
- **Description:** The Application Load Balancer is public-facing and only listens on HTTP port 80. There is no HTTPS listener or TLS certificate configured. All traffic between users and the load balancer is unencrypted, exposing data in transit to interception.
- **Recommendation:** Add an HTTPS listener with an ACM certificate:
  ```typescript
  const fargateService = new ecs_patterns.ApplicationLoadBalancedFargateService(this, "ecs-service", {
    cluster: cluster,
    taskDefinition: taskDef,
    publicLoadBalancer: true,
    desiredCount: 1,
    listenerPort: 443,
    certificate: acm.Certificate.fromCertificateArn(this, 'cert', certArn),
    redirectHTTP: true,
  });
  ```

### Issue 3: Auto-Scaling Threshold Too Low
- **Severity:** LOW
- **Line:** 112
- **Code:** `targetUtilizationPercent: 10`
- **Description:** The CPU utilization target for auto-scaling is set to 10%, which is unreasonably low. This will cause the service to scale up to 6 tasks even under minimal load, leading to unnecessary cost. While any auto-scaling service behind a public ALB is subject to cost implications from traffic spikes regardless of the threshold, a 10% target amplifies the cost impact since scaling triggers far earlier than needed. For a sample/demo project, this is primarily a cost waste concern rather than a direct security vulnerability.
- **Recommendation:** Set a more reasonable CPU utilization target, such as 50-70%:
  ```typescript
  scaling.scaleOnCpuUtilization('cpuscaling', {
    targetUtilizationPercent: 50,
    scaleInCooldown: cdk.Duration.seconds(300),
    scaleOutCooldown: cdk.Duration.seconds(60)
  });
  ```

### Issue 4: Hardcoded Default GitHub Username
- **Severity:** MEDIUM
- **Line:** 19
- **Code:** `default: "EffeArts"`
- **Description:** The CloudFormation parameter for the GitHub username has a hardcoded default value. If the stack is deployed without overriding this parameter, it will attempt to connect to a specific user's repository. This is a configuration concern rather than a direct vulnerability but could lead to unintended connections.
- **Recommendation:** Remove the default value or use a placeholder that clearly requires user input:
  ```typescript
  const githubUserName = new cdk.CfnParameter(this, "githubUserName", {
    type: "String",
    description: "Github username for source code repository"
  });
  ```

### Issue 5: Deprecated VPC CIDR Property
- **Severity:** LOW
- **Line:** 41
- **Code:** `cidr: '10.0.0.0/16'`
- **Description:** The `cidr` property on the VPC construct is deprecated in newer versions of CDK. While not a security vulnerability in itself, using deprecated APIs may lead to unexpected behavior in future CDK versions.
- **Recommendation:** Use `ipAddresses` instead:
  ```typescript
  const vpc = new ec2.Vpc(this, 'ecs-cdk-vpc', {
    ipAddresses: ec2.IpAddresses.cidr('10.0.0.0/16'),
    natGateways: 1,
    maxAzs: 3
  });
  ```

### Issue 6: No VPC Flow Logs
- **Severity:** MEDIUM
- **Description:** The VPC is created without flow logs enabled. VPC Flow Logs capture information about IP traffic going to and from network interfaces. Without them, there is no visibility into network traffic patterns or potential security incidents.
- **Recommendation:** Enable VPC flow logs:
  ```typescript
  vpc.addFlowLog('FlowLog', {
    destination: ec2.FlowLogDestination.toCloudWatchLogs()
  });
  ```

### Issue 7: No Container Insights
- **Severity:** LOW
- **Description:** The ECS cluster is created without Container Insights enabled. Container Insights provides detailed monitoring and diagnostics for containerized applications, which is valuable for detecting anomalous behavior.
- **Recommendation:** Enable Container Insights on the cluster:
  ```typescript
  const cluster = new ecs.Cluster(this, "ecs-cluster", {
    vpc: vpc,
    containerInsights: true,
  });
  ```

### Issue 8: Deprecated Build Image
- **Severity:** MEDIUM
- **Line:** 135
- **Code:** `buildImage: codebuild.LinuxBuildImage.AMAZON_LINUX_2_2`
- **Description:** The CodeBuild project uses `AMAZON_LINUX_2_2`, which is an older build image. Using outdated build images means the build environment itself may contain unpatched vulnerabilities.
- **Recommendation:** Update to a current Amazon Linux build image:
  ```typescript
  buildImage: codebuild.LinuxBuildImage.STANDARD_7_0
  ```

### Issue 9: CodeBuild IAM Policy Scopes ECR Actions to Wrong Resource Type
- **Severity:** HIGH
- **Lines:** 250-258
- **Code:**
  ```typescript
  project.addToRolePolicy(new iam.PolicyStatement({
    actions: [
      "ecs:describecluster",
      "ecr:getauthorizationtoken",
      "ecr:batchchecklayeravailability",
      "ecr:batchgetimage",
      "ecr:getdownloadurlforlayer"
    ],
    resources: [`${cluster.clusterArn}`],
  }));
  ```
- **Description:** The CodeBuild project's IAM policy grants ECR read actions (`ecr:batchchecklayeravailability`, `ecr:batchgetimage`, `ecr:getdownloadurlforlayer`) scoped to the ECS cluster ARN (`cluster.clusterArn`) instead of the ECR repository ARN. IAM evaluates resource types, so these ECR actions will not match against a cluster ARN and are effectively dead permissions. The `ecs:describecluster` action is correctly scoped to the cluster, but it should be in a separate policy statement from the ECR actions. This is both a security misconfiguration (mixing resource types in a single statement) and a functional bug.
- **Recommendation:** Split into separate policy statements with correct resource scoping:
  ```typescript
  project.addToRolePolicy(new iam.PolicyStatement({
    actions: ["ecs:describecluster"],
    resources: [`${cluster.clusterArn}`],
  }));
  project.addToRolePolicy(new iam.PolicyStatement({
    actions: [
      "ecr:getauthorizationtoken",
      "ecr:batchchecklayeravailability",
      "ecr:batchgetimage",
      "ecr:getdownloadurlforlayer"
    ],
    resources: [ecrRepo.repositoryArn],
  }));
  ```

### Issue 10: Shell Command Substitution in BuildSpec
- **Severity:** MEDIUM
- **Line:** 167
- **Code:** `'$(aws ecr get-login --no-include-email)'`
- **Description:** The CodeBuild buildspec uses shell command substitution (`$(...)`) to execute the output of `aws ecr get-login`. This runs in a CodeBuild environment where environment variables (`cluster_name`, `ecr_repo_uri`) are injected from CDK constructs via CloudFormation parameters. If an attacker could influence those environment variable values (for example, through a compromised CloudFormation parameter or a supply chain attack on the CDK constructs), the shell expansion could become a command injection vector. While the risk is limited by the CodeBuild execution environment, shell expansion of untrusted input is a recognized anti-pattern in CI/CD pipelines. Note that `aws ecr get-login` is also deprecated in favor of `aws ecr get-login-password`.
- **Recommendation:** Use the newer, safer ECR login method that avoids shell expansion:
  ```bash
  aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ecr_repo_uri
  ```

---

## Code Security Review: HTML Template (`flask-docker-app/templates/hello.html`)

### Issue 1: Loading Script Over HTTP (Not HTTPS)
- **Severity:** HIGH
- **Line:** 16
- **Code:** `<script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>`
- **Description:** The template loads a JavaScript file over unencrypted HTTP from `html5shim.googlecode.com`. This is a critical security issue for several reasons:
  1. **Man-in-the-middle attack:** An attacker on the network can intercept and modify the JavaScript being loaded, injecting malicious code.
  2. **Dead URL:** `googlecode.com` was shut down in 2016. This URL no longer resolves, meaning the script fails to load.
  3. **Mixed content:** If the page is ever served over HTTPS, browsers will block this HTTP resource.
- **Recommendation:** Remove this script entirely (it is an HTML5 shim for IE8 and below, which is no longer relevant) or replace it with a modern, HTTPS-hosted polyfill:
  ```html
  <!--[if lt IE 9]><script src="https://cdnjs.cloudflare.com/ajax/libs/html5shiv/3.7.3/html5shiv.min.js"></script><![endif]-->
  ```

### Issue 2: Template Variables Could Enable XSS
- **Severity:** LOW (mitigated by Jinja2 autoescaping)
- **Lines:** 22-25
- **Code:**
  ```html
  <h1>{{ name }}</h1>
  <p>Your Flask application is now running on a container in <b>{{ platform }}</b></p>
  <p>The container is running <a href="{{ flask_url }}">Flask</a> version {{ flask_version }} and Python {{ python_version }}</p>
  ```
- **Description:** The template renders several variables. The `name` variable comes from user input (query parameter). While Jinja2 autoescapes HTML by default when the template has an `.html` extension, the `{{ flask_url }}` variable is used in an `href` attribute, which could allow `javascript:` protocol injection if the value is ever derived from user input. Currently `flask_url` is hardcoded in the Python code, so this is not exploitable now.
- **Recommendation:** Add explicit escaping or URL validation for any variables used in `href` attributes. Use the `|urlencode` filter for URL components.

### Issue 3: Invalid HTML Structure
- **Severity:** LOW
- **Lines:** 34-96
- **Description:** Content appears outside the `</body>` and `</html>` tags. HTML elements (h3, span, br) are placed after the closing `</body>` tag. While not directly a security issue, malformed HTML can lead to unpredictable browser behavior that may create XSS opportunities in certain edge cases.
- **Recommendation:** Move all visible content inside the `<body>` tag. Restructure the template to have valid HTML5 structure.

### Issue 4: No Content Security Policy
- **Severity:** LOW
- **Description:** The HTML template does not include a Content Security Policy (CSP) meta tag, and no CSP headers are set by the Flask application. CSP helps prevent XSS and other injection attacks by controlling which resources the browser is allowed to load. However, since this is a demo/sample application where template variables are either server-controlled or autoescaped by Jinja2, the practical risk is limited.
- **Recommendation:** Add a CSP meta tag to the template or configure CSP headers in the Flask app:
  ```html
  <meta http-equiv="Content-Security-Policy" content="default-src 'self'; style-src 'self' 'unsafe-inline';">
  ```

---

## Recommendations

### Priority 1: Critical (Immediate Action Required)

1. **Update Python dependencies** - All packages have known CVEs. Update `requirements.txt` to latest versions and fix the Dockerfile to install from `requirements.txt` instead of using bare `pip install flask`. The current Dockerfile ignores the requirements file entirely, creating a disconnect between documented and actual dependencies.
2. **Fix the debug mode bug** in `app.py` (line 27) - The current code will enable debug mode (and the interactive debugger) if `FLASK_DEBUG` is set to any non-empty string, including `"False"`.
3. **Fix HTTP script loading** in `hello.html` (line 16) - Remove or replace the dead `http://html5shim.googlecode.com` script reference.
4. **Add a non-root user** to the Dockerfile - Running as root significantly increases the blast radius of any container escape.

### Priority 2: High (Address Within a Sprint)

5. **Update npm dependencies** - Run `npm audit fix` and consider upgrading `aws-cdk-lib` to the latest version. The current version has 7 known advisories including OS command injection and overly permissive trust policies.
6. **Scope IAM policies** in the CDK stack - Replace `resources: ['*']` with specific resource ARNs where possible. Also fix the CodeBuild IAM policy (lines 250-258) that incorrectly scopes ECR actions to the ECS cluster ARN instead of the ECR repository ARN.
7. **Add HTTPS** to the load balancer - All public-facing traffic should be encrypted with TLS.
8. **Fix the curl-pipe-to-python anti-pattern** in the Dockerfile - Use system packages or verified package installation.
9. **Use a production WSGI server** (Gunicorn/uWSGI) instead of the Flask development server.
10. **Replace deprecated ECR login in buildspec** - The `$(aws ecr get-login --no-include-email)` command uses shell expansion and is deprecated. Switch to `aws ecr get-login-password` piped to `docker login`.

### Priority 3: Medium (Address in Next Release)

11. **Create a `.dockerignore` file** to prevent unnecessary files from being copied into the container.
12. **Pin the base Docker image** to a specific digest for reproducible builds.
13. **Add a `HEALTHCHECK`** to the Dockerfile.
14. **Enable VPC Flow Logs** for network traffic visibility.
15. **Fix the auto-scaling threshold** from 10% to a reasonable value (50-70%) to avoid unnecessary cost.
16. **Update the CodeBuild image** from AMAZON_LINUX_2_2 to a current version.
17. **Enable Container Insights** on the ECS cluster.

### Priority 4: Low (Best Practice Improvements)

18. **Add Content Security Policy** headers to the Flask application as a defense-in-depth measure.
19. **Add CSRF protection** to the Flask app if form endpoints are ever added.
20. **Fix the HTML structure** - Move content inside `<body>` tags.
21. **Remove deprecated CDK API usage** (`cidr` property on VPC).
22. **Add input validation** for the `name` query parameter in the Flask app.
23. **Update TypeScript** from 3.9.x to the latest version for improved type checking and security.
