# 🚀 Contributing to Audiobookshelf

Thank you for contributing! This guide explains how to set up the project, make changes, run security checks, and submit a pull request.

---

# Table of Contents

- [Prerequisites](#prerequisites)
- [Repository Setup](#repository-setup)
- [Installation](#installation)
- [Running the Application](#running-the-application)
- [Security Testing](#security-testing)
- [Making Changes](#making-changes)
- [Opening a Pull Request](#opening-a-pull-request)
- [Keeping Your Fork Updated](#keeping-your-fork-updated)
- [Troubleshooting](#troubleshooting)
- [Useful Commands](#useful-commands)
- [Checklist Before Submitting](#checklist-before-submitting)

---

# Prerequisites

Install the following tools before getting started.

| Tool | Recommended Version |
|------|----------------------|
| Git | Latest |
| Node.js | 20.x |
| npm | 10.x |
| Docker | Latest (Optional) |
| Docker Compose | Latest (Optional) |
| GitHub CLI (`gh`) | Latest (Recommended) |

## Install Node.js 20 (Ubuntu/Debian)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version
npm --version
```

## Install Docker (Optional)

```bash
sudo apt update
sudo apt install -y docker.io docker-compose

sudo systemctl enable docker
sudo systemctl start docker

sudo usermod -aG docker $USER

# Log out and back in
# or run
newgrp docker
```

---

# Repository Setup

## 1. Fork the Repository

Fork this repository using the **Fork** button on GitHub.

## 2. Clone Your Fork

Replace `YOUR_USERNAME` with your GitHub username.

```bash
git clone https://github.com/YOUR_USERNAME/audiobookshelf.git

cd audiobookshelf
```

Add the original repository as the upstream remote.

```bash
git remote add upstream https://github.com/PawKoi/audiobookshelf.git

git remote -v
```

Example output:

```text
origin    https://github.com/YOUR_USERNAME/audiobookshelf.git (fetch)
origin    https://github.com/YOUR_USERNAME/audiobookshelf.git (push)
upstream  https://github.com/PawKoi/audiobookshelf.git (fetch)
upstream  https://github.com/PawKoi/audiobookshelf.git (push)
```

---

# Installation

Install project dependencies.

```bash
npm install
```

Install frontend dependencies.

```bash
cd client
npm install
cd ..
```

Verify installation.

```bash
npm list --depth=0
```

---

# Running the Application

## Development Mode

Runs with hot reload.

```bash
npm run dev
```

Application:

```
http://localhost:3000
```

---

## Production Mode

```bash
npm start
```

Application:

```
http://localhost:80
```

---

## Docker

Build the image.

```bash
docker build -t audiobookshelf .
```

Run the container.

```bash
docker run -d \
    -p 80:80 \
    --name abs-test \
    audiobookshelf
```

Stop the container.

```bash
docker stop abs-test
docker rm abs-test
```

---

## Docker Compose

```bash
docker-compose up -d
```

View logs.

```bash
docker-compose logs -f
```

Stop.

```bash
docker-compose down
```

---

# Security Testing

Run security tools before opening a pull request whenever possible.

## npm Audit

```bash
npm audit

npm audit --json > npm-audit.json
```

## Semgrep

```bash
semgrep scan \
    --config p/owasp-top-ten \
    --exclude node_modules
```

## Gitleaks

```bash
gitleaks detect \
    --source . \
    --report-format json \
    --report-path gitleaks-report.json
```

## Trivy

```bash
trivy fs .
```

---

# GitHub Actions

List workflow runs.

```bash
gh run list --workflow "DevSecOps Security Pipeline"
```

Watch the latest workflow.

```bash
gh run watch
```

Download workflow artifacts.

```bash
gh run download
```

---

# Making Changes

## Create a Feature Branch

Always create a new branch.

```bash
git checkout -b feature/your-feature
```

Examples:

```bash
git checkout -b feature/update-docs
git checkout -b feature/improve-auth
git checkout -b security/fix-sql-injection
```

---

## Make Your Changes

Format code if available.

```bash
npm run format
```

Run tests.

```bash
npm test
```

---

## Commit Your Changes

Stage your work.

```bash
git add .
```

Commit using conventional commits.

```bash
git commit -m "feat: add new feature"
```

Security fixes.

```bash
git commit -m "fix: patch SQL injection vulnerability"
```

---

## Push Your Branch

```bash
git push -u origin feature/your-feature
```

---

# Opening a Pull Request

Open GitHub and create a new Pull Request.

Use the following configuration.

| Setting | Value |
|----------|-------|
| Base Repository | PawKoi/audiobookshelf |
| Base Branch | security/devsecops-pipeline |
| Head Repository | YOUR_USERNAME/audiobookshelf |
| Compare Branch | feature/your-feature |

Include:

- Clear title
- Description of changes
- Testing performed
- Related issues (if any)

---

# CI/CD Pipeline

Every pull request automatically runs:

- ✅ Static Application Security Testing (SAST)
- ✅ Dependency & Container Scanning (SCA)
- ✅ Dynamic Security Testing (DAST)
- ✅ Deployment Validation
- ✅ Notifications

Your pull request should only be merged after all checks pass.

---

# Keeping Your Fork Updated

Fetch upstream changes.

```bash
git fetch upstream
```

Switch to the development branch.

```bash
git checkout security/devsecops-pipeline
```

Merge upstream.

```bash
git merge upstream/security/devsecops-pipeline
```

Push to your fork.

```bash
git push origin security/devsecops-pipeline
```

---

# Troubleshooting

## npm install fails

```bash
rm -rf node_modules package-lock.json

npm install
```

---

## Docker build fails

Check available disk space.

```bash
df -h
```

Clean Docker.

```bash
docker system prune -a -f
```

---

## Port Already in Use

Use another port or modify your configuration.

Example:

```bash
docker run -p 8080:80 ...
```

---

## Pipeline Fails

View failed logs.

```bash
gh run view --log-failed
```

---

## Docker Service Status

```bash
sudo systemctl status docker
```

---

# Useful Commands

## Clone

```bash
git clone https://github.com/YOUR_USERNAME/audiobookshelf.git

cd audiobookshelf
```

## Install

```bash
npm install
```

## Start Development

```bash
npm run dev
```

## Run Security Audit

```bash
npm audit --json > npm-audit.json
```

## Create Feature Branch

```bash
git checkout -b feature/my-feature
```

## Push Changes

```bash
git push -u origin feature/my-feature
```

## View GitHub Actions

```bash
gh run list --workflow "DevSecOps Security Pipeline"
```

## Update Fork

```bash
git fetch upstream

git merge upstream/security/devsecops-pipeline

git push origin security/devsecops-pipeline
```

## Clean Docker

```bash
docker system prune -a -f
```

---

# Checklist Before Submitting

- [ ] Code builds successfully
- [ ] Tests pass
- [ ] Security scans completed
- [ ] No secrets committed
- [ ] Commit messages follow Conventional Commits
- [ ] Branch is up to date
- [ ] CI/CD pipeline passes
- [ ] Pull request description is complete

---

# After Your Pull Request is Merged

Update your local repository.

```bash
git checkout security/devsecops-pipeline

git pull upstream security/devsecops-pipeline
```

Delete your feature branch.

```bash
git branch -d feature/your-feature
```

Delete the remote branch.

```bash
git push origin --delete feature/your-feature
```

---

# Need Help?

If you encounter issues:

- Open a GitHub Issue
- Start a GitHub Discussion
- Review the workflow logs using:

```bash
gh run view --log
```

---

## Thank You ❤️

Thank you for helping improve **Audiobookshelf**.

Your contributions—whether code, documentation, testing, or security improvements—are greatly appreciated.
