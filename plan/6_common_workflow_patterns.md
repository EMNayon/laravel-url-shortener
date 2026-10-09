# 🗂️ Common GitHub Actions Workflow Patterns

Real-world প্রজেক্টে সবচেয়ে বেশি ব্যবহৃত Workflow Pattern গুলো এখানে রেফারেন্স হিসেবে রাখা হয়েছে।

---

## Pattern 1: PR-Only Testing (PR তে শুধু Test চালানো)

`main` এ সরাসরি push হলে নয়, শুধু Pull Request-এ Test চলবে:

```yaml
name: PR Tests

on:
  pull_request:
    branches: [ "main" ]    # শুধু main-এ PR দিলে trigger হবে

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Tests
        run: php artisan test
```

---

## Pattern 2: Scheduled / Cron Workflow (নির্দিষ্ট সময়ে চালানো)

প্রতিদিন রাত ২টায় স্বয়ংক্রিয়ভাবে টেস্ট করার উদাহরণ:

```yaml
name: Nightly Tests

on:
  schedule:
    - cron: '0 20 * * *'    # UTC 20:00 = Bangladesh 02:00 রাত
    # cron format: minute hour day month weekday

jobs:
  nightly-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Full Test Suite
        run: php artisan test --coverage
```

**Cron সময় হিসাব:**
| Cron Expression | অর্থ |
|:---|:---|
| `0 * * * *` | প্রতি ঘণ্টায় |
| `0 0 * * *` | প্রতিদিন UTC midnight (BD রাত ৬টা) |
| `0 20 * * *` | প্রতিদিন UTC 20:00 (BD রাত ২টা) |
| `0 0 * * 1` | প্রতি সোমবার |
| `*/15 * * * *` | প্রতি ১৫ মিনিটে |

---

## Pattern 3: On-Demand / Manual Workflow (হাতে বাটন চেপে চালানো)

GitHub Actions Tab থেকে ম্যানুয়ালি চালানোর Workflow:

```yaml
name: Manual Deploy

on:
  workflow_dispatch:             # GitHub UI থেকে "Run workflow" বাটন দিয়ে চালানো যাবে
    inputs:
      environment:
        description: 'কোথায় Deploy করবেন?'
        required: true
        type: choice
        options:
          - staging
          - production
      debug_mode:
        description: 'Debug Mode চালু করবেন?'
        required: false
        type: boolean
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Show selected options
        run: |
          echo "Environment: ${{ inputs.environment }}"
          echo "Debug: ${{ inputs.debug_mode }}"
```

---

## Pattern 4: Composer Caching (দ্রুত Build করা)

প্রতিবার নতুন করে `composer install` না করে Cache ব্যবহার করা:

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Get Composer Cache Directory
    id: composer-cache
    run: echo "dir=$(composer config cache-files-dir)" >> $GITHUB_OUTPUT

  - name: Cache Composer Dependencies
    uses: actions/cache@v4
    with:
      path: ${{ steps.composer-cache.outputs.dir }}
      key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.lock') }}
      restore-keys: |
        ${{ runner.os }}-composer-

  - name: Install Dependencies
    run: composer install --prefer-dist --no-progress
```

---

## Pattern 5: Matrix Build (একাধিক PHP Version-এ Test)

```yaml
name: Matrix Tests

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false        # একটা version fail করলে বাকিগুলো বন্ধ হবে না
      matrix:
        php-version: ['8.1', '8.2', '8.3']

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP ${{ matrix.php-version }}
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-version }}

      - name: Install & Test
        run: |
          composer install
          php artisan test
```

---

## Pattern 6: Slack/Email Notification (Build Success/Fail জানানো)

```yaml
steps:
  - name: Run Tests
    id: tests
    run: php artisan test

  - name: Notify Slack on Failure
    if: failure()
    uses: slackapi/slack-github-action@v2
    with:
      webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
      webhook-type: incoming-webhook
      payload: |
        {
          "text": "❌ Build Failed on `${{ github.ref_name }}` by *${{ github.actor }}*"
        }

  - name: Notify Slack on Success
    if: success()
    uses: slackapi/slack-github-action@v2
    with:
      webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
      webhook-type: incoming-webhook
      payload: |
        {
          "text": "✅ Build Passed on `${{ github.ref_name }}` by *${{ github.actor }}*"
        }
```

---

## Pattern 7: Deploy শুধু Tag Push-এ (Release Deploy)

`v1.0.0` এর মতো Tag দিলে তবেই Production-এ Deploy হবে:

```yaml
name: Release Deploy

on:
  push:
    tags:
      - 'v*.*.*'             # v1.0.0, v2.1.3 এই ধরনের Tag-এ Trigger হবে

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Get Release Version
        run: echo "Deploying version ${{ github.ref_name }}"

      - name: Deploy to Production
        run: ./deploy.sh production
```

---

## Pattern 8: Job Output দিয়ে একাধিক Job এ Data শেয়ার করা

```yaml
jobs:
  prepare:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get-ver.outputs.version }}    # Output declare করা

    steps:
      - name: Get App Version
        id: get-ver
        run: echo "version=$(cat VERSION)" >> $GITHUB_OUTPUT

  deploy:
    needs: prepare                                     # prepare job এর পরে চলবে
    runs-on: ubuntu-latest
    steps:
      - name: Use version from previous job
        run: echo "Deploying ${{ needs.prepare.outputs.version }}"
```

---

## 🔑 Quick Reference Summary

| Pattern | Trigger | ব্যবহারের ক্ষেত্র |
|:---|:---|:---|
| PR Testing | `pull_request` | PR মার্জের আগে কোড যাচাই করা |
| Scheduled | `schedule` (cron) | নিয়মিত Health Check বা Backup |
| Manual | `workflow_dispatch` | হাতে Deploy বা Test চালানো |
| Caching | `actions/cache@v4` | Build সময় কমানো |
| Matrix | `strategy.matrix` | একাধিক ভার্সনে Compatibility টেস্ট |
| Notification | `if: failure()` + Slack Action | বিল্ড স্ট্যাটাস জানানো |
| Tag Deploy | `push.tags` | Production Release Deployment |
| Job Output | `outputs` + `needs` | Jobs এর মধ্যে Data আদান-প্রদান |
