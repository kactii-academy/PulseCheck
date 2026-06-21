# 🚀 PulseCheck — CI/CD with Docker & AWS EC2

PulseCheck is a lightweight FastAPI-based health monitoring service built to **demonstrate real-world CI/CD principles**, Dockerized deployments, and SSH-based Continuous Deployment on AWS EC2 using GitHub Actions.

This project focuses on **proof of automation**, not just configuration.

---

## 🔍 Project Goals

* Build a simple health-check API (`/health`)
* Containerize the application using Docker
* Deploy on AWS EC2 (Free Tier)
* Implement **Continuous Deployment** using GitHub Actions
* Validate that deployments actually reach production
* Learn and document real-world DevOps failure scenarios

---

## 🧱 Tech Stack

* **Language**: Python
* **Framework**: FastAPI
* **Containerization**: Docker
* **Cloud**: AWS EC2 (Ubuntu LTS)
* **CI/CD**: GitHub Actions
* **Deployment Method**: SSH-based CD (no Kubernetes, no Terraform)

---

## 📁 Repository Structure

```
PulseCheck/
├── app.py                 # FastAPI application
├── Dockerfile             # Docker image definition
├── .github/
│   └── workflows/
│       ├── ci.yml         # CI: tests & health checks
│       └── cd.yml         # CD: deploy to EC2 via SSH
└── README.md
```

---

## ⚙️ Application Endpoints

### Health Check

```
GET /health
```

**Response:**

```json
{
  "status": "ok",
  "version": "PulseCheck v1.1 – deployed via CI/CD"
}
```

> The version marker is intentionally included to **prove real deployments**.

---

## 🧪 CI Pipeline (GitHub Actions)

**Triggered on:** Push to `main`

**What CI does:**

* Checks out code
* Installs dependencies
* Runs basic health validation
* Ensures app starts correctly

✅ CI must pass before CD executes

---

## 🚢 CD Pipeline (GitHub Actions → EC2)

**Triggered on:** Push to `main` (after CI)

### Deployment Flow

1. GitHub Actions sets up SSH securely
2. Connects to EC2 using a **dedicated deployment key**
3. Pulls latest code on EC2
4. Builds Docker image
5. Stops old container (if exists)
6. Runs new container with:

   * Port mapping
   * Restart policy
7. Application becomes live automatically

---

## 🔐 Security Highlights

* SSH access restricted to **My IP**
* EC2 access via key-based authentication only
* Secrets stored securely in GitHub Actions:

  * `EC2_HOST`
  * `EC2_USER`
  * `EC2_SSH_KEY`
  * `APP_PORT`

---

## ✅ Deployment Validation (Proof)

The deployment is considered successful only when:

* GitHub Actions CD pipeline turns **green**
* No manual SSH is used for deployment
* Browser shows **updated version string**
* Docker container restarts automatically
* Health endpoint responds correctly

Example:

```
http://<EC2_PUBLIC_IP>:8000/health
```

---

## ⚠️ What Went Wrong (Real Issues Faced)

This project intentionally documents **real failures**, because DevOps is about handling them.

### Common Issues Encountered

#### 1. Security Group Misconfiguration

* Port `8000` blocked → browser timeout
* Fixed by explicitly allowing `8000` from my IP

#### 2. SSH Key Confusion

* Mixed local SSH key vs GitHub Actions SSH key
* Learned to create **separate deployment keys**

#### 3. Docker Permission Errors

* `permission denied while trying to connect to docker.sock`
* Fixed by:

  ```bash
  sudo usermod -aG docker ubuntu
  ```

#### 4. Container Running but Not Accessible

* App bound to `localhost` instead of `0.0.0.0`
* Fixed in app / Docker configuration

#### 5. GitHub Secrets Formatting Errors

* Newlines in SSH private key broke authentication
* Learned that **secrets can fail silently**

#### 6. Pipeline Passed but App Didn’t Update

* Old container still running
* Container name mismatch (`pulsecheck` vs random name)
* Fixed by explicitly stopping & removing containers

#### 7. CD Pipeline Failed on Re-run

* `mkdir ~/.ssh` failed because directory existed
* Fixed by making steps **idempotent**

---

## 🧠 Engineering Questions I Asked & What I Learned

This section captures the key questions I consciously asked myself during development, the decisions I made, and the practical DevOps lessons learned while building and deploying PulseCheck.

### ❓ How does my CI/CD pipeline work end-to-end — and how do I prove it actually deploys?

**What I asked myself**

* Is my pipeline only green, or is it truly deploying to production?
* How can I verify that the latest commit is what’s running on the EC2 instance?

**What I implemented**

* A GitHub Actions pipeline triggered on every push to main
* CI stage to validate application startup
* CD stage that:
  * Connects to AWS EC2 via SSH using GitHub Secrets
  * Builds a Docker image on the server
  * Stops and removes any existing container
  * Runs the updated container with proper port binding
* A public `/health` endpoint returning a deployment version marker

**What I learned**

* A successful pipeline run does not guarantee a successful deployment
* Versioned health endpoints are a simple but powerful way to verify real production updates
* CI/CD should be observable from outside the server, not just from logs

---

### ❓ Why did my deployment succeed but the app was unreachable?

**What I asked myself**

* If Docker says the container is running, why does the browser fail?
* Is this an app issue, container issue, or infrastructure issue?

**What I discovered**

* Missing or incorrect EC2 Security Group inbound rules
* Application not bound to `0.0.0.0`
* Port exposed in Docker but blocked at the network level

**What I learned**

* Deployment spans application, container, OS, and cloud networking
* Debugging DevOps issues requires moving layer by layer, not guessing
* Tools don’t fail alone — integrations do

---

### ❓ Why did my CD job fail even though the code was correct?

**What I asked myself**

* Why is GitHub Actions failing before deployment even starts?
* Why does SSH setup break on re-runs?

**What I fixed**

* Made SSH setup idempotent
* Ensured directories and keys don’t error if they already exist
* Handled container stop/remove commands safely

**What I learned**

* CI/CD pipelines must be re-runnable by design
* Idempotency is a core DevOps principle, not an optimization
* Pipelines should assume partial failure is normal

---

### ❓ What does this project really teach beyond tools?

**Key takeaways**

* DevOps is less about YAML and more about systems thinking
* “Works on my machine” means nothing without production validation
* Failures are signals — each one improved the pipeline design
* Simple projects executed end-to-end are more valuable than complex demos

---

✅ **Final Reflection**

This project helped me move from writing pipelines to owning deployments. I now approach CI/CD with a production mindset — validating not just execution, but outcome.

---

## 📌 Current Limitations (Known Gaps)

* No zero-downtime deployment strategy
* No rollback mechanism
* No HTTPS / Load Balancer
* No monitoring or alerting
* Single EC2 instance only

> These are **intentional exclusions** for learning clarity.

---

## 🎯 Future Improvements

* Add Nginx reverse proxy
* Implement blue-green or rolling deployments
* Add monitoring (Prometheus / CloudWatch)
* Introduce rollback strategy
* Migrate to Infrastructure as Code

---

## 📄 License

MIT

---

## Credits

This repository was mirrored from [https://github.com/Kirankumarvel/PulseCheck](https://github.com/Kirankumarvel/PulseCheck).
All credit goes to the original authors.
