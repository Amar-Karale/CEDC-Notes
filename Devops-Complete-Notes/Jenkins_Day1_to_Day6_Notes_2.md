# Jenkins — Complete Notes (Day 1 to Day 6)

> Made from instructor's day-wise notes + your handwritten notes, cleaned up, corrected, and explained in simple language.

---

## 📑 Index — Click a Topic to Jump There

| Day | Topics |
|---|---|
| **[Day 1 — Introduction, CI/CD, Installation](#-day-1--introduction-to-jenkins-cicd-and-installation)** | [What is CI/CD?](#1-what-is-cicd) • [What is Jenkins?](#2-what-is-jenkins) • [Server Requirements (Agent)](#3-jenkins-server-requirements-agent) • [Installing Jenkins (LTS)](#4-installing-jenkins-long-term-support--lts-version) • [CI/CD Architecture Diagram](#5-the-cicd-architecture-big-picture-diagram) |
| **[Day 2 — Jenkins Jobs (Project Types)](#-day-2--jenkins-jobs-project-types)** | [What is a "Job"?](#what-is-a-job-in-jenkins) • [Types of Jobs](#types-of-jenkins-jobs) • [Freestyle Project Config](#freestyle-project--key-configuration-sections) • [Advanced Job Config: SCM/Triggers/Build Env/Steps](#advanced-job-configuration) • [Where Job Data is Stored](#where-job-data-is-stored-on-the-jenkins-server) |
| **[Day 3 — Jenkinsfile, Pipeline Syntax & Master-Agent](#-day-3--jenkinsfile-pipeline-syntax--master-agent-architecture)** | [Freestyle → Pipeline (Why)](#why-move-from-freestyle-to-pipeline) • [Basic Jenkinsfile Syntax](#basic-jenkinsfile-syntax-declarative-pipeline) • [Where to Write a Pipeline](#where-do-you-write-a-pipeline) • [Pipeline Build Settings](#options--build-settings-for-pipelines) • [Master–Agent Architecture](#masteragent-masterslave-architecture) • [Setting Up an Agent via SSH](#setting-up-a-jenkins-agent-via-ssh--step-by-step) |
| **[Day 4 — Credentials, Maven, Real Pipeline](#-day-4--credentials-maven-and-building-a-real-pipeline)** | [What is Maven?](#what-is-maven) • [What a Build Tool Does](#what-exactly-does-a-build-tool-do) • [Maven Build Lifecycle](#mavens-build-lifecycle-phases) • [Maven Build Stage in Pipeline](#adding-a-maven-build-stage-to-a-jenkins-pipeline) • [Jenkins Credentials Deep Dive](#jenkins-credentials--deep-explanation) |
| **[Day 5 — SonarQube: Code Quality Scanning](#-day-5--sonarqube-code-quality-scanning)** | [What is SonarQube?](#what-is-sonarqube) • [What SonarQube Detects](#what-can-sonarqube-detect) • [Installing SonarQube via Docker](#installing-sonarqube-via-docker) • [Connecting SonarQube to Jenkins](#connecting-sonarqube-to-jenkins) • [SonarQube Scan Stage in Pipeline](#adding-a-sonarqube-scan-stage-to-a-pipeline) |
| **[Day 6 — Trivy, Docker Integration, Full DevSecOps Pipeline](#-day-6--trivy-docker-integration--full-cicddevsecops-pipeline)** | [What is Trivy?](#what-is-trivy) • [Installing Trivy](#installing-trivy) • [Docker Installation](#docker-installation-needed-on-the-agent-for-buildingscanning-images) • [Trivy Plugins & Setup](#trivy-plugins--setup-in-jenkins) • [Full DevSecOps Pipeline Example](#full-devsecops-pipeline-example-build--scan--push--deploy) • [GitHub Webhook for Auto-Trigger](#github-webhook-for-auto-trigger) • [DevSecOps Task Checklist](#devsecops-task-checklist-practice-project-blueprint) |
| **Quick Reference** | [Declarative vs Scripted Pipeline](#-quick-reference--declarative-vs-scripted-pipeline) • [Important File Paths & Ports](#-quick-reference--important-file-paths--ports) |

---

# 📘 DAY 1 — Introduction to Jenkins, CI/CD, and Installation

## 1. What is CI/CD?

<img width="2050" height="480" alt="image" src="https://github.com/user-attachments/assets/250f916c-95b0-491b-93c8-efbf47e7f5d8" />

**CI/CD** stands for **Continuous Integration / Continuous Delivery / Continuous Deployment**. It is a *process* — a set of automated steps — that takes code from a developer's laptop all the way to a running application, without manual work, error steps in between.

Think of it like a factory assembly line for software:

The flow is like
``` 
Developer writes code → pushes to GitHub → server pulls the code
   → code is built (Maven/Gradle/npm) → an "artifact" is created
   → artifact is tested → artifact is stored (e.g. AWS S3)
   → artifact is deployed to a production server
```

### Breaking it into CI, CD (Delivery), and CD (Deployment)

| Stage | What happens | What it's called |
|---|---|---|
| Dev → GitHub → Server pulls code | Moving/transferring code from developer to server | **CI (Continuous Integration)** |
| Server → Build → Artifact → Test → Store (e.g. S3) | Compiling, testing, packaging the code | **Continuous Delivery** |
| Stored artifact → Deployed to Production server automatically | The final release step running automatically | **Continuous Deployment** |

**Simple definitions:**
- **CI (Continuous Integration):** Developers frequently push small code changes to a shared repo (like GitHub), and the code is automatically pulled, built, and tested. It catches integration bugs early instead of a giant "merge nightmare" at the end.
- **CD (Continuous Delivery):** After CI, the tested code is automatically packaged into a deployable "artifact" (like a `.jar`, `.war`, or Docker image) and stored somewhere (S3, Docker Hub, Nexus, etc.) ready to deploy — but a human may still click a button to release it.
- **CD (Continuous Deployment):** The stage after delivery — the artifact is automatically deployed and made live on the production server with **no manual approval** needed.

## 2. What is Jenkins?

Jenkins is an **automation server** used to implement CI/CD. It automates the repetitive process of pulling code, building it, testing it, and deploying it.

Key facts:
- Jenkins is **open source** (free to use).
- It was **originally called "Hudson"** — it is considered the "mother" (originator) of most modern CI/CD tools.
- Jenkins is written in **Java**, so a Java Runtime is required to run it.
- Jenkins provides a **GUI (browser-based dashboard)** to configure and monitor jobs — you don't have to write everything from a terminal.
- Jenkins is created in the **Java language**, but it can build projects in *any* language (Python, Node.js, Java, Go, etc.) because it just runs shell commands/scripts for you.

### Why is Jenkins needed?
Without Jenkins, someone would have to *manually*:
1. Pull the latest code
2. Run the build tool (Maven/Gradle/npm)
3. Run tests
4. Package it
5. Copy it to a server or S3
6. Restart the application

Jenkins automates all of this into a repeatable **pipeline**.

## 3. Jenkins Server Requirements ("Agent")

An **agent** is the machine (server) where your actual project/build runs.

- Minimum recommended: **4 GB RAM**, **20 GB storage**
- Jenkins runs by default on **port 8080**
- You should **update Jenkins roughly every 12 weeks**, because old versions get deprecated (lose security support and plugin compatibility).

## 4. Installing Jenkins (Long Term Support / LTS version)

These are the official Debian/Ubuntu installation steps (matches Jenkins' official docs — the version-signing key method):

```bash
sudo apt update

# 1. Add Jenkins' GPG key for package verification
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# 2. Add the Jenkins repository to your system's sources list
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  "https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# 3. Update package index again (to pick up the new repo)
sudo apt update

# 4. Install Java (Jenkins requires Java to run)
sudo apt install fontconfig openjdk-21-jre -y

# 5. Install Jenkins
sudo apt install jenkins -y
```

> 📝 **Note:** Your handwritten command had a small typo (`sudo do` and a wrong flag order) — the corrected, working version is above. Also note Jenkins today recommends **Java 17 or 21**, not older versions like Java 11.

### Starting and checking Jenkins

```bash
sudo systemctl enable jenkins   # start Jenkins automatically on boot
sudo systemctl start jenkins    # start Jenkins now
sudo systemctl status jenkins   # check if Jenkins is running
```

If the status shows **active (running)**, Jenkins started successfully.

### Opening Jenkins in the browser

Jenkins runs on **port 8080** by default. Open:

```
http://<your-server-public-ip>:8080
```

Example from your notes: `http://<public-ip>:8080`
++
> ⚠️ Make sure port `8080` is **open in your Security Group** (if on AWS EC2) or firewall, otherwise the page won't load.

### Unlocking Jenkins (first-time setup)

The first time you open Jenkins, it asks for an **Initial Admin Password**. Get it by running:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy that password, paste it into the browser page, and continue.

### Plugin Installation Screen

After unlocking, Jenkins gives you a choice:
- **Install suggested plugins** (recommended for beginners — installs the most commonly used plugins automatically)
- **Select plugins to install** (choose manually)

You can always install more plugins **later** too, from:
`Manage Jenkins → Plugins → Available Plugins`

### Create First Admin User

After plugins finish installing, Jenkins asks you to create your **own admin username and password** (instead of using the temporary initial password). Fill in:
- Username
- Password
- Full name
- Email

Then click **Save and Continue → Save and Finish → Start using Jenkins**.

## 5. The CI/CD Architecture (Big Picture Diagram)

```
 dev ──Git Pull──▶ [Jenkins] ──▶ Build (Maven/Gradle/Node.js)
                        │
                        ▼
                 SonarQube / Trivy (code quality + security scan)
                        │
                        ▼
                    Artifact created
                        │
                        ▼
              Stored in S3 / Docker Hub / ECR
                        │
                        ▼
             Deployed to Main/Production Server
                        │
                        ▼
                 Domain.com ──▶ Actor (End User)
                        ▲
                 release (manual trigger, optional)
```

- **CI** = Git Pull + Build stages (left side)
- **CD Delivery** = Build → Artifact → Store
- **CD Deployment** = Artifact → Main server → Domain (live to users)

---

# 📘 DAY 2 — Jenkins Jobs (Project Types)

## What is a "Job" in Jenkins?

A **Job** (also called a "Project") is a single task or pipeline configured in Jenkins — for example, "build my app," "run my tests," or "deploy my app." Every automated process you want Jenkins to run (build, copy files, run scripts) is created as a **Job**.

To create a job: `Jenkins Dashboard → New Item → Enter a name → Select job type → OK`

## Types of Jenkins Jobs

| # | Job Type | What it's used for |
|---|---|---|
| 1 | **Freestyle Project** | The simplest job type — a GUI-based job where you configure build steps (like "run this shell script") one by one via checkboxes/fields. Good for beginners and simple tasks. |
| 2 | **Pipeline** | A job defined as **code** (Groovy-based script called a `Jenkinsfile`), broken into "stages." This is the industry-standard way for real CI/CD. |
| 3 | **Multi-configuration project** | Used to run the *same* job across multiple configurations/environments (e.g., testing on multiple OS versions or Java versions) at once. |
| 4 | **Folder** | Not a "job" itself — it's a way to **organize** multiple jobs into groups, like a folder on your computer holds files. |
| 5 | **Multibranch Pipeline** | Automatically creates a pipeline job **per branch** in your Git repository — great when your team uses feature branches, since each branch gets its own automatic pipeline. |
| 6 | **Organization Folder** | Scans an entire GitHub organization/account and automatically creates jobs for every repository (and every branch) it finds. |

## Freestyle Project — Key Configuration Sections

When you create a Freestyle Project, the configuration page has these important sections:

### 1. Discard Old Builds
Controls how many old builds Jenkins keeps, so your disk doesn't fill up.
- **Days to keep builds:** e.g., keep builds for the last 30 days
- **Max # of builds to keep:** e.g., keep only the last 10 builds

### 2. This project is parameterized
Lets you add **input parameters** the user can choose before running a build — for example:
- **Choice Parameter** — a dropdown, e.g. a parameter called `action` with choices `apply` / `destroy` (useful for Terraform-style jobs).
- **String Parameter** — free text input.

### 3. Throttle builds
Limits how many builds of this job can run at the same time.

## Advanced Job Configuration

### Source Code Management (SCM)
Tells Jenkins **where your code lives** — most commonly **Git**. You provide the repo URL and branch (e.g. `main`).
- Option: **None** (if the job doesn't need to pull any code from Git — e.g. a simple script-only job)

### Triggers
Controls **when** the job should automatically run:

| Trigger | Meaning |
|---|---|
| **Trigger builds remotely** | Starts a build via a special URL/API call — e.g., from an external script. |
| **Build after other projects are built** | Runs this job only after another specified job finishes successfully (chaining jobs together). |
| **Build periodically** | Runs on a schedule, like a cron job (e.g., every night at 2 AM), regardless of whether code changed. |
| **GitHub hook trigger for GITScm polling** | Automatically triggers a build the moment new code is **pushed to GitHub** (via a "webhook"). This is the most common trigger for real CI/CD. |
| **Poll SCM** | Jenkins periodically **checks** the Git repo for changes (instead of waiting for a webhook push) and builds only if there's a change. |

> 💡 **GitHub Hook vs Poll SCM:** A webhook is *instant* (GitHub tells Jenkins "code changed!" the moment it happens). Poll SCM is *Jenkins asking* GitHub "did anything change?" every few minutes — less efficient, but useful when webhooks aren't possible (e.g., Jenkins server not reachable from the internet).

### Build Environment
| Option | Meaning |
|---|---|
| **Delete workspace before build starts** | Wipes the local working folder clean before every build — ensures no leftover files from a previous build cause issues. |
| **Use secret text(s) or file(s)** | Injects stored credentials (like API keys or passwords) as environment variables into the build, safely. |
| **Add timestamps to the console output** | Adds a timestamp next to every line of build log output — useful for debugging how long each step took. |

### Build Steps
This is where you add the actual **commands** Jenkins should run. The most common one:

**Execute Shell** — lets you write raw shell/bash commands.

Example (from your notes):
```bash
echo "hello world"
mkdir -p devops
```

### Post-build Actions
Runs actions **after** the build finishes — for example:
- Send an email if the build fails
- Archive build artifacts
- Trigger another job

## Where Job Data is Stored on the Jenkins Server

Every job you create is stored on disk under:

```
/var/lib/jenkins/workspace/<job-name>
```

This is the actual folder ("workspace") where your job's files, cloned repo, and build output live. You can `cd` into it and see everything Jenkins downloaded/created for that job.

---

# 📘 DAY 3 — Jenkinsfile, Pipeline Syntax & Master-Agent Architecture

## Why move from Freestyle to Pipeline?

Freestyle jobs are GUI-based and hard to version-control. **Pipelines** let you write your entire CI/CD process **as code** in a file called a **Jenkinsfile**, which you can commit to Git alongside your project — this is called **"Pipeline as Code."**

## Basic Jenkinsfile Syntax (Declarative Pipeline)

```groovy
pipeline {
    agent any                 // run on any available agent

    stages {
        stage('Hello') {
            steps {
                echo 'hello world'
                sh 'mkdir -p devops'
            }
        }
    }
}
```

### Breaking down every part:

| Keyword | Meaning |
|---|---|
| `pipeline { }` | The outer wrapper — everything in a Declarative Pipeline goes inside this block. |
| `agent` | Tells Jenkins **where** to run this pipeline. `agent any` = run on any available agent/node. You can also use `agent { label 'agent-name' }` to run on a specific labeled machine. |
| `stages { }` | A container that holds one or more `stage` blocks — represents the major phases of your pipeline (e.g. Build, Test, Deploy). |
| `stage('name') { }` | A single named phase/step-group in your pipeline. Shows up as a separate box in the Jenkins "Pipeline Stage View." |
| `steps { }` | The actual list of commands/actions to run inside a stage. |
| `echo` | Prints text to the console log (like `print()`). |
| `sh` | Runs a shell/bash command on Linux agents. |

## Where do you write a Pipeline?

Two ways:
1. **Pipeline script (inline)** — write the Groovy code directly in the Jenkins job's configuration page, in a text box.
2. **Pipeline script from SCM (recommended)** — store a file named `Jenkinsfile` in your Git repo root, and tell Jenkins to fetch and run *that* file. This way your pipeline is version-controlled with your code.

## Options / Build Settings for Pipelines

| Option | Meaning |
|---|---|
| **Discard old builds** | Same as Freestyle — controls how many old build records/logs Jenkins keeps. |
| **Delete build** (specific build #) | Manually delete one particular build's history/logs. |
| **Replay** | Re-run a *previous* pipeline run's exact script — useful for testing small script edits without committing to Git. |
| **Pipeline Syntax / Overview / Console Output** | Built-in tools inside Jenkins to help you *generate* correct pipeline syntax and view logs of a run. |

## Master–Agent (Master–Slave) Architecture

Jenkins can run on a single machine, but in real production, work is distributed:

- **Master (Controller):** The main Jenkins server. It hosts the **web dashboard**, schedules jobs, and distributes the actual build work to agents. It does **not** run the heavy build work itself.
- **Agent (Slave/Node):** Separate machine(s) connected to the Jenkins master. Agents **actually run** the jobs/builds — this is where your project's real work (compiling, testing, deploying) happens.

**Why use agents instead of running everything on master?**
- Distributes load across multiple machines (scaling)
- Lets you run builds on different operating systems (e.g., Windows agent + Linux agent)
- Keeps the master server light and stable (crashing a build shouldn't crash your whole Jenkins dashboard)

### Setting Up a Jenkins Agent (via SSH) — Step by Step

This is the most common, professional way to connect a Linux agent to a Jenkins master.

#### Step 1 — Prepare the Agent Machine

Install Java on the **agent** (agent needs Java to run Jenkins' remoting client):

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
java -version
```

> ⚠️ Master and agent should ideally use the **same Java version** to avoid compatibility issues.

#### Step 2 — Generate an SSH Key Pair (on the Master)

```bash
cd ~/.ssh
ssh-keygen        # creates a new key pair
```

This creates:
- A **private key** — stays on the **master**
- A **public key** — copied to the **agent**

```bash
cat id_rsa.pub    # copy this — it's the public key
```

Copy the public key onto the agent machine's `~/.ssh/authorized_keys` file, so the master can log in via SSH without a password.

```bash
cat id_rsa        # copy this — it's the private key, needed in Step 3
```

#### Step 3 — Add SSH Credentials in Jenkins

`Manage Jenkins → Credentials → System → Global credentials → Add Credentials`

| Field | Value |
|---|---|
| **Kind** | SSH Username with private key |
| **ID** | A memorable ID, e.g. `jenkins-agent-key` |
| **Description** | Something explaining what this credential is for |
| **Username** | The Linux username on the agent machine, e.g. `ubuntu` |
| **Private Key** | Select "Enter directly" → paste the **private key** you copied in Step 2 |
| **Passphrase** | Leave blank if the key has none |

Click **Create**.

#### Step 4 — Create the Agent Node in Jenkins

`Manage Jenkins → Nodes → New Node`

1. Give it a **name** (e.g., `agent-1`)
2. Select **Permanent Agent**
3. Configure:

| Field | Value / Meaning |
|---|---|
| **Remote root directory** | Where Jenkins will store its files on the agent, e.g. `/home/ubuntu` |
| **Labels** | A tag/name used to refer to this agent in pipelines (e.g. `agent`), e.g. `agent { label 'agent' }` in a Jenkinsfile |
| **Usage** | "Use this node as much as possible" (default recommended) |
| **Launch method** | Select **"Launch agents via SSH"** |
| **Host** | The IP address or domain name of the agent machine |
| **Credentials** | Select the SSH credential you created in Step 3 |
| **Host Key Verification Strategy** | For beginners: **Non-verifying Verification Strategy** (in production, use "Known hosts file" for better security) |
| **Number of executors** | How many builds/jobs this agent can run at the same time (default is 1) |

4. Click **Save**.

If everything is correct, the agent shows as **connected** (green) in `Manage Jenkins → Nodes`.

---

# 📘 DAY 4 — Credentials, Maven, and Building a Real Pipeline

## What is Maven?

**Maven** is a **build automation tool**, primarily used for **Java** projects.

It handles:
- Compiling source code
- Managing dependencies (external libraries your project needs)
- Running tests
- Packaging the final application into a distributable file

> Maven checks all the dependencies the project needs from a file called **`pom.xml`** — this is Maven's configuration file, similar to how Node.js projects use `package.json` or Python projects use `requirements.txt`.

Maven produces a **deployable artifact** — commonly a `.jar` (Java Archive) or `.war` (Web Application Archive) file.

Other build tools exist too, e.g. **Gradle**, **Ant** (for Java), or `npm`/`yarn` (for Node.js).

## What Exactly Does a Build Tool Do?

A build tool automates the process of converting **raw source code** into an **executable, deployable application**, by running tasks in a fixed, correct order (compile → test → package → deploy).

## Maven's Build Lifecycle (Phases)

Maven runs a fixed sequence of **phases**. Running a later phase automatically runs all the earlier ones too.

| Phase | What it does |
|---|---|
| **validate** | Checks the project structure is correct and all necessary info is available. |
| **compile** | Compiles the source code. |
| **test** | Runs unit tests on the compiled code (doesn't require packaging). |
| **package** | Packages the compiled code into a distributable file, like a `.jar` or `.war`. |
| **integration-test** | Deploys the package into an environment to run integration tests. |
| **verify** | Runs checks to confirm the package meets quality standards. |
| **install** | Installs the package into your **local** repository, so other local projects can use it as a dependency. |
| **deploy** | Copies the final package to a **remote** repository, sharing it with other developers/projects. |

There are two other important lifecycles:
- **clean** — deletes artifacts/files created by previous builds.
- **site** — generates project documentation.

> 💡 Behind the scenes, phases map to Maven "goals" — e.g., the `package` phase runs `jar:jar` if your project is a JAR, or `war:war` if it's a WAR.

## Adding a Maven Build Stage to a Jenkins Pipeline

```groovy
pipeline {
    agent { label 'project' }

    tools {
        maven 'maven'          // must match the Maven tool name configured in Jenkins
    }

    environment {
        S3_BUCKET = "project-insure-me-build-artifacts-store"
        REGION    = "ap-south-1"
        ARTIFACT  = "target/Insurance-0.0.1-SNAPSHOT.jar"
    }

    stages {

        stage('Code Pull') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/mukundDeo9325/Project-InsureMe1.git']]
                )
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([
                    aws(
                        credentialsId: 'aws-cred',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        aws s3 cp ${ARTIFACT} s3://${S3_BUCKET}/Artifacts/ --region ${REGION}
                    '''
                }
            }
        }
    }
}
```

### Explaining every new part:

| Line | Meaning |
|---|---|
| `tools { maven 'maven' }` | Tells Jenkins to use a Maven installation you've configured under `Manage Jenkins → Tools`, named `maven`. Jenkins auto-adds it to the `PATH` for this pipeline. |
| `environment { }` | Defines reusable variables (like `S3_BUCKET`) available throughout the pipeline. |
| `checkout scmGit(...)` | A more explicit/modern way (compared to the plain `git` step) to pull code from a specific branch and repo URL. |
| `sh 'mvn clean package'` | Runs Maven: `clean` removes old build files, `package` compiles + tests + packages into a `.jar`. |
| `withCredentials([aws(...)])` | Safely injects AWS access keys (stored in Jenkins Credentials) as environment variables **only within this block**, so secrets aren't hardcoded or exposed in logs. |
| `aws s3 cp ... s3://...` | The AWS CLI command that uploads (copies) your built artifact to an S3 bucket. |

## Jenkins Credentials — Deep Explanation

Credentials let Jenkins securely store secrets (passwords, keys, tokens) instead of hardcoding them into scripts.

`Manage Jenkins → Credentials → System → Global credentials → Add Credentials`

| Field | Meaning |
|---|---|
| **Kind** | The *type* of credential — e.g. "SSH Username with private key," "Username with password," "Secret text," "AWS Credentials." |
| **Scope** | `Global` = usable by any job. `System` = only usable by Jenkins internals. |
| **ID** | A unique short name used to **reference** this credential inside your Jenkinsfile (e.g. `aws-cred`). |
| **Description** | A human-readable note about what this credential is for. |

For **AWS credentials** specifically, Jenkins asks for:
- Access Key ID
- Secret Access Key

These get referenced in a pipeline using `credentialsId: 'aws-cred'` as shown above.

---

# 📘 DAY 5 — SonarQube: Code Quality Scanning

## What is SonarQube?

**SonarQube** is an **automated code-quality and security-analysis tool**. It's **open source**, developed using **Java**, and supports **~30 programming languages** through plugins.

Think of SonarQube as a **digital code reviewer** — it scans your source code and reports problems *before* they reach production.

## What Can SonarQube Detect?

| # | Category | Meaning |
|---|---|---|
| 1 | **Bugs** | Actual coding mistakes likely to cause incorrect behavior. |
| 2 | **Vulnerabilities** | Security weaknesses that could be exploited by attackers. |
| 3 | **Code Smells** | Code that "works" but is poorly written/repeated, making it hard to maintain. |
| 4 | **Code Duplication** | Identical or near-identical code blocks repeated across the project. |
| 5 | **Security Hotspots** | Sensitive areas of code that *need a human review* — not confirmed vulnerabilities yet, but risky patterns. |
| 6 | **Code Coverage** | The percentage of your code that is actually executed by automated tests. |
| 7 | **Integration with CI/CD** | SonarQube plugs directly into pipelines (Jenkins, Azure DevOps, Bamboo, etc.) to run automatically on every build. |
| 8 | **Plugin Ecosystem** | Additional plugins extend SonarQube to support more languages and tools. |

## Installing SonarQube (via Docker)

First, install Docker on the machine where SonarQube will run (commonly the **Jenkins agent**):

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker jenkins   # let the "jenkins" user run docker commands
sudo usermod -aG docker ubuntu    # let the "ubuntu" user run docker commands
newgrp docker                     # refresh group membership in current shell
sudo chmod 777 /var/run/docker.sock
```

Then run SonarQube as a Docker container:

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

**Breaking this down:**
- `-d` → run in detached mode (in the background)
- `--name sonar` → names the container "sonar"
- `-p 9000:9000` → maps port 9000 on your host to port 9000 inside the container (SonarQube's web UI default port)
- `sonarqube:lts-community` → uses the official, free "Community" long-term-support image

Access it at:
```
http://<server-ip>:9000
```

**Default login:** Username `admin`, Password `admin` (you'll be asked to change it on first login).

## Connecting SonarQube to Jenkins

### 1. Install Plugins in Jenkins
`Manage Jenkins → Plugins → Available` — install:
- **SonarQube Scanner** plugin

### 2. Generate a SonarQube Token
Inside SonarQube: `My Account → Security → Generate Token`
- Give it a **name**
- Choose a **type** (Global Analysis Token, or Project Analysis Token)
- Set an **expiry** (e.g., 30 days)
- Click **Generate**, then **copy** the token immediately (you can't view it again later!)

### 3. Add SonarQube Token as a Jenkins Credential
`Manage Jenkins → Credentials → Add Credentials`
- **Kind:** Secret text
- **Secret:** paste the token
- **Description:** something recognizable, e.g. "sonar-cred"

### 4. Configure the SonarQube Server in Jenkins
`Manage Jenkins → System → SonarQube servers`
- **Name:** e.g. `sonar-server` (this exact name is referenced in your Jenkinsfile)
- **Server URL:** e.g. `http://<sonarqube-ip>:9000`
- **Server authentication token:** select the credential you added above

### 5. Configure the SonarQube Scanner Tool
`Manage Jenkins → Tools → SonarQube Scanner installations`
- **Name:** e.g. `sonar-scanner` (matches the `tool 'sonar-scanner'` reference in the Jenkinsfile)
- Let Jenkins auto-install it, or point to an existing installation

### 6. Webhooks (optional but recommended)
Inside SonarQube: `Administration → Configuration → Webhooks`
Add a webhook pointing back to Jenkins, so SonarQube can **notify Jenkins the instant analysis finishes** (needed for the Quality Gate step below to work efficiently, instead of Jenkins polling).

## Adding a SonarQube Scan Stage to a Pipeline

```groovy
pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('code-pull') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/mukundDeo9325/Project-InsureMe1.git']])
            }
        }
        stage('code-build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage("code-test") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=InsureMe \
                        -Dsonar.projectName=InsureMe \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }
        stage("code-test-quality-gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
                }
            }
        }
    }
}
```

### Explaining the new parts:

| Line | Meaning |
|---|---|
| `SCANNER_HOME = tool 'sonar-scanner'` | Fetches the path to the SonarQube Scanner tool you configured, and stores it in a variable. |
| `withSonarQubeEnv('sonar-server')` | Wraps the scan command with the SonarQube server connection details (URL + token) you configured under that server name. |
| `-Dsonar.projectKey` | A unique identifier for this project inside SonarQube. |
| `-Dsonar.projectName` | The display name shown in the SonarQube dashboard. |
| `-Dsonar.sources=src` | The folder containing your source code to scan. |
| `-Dsonar.java.binaries` | Path to the compiled `.class` files (SonarQube needs these for deeper Java analysis). |
| `waitForQualityGate` | Pauses the pipeline and waits for SonarQube to finish analyzing and return a **Pass/Fail "Quality Gate"** result. |
| `abortPipeline: false` | If the quality gate fails, the pipeline **continues anyway** instead of stopping (set to `true` in stricter setups to block bad code from deploying). |

---

# 📘 DAY 6 — Trivy, Docker Integration & Full CI/CD/DevSecOps Pipeline

## What is Trivy?

**Trivy** is an open-source **vulnerability scanner**, mainly used to find security errors/vulnerabilities inside **Docker images** (it can also scan filesystems, Git repos, and Infrastructure-as-Code files).

## Installing Trivy

```bash
sudo apt-get install wget gnupg -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

**What each step does:**
1. Install `wget` and `gnupg` (tools needed to download and verify the signing key)
2. Download Trivy's public GPG key and convert it to Jenkins-readable binary format (`gpg --dearmor`)
3. Add Trivy's official package repository to your system's sources list, tied to that signing key (so `apt` trusts it)
4. Refresh the package list
5. Install Trivy

## Docker Installation (needed on the agent, for building/scanning images)

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

> 📝 Adding `jenkins` (and your login user) to the `docker` group lets Jenkins run `docker` commands **without needing `sudo`** every time — otherwise your pipeline's `sh` steps will fail with permission errors.

## Trivy Plugins & Setup in Jenkins

**Plugins to install:** Docker, AWS Credentials, Maven Integration, Stage View

**Credentials needed:**
- **Docker credentials** — your Docker Hub username & password (kind: "Username with password")
- **AWS credentials** — Access Key + Secret Key (for uploading scan reports to S3, if desired)
- **SonarQube credentials** — from Day 5

**Tools needed:** Maven, Docker (downloaded from Docker's own site/repo, not from a generic apt mirror, to guarantee you get the latest stable version)

### S3 Bucket for Trivy Reports (optional)
You can configure a dedicated S3 bucket to store your Trivy scan reports:
- Bucket name → change it to your own unique bucket name in the script
- **Region** → must match your AWS credentials' region

## Full DevSecOps Pipeline Example (Build → Scan → Push → Deploy)

```groovy
pipeline {
    agent { label 'agent' }
    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME  = tool 'sonar-scanner'
        S3_BUCKET     = "newbukkforjenkinstest"
        REGION        = "ap-northeast-1"
        warFile       = "target/Insurance-0.0.1-SNAPSHOT.jar"
        DOCKER_IMAGE  = "mukunddeo9325/insureme"
    }
    stages {
        stage('code-pull') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/mukundDeo9325/Project-InsureMe1.git']])
            }
        }
        stage('code-build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage("code-test") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=InsureMe \
                        -Dsonar.projectName=InsureMe \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }
        stage("code-test-quality-gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
                }
            }
        }
        stage('code-push') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'aws s3 cp ${warFile} s3://${S3_BUCKET}/Artifacts/ --region ${REGION}'
                }
            }
        }
        stage('docker-image') {
            steps {
                sh 'docker build -t mukunddeo9325/insureme .'
            }
        }
        stage('image-push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred',
                    passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh 'docker push mukunddeo9325/insureme'
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image ${DOCKER_IMAGE}:latest > trivy-report.txt
                '''
            }
        }
        stage('code-deploy') {
            steps {
                sh 'docker run -itd --name insure-me -p 8089:8081 mukunddeo9325/insureme'
            }
        }
    }
}
```

### New parts explained:

| Line | Meaning |
|---|---|
| `docker build -t <image-name> .` | Builds a Docker image from the `Dockerfile` in the current directory (`.`), tagging it with a name. |
| `usernamePassword(credentialsId: ...)` | A credential type that injects a **username and password pair** as two separate environment variables. |
| `docker login -u ... -p ...` | Logs in to Docker Hub using injected credentials (so `docker push` is authorized). |
| `docker push <image>` | Uploads the built image to Docker Hub. |
| `trivy image <image>:latest > trivy-report.txt` | Scans the Docker image for vulnerabilities and saves the results into a text file. |
| `docker run -itd --name ... -p 8089:8081 <image>` | Runs the pushed image as a live container: `-i` interactive, `-t` allocate terminal, `-d` detached (background), `-p 8089:8081` maps host port 8089 → container port 8081. |

# 🔗 Jenkins Webhooks — Deep Dive (Simple Words)

> Extra notes to go alongside Day 6 (GitHub Webhook for Auto-Trigger).

---

## 1. The Core Idea (One Line)

A **webhook** is GitHub **calling Jenkins on the phone** the moment something happens (like a push) — instead of Jenkins repeatedly **calling GitHub to ask "anything new? anything new?"**

---

## 2. Without a Webhook: Polling (the "Annoying" Way)

Before webhooks, Jenkins had to keep checking GitHub itself:

```
Jenkins: "Any new commits?" → GitHub: "No"
(wait 2 minutes)
Jenkins: "Any new commits?" → GitHub: "No"
(wait 2 minutes)
Jenkins: "Any new commits?" → GitHub: "Yes! 3 new commits"
```

This is called **polling** (`Poll SCM` in Jenkins). Problems with it:

| Problem | Why it hurts |
|---|---|
| **Wastes resources** | Jenkins asks even when nothing changed |
| **Delay** | Push right after Jenkins just checked? You wait for the *next* check |
| **Doesn't scale** | 100 jobs all polling every 2 minutes = a lot of wasted requests |

---

## 3. With a Webhook: Push Notification (the Smart Way)

Now flip the direction. **GitHub tells Jenkins**, instantly, the moment a push happens:

```
Dev pushes code → GitHub instantly sends an HTTP request to Jenkins
                → Jenkins receives it → Jenkins starts the build immediately
```

No waiting, no wasted checks.

> 📱 **Analogy:** This is exactly like a phone notification. WhatsApp's server doesn't make your phone ask "any new messages?" every 2 minutes — WhatsApp **pushes** the notification to you the instant a message arrives. GitHub does the same thing to Jenkins.

---

## 4. What Is a Webhook, Technically?

It's just an **HTTP POST request** that one system automatically sends to another system's URL, whenever a specific event happens.

| Part | What it means here |
|---|---|
| **Event** | A `git push` to your repo |
| **URL Jenkins gives GitHub** | The "phone number" GitHub calls |
| **Payload** | The data GitHub sends — which repo, which branch, which commits |

---

## 5. How It's Wired Up (Step by Step)

### Step 1 — Jenkins side: tell the job to listen

In your Jenkins job → **Build Triggers** → check:

```
✅ GitHub hook trigger for GITScm polling
```

This means: *"Don't poll on a timer — wait for GitHub to ping me instead."*

### Step 2 — GitHub side: tell GitHub where to send the ping

In your GitHub repo → **Settings → Webhooks → Add webhook**:

| Field | Value |
|---|---|
| **Payload URL** | `http://<jenkins-server-ip>:8080/github-webhook/` |
| **Content type** | `application/json` |
| **Which events** | "Just the push event" (or as needed) |

- **Payload URL** — a special built-in Jenkins endpoint that exists purely to receive GitHub's pings.
- **Content type** — the format of the data GitHub will send.
- **Which events** — you're telling GitHub: only ping me on a push, not on every issue comment or star.

### Step 3 — The actual flow when you push code

```
You: git push
      ↓
GitHub detects the push
      ↓
GitHub sends POST request → http://<jenkins-ip>:8080/github-webhook/
      ↓
Jenkins receives it, checks: "which job has GitHub-hook-trigger
                               enabled for this repo?"
      ↓
Jenkins finds the matching job → triggers a build automatically
```

---

## 6. Simple Analogy to Lock It In

Think of a **restaurant buzzer** they give you while waiting for a table:

- **Polling** = you walking up to the host every 5 minutes asking "is my table ready?"
- **Webhook** = the buzzer vibrating in your hand the instant your table is ready — you don't lift a finger, it comes to you

---

## 7. Important Gotcha: Public Reachability

For GitHub to reach `http://<jenkins-ip>:8080/github-webhook/`, your Jenkins server's port **8080 must be publicly reachable** — open in your AWS Security Group / firewall.

If Jenkins sits behind a private network with no public IP, GitHub's servers can never deliver the ping. In that case you'd need something like:
- **`ngrok`** (for local testing / temporary public tunnel)
- A **reverse proxy** / load balancer that exposes Jenkins publicly

---

## 8. Quick Comparison Table

| | Polling | Webhook |
|---|---|---|
| **Who initiates** | Jenkins asks GitHub | GitHub tells Jenkins |
| **Speed** | Delayed (based on poll interval) | Instant |
| **Resource use** | Wasteful (constant checking) | Efficient (only fires on real events) |
| **Setup** | Just a checkbox in Jenkins | Checkbox in Jenkins + webhook config in GitHub |
| **Best for** | Jenkins server not reachable from internet | Standard production CI/CD setups |

---

## 9. Related: SonarQube Webhooks (bonus, ties into Day 5)

The same push-notification concept is used elsewhere in your pipeline. In Day 5, SonarQube can also be configured with a webhook (`Administration → Configuration → Webhooks`) pointing back to Jenkins — so SonarQube tells Jenkins the instant analysis finishes, instead of Jenkins polling for the Quality Gate result. Same idea, different tool.

## DevSecOps Task Checklist (Practice Project Blueprint)

A good practice task: build a **CI/CD pipeline with a specific failure stage**, so that if any *one* stage fails, only **that stage** shows as failed (not the whole pipeline silently dying) — this is one of the key benefits of **Declarative Pipelines** over Scripted ones (see comparison table in Day 3).

**Example flow for a Django app project:**

1. **Name** → e.g. `Djangoapp`
2. **Description** → e.g. "This is a Django project CI/CD pipeline"
3. **GitHub hook trigger** → for auto-triggering on push
4. **Pipeline → Definition** → "Pipeline script from SCM"
5. **SCM** → Git → Repository URL (your GitHub repo)
6. **Script Path** → `Jenkinsfile` (the file Jenkins should pull and run)

**Basic Jenkinsfile skeleton for this:**

```groovy
pipeline {
    agent { label 'spy' }

    stages {
        stage('code') {
            steps {
                echo 'pulling code'
            }
        }
    }
}
```

Complete workflow for a containerized deployment task:

```
git clone (repo) → docker build → docker push (Docker Hub) → deploy on EC2
```

```bash
# On the target EC2/deployment server:
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
git clone <repo-url>
cd <repo-folder>
docker build -t <your-dockerhub-username>/<image-name> .
docker push <your-dockerhub-username>/<image-name>
```

---

# 🔑 Quick Reference — Declarative vs Scripted Pipeline

| Feature | Scripted Pipeline | Declarative Pipeline |
|---|---|---|
| Syntax | Groovy-based, full flexibility | Structured, predefined schema |
| Starting block | `node { }` | `pipeline { }` |
| Learning curve | Steeper, more complex | Easier, beginner-friendly |
| Error handling | `try / catch / finally` | `post { }` block |
| Conditionals | Full Groovy `if/else` | `when { }` directive |
| Validation | Only found at runtime | Validated before running |
| Failure behavior | If one step fails, entire script fails | If one **stage** fails, only that stage fails — other stages/logic can still be evaluated |
| Best for | Complex/dynamic custom logic | Standard, repeatable CI/CD pipelines (recommended default) |

> ✅ **Recommendation (industry standard):** Use **Declarative Pipeline** unless you specifically need advanced dynamic Groovy logic.

# 🔑 Quick Reference — Important File Paths & Ports

| Item | Value |
|---|---|
| Initial admin password | `/var/lib/jenkins/secrets/initialAdminPassword` |
| Jenkins default port | `8080` |
| Jenkins job workspace | `/var/lib/jenkins/workspace/<job-name>` |
| SonarQube default port | `9000` |
| Docker socket permission fix | `sudo chmod 777 /var/run/docker.sock` |
