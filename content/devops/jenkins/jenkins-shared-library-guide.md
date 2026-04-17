---
title: "Jenkins Shared Library — Complete Guide for AWS & DevOps Engineers"
date: 2026-04-17
description: "A comprehensive, real-world reference for building, structuring, and consuming Jenkins Shared Libraries in enterprise CI/CD pipelines. Covers vars/, src/, resources/, testing, security, and full microservices examples."
author: "DB"
tags: ["Jenkins", "CI/CD", "DevOps", "Groovy", "Kubernetes", "Docker", "Shared Library", "Pipeline"]
categories: ["DevOps"]
series: "Jenkins Mastery"
level: "Intermediate"
---

> A comprehensive, real-world reference for building, structuring, and consuming Jenkins Shared Libraries in enterprise CI/CD pipelines — from first principles to a production-grade microservices setup.

---

## 🌐 What is a Jenkins Shared Library?

A **Jenkins Shared Library** is a mechanism that lets you extract reusable Pipeline code — Groovy scripts, helper classes, and static resources — into a **separate Git repository** and import them into any Jenkins Pipeline across your organisation.

Think of it like an **npm package** or a **Python library**, but for Jenkins Pipelines. Instead of copy-pasting 200 lines of Docker-build-and-push logic into every Jenkinsfile, you write it once in the shared library and call a single function:

```groovy
// ❌ Before shared library — this block repeated across 40 repos
stage('Build Docker Image') {
    sh "docker build -t myrepo/myapp:${env.BUILD_NUMBER} ."
    sh "docker push myrepo/myapp:${env.BUILD_NUMBER}"
    sh "docker tag myrepo/myapp:${env.BUILD_NUMBER} myrepo/myapp:latest"
    sh "docker push myrepo/myapp:latest"
}

// ✅ After shared library — one line replaces all of the above
dockerBuildAndPush(image: 'myrepo/myapp', tag: env.BUILD_NUMBER)
```

The library lives in its own Git repository (e.g., `jenkins-shared-library`) and is versioned independently from application code.

---

## 🤔 Why Use a Shared Library?

| Problem Without Shared Libraries | Solution With Shared Libraries |
|----------------------------------|-------------------------------|
| Duplicate CI/CD logic across 50+ repos | Single source of truth |
| Bug fix requires PRs in every repo | Fix once, all pipelines inherit it |
| Inconsistent deployment practices | Enforced standards via shared steps |
| Security checks scattered everywhere | Central, auditable security gates |
| Onboarding new teams is slow | New team uses a 5-line Jenkinsfile |

{{< callout type="info" title="Real-World Impact" >}}
A fintech company has 80 microservices. Without a shared library, a change to the SonarQube quality gate threshold requires **80 pull requests**. With a shared library, it requires exactly **one commit** in the library repository — and all 80 pipelines pick it up on their next run.
{{< /callout >}}

---

## 🗂️ Repository & Directory Structure

The structure is **strictly enforced by Jenkins** — directory names and file conventions are not optional.

```
jenkins-shared-library/
├── src/                          # Groovy source classes (object-oriented helpers)
│   └── com/
│       └── mycompany/
│           ├── Docker.groovy
│           ├── Kubernetes.groovy
│           ├── Notification.groovy
│           └── Utils.groovy
│
├── vars/                         # Global variables / custom Pipeline steps
│   ├── dockerBuildAndPush.groovy
│   ├── dockerBuildAndPush.txt    # (optional) Step docs shown in Jenkins UI
│   ├── deployToKubernetes.groovy
│   ├── sendSlackNotification.groovy
│   ├── runSonarAnalysis.groovy
│   └── standardPipeline.groovy   # Full opinionated pipeline template
│
├── resources/                    # Non-Groovy static files (templates, configs)
│   ├── com/
│   │   └── mycompany/
│   │       ├── k8s-deployment-template.yaml
│   │       ├── sonar-project.properties
│   │       └── pod-templates/
│   │           └── maven-agent.yaml
│   └── scripts/
│       └── db-migrate.sh
│
├── test/                         # Unit tests (Spock / JenkinsPipelineUnit)
│   └── groovy/
│       └── com/
│           └── mycompany/
│               ├── DockerSpec.groovy
│               └── DeploySpec.groovy
│
├── build.gradle                  # Build file for running tests
└── README.md
```

### `src/` — Groovy Classes

- Contains **Groovy classes** organised in Java-style packages.
- Compiled and added to the classpath when the library is loaded.
- **Cannot** directly call Jenkins DSL steps (`sh`, `echo`, `stage`) unless you pass the `steps` object into them.
- Best for: pure logic, API clients, data transformation, string manipulation.

### `vars/` — Global Pipeline Steps

- Contains **Groovy scripts** that become **globally available functions** in any Pipeline that loads the library.
- Each `.groovy` file becomes a callable step named after the file — `vars/dockerBuildAndPush.groovy` → callable as `dockerBuildAndPush(...)` in a Jenkinsfile.
- **CAN** call Jenkins DSL steps directly — they run in the Pipeline's CPS (Continuation Passing Style) context.
- Optionally, pair each `.groovy` with a `.txt` file of the same name — its content appears as the step's help text in the Jenkins "Pipeline Syntax" UI.

### `resources/` — Static Files

- Stores **static, non-Groovy files**: YAML templates, shell scripts, config files.
- Loaded at runtime via `libraryResource('path/relative/to/resources/')`.
- Jenkins does **not** treat files here as Groovy — they are served as plain text.

### `test/` — Unit Tests

- Not a Jenkins-enforced directory — convention for housing **JenkinsPipelineUnit** or **Spock** tests.
- Wired up via `build.gradle` or `pom.xml`.

---

## ⚙️ Library Registration in Jenkins

### Global Library (Manage Jenkins)

Registered once — available to **every Pipeline** on the Jenkins instance without any `@Library` annotation if marked as "Load implicitly".

**Path:** `Manage Jenkins → Configure System → Global Pipeline Libraries`

| Field | Example Value |
|-------|--------------|
| Name | `company-shared-lib` |
| Default version | `main` |
| Load implicitly | `true` (auto-imported to all jobs) |
| Retrieval method | Modern SCM → Git |
| Repository URL | `https://github.com/myorg/jenkins-shared-library.git` |
| Credentials | `github-service-account` |

### Folder-Level Library

Registered on a Jenkins **Folder** item. Only Pipelines inside that folder (and sub-folders) can use it. Ideal for team-specific or product-specific libraries.

**Path:** `<Folder> → Configure → Pipeline Libraries`

Use this to give the Platform team a different library than the Data team.

### Pipeline-Level `@Library` Annotation

Any Pipeline can import a library directly, specifying a version, branch, or tag:

```groovy
// Pin to a specific release tag — recommended for production
@Library('company-shared-lib@v2.3.1') _

// Use the main branch
@Library('company-shared-lib') _

// Use a feature branch during development
@Library('company-shared-lib@feature/new-deploy-step') _

// Import multiple libraries simultaneously
@Library(['company-shared-lib@v2.3.1', 'security-lib@main']) _
```

{{< callout type="tip" title="The Trailing Underscore" >}}
The trailing `_` after `@Library(...)` is **mandatory** when you don't import any specific class. It satisfies the Groovy import statement syntax. Without it your pipeline will fail with a syntax error.
{{< /callout >}}

---

## 📁 `vars/` — Global Variables & Steps

### Simple Step

**`vars/helloWorld.groovy`**

```groovy
def call(String name = 'World') {
    echo "Hello, ${name}! This ran from the shared library."
}
```

**Usage in Jenkinsfile:**

```groovy
@Library('company-shared-lib') _

pipeline {
    agent any
    stages {
        stage('Greet') {
            steps {
                helloWorld('Jenkins')
                // Output: Hello, Jenkins! This ran from the shared library.
            }
        }
    }
}
```

### `call()` Convention

Every `vars/` file **must expose a `call()` method** — this is what Jenkins invokes when you use the step name as a function. You can overload `call()` with different signatures:

```groovy
// vars/myStep.groovy

// Called as: myStep()
def call() {
    call([:])  // delegate to map version with empty map
}

// Called as: myStep(env: 'prod', region: 'us-east-1')
def call(Map config) {
    def environment = config.get('env',    'dev')
    def region      = config.get('region', 'us-east-1')
    echo "Deploying to ${environment} in ${region}"
}
```

### Accepting Parameters & Maps

Using a **Map parameter** is the idiomatic Jenkins Shared Library pattern — it gives callers named, optional arguments without overloading hell:

**`vars/runSonarAnalysis.groovy`**

```groovy
def call(Map config = [:]) {
    // Provide defaults, allow caller to override any value
    def projectKey  = config.get('projectKey',  env.JOB_NAME)
    def projectName = config.get('projectName', env.JOB_NAME)
    def sonarServer = config.get('sonarServer', 'SonarQubeServer')
    def qualityGate = config.get('qualityGate', true)
    def exclusions  = config.get('exclusions',  '**/test/**,**/vendor/**')

    withSonarQubeEnv(sonarServer) {
        sh """
            sonar-scanner \
              -Dsonar.projectKey=${projectKey} \
              -Dsonar.projectName=${projectName} \
              -Dsonar.exclusions=${exclusions}
        """
    }

    if (qualityGate) {
        timeout(time: 5, unit: 'MINUTES') {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "SonarQube Quality Gate failed: ${qg.status}"
            }
        }
    }
}
```

**Usage:**

```groovy
// Minimal — all defaults apply
runSonarAnalysis()

// Override specific options only
runSonarAnalysis(
    projectKey:  'payment-service',
    projectName: 'Payment Service',
    qualityGate: true,
    exclusions:  '**/generated/**'
)
```

---

## 🧱 `src/` — Groovy Classes

Classes in `src/` are standard Groovy with package declarations. They **cannot** call Jenkins DSL steps directly — you must pass the `steps` reference from the calling `vars/` script.

### Utility Class

**`src/com/mycompany/Docker.groovy`**

```groovy
package com.mycompany

class Docker implements Serializable {

    private def steps           // Reference to Jenkins Pipeline steps
    private String registry
    private String credentialsId

    Docker(steps, String registry, String credentialsId) {
        this.steps         = steps
        this.registry      = registry
        this.credentialsId = credentialsId
    }

    /**
     * Build a Docker image and return the full image reference.
     */
    String build(String imageName, String tag, String context = '.') {
        def fullImage = "${registry}/${imageName}:${tag}"
        steps.sh "docker build -t ${fullImage} ${context}"
        return fullImage
    }

    /**
     * Push a Docker image to the registry using stored credentials.
     */
    void push(String fullImage) {
        steps.withCredentials([
            steps.usernamePassword(
                credentialsId: credentialsId,
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )
        ]) {
            steps.sh """
                echo "\$DOCKER_PASS" | docker login ${registry} -u "\$DOCKER_USER" --password-stdin
                docker push ${fullImage}
            """
        }
    }

    /**
     * Tag an existing image with a new name.
     */
    void tag(String sourceImage, String targetImage) {
        steps.sh "docker tag ${sourceImage} ${targetImage}"
    }
}
```

{{< callout type="warning" title="implements Serializable — Never Skip This" >}}
`implements Serializable` is **required** on all `src/` classes used in Pipelines. Jenkins checkpoints pipeline state to disk between stages — non-serializable objects cause `NotSerializableException` and crash the build at runtime.
{{< /callout >}}

**`src/com/mycompany/Utils.groovy`** — Validation helper used across all steps:

```groovy
package com.mycompany

class Utils implements Serializable {

    /**
     * Validate that all required keys are present in a config map.
     * Throws a descriptive error if any are missing.
     */
    static void requireKeys(Map config, List<String> keys) {
        def missing = keys.findAll { !config.containsKey(it) || !config[it] }
        if (missing) {
            throw new IllegalArgumentException(
                "Missing required parameters: ${missing.join(', ')}"
            )
        }
    }

    /**
     * Convert a branch name to a safe Docker tag.
     * feature/my-feature → feature-my-feature
     */
    static String safeBranchTag(String branch) {
        return branch.replaceAll('[^a-zA-Z0-9._-]', '-').toLowerCase()
    }

    /**
     * Return true if the current branch is a production branch.
     */
    static boolean isProductionBranch(String branch) {
        return branch in ['main', 'master']
    }
}
```

### Using `src/` Classes in `vars/`

**`vars/dockerBuildAndPush.groovy`**

```groovy
import com.mycompany.Docker
import com.mycompany.Utils

def call(Map config = [:]) {
    Utils.requireKeys(config, ['image'])  // Fail fast with a clear error message

    def imageName  = config.image
    def tag        = config.get('tag',        env.BUILD_NUMBER)
    def registry   = config.get('registry',   'registry.mycompany.com')
    def credId     = config.get('credId',     'harbor-robot-credentials')
    def pushLatest = config.get('pushLatest', env.BRANCH_NAME == 'main')
    def context    = config.get('context',    '.')
    def dockerfile = config.get('dockerfile', 'Dockerfile')
    def buildArgs  = config.get('buildArgs',  [:])

    def buildArgsStr = buildArgs.collect { k, v -> "--build-arg ${k}=${v}" }.join(' ')

    // Pass 'this' so the class can call Jenkins DSL steps
    def docker = new Docker(this, registry, credId)

    stage("Build Image: ${imageName}:${tag}") {
        def fullImage = "${registry}/${imageName}:${tag}"
        sh "docker build -f ${dockerfile} ${buildArgsStr} -t ${fullImage} ${context}"
        docker.push(fullImage)

        if (pushLatest) {
            def latestImage = "${registry}/${imageName}:latest"
            docker.tag(fullImage, latestImage)
            docker.push(latestImage)
        }

        // Export for downstream stages to use
        env.DOCKER_IMAGE = "${registry}/${imageName}:${tag}"
        echo "✅ Image pushed: ${env.DOCKER_IMAGE}"
    }
}
```

---

## 📦 `resources/` — Static Files

### Loading a Resource File

Use `libraryResource()` to read any file under `resources/` as a string at runtime.

**`resources/com/mycompany/k8s-deployment-template.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: APP_NAME
  namespace: NAMESPACE
  labels:
    app: APP_NAME
    version: IMAGE_TAG
    environment: ENVIRONMENT
spec:
  replicas: REPLICA_COUNT
  selector:
    matchLabels:
      app: APP_NAME
  template:
    metadata:
      labels:
        app: APP_NAME
        version: IMAGE_TAG
    spec:
      containers:
        - name: APP_NAME
          image: REGISTRY/APP_NAME:IMAGE_TAG
          ports:
            - containerPort: 8080
          env:
            - name: ENVIRONMENT
              value: "ENVIRONMENT"
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          resources:
            requests:
              memory: "256Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: APP_NAME
  namespace: NAMESPACE
spec:
  selector:
    app: APP_NAME
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

**`vars/deployToKubernetes.groovy`** — Loading and substituting the template:

```groovy
def call(Map config = [:]) {
    def appName     = config.appName     ?: error('appName is required')
    def environment = config.environment ?: 'dev'
    def imageTag    = config.imageTag    ?: env.BUILD_NUMBER
    def namespace   = config.get('namespace',   environment)
    def replicas    = config.get('replicas',     environmentDefaults(environment).replicas)
    def kubeConfig  = config.get('kubeConfig',  "kubeconfig-${environment}")
    def registry    = config.get('registry',    'registry.mycompany.com')
    def waitTimeout = config.get('waitTimeout', '5m')

    stage("Deploy to ${environment.toUpperCase()}") {
        // Load the YAML template from the library's resources/ directory
        def template = libraryResource('com/mycompany/k8s-deployment-template.yaml')

        // Token substitution — replace placeholders with real values
        def manifest = template
            .replace('APP_NAME',      appName)
            .replace('IMAGE_TAG',     imageTag)
            .replace('NAMESPACE',     namespace)
            .replace('REGISTRY',      registry)
            .replace('ENVIRONMENT',   environment)
            .replace('REPLICA_COUNT', replicas.toString())

        writeFile file: "k8s-${environment}.yaml", text: manifest

        withCredentials([file(credentialsId: kubeConfig, variable: 'KUBECONFIG')]) {
            sh """
                kubectl apply -f k8s-${environment}.yaml
                kubectl rollout status deployment/${appName} \\
                    -n ${namespace} \\
                    --timeout=${waitTimeout}
            """
        }
        echo "✅ ${appName} deployed to ${environment} (image: ${imageTag})"
    }
}

// Helper: environment-specific defaults
private Map environmentDefaults(String env) {
    def defaults = [
        dev:     [replicas: 1],
        staging: [replicas: 2],
        prod:    [replicas: 4],
    ]
    return defaults.get(env, [replicas: 1])
}
```

---

## 🏗️ Real-World Application — Microservices CI/CD

### Project Layout

Imagine **MyCompany** has the following ecosystem:

```
mycompany/
├── jenkins-shared-library/     ← This shared library repo
├── payment-service/            ← Java Spring Boot microservice
├── order-service/              ← Node.js microservice
├── inventory-service/          ← Python FastAPI microservice
└── frontend-app/               ← React single-page application
```

All four applications need the same CI/CD pipeline stages:

1. Checkout source code
2. Run unit tests + code coverage
3. SonarQube static analysis
4. Build & push Docker image to Harbor registry
5. Deploy to Kubernetes (dev → staging → prod with manual approval)
6. Send Slack notification with build status

Without a shared library, this logic is duplicated across four Jenkinsfiles — 200+ lines each. With a shared library, each Jenkinsfile is under 50 lines.

### Notification Helper

**`vars/sendSlackNotification.groovy`**

```groovy
def call(Map config = [:]) {
    def channel  = config.get('channel',  '#ci-cd-alerts')
    def status   = config.get('status',   'SUCCESS')   // SUCCESS | FAILURE | UNSTABLE
    def appName  = config.get('appName',  env.JOB_NAME)
    def buildUrl = config.get('buildUrl', env.BUILD_URL)
    def message  = config.get('message',  '')

    def colorMap = [
        SUCCESS:  '#36a64f',   // green
        FAILURE:  '#dc143c',   // red
        UNSTABLE: '#ffa500',   // orange
    ]

    def emoji = status == 'SUCCESS' ? '✅' : (status == 'FAILURE' ? '❌' : '⚠️')
    def color = colorMap.get(status, '#808080')

    def text = """
${emoji} *${appName}* — Build #${env.BUILD_NUMBER} — *${status}*
${message ? "• ${message}\n" : ''}• Branch: `${env.BRANCH_NAME ?: 'N/A'}`
• Duration: ${currentBuild.durationString}
• <${buildUrl}|View Build>
    """.trim()

    slackSend(
        channel: channel,
        color:   color,
        message: text
    )
}
```

### Complete `standardPipeline` Template Step

This is the most powerful pattern — one opinionated step that wraps the entire pipeline. Application teams call it with a config map and the library does everything.

**`vars/standardPipeline.groovy`**

```groovy
import com.mycompany.Utils

def call(Map config = [:]) {
    Utils.requireKeys(config, ['appName', 'registry'])

    def appName    = config.appName
    def registry   = config.registry
    def language   = config.get('language',  'java')   // java | node | python
    def buildCmd   = config.get('buildCmd',  defaultBuildCmd(language))
    def testCmd    = config.get('testCmd',   defaultTestCmd(language))
    def sonarKey   = config.get('sonarKey',  appName)
    def prodBranch = config.get('prodBranch', 'main')

    pipeline {
        agent {
            kubernetes {
                yaml libraryResource("com/mycompany/pod-templates/${language}-agent.yaml")
            }
        }

        environment {
            APP_NAME = appName
            REGISTRY = registry
        }

        stages {
            stage('Build & Test') {
                steps {
                    sh buildCmd
                    sh testCmd
                }
                post {
                    always {
                        junit allowEmptyResults: true, testResults: '**/test-results/**/*.xml'
                    }
                }
            }

            stage('Code Quality') {
                steps {
                    runSonarAnalysis(projectKey: sonarKey, qualityGate: true)
                }
            }

            stage('Container Build') {
                steps {
                    dockerBuildAndPush(
                        image:      appName,
                        registry:   registry,
                        pushLatest: env.BRANCH_NAME == prodBranch
                    )
                }
            }

            stage('Deploy Dev') {
                steps {
                    deployToKubernetes(appName: appName, environment: 'dev', imageTag: env.BUILD_NUMBER)
                }
            }

            stage('Deploy Staging') {
                when { branch prodBranch }
                steps {
                    deployToKubernetes(appName: appName, environment: 'staging', imageTag: env.BUILD_NUMBER)
                }
            }

            stage('Deploy Production') {
                when { branch prodBranch }
                steps {
                    input message: "Approve deployment of ${appName} build #${env.BUILD_NUMBER} to PRODUCTION?"
                    deployToKubernetes(appName: appName, environment: 'prod', imageTag: env.BUILD_NUMBER, replicas: 4)
                }
            }
        }

        post {
            success {
                sendSlackNotification(channel: '#deployments', status: 'SUCCESS', appName: appName)
            }
            failure {
                sendSlackNotification(channel: '#ci-cd-alerts', status: 'FAILURE', appName: appName,
                    message: 'Check the build logs for errors.')
            }
            always {
                cleanWs()
            }
        }
    }
}

private String defaultBuildCmd(String lang) {
    return [java: 'mvn package -DskipTests', node: 'npm ci && npm run build', python: 'pip install -r requirements.txt'].get(lang, 'echo No build command')
}

private String defaultTestCmd(String lang) {
    return [java: 'mvn test', node: 'npm test', python: 'pytest --junitxml=test-results/results.xml'].get(lang, 'echo No test command')
}
```

### Full Jenkinsfile for Each Microservice

With the shared library, every application's Jenkinsfile becomes this simple:

**`payment-service/Jenkinsfile`** — Java Spring Boot service:

```groovy
@Library('company-shared-lib@v2.3.1') _

standardPipeline(
    appName:  'payment-service',
    registry: 'registry.mycompany.com',
    language: 'java',
    sonarKey: 'payment-service'
)
```

**`order-service/Jenkinsfile`** — Node.js service:

```groovy
@Library('company-shared-lib@v2.3.1') _

standardPipeline(
    appName:  'order-service',
    registry: 'registry.mycompany.com',
    language: 'node'
)
```

**`inventory-service/Jenkinsfile`** — Python FastAPI service:

```groovy
@Library('company-shared-lib@v2.3.1') _

standardPipeline(
    appName:  'inventory-service',
    registry: 'registry.mycompany.com',
    language: 'python',
    testCmd:  'pytest --junitxml=test-results/results.xml --cov=app'
)
```

{{< callout type="tip" title="The Power of This Pattern" >}}
Four microservices. Four Jenkinsfiles. Each is **4 lines**. When a new team joins, they copy any one of these files, change the `appName`, and have a full production CI/CD pipeline in 5 minutes. When the DevOps team updates the deployment strategy or adds a security scan, **all four pipelines get the update automatically**.
{{< /callout >}}

---

## 🌿 Versioning & Branching Strategy

Treat the shared library like a product with semantic versioning:

```
main           ← stable, production-ready
develop        ← integration branch
feature/*      ← new steps / features
hotfix/*       ← urgent bug fixes
v1.0.0         ← tagged release
v2.0.0         ← tagged release (breaking changes)
```

**Rules to follow:**

```groovy
// ✅ CORRECT — pinned to a tag, safe for production
@Library('company-shared-lib@v2.3.1') _

// ⚠️  ACCEPTABLE — for development or staging environments only
@Library('company-shared-lib@develop') _

// ❌ DANGEROUS — a bad push to main breaks ALL pipelines simultaneously
@Library('company-shared-lib') _
```

| Rule | Reason |
|------|--------|
| Application teams pin to a **tag** | Protected from breaking changes in `main` |
| DevOps team owns the library repo | Clear accountability and review process |
| Maintain a `CHANGELOG.md` | Application teams know what changed between versions |
| Test new steps in a **sandbox folder** | Validate before merging to `main` |
| Use **semantic versioning** (MAJOR.MINOR.PATCH) | MAJOR = breaking change, MINOR = new feature, PATCH = bug fix |

---

## 🧪 Testing the Shared Library

### JenkinsPipelineUnit Setup

Add to your `build.gradle`:

```groovy
plugins {
    id 'groovy'
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'com.lesfurets:jenkins-pipeline-unit:1.23'
    testImplementation 'org.spockframework:spock-core:2.3-groovy-4.0'
    testImplementation 'org.codehaus.groovy:groovy-all:4.0.6'
}

test {
    useJUnitPlatform()
}

sourceSets {
    main {
        groovy {
            srcDirs = ['src', 'vars']
        }
    }
    test {
        groovy {
            srcDirs = ['test/groovy']
        }
    }
}
```

### Writing Unit Tests

**`test/groovy/com/mycompany/DockerSpec.groovy`**

```groovy
package com.mycompany

import com.lesfurets.jenkins.unit.BasePipelineTest
import org.junit.Before
import org.junit.Test
import static org.assertj.core.api.Assertions.assertThat

class DockerBuildAndPushSpec extends BasePipelineTest {

    def script

    @Override
    @Before
    void setUp() throws Exception {
        super.setUp()
        // Register mocks for Jenkins steps used inside the shared library
        helper.registerAllowedMethod('stage',           [String, Closure])
        helper.registerAllowedMethod('sh',              [String])
        helper.registerAllowedMethod('echo',            [String])
        helper.registerAllowedMethod('error',           [String])
        helper.registerAllowedMethod('withCredentials', [List, Closure])
        helper.registerAllowedMethod('usernamePassword',[Map], { args -> args })

        // Load the vars/ step under test
        script = loadScript('vars/dockerBuildAndPush.groovy')
    }

    @Test
    void 'should build and push image with default tag'() {
        // Arrange
        binding.setVariable('env', [BUILD_NUMBER: '42', BRANCH_NAME: 'feature/test'])

        // Act
        script.call(image: 'my-app')

        // Assert — verify sh commands were called with correct arguments
        def calls = helper.callStack.findAll { it.methodName == 'sh' }
        assertThat(calls.any { it.args[0].contains('docker build') }).isTrue()
        assertThat(calls.any { it.args[0].contains('docker push') }).isTrue()
    }

    @Test
    void 'should push latest tag when on main branch'() {
        binding.setVariable('env', [BUILD_NUMBER: '99', BRANCH_NAME: 'main'])

        script.call(image: 'my-app', pushLatest: true)

        def shCalls = helper.callStack
            .findAll { it.methodName == 'sh' }
            .collect { it.args[0] }

        assertThat(shCalls.any { it.contains(':latest') }).isTrue()
    }

    @Test(expected = Exception)
    void 'should throw when image parameter is not provided'() {
        script.call([:])  // Missing required 'image' key — must throw
    }
}
```

**Run tests locally:**

```bash
# Run all tests
./gradlew test

# Verbose output — shows each test name and result
./gradlew test --info

# Run a specific test class
./gradlew test --tests "com.mycompany.DockerBuildAndPushSpec"
```

---

## 🔐 Security Best Practices

### Credential Management

Never hardcode credentials. Always use Jenkins Credential Store:

```groovy
// ✅ CORRECT — credentials never appear in logs or source control
withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
    sh "sonar-scanner -Dsonar.login=${SONAR_TOKEN}"
}

// ✅ CORRECT — Docker registry login with masked credentials
withCredentials([usernamePassword(
    credentialsId: 'harbor-robot-credentials',
    usernameVariable: 'DOCKER_USER',
    passwordVariable: 'DOCKER_PASS'
)]) {
    sh 'echo "$DOCKER_PASS" | docker login registry.mycompany.com -u "$DOCKER_USER" --password-stdin'
}

// ❌ WRONG — token visible in logs, SCM history, and build artifacts
def sonarToken = 'squ_abc123secret'
sh "sonar-scanner -Dsonar.login=${sonarToken}"
```

### Sandbox vs. Trusted Libraries

Jenkins runs Pipelines in a **Groovy Sandbox** by default. Shared library code in `vars/` runs **in-process** and can be granted **trusted** status if the library is configured as "trusted" in Global Library settings.

- `src/` classes in a **trusted library** can call any Java/Groovy API.
- **Untrusted** (default) libraries are sandboxed — disallowed method calls require admin approval under "In-process Script Approval."

### Script Approval Gotchas

If you see `org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException`:

1. Go to `Manage Jenkins → In-process Script Approval` and approve the specific method signature.
2. Refactor the code to use an approved equivalent.
3. Mark the library as **trusted** if you fully control and own it.

{{< callout type="warning" title="Trusted Library Warning" >}}
Marking a library as **trusted** gives it the ability to run any Java/Groovy code with no restrictions. Only do this for libraries you fully own and have code-reviewed. A compromised trusted library can exfiltrate all Jenkins credentials.
{{< /callout >}}

---

## 🐛 Common Pitfalls & Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `NotSerializableException` | `src/` class not implementing `Serializable` | Add `implements Serializable` to every `src/` class |
| `RejectedAccessException` | Sandboxed method not approved | Approve in Script Approval or mark library as trusted |
| Step not found | File in wrong directory or wrong method name | Verify file is in `vars/`, has a `call()` method, and the filename matches the function name exactly |
| `libraryResource` returns null | Wrong path | Path is relative to `resources/` root — check spelling and forward slashes |
| Changes not picked up | Library cached by Jenkins | Use `@Library('lib@branch')` to force re-load, or clear the workspace |
| `ConcurrentModificationException` | Iterating and modifying a collection simultaneously | Use `.collect{}` to create a new list instead of mutating in-place |
| Infinite recursion on `call()` | `call()` accidentally calling itself | Check that your `call()` doesn't invoke the same step name recursively |
| Build hangs at quality gate | `waitForQualityGate()` waiting forever | Add a `timeout(time: 5, unit: 'MINUTES')` block around it |

{{< callout type="tip" title="Debug Tip" >}}
Add `echo "DEBUG: config = ${config.inspect()}"` at the top of any `vars/` step to dump all incoming parameters during a failing build. Remove it before merging.
{{< /callout >}}

---

## 📋 Quick Reference Cheat Sheet

```groovy
// ── IMPORT ──────────────────────────────────────────────────────────────────
@Library('company-shared-lib@v2.3.1') _                  // Pin to tag (production)
@Library('company-shared-lib@feature/new-step') _        // Use branch (development)
@Library(['lib-a@v1.0', 'lib-b@main']) _                 // Multiple libraries

// ── VARS/ STEP SKELETON ─────────────────────────────────────────────────────
// File: vars/myStep.groovy
def call(Map config = [:]) {
    def required = config.requiredParam ?: error('requiredParam is missing')
    def optional = config.get('optionalParam', 'defaultValue')
    stage("My Stage") {
        sh "echo running with ${optional}"
    }
}

// ── SRC/ CLASS SKELETON ─────────────────────────────────────────────────────
// File: src/com/mycompany/MyHelper.groovy
package com.mycompany
class MyHelper implements Serializable {        // Serializable is mandatory
    private def steps
    MyHelper(steps) { this.steps = steps }      // Accept Jenkins steps context
    void doSomething() { steps.sh "echo hello" }
}

// ── USE SRC/ CLASS IN VARS/ ─────────────────────────────────────────────────
import com.mycompany.MyHelper
def call() {
    def helper = new MyHelper(this)   // 'this' passes the Jenkins steps context
    helper.doSomething()
}

// ── LOAD RESOURCE FILE ───────────────────────────────────────────────────────
def template = libraryResource('com/mycompany/my-template.yaml')
def rendered = template.replace('TOKEN', 'actual-value')
writeFile file: 'output.yaml', text: rendered
sh 'kubectl apply -f output.yaml'

// ── CREDENTIALS ──────────────────────────────────────────────────────────────
withCredentials([usernamePassword(
    credentialsId: 'my-cred',
    usernameVariable: 'USER',
    passwordVariable: 'PASS'
)]) {
    sh 'docker login -u "$USER" -p "$PASS" registry.example.com'
}

// ── ENVIRONMENT-AWARE DEFAULTS ───────────────────────────────────────────────
private Map envDefaults(String env) {
    return [dev: [replicas: 1], staging: [replicas: 2], prod: [replicas: 4]]
        .get(env, [replicas: 1])
}

// ── POST / NOTIFICATIONS ─────────────────────────────────────────────────────
post {
    success  { sendSlackNotification(status: 'SUCCESS', channel: '#deploys') }
    failure  { sendSlackNotification(status: 'FAILURE', channel: '#alerts')  }
    always   { cleanWs() }
}

// ── QUALITY GATE WITH TIMEOUT ────────────────────────────────────────────────
timeout(time: 5, unit: 'MINUTES') {
    def qg = waitForQualityGate()
    if (qg.status != 'OK') { error "Quality Gate failed: ${qg.status}" }
}

// ── MANUAL APPROVAL GATE ─────────────────────────────────────────────────────
input message: 'Approve deployment to PRODUCTION?',
      submitter: 'release-managers',
      ok: 'Deploy'
```

---

*Written for Jenkins LTS 2.x+ and Pipeline Plugin 2.x+. All code examples are production-ready and follow Jenkins community conventions.*
