# gitlab


# 🌟 **GITLAB CI/CD CRASH COURSE – FULL EASY NOTES (NO IMPORTANT PART MISSED!)**

## 🚀 **1. What You Will Learn**

* Build a **full CI/CD pipeline** in GitLab
* Pipeline does:
  ✅ Run **tests**
  ✅ Build **Docker image**
  ✅ Push to **Docker Hub private repo**
  ✅ Deploy to **Ubuntu server**
* Learn core GitLab CI/CD components:
  ⭐ Jobs
  ⭐ Stages
  ⭐ Runners
  ⭐ Variables
  ⭐ Docker-in-Docker

---

# 🧠 **2. What is GitLab CI/CD?**

### 💡 Simple Definition

➡️ **CI/CD = Continuous Integration + Continuous Deployment**
➡️ “Automatically test, build & release code whenever developers push changes.”

### 🔥 Why GitLab CI/CD?

* Your code is already in GitLab 🟣 → Easy to extend workflows
* No need to install Jenkins/extra tools
* GitLab provides **managed runners** → You can run pipelines instantly
* Pipeline is written as **code** = `.gitlab-ci.yml`

---

# 🏗️ **3. GitLab CI/CD Architecture**

![Image](https://docs.gitlab.com/development/cicd/img/ci_architecture_v13_0.png?utm_source=chatgpt.com)

![Image](https://developer.ibm.com/developer/default/tutorials/build-multi-architecture-x86-and-power-container-images-using-gitlab/images/fig1.png?utm_source=chatgpt.com)

![Image](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2022/12/15/devops-2115_1.png?utm_source=chatgpt.com)

![Image](https://docs.gitlab.co.jp/ee/architecture/blueprints/runner_scaling/gitlab-autoscaling-overview.png?utm_source=chatgpt.com)

### 🟣 **GitLab Server**

* Stores your code
* Stores your pipeline YAML
* Decides what jobs must run

### 🔵 **GitLab Runners**

* Actual machine that EXECUTES your pipeline
* Can be:

  * Managed by GitLab (default, free)
  * Self-managed (your company server)

### ⚙️ Default Execution

* GitLab managed runner runs each job **inside a Docker container**
* Default image = **Ruby**
* You can override using `image:` keyword

---

# 🐍 **4. Demo App Overview**

* Python web application
* Has **tests** in `/app/tests`
* Uses **Makefile** commands:

  * `make test` → run tests
  * `make run` → run app locally
* App runs on **port 5000**

---

# 🛠️ **5. Running the App Locally**

### ✔ Run tests:

```
make test
```

→ Installs dependencies (requirements.txt)
→ Runs pytest
→ 4 tests passed

### ✔ Run the app:

```
PORT=5004 make run
```

→ Open browser: `localhost:5004`

---

# 🧾 **6. Create CI/CD Pipeline**

### File needed:

📄 **`.gitlab-ci.yml`**

### Pipeline has **jobs**

Example:

```yaml
run_tests:
  script:
    - make test
```

---

# ⚡ **7. First Job: Run Tests**

### Problem:

`make test` requires:

* Python
* Pip
* Make

### Default runner image = Ruby ❌

So we override with Python image ✔

```yaml
run_tests:
  image: python:3.9
  before_script:
    - apt-get update
    - apt-get install make
  script:
    - make test
```

### What happens?

* Runner launches container with Python 3.9
* Installs make
* Runs tests

### Pipeline automatically starts when you commit YAML.

---

# 🔍 **8. Understanding Job Logs**

* Shows container started with Python image
* Shows repo clone
* Shows before_script
* Shows tests passed

---

# 🐳 **9. Build + Push Docker Image Job**

![Image](https://assets.bytebytego.com/diagrams/0414-how-does-docker-work.png?utm_source=chatgpt.com)

![Image](https://docs.docker.com/get-started/images/docker-architecture.webp?utm_source=chatgpt.com)

![Image](https://www.fosstechnix.com/wp-content/uploads/2023/08/Push-Docker-Image-to-GitLab-Container-Registry-e1691645222548.png?utm_source=chatgpt.com)

![Image](https://stanislas.blog/2018/09/build-push-docker-images-gitlab-ci/docker-build-push-gitlab-ci.png?utm_source=chatgpt.com)

### Steps:

1. Build Docker image
2. Login to Docker Hub
3. Push image to **private repo**

### Store sensitive info in GitLab →

🟩 **CI/CD → Settings → Variables**
Create:

* `REGISTRY_USER`
* `REGISTRY_PASS`

### Build Job:

```yaml
build_image:
  image: docker:20.10
  services:
    - docker:20.10-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $REGISTRY_USER -p $REGISTRY_PASS
  script:
    - docker build -t $IMAGE_NAME:$IMAGE_TAG .
    - docker push $IMAGE_NAME:$IMAGE_TAG
```

### Why Docker-in-Docker?

* Need docker client + docker daemon
* GitLab starts 2 containers:

  * job container → docker client
  * service container → docker daemon

---

# 🪜 **10. Control Job Order Using Stages**

By default jobs run in parallel ❌
But we want:
1️⃣ Test
2️⃣ Build
3️⃣ Deploy

### Define Stages:

```yaml
stages:
  - test
  - build
  - deploy
```

Then assign:

```yaml
run_tests:
  stage: test

build_image:
  stage: build
```

---

# 🌐 **11. Deploy Stage Setup**

### Requirements:

* A remote **Ubuntu server**
* Install **Docker** on server
* GitLab can SSH into server

### Platform used in demo:

💙 **DigitalOcean Droplet**

### Steps:

1. Create SSH key
2. Upload public key to DigitalOcean
3. Create droplet
4. SSH into server
5. Install Docker
6. Add SSH private key as CI/CD variable (file type)

Variable name:

* `SSH_KEY`

---

# 🚚 **12. Deploy Job**

Goal:

* SSH into server
* Docker login
* Stop old container
* Remove old container
* Pull new image
* Run new container

### Deploy Job:

```yaml
deploy:
  stage: deploy
  before_script:
    - chmod 400 $SSH_KEY
  script:
    - ssh -i $SSH_KEY -o StrictHostKeyChecking=no root@SERVER_IP "
        docker login -u $REGISTRY_USER -p $REGISTRY_PASS &&
        docker ps -aq | xargs docker stop || true &&
        docker ps -aq | xargs docker rm || true &&
        docker pull $IMAGE_NAME:$IMAGE_TAG &&
        docker run -d -p 5000:5000 $IMAGE_NAME:$IMAGE_TAG
      "
```

---

# 🧪 **13. Final Pipeline Flow**

```
🟣 Stage 1 → run_tests
🟡 Stage 2 → build_image
🟢 Stage 3 → deploy
```

All stages run **in sequence**.

---

# 🎉 **14. Validate Deployment**

### On server:

```
docker ps
```

→ Should show your Python app running

### In browser:

Open:

```
http://<server-ip>:5000
```

---

# 💡 **15. Things You MUST Remember for Interview**

### 🟩 GitLab Key Terms

* **Job** = single task
* **Stage** = group of jobs
* **Runner** = machine that runs jobs
* **Pipeline** = collection of stages
* **Variables** = pass secrets or config
* **Docker-in-Docker** = required for building docker images

### 🟧 Why GitLab CI/CD is good?

* No server setup
* Integrated with GitLab repo
* Easy pipelines
* Managed runners

### 🟥 Why we used:

* `image:` → select container base
* `before_script:` → setup environment
* `services:` → start additional containers
* `stages:` → job order
* CI/CD variables → hide secrets
* SSH key → connect for deployment

---

GitLab CI/CD 🟣
├─ Concepts 🔑
│ ├─ Jobs 🧩
│ │ ├─ script: commands to run
│ │ ├─ before_script: prepare env
│ │ └─ image: docker image used
│ ├─ Stages 🪜
│ │ ├─ test
│ │ ├─ build
│ │ └─ deploy
│ ├─ Runners 🏃‍♂️
│ │ ├─ managed (gitlab.com)
│ │ └─ self-managed
│ ├─ Variables 🔐
│ │ ├─ project settings → secret variables
│ │ └─ file type (for ssh key)
│ ├─ Images & Executors 🐳
│ │ ├─ default image (ruby) — override with python/node/etc.
│ │ └─ docker executor (containers)
│ ├─ Services (e.g., dind) ⚙️
│ └─ Artifacts / Cache 📦
|
├─ Pipeline Flow ▶️
│ ├─ Stage: test ✅ → run unit tests (make test)
│ ├─ Stage: build ✅ → build docker image + push to registry
│ └─ Stage: deploy ✅→ ssh to server, pull image, run container
|
├─ Demo App (Python) 🐍
│ ├─ tests: app/tests (pytest via make test)
│ ├─ Dockerfile: builds python image
│ └─ Makefile: helpers (test, run)
|
├─ Build & Push (Docker-in-Docker) 🐳🐳
│ ├─ image: docker:20.10
│ ├─ services:
│ │ └─ docker:20.10-dind
│ ├─ variable: DOCKER_TLS_CERTDIR=/certs
│ └─ before_script: docker login -u $REGISTRY_USER -p $REGISTRY_PASS
|
├─ Deploy (SSH → Droplet) 🔐➡️🖥️
│ ├─ Create droplet (DigitalOcean) + add SSH public key
│ ├─ Add private key to CI/CD variables (file type: SSH_KEY)
│ ├─ before_script: chmod 400 $SSH_KEY
│ └─ script: ssh -i $SSH_KEY -o StrictHostKeyChecking=no root@IP "docker login && stop/remove old && docker run -d"
|
└─ Interview Flash Points ✨
├─ What is CI vs CD? (auto test/build vs auto deploy)
├─ Where do jobs run? (runners)
├─ How to keep secrets safe? (CI/CD variables masked)
├─ Why use stages? (control order / parallelism)
└─ Docker-in-docker? (client + daemon via service dind)
