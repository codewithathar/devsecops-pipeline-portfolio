# End-to-End DevSecOps Pipeline for Java (Spring Boot)

![Build Status](https://img.shields.io/badge/Build-Passing-brightgreen)
![Security Scan](https://img.shields.io/badge/Security-Trivy%20Scan-blue)
![Language](https://img.shields.io/badge/Language-Java%20%7C%20Docker-orange)

## 📋 Project Overview
This project demonstrates a **"Shift-Left" Security approach** by integrating automated security gates directly into the CI/CD pipeline. 

Instead of waiting for a security team to scan the application in production (using tools like **Prisma Cloud** or **Qualys**), this pipeline detects and **blocks** Critical vulnerabilities inside the developer workflow using **GitHub Actions** and **Trivy**.

### 🎯 Objective
To prevent vulnerable code and container images from ever reaching the deployment stage by enforcing strict quality gates.

---

## 🏗 Architecture & Workflow

The pipeline follows a standard **DevSecOps** workflow:

1.  **Code Commit:** Developer pushes Java code.
2.  **Build (Maven):** Application is compiled and unit tests are run.
3.  **SCA Scan (Trivy FS):** Scans dependencies (`pom.xml`) for known vulnerabilities.
4.  **Containerization:** Docker image is built.
5.  **Image Scan (Trivy Image):** Scans the final OS/Distroless base layer for vulnerabilities.
6.  **Quality Gate:** * If `Severity = CRITICAL` → **BUILD FAILS** (Deployment Blocked).
    * If `Severity < CRITICAL` → **BUILD PASSES** (Report Generated).

---

## 🛠 Tech Stack

| Component | Tool Used | Enterprise Equivalent (My Experience) |
| :--- | :--- | :--- |
| **Application** | Java 17, Spring Boot | Enterprise Java Apps |
| **CI/CD** | GitHub Actions | Jenkins, Azure DevOps |
| **Containerization** | Docker | Docker, Containerd |
| **Security Scanner** | **Trivy** (Open Source) | **Prisma Cloud, Qualys, JFrog Xray** |
| **Orchestration** | Kubernetes (Manifests included) | AKS, EKS, OpenShift |

---

## 🚀 Key Features Implemented

### 1. Automated Dependency Scanning (SCA)
The pipeline parses the `pom.xml` file to identify insecure Java libraries (e.g., Log4j, Spring4Shell) before the application is even packaged.

### 2. Container Image Hardening
Post-build, the Docker image is scanned for OS-level vulnerabilities (e.g., `glibc`, `openssl` issues). The pipeline enforces a threshold to **fail the build** if high-severity CVEs are detected.

### 3. "Breaker" Logic (Compliance Gate)
Custom logic in the YAML configuration ensures that security is not just "informative" but "enforced."
```yaml
# Snippet from my workflow
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'my-app:latest'
    format: 'table'
    exit-code: '1'   # <--- This forces the pipeline to fail
    severity: 'CRITICAL,HIGH'
