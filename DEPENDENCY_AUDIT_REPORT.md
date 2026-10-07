# Dependency Audit Report

**Repository:** amazon-ecs-fargate-cdk-v2-cicd
**Date:** 2025-07-15
**Components Audited:** CDK v2 TypeScript Project (`cdk-v2/`), Flask Python App (`flask-docker-app/`)

---

## 1. Executive Summary

This report presents the findings of a comprehensive dependency and security audit across both components of this repository. The audit covers known vulnerabilities, outdated packages, Dockerfile security, and code-level security observations.

**Key Findings:**

| Category | Count |
|---|---|
| Node.js (npm) vulnerabilities | **31** (2 critical, 11 high, 7 moderate, 11 low) |
| Node.js outdated packages | **10** packages behind latest |
| Python (pip) vulnerabilities | **17** known CVEs across 4 packages |
| Python outdated packages | **5 of 6** packages behind latest |
| Dockerfile security issues | **4** findings |
| Code security observations | **3** findings |

The overall risk posture is **high**. Both the Node.js and Python dependency sets contain critical and high-severity vulnerabilities, several with known CVE identifiers. The Dockerfile also has security weaknesses that should be addressed.

---

## 2. Node.js / CDK Dependencies (`cdk-v2/`)

### 2.1 Outdated Packages

All direct dependencies were checked against the npm registry. The table below shows current pinned versions versus the latest available.

| Package | Current | Wanted | Latest | Status |
|---|---|---|---|---|
| `aws-cdk` (dev) | 2.37.1 | 2.37.1 | 2.1144.0 | :red_circle: Severely outdated |
| `aws-cdk-lib` | 2.37.1 | 2.37.1 | 2.272.0 | :red_circle: Severely outdated |
| `constructs` | 10.1.77 | 10.8.1 | 10.8.1 | :yellow_circle: Behind latest |
| `typescript` (dev) | 3.9.10 | 3.9.10 | 7.0.2 | :red_circle: Severely outdated |
| `jest` (dev) | 27.5.1 | 27.5.1 | 30.5.2 | :red_circle: Major versions behind |
| `ts-jest` (dev) | 27.1.5 | 27.1.5 | 29.4.14 | :red_circle: Major versions behind |
| `ts-node` (dev) | 10.9.1 | 10.9.2 | 10.9.2 | :green_circle: Patch behind |
| `@types/jest` (dev) | 27.5.2 | 27.5.2 | 30.0.0 | :red_circle: Major versions behind |
| `@types/node` (dev) | 10.17.27 | 10.17.27 | 26.6.4 | :red_circle: Severely outdated |
| `@types/prettier` (dev) | 2.6.0 | 2.6.0 | 2.7.3 | :yellow_circle: Behind latest |

### 2.2 Vulnerability Scan (npm audit)

**Summary:** 31 vulnerabilities found (2 critical, 11 high, 7 moderate, 11 low)

#### Critical Severity

| Package | Advisory | Description |
|---|---|---|
| `@babel/traverse` | [GHSA-67hx-6x53-jw92](https://github.com/advisories/GHSA-67hx-6x53-jw92) | Arbitrary code execution when compiling specifically crafted malicious code |
| `form-data` | [GHSA-fjxv-7rqg-78g4](https://github.com/advisories/GHSA-fjxv-7rqg-78g4) | Unsafe random function used for choosing multipart boundary |
| `form-data` | [GHSA-hmw2-7cc7-3qxx](https://github.com/advisories/GHSA-hmw2-7cc7-3qxx) | CRLF injection via unescaped multipart field names and filenames |

#### High Severity

| Package | Advisory | Description |
|---|---|---|
| `aws-cdk-lib` | [GHSA-rx28-r23p-2qc3](https://github.com/advisories/GHSA-rx28-r23p-2qc3) | EKS overly permissive trust policies |
| `aws-cdk-lib` | [GHSA-v4mq-x674-ff73](https://github.com/advisories/GHSA-v4mq-x674-ff73) | IAM OIDC custom resource allows connection to unauthorized OIDC provider |
| `aws-cdk-lib` | [GHSA-qq4x-c6h6-rfxh](https://github.com/advisories/GHSA-qq4x-c6h6-rfxh) | Insertion of Sensitive Information into Log File (Cognito UserPoolClient) |
| `aws-cdk-lib` | [GHSA-5pq3-h73f-66hr](https://github.com/advisories/GHSA-5pq3-h73f-66hr) | CodePipeline trusted entities are too broad |
| `aws-cdk-lib` | [GHSA-999r-qq7v-r334](https://github.com/advisories/GHSA-999r-qq7v-r334) | OS Command Injection in NodejsFunction Bundling |
| `aws-cdk-lib` | [GHSA-vcrf-j523-4mrf](https://github.com/advisories/GHSA-vcrf-j523-4mrf) | OS Command Injection in NodejsFunction Docker Bundling |
| `aws-cdk-lib` | [GHSA-464c-974j-9xm6](https://github.com/advisories/GHSA-464c-974j-9xm6) | CodeBuild S3 Log Encryption Boolean Inversion |
| `brace-expansion` | [GHSA-v6h2-p8h4-qcjw](https://github.com/advisories/GHSA-v6h2-p8h4-qcjw) | Regular Expression Denial of Service |
| `brace-expansion` | [GHSA-f886-m6hf-6m8v](https://github.com/advisories/GHSA-f886-m6hf-6m8v) | Zero-step sequence causes process hang and memory exhaustion |
| `brace-expansion` | [GHSA-3jxr-9vmj-r5cp](https://github.com/advisories/GHSA-3jxr-9vmj-r5cp) | DoS via exponential-time expansion |
| `brace-expansion` | [GHSA-mh99-v99m-4gvg](https://github.com/advisories/GHSA-mh99-v99m-4gvg) | DoS via unbounded expansion length (out-of-memory) |
| `brace-expansion` | [GHSA-rgw5-rvv9-x895](https://github.com/advisories/GHSA-rgw5-rvv9-x895) | DoS bypassing CVE-2026-14257 mitigation |
| `brace-expansion` | [GHSA-q2hr-2g5m-vwhr](https://github.com/advisories/GHSA-q2hr-2g5m-vwhr) | Quadratic-time expansion causes CPU DoS |
| `brace-expansion` | [GHSA-qhr7-859c-m2p7](https://github.com/advisories/GHSA-qhr7-859c-m2p7) | DoS via uncontrolled recursion (stack exhaustion) |
| `brace-expansion` | [GHSA-6j4f-fj2g-mc7p](https://github.com/advisories/GHSA-6j4f-fj2g-mc7p) | DoS via uncontrolled recursion in parseCommaParts |
| `braces` | [GHSA-grv7-fg5c-xmjg](https://github.com/advisories/GHSA-grv7-fg5c-xmjg) | Uncontrolled resource consumption |
| `braces` | [GHSA-vfj7-8cjw-p6xm](https://github.com/advisories/GHSA-vfj7-8cjw-p6xm) | Stack-exhaustion DoS through deeply nested patterns |
| `browserslist` | [GHSA-c83g-rgw3-j3cx](https://github.com/advisories/GHSA-c83g-rgw3-j3cx) | Unbounded memory growth leading to OOM |
| `browserslist` | [GHSA-73wf-gq98-2v4g](https://github.com/advisories/GHSA-73wf-gq98-2v4g) | Uncaught crash via untrusted custom stats |
| `cross-spawn` | [GHSA-3xgq-45jj-v275](https://github.com/advisories/GHSA-3xgq-45jj-v275) | Regular Expression Denial of Service (ReDoS) |
| `js-yaml` | [GHSA-mh29-5h37-fv8m](https://github.com/advisories/GHSA-mh29-5h37-fv8m) | Prototype pollution in merge key |
| `js-yaml` | [GHSA-h67p-54hq-rp68](https://github.com/advisories/GHSA-h67p-54hq-rp68) | Quadratic-complexity DoS in merge key handling |
| `js-yaml` | [GHSA-52cp-r559-cp3m](https://github.com/advisories/GHSA-52cp-r559-cp3m) | YAML merge-key chains force quadratic CPU consumption |
| `js-yaml` | [GHSA-5p4m-2wfm-xmqj](https://github.com/advisories/GHSA-5p4m-2wfm-xmqj) | Quadratic CPU consumption in !!omap resolution |
| `js-yaml` | [GHSA-2883-xcg3-v3hh](https://github.com/advisories/GHSA-2883-xcg3-v3hh) | maxTotalMergeKeys does not limit CPU for empty merge sources |
| `lodash` | [GHSA-r5fr-rjxr-66jc](https://github.com/advisories/GHSA-r5fr-rjxr-66jc) | Code Injection via `_.template` imports key names |
| `lodash` | [GHSA-f23m-r3pf-42rh](https://github.com/advisories/GHSA-f23m-r3pf-42rh) | Prototype Pollution via array path bypass in `_.unset`/`_.omit` |
| `lodash` | [GHSA-xxjr-mmjv-4gpg](https://github.com/advisories/GHSA-xxjr-mmjv-4gpg) | Prototype Pollution in `_.unset` and `_.omit` functions |
| `minimatch` | [GHSA-3ppc-4f35-3m26](https://github.com/advisories/GHSA-3ppc-4f35-3m26) | ReDoS via repeated wildcards |
| `minimatch` | [GHSA-7r86-cg39-jmmj](https://github.com/advisories/GHSA-7r86-cg39-jmmj) | ReDoS: matchOne() combinatorial backtracking |
| `minimatch` | [GHSA-23c5-xmqv-rm74](https://github.com/advisories/GHSA-23c5-xmqv-rm74) | ReDoS: nested extglobs generate catastrophic backtracking |
| `picomatch` | [GHSA-3v7f-55p6-f55p](https://github.com/advisories/GHSA-3v7f-55p6-f55p) | Method Injection in POSIX Character Classes |
| `picomatch` | [GHSA-c2c7-rcm5-vvqj](https://github.com/advisories/GHSA-c2c7-rcm5-vvqj) | ReDoS via extglob quantifiers |
| `semver` | [GHSA-c2qf-rxjj-qqgw](https://github.com/advisories/GHSA-c2qf-rxjj-qqgw) | Regular Expression Denial of Service |
| `ws` | [GHSA-3h5v-q93c-6h6q](https://github.com/advisories/GHSA-3h5v-q93c-6h6q) | DoS when handling a request with many HTTP headers |
| `ws` | [GHSA-96hv-2xvq-fx4p](https://github.com/advisories/GHSA-96hv-2xvq-fx4p) | Memory exhaustion DoS from tiny fragments and data chunks |

#### Moderate Severity

| Package | Advisory | Description |
|---|---|---|
| `@babel/helpers` | [GHSA-968p-4wvh-cqc8](https://github.com/advisories/GHSA-968p-4wvh-cqc8) | Inefficient RegExp complexity in generated code |
| `micromatch` | [GHSA-952p-6rrq-rcjv](https://github.com/advisories/GHSA-952p-6rrq-rcjv) | Regular Expression Denial of Service (ReDoS) |
| `sprintf-js` | [GHSA-hp3w-g68c-fv3c](https://github.com/advisories/GHSA-hp3w-g68c-fv3c) | DoS through unbounded precision specifiers |
| `tough-cookie` | [GHSA-72xf-g2v4-qvf3](https://github.com/advisories/GHSA-72xf-g2v4-qvf3) | Prototype Pollution vulnerability |
| `word-wrap` | [GHSA-j8xg-fqg3-53r7](https://github.com/advisories/GHSA-j8xg-fqg3-53r7) | Regular Expression Denial of Service |
| `yaml` | [GHSA-48c2-rrv3-qjmp](https://github.com/advisories/GHSA-48c2-rrv3-qjmp) | Stack Overflow via deeply nested YAML collections |

#### Low Severity

| Package | Advisory | Description |
|---|---|---|
| `@babel/core` | [GHSA-4x5r-pxfx-6jf8](https://github.com/advisories/GHSA-4x5r-pxfx-6jf8) | Arbitrary File Read via sourceMappingURL Comment |
| `@tootallnate/once` | [GHSA-vpq2-c234-7xj6](https://github.com/advisories/GHSA-vpq2-c234-7xj6) | Incorrect Control Flow Scoping |
| `diff` | [GHSA-73rr-hh4g-fpgx](https://github.com/advisories/GHSA-73rr-hh4g-fpgx) | DoS in parsePatch and applyPatch |

> **Note:** Several low-severity packages (e.g., `http-proxy-agent`, `jsdom`, `jest-environment-jsdom`, `jest-config`, `@jest/core`, `jest`, `jest-cli`, `jest-runner`, `argparse`) are flagged because they transitively depend on the vulnerable packages listed above. These transitive dependencies account for the remaining vulnerability count entries reaching 31 total.

---

## 3. Python / Flask Dependencies (`flask-docker-app/`)

### 3.1 Outdated Packages

| Package | Current (Pinned) | Latest Available | Status |
|---|---|---|---|
| `Click` | 7.0 | 8.1.8 | :red_circle: Major version behind |
| `Flask` | 1.1.1 | 3.1.3 | :red_circle: Multiple major versions behind |
| `itsdangerous` | 1.1.0 | 2.2.0 | :red_circle: Major version behind |
| `Jinja2` | 2.11.3 | 3.1.6 | :red_circle: Major version behind |
| `MarkupSafe` | 1.1.1 | 3.0.4 | :red_circle: Multiple major versions behind |
| `Werkzeug` | 0.15.6 | 3.1.9 | :red_circle: Multiple major versions behind |

### 3.2 Vulnerability Scan (pip-audit)

**Summary:** 17 known vulnerabilities found across 4 packages (Click, Flask, Jinja2, Werkzeug). 2 packages (itsdangerous, MarkupSafe) have no known vulnerabilities at their pinned versions.

#### Click 7.0 (1 vulnerability)

| CVE / ID | Fix Version | Description |
|---|---|---|
| [CVE-2026-7246](https://osv.dev/vulnerability/PYSEC-2026-2132) (PYSEC-2026-2132) | 8.3.3 | Command injection vulnerability in `click.edit()` allowing attackers to pass arbitrary OS commands from an unprivileged account. |

#### Flask 1.1.1 (2 vulnerabilities)

| CVE / ID | Fix Version | Description |
|---|---|---|
| [CVE-2023-30861](https://osv.dev/vulnerability/PYSEC-2023-62) (PYSEC-2023-62) | 2.2.5 / 2.3.2 | Response intended for one client may be cached and sent to other clients by a caching proxy. If cookies are also cached, session cookies may leak across clients. |
| [CVE-2026-27205](https://osv.dev/vulnerability/PYSEC-2026-2151) (PYSEC-2026-2151) | 3.1.3 | When the session object is accessed, Flask fails to set the `Vary: Cookie` header in certain code paths, leading to cache poisoning of sensitive information. |

#### Jinja2 2.11.3 (4 vulnerabilities)

| CVE / ID | Fix Version | Description |
|---|---|---|
| [CVE-2024-22195](https://osv.dev/vulnerability/PYSEC-2026-1473) (PYSEC-2026-1473) | 3.1.3 | `xmlattr` filter accepts keys containing spaces, enabling XSS if user-controlled keys are rendered in templates. |
| [CVE-2025-27516](https://osv.dev/vulnerability/PYSEC-2026-1471) (PYSEC-2026-1471) | 3.1.6 | The `|attr` filter bypasses the sandbox, allowing arbitrary Python code execution when an attacker controls template content. |
| [CVE-2024-34064](https://osv.dev/vulnerability/PYSEC-2026-1474) (PYSEC-2026-1474) | 3.1.4 | `xmlattr` filter accepts keys containing `/`, `>`, `=` and other non-attribute characters, enabling XSS. |
| [CVE-2024-56326](https://osv.dev/vulnerability/PYSEC-2026-1475) (PYSEC-2026-1475) | 3.1.5 | Indirect calls to `str.format` bypass sandbox detection, allowing arbitrary Python code execution when an attacker controls template content. |

#### Werkzeug 0.15.6 (11 vulnerabilities)

| CVE / ID | Fix Version | Description |
|---|---|---|
| [CVE-2022-29361](https://osv.dev/vulnerability/PYSEC-2022-203) (PYSEC-2022-203) | 2.1.1 | Improper parsing of HTTP requests allows HTTP Request Smuggling via crafted requests (disputed by vendor for unsupported configurations). |
| [CVE-2023-25577](https://osv.dev/vulnerability/PYSEC-2023-58) (PYSEC-2023-58) | 2.2.3 | Multipart form data parser processes unlimited parts, enabling DoS through CPU and memory exhaustion. |
| [CVE-2023-23934](https://osv.dev/vulnerability/PYSEC-2023-57) (PYSEC-2023-57) | 2.2.3 | Nameless cookie parsing allows a compromised adjacent subdomain to set malicious cookies. |
| [CVE-2023-46136](https://osv.dev/vulnerability/PYSEC-2023-221) (PYSEC-2023-221) | 2.3.8 / 3.0.1 | Crafted multipart uploads with CR/LF followed by large data cause DoS via unbounded buffer growth. |
| [CVE-2024-49766](https://osv.dev/vulnerability/PYSEC-2026-2045) (PYSEC-2026-2045) | 3.0.6 | `safe_join()` does not catch UNC paths on Python < 3.11 on Windows, potentially allowing unintended data access. |
| [CVE-2024-34069](https://osv.dev/vulnerability/PYSEC-2026-2043) (PYSEC-2026-2043) | 3.0.3 | Debugger allows remote code execution if an attacker gets the developer to interact with a controlled domain and enter the debugger PIN. |
| [CVE-2025-66221](https://osv.dev/vulnerability/PYSEC-2026-2046) (PYSEC-2026-2046) | 3.1.4 | `safe_join` allows Windows device names, causing `send_from_directory` to hang indefinitely when serving such paths. |
| [CVE-2026-21860](https://osv.dev/vulnerability/PYSEC-2026-2044) (PYSEC-2026-2044) | 3.1.5 | `safe_join` allows Windows device names with file extensions or trailing spaces, causing hangs. |
| [CVE-2026-27199](https://osv.dev/vulnerability/PYSEC-2026-2320) (PYSEC-2026-2320) | 3.1.6 | `safe_join` allows Windows device names if preceded by other path segments. |
| [CVE-2026-102598](https://osv.dev/vulnerability/GHSA-g6x2-hccm-hh4m) (GHSA-g6x2-hccm-hh4m) | 3.1.9 | `safe_join` allows Windows device names with empty ADS markers on NTFS. |

> **Note:** itsdangerous 1.1.0 and MarkupSafe 1.1.1 have no known vulnerabilities at their pinned versions, but are still significantly outdated.

---

## 4. Dockerfile Security Review (`flask-docker-app/Dockerfile`)

```dockerfile
FROM public.ecr.aws/amazonlinux/amazonlinux:2022
RUN yum install python3 && \
curl -O https://bootstrap.pypa.io/get-pip.py && \
python3 get-pip.py && \
pip install --upgrade pip && \
pip install flask

RUN alias python=python3

ENV FLASK_APP app.py
ENV PLATFORM 'Amazon ECS'

WORKDIR /app
COPY . /app/

CMD ["python3", "app.py"]
```

### Findings

| # | Issue | Severity | Description |
|---|---|---|---|
| 1 | **Unpinned base image** | High | `amazonlinux:2022` uses no digest or specific tag version. Builds are not reproducible and may silently pull different images over time, potentially introducing supply-chain risk. |
| 2 | **Flask version not pinned in `pip install`** | High | `pip install flask` in the Dockerfile does not use a version pin, meaning any build will pull the latest Flask version from PyPI. This contradicts the `requirements.txt` pinning and may produce inconsistent or broken images. The Dockerfile should use `pip install -r requirements.txt` instead. |
| 3 | **Container runs as root** | Medium | No `USER` directive is set, so the application runs as root inside the container. If the application is compromised, the attacker gains root-level access within the container. A non-root user should be created and used. |
| 4 | **pip bootstrap via curl** | Low | Downloading `get-pip.py` via `curl -O` lacks checksum verification. While the download uses HTTPS, pinning a known hash would improve supply-chain integrity. |

---

## 5. Code Security Observations

### 5.1 Flask Debug Mode (`flask-docker-app/app.py`)

```python
app.run(
    debug=os.getenv('FLASK_DEBUG', False),
    host='0.0.0.0',
    port=5000
)
```

**Finding (Medium):** The `debug` parameter is set via `os.getenv('FLASK_DEBUG', False)`. While the default is `False`, `os.getenv()` returns a string, so any non-empty string value (including `"False"` or `"0"`) will be truthy in Python. If `FLASK_DEBUG` is set to any value at all, debug mode will be enabled. Debug mode in production exposes the interactive Werkzeug debugger, which can lead to remote code execution. The code should explicitly check for a specific value like `"1"` or `"true"`.

### 5.2 Wildcard IAM Resources (`cdk-v2/lib/ecs_cdk-stack.ts`)

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

**Finding (Medium):** The execution role policy uses a wildcard resource (`'*'`) for all listed actions. While `ecr:GetAuthorizationToken` requires `*` as the resource (it is an account-level action), the other ECR actions (`BatchCheckLayerAvailability`, `GetDownloadUrlForLayer`, `BatchGetImage`) should be scoped to the specific ECR repository ARN. Similarly, the CloudWatch Logs actions (`CreateLogStream`, `PutLogEvents`) should be scoped to the specific log group ARN. This follows the AWS principle of least privilege.

### 5.3 Application Listening on All Interfaces (`flask-docker-app/app.py`)

```python
host='0.0.0.0'
```

**Finding (Low):** The application binds to all network interfaces. While this is expected and necessary for a containerized application behind an ALB, it is worth noting that there is no application-level authentication or rate limiting. All security depends on the ALB and VPC configuration defined in the CDK stack.

---

## 6. Recommendations (Prioritized by Severity)

### Critical Priority

1. **Upgrade `aws-cdk-lib` and `aws-cdk` to latest v2** (currently 2.37.1, latest 2.272.0+)
   - Addresses 7 security advisories in aws-cdk-lib itself, plus transitive dependency vulnerabilities in `minimatch`, `semver`, and `yaml`.
   - Run: `npm install aws-cdk-lib@latest aws-cdk@latest`

2. **Upgrade all Python dependencies to their latest versions**
   - Werkzeug 0.15.6 has 11 known vulnerabilities. Update to 3.1.9.
   - Jinja2 2.11.3 has 4 known vulnerabilities including sandbox escapes. Update to 3.1.6.
   - Flask 1.1.1 has 2 known vulnerabilities. Update to 3.1.3.
   - Click 7.0 has 1 command injection vulnerability. Update to 8.1.8.
   - Update `requirements.txt` to:
     ```
     Click==8.1.8
     Flask==3.1.3
     itsdangerous==2.2.0
     Jinja2==3.1.6
     MarkupSafe==3.0.4
     Werkzeug==3.1.9
     ```
   - Note: Upgrading from Flask 1.x to 3.x is a major version jump and may require code changes.

3. **Fix the Dockerfile to use `requirements.txt`**
   - Replace `pip install flask` with `pip install -r requirements.txt` so that pinned dependency versions are respected.

### High Priority

4. **Upgrade Jest ecosystem** (jest, ts-jest, @types/jest)
   - Resolves transitive vulnerabilities through `@babel/traverse` (critical), `form-data` (critical), and multiple high-severity dependencies.
   - These are dev dependencies, so the risk is limited to the build/CI environment.

5. **Upgrade TypeScript** from 3.9.x to the latest supported version
   - TypeScript 3.9 is end-of-life. The latest version (7.0.2) offers significant security and performance improvements.

6. **Scope IAM policy resources** in `ecs_cdk-stack.ts`
   - Replace `resources: ['*']` with specific ARNs for ECR repository and CloudWatch Logs log group.
   - Keep `'*'` only for `ecr:GetAuthorizationToken` which requires it.

7. **Add a non-root user to the Dockerfile**
   - Add `RUN useradd -r appuser` and `USER appuser` before the `CMD` directive.

### Medium Priority

8. **Fix debug mode check** in `app.py`
   - Change `debug=os.getenv('FLASK_DEBUG', False)` to `debug=os.getenv('FLASK_DEBUG', '').lower() in ('1', 'true')` to prevent accidental activation.

9. **Pin the Dockerfile base image** to a specific digest or version tag for reproducible builds.

10. **Add hash verification for pip bootstrap** in the Dockerfile, or switch to a base image that includes pip.

### Low Priority

11. **Run `npm audit fix`** to automatically resolve the subset of vulnerabilities that have compatible semver fixes (non-breaking).

12. **Update `@types/node`** from 10.17.27 to a version matching the Node.js runtime in use (Node.js 22 types).

13. **Consider adding Python tests** and a Python linter (e.g., `bandit` for security scanning) to the CI pipeline.

14. **Consider adding `npm audit` and `pip-audit`** as steps in the CI/CD pipeline to catch vulnerabilities before deployment.

---

*Report generated using `npm audit`, `npm outdated`, `pip-audit v2.9.0`, and manual code review.*
