# 🚀 Step-by-Step GitHub Actions Practice Plan with Laravel

## ❓ Question: Private Repo vs Public Repo for GitHub Actions?
> **Answer:** **Yes! GitHub Actions supports both Private and Public repositories.**
> - **Public Repository:** 100% Free, Unlimited build minutes.
> - **Private Repository:** Free tier includes **2,000 free build minutes/month** (which is more than enough for practice and personal projects).

---

## 📌 Phase 1: Project & Repository Setup
- [ ] **Step 1.1:** Create a fresh Laravel application in the workspace.
- [ ] **Step 1.2:** Initialize Git repository (`git init`).
- [ ] **Step 1.3:** Create a new repository on GitHub (Public or Private) and connect local repo to remote.

---

## 📌 Phase 2: GitHub Actions Basics (First Workflow)
- [ ] **Step 2.1:** Create `.github/workflows/` folder structure.
- [ ] **Step 2.2:** Create a simple workflow file `hello-world.yml`.
- [ ] **Step 2.3:** Understand basic concepts:
  - `name`: Workflow title
  - `on`: Triggers (`push`, `pull_request`, `workflow_dispatch`)
  - `jobs` & `steps`: Task execution flow
  - `uses` & `run`: Action market actions vs custom bash shell scripts
- [ ] **Step 2.4:** Push and verify execution on GitHub Actions tab.

---

## 📌 Phase 3: Laravel CI (Continuous Integration) Workflow
- [ ] **Step 3.1:** Create `laravel-ci.yml`.
- [ ] **Step 3.2:** Configure PHP environment setup using `shivammathur/setup-php@v2` (PHP 8.2/8.3, extensions like `mbstring`, `pdo_sqlite`, etc.).
- [ ] **Step 3.3:** Install Composer dependencies efficiently.
- [ ] **Step 3.4:** Set up environment configuration (`.env` setup & `artisan key:generate`).
- [ ] **Step 3.5:** Run Code Style check using Laravel Pint (`./vendor/bin/pint --test`).
- [ ] **Step 3.6:** Run automated tests (`php artisan test`).

---

## 📌 Phase 4: Optimization & Advanced Workflows
- [ ] **Step 4.1:** **Dependency Caching:** Cache `composer` dependencies using GitHub Actions cache to speed up workflows.
- [ ] **Step 4.2:** **Matrix Testing:** Run tests across multiple PHP versions (e.g., 8.2, 8.3) simultaneously.
- [ ] **Step 4.3:** **Database Services:** Configure MySQL service container in GitHub Actions for integration tests.
- [ ] **Step 4.4:** **Branch Protection & PR Checks:** Learn how to block merging pull requests if CI tests fail.

---

## 📌 Phase 5: Automated Deployment (CD - Continuous Deployment) Concepts
- [ ] **Step 5.1:** Learn GitHub Secrets (`secrets.SSH_HOST`, `secrets.SSH_KEY`).
- [ ] **Step 5.2:** Create a deployment workflow (`deploy.yml`) for deploying code via SSH / Rsync to a server upon `push` to `main` branch.

---

## 🎯 Next Step
We will execute **Phase 1** first by creating the Laravel application!
