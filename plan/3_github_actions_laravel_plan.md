# 🚀 Step-by-Step GitHub Actions Practice Plan with Laravel

> এই ফাইলটি আমাদের সম্পূর্ণ প্র্যাকটিস রোডম্যাপ। প্রতিটি Phase সম্পন্ন করার পর checkbox টিক করুন।

---

## ❓ Private Repo vs Public Repo for GitHub Actions?

> - **Public Repository:** ১০০% ফ্রি, **Unlimited** build minutes।
> - **Private Repository:** Free account-এ প্রতি মাসে **২,০০০ মিনিট** বিনামূল্যে (Practice এর জন্য যথেষ্ট)।

---

## 📌 Phase 1: Project & Repository Setup

**লক্ষ্য:** Laravel প্রজেক্ট তৈরি করে GitHub-এ Push করা।

- [ ] **Step 1.1:** Workspace-এ ফ্রেশ Laravel Project তৈরি করা।
  ```bash
  composer create-project laravel/laravel laravel-app
  cd laravel-app
  ```
- [ ] **Step 1.2:** Local Git repository initialize করা।
  ```bash
  git init
  git add .
  git commit -m "Initial Laravel project setup"
  ```
- [ ] **Step 1.3:** GitHub-এ নতুন Public/Private Repo তৈরি করে Push করা।
  ```bash
  git remote add origin https://github.com/YOUR-USERNAME/REPO-NAME.git
  git branch -M main
  git push -u origin main
  ```

---

## 📌 Phase 2: GitHub Actions Basics (First Workflow)

**লক্ষ্য:** প্রথম Workflow তৈরি করে GitHub Actions কীভাবে কাজ করে তা দেখা।

- [ ] **Step 2.1:** প্রজেক্টের ভেতরে `.github/workflows/` ফোল্ডার তৈরি করা।
  ```bash
  mkdir -p .github/workflows
  ```
- [ ] **Step 2.2:** প্রথম simple workflow ফাইল তৈরি করা।
  ```yaml
  # .github/workflows/hello-world.yml
  name: Hello World Workflow

  on:
    push:
      branches: [ "main" ]

  jobs:
    say-hello:
      runs-on: ubuntu-latest
      steps:
        - name: Print Hello Message
          run: echo "Hello from GitHub Actions! 🚀"

        - name: Show Runner OS Info
          run: uname -a
  ```
- [ ] **Step 2.3:** Push করে GitHub-এর **Actions Tab** এ গিয়ে Workflow execution দেখা।
  ```bash
  git add .
  git commit -m "Add first hello-world GitHub Actions workflow"
  git push
  ```

---

## 📌 Phase 3: Laravel CI (Continuous Integration) Workflow

**লক্ষ্য:** কোড Push হলে স্বয়ংক্রিয়ভাবে Laravel Project Build, Code Style Check ও Test Run করানো।

- [ ] **Step 3.1:** `laravel-ci.yml` ফাইল তৈরি করা।
- [ ] **Step 3.2:** PHP Environment Setup করা (`shivammathur/setup-php@v2` দিয়ে PHP 8.2/8.3, mbstring, pdo_sqlite ইত্যাদি extension সহ)।
- [ ] **Step 3.3:** Composer dependencies install করা।
  ```bash
  composer install --prefer-dist --no-progress --no-interaction
  ```
- [ ] **Step 3.4:** `.env` সেটআপ ও App Key Generate করা।
  ```bash
  cp .env.example .env
  php artisan key:generate
  ```
- [ ] **Step 3.5:** Laravel Pint দিয়ে Code Style Check করা।
  ```bash
  ./vendor/bin/pint --test
  ```
- [ ] **Step 3.6:** Automated Tests রান করা।
  ```bash
  php artisan test
  ```

**সম্পূর্ণ `laravel-ci.yml` উদাহরণ:**
```yaml
name: Laravel CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  laravel-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          extensions: mbstring, pdo_sqlite, sqlite3

      - name: Install Dependencies
        run: composer install --prefer-dist --no-progress --no-interaction

      - name: Setup Environment
        run: |
          cp .env.example .env
          php artisan key:generate

      - name: Run Pint (Code Style)
        run: ./vendor/bin/pint --test

      - name: Run Tests
        run: php artisan test
```

---

## 📌 Phase 4: Optimization & Advanced Workflows

**লক্ষ্য:** CI Pipeline কে দ্রুততর ও আরও শক্তিশালী করা।

- [ ] **Step 4.1:** **Composer Caching** যুক্ত করা (Build Time কমানো)।
  ```yaml
  - name: Cache Composer dependencies
    uses: actions/cache@v4
    with:
      path: vendor
      key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.lock') }}
      restore-keys: ${{ runner.os }}-composer-
  ```
- [ ] **Step 4.2:** **Matrix Testing** দিয়ে একাধিক PHP version (8.2, 8.3) এ একসাথে Test করা।
  ```yaml
  strategy:
    matrix:
      php-version: ['8.2', '8.3']
  ```
- [ ] **Step 4.3:** **MySQL Service Container** দিয়ে Real Database-এ Integration Test চালানো।
  ```yaml
  services:
    mysql:
      image: mysql:8.0
      env:
        MYSQL_DATABASE: laravel_test
        MYSQL_ROOT_PASSWORD: password
      ports:
        - 3306:3306
  ```
- [ ] **Step 4.4:** **Branch Protection Rule** সেটআপ করা — CI ফেল করলে PR মার্জ ব্লক করা।
  - GitHub → Settings → Branches → Add Branch Protection Rule → "Require status checks to pass"

---

## 📌 Phase 5: Automated Deployment (CD - Continuous Deployment)

**লক্ষ্য:** `main` ব্রাঞ্চে Push হলে স্বয়ংক্রিয়ভাবে VPS/Server-এ Deploy করা।

- [ ] **Step 5.1:** GitHub Secrets সেটআপ করা (Settings → Secrets and variables → Actions):
  - `SSH_HOST` — Server এর IP Address
  - `SSH_USERNAME` — SSH Username
  - `SSH_PRIVATE_KEY` — SSH Private Key

- [ ] **Step 5.2:** `deploy.yml` Workflow তৈরি করা।
  ```yaml
  name: Deploy to Production

  on:
    push:
      branches: [ "main" ]

  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4

        - name: Deploy via SSH
          uses: appleboy/ssh-action@v1
          with:
            host: ${{ secrets.SSH_HOST }}
            username: ${{ secrets.SSH_USERNAME }}
            key: ${{ secrets.SSH_PRIVATE_KEY }}
            script: |
              cd /var/www/laravel-app
              git pull origin main
              composer install --no-dev --optimize-autoloader
              php artisan migrate --force
              php artisan config:cache
              php artisan route:cache
  ```

---

## 📋 Practice Progress Tracker

| Phase | বিষয় | স্ট্যাটাস |
|:---:|:---|:---:|
| 1 | Laravel Project & Git Setup | ⬜ |
| 2 | First Hello World Workflow | ⬜ |
| 3 | Laravel CI Pipeline | ⬜ |
| 4 | Caching + Matrix + MySQL | ⬜ |
| 5 | CD Auto Deployment | ⬜ |
