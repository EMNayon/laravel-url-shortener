# 📚 GitHub Actions: বেসিক ধারণাসমূহ (Fundamental Concepts & Syntax Guide)

GitHub Actions দিয়ে কাজ শুরু করার পূর্বে নিচের মূল বিষয়গুলো জানা অত্যন্ত জরুরি:

---

## 📂 ১. ফাইল লোকেশন ও ফরম্যাট (File Structure)
* **ফাইল লোকেশন:** সমস্ত Workflow ফাইল অবশ্যই প্রজেক্টের **`.github/workflows/`** ফোল্ডারে থাকতে হবে।
* **ফাইল টাইপ:** ফাইল ফরম্যাট **`.yml`** বা **`.yaml`** হতে হবে।
* **YAML কড়াকড়ি:** YAML ফাইলে স্পেস (Indentation) ঠিক রাখা আবশ্যক। ট্যাব (Tab) ব্যবহার করা যাবে না, সাধারণত **২ টি স্পেস** ব্যবহার করতে হয়।

---

## 🧱 ২. Workflow ফাইলের মূল ৫টি অংশ (Key Components)

একটি সাধারণ Workflow ফাইলের উদাহরণ:

```yaml
name: Laravel CI Workflow           # ১. Workflow এর নাম

on:                                 # ২. Trigger (কখন এই workflow রান হবে)
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:                               # ৩. Job ডিক্লেয়ারেশন
  build-and-test:                   # Job ID
    runs-on: ubuntu-latest          # ৪. Runner OS (GitHub Virtual Server)

    steps:                          # ৫. স্টেপস (কাজের ক্রম)
      - name: Code checkout করা
        uses: actions/checkout@v4

      - name: PHP Setup করা
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'

      - name: Dependencies Install করা
        run: composer install --prefer-dist --no-progress

      - name: Test রান করা
        run: php artisan test
```

---

## 🔍 ৩. `uses` এবং `run` এর পার্থক্য

| বিষয় | `uses` | `run` |
| :--- | :--- | :--- |
| **কাজ** | GitHub Marketplace থেকে তৈরি করা রেডিমেড Action ব্যবহার করা। | টার্মিনাল বা Shell কমান্ড রান করা। |
| **উদাহরণ** | `uses: actions/checkout@v4` | `run: php artisan test` |
| **কখন লাগে** | PHP Setup, Git Checkout, Caching, Cloud Auth ইত্যাদি জটিল কাজে। | Custom Script, Composer Install, Artisan Command ইত্যাদি সাধারণ কাজে। |

---

## 🔐 ৪. Secrets এবং Environment Variables

কখনোই সিক্রেট তথ্য (DB Password, SSH Key, API Token) কোডের মধ্যে সরাসরি লিখবেন না।

1. **GitHub Secrets:** 
   - GitHub Repo > Settings > Secrets and variables > Actions এ গিয়ে সিক্রেট সেভ করতে হয়।
   - Workflow-এ ব্যবহার করার নিয়ম: `${{ secrets.MY_API_KEY }}`

2. **Environment Variables (`env`):**
   - নরমাল কনফিগারেশন ভ্যারিয়েবল সেট করতে ব্যবহৃত হয়।
   ```yaml
   env:
     APP_ENV: testing
     DB_CONNECTION: sqlite
   ```

---

## 🔗 ৫. জবের নির্ভরতা (`needs`) ও প্যারালাল এক্সিকিউশন

ডিফল্টভাবে একাধিক Job থাকলে সেগুলো **Parallel (একসাথে)** রান করে। কিন্তু যদি আপনি চান যে Build শেষ হওয়ার পরই কেবল Test রান করবে, তখন **`needs`** ব্যবহার করতে হয়:

```yaml
jobs:
  lint-check:
    runs-on: ubuntu-latest
    steps: ...

  run-tests:
    needs: lint-check          # lint-check সফলভাবে পাস করার পর এই Job রান করবে
    runs-on: ubuntu-latest
    steps: ...
```

---

## 🎛️ ৬. ম্যানুয়াল ট্রিগার (`workflow_dispatch`)

মাঝে মাঝে গিটহাবে পুশ না করেও হাত দিয়ে টেস্ট বা ডেপ্লয় বাটন চেপে চালানের প্রয়োজন হয়। এজন্য `workflow_dispatch` ব্যবহার করা হয়:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target Environment'
        required: true
        default: 'staging'
```

---

## 💡 ৭. প্রয়োজনীয় এক্সপ্রেশন ও ফিল্টার (`if` Condition)

নির্দিষ্ট শর্তে কোনো Step বা Job চালাতে `if` ব্যবহার করা যায়:

```yaml
steps:
  - name: Deploy to Server
    if: github.ref == 'refs/heads/main'   # শুধুমাত্র main ব্রাঞ্চে কন্ডিশন সত্য হলে চলবে
    run: ./deploy.sh
```

---

## 📝 সারসংক্ষেপ
এগুলোই হলো GitHub Actions এর প্রধান মৌলিক উপাদানসমূহ। এই বিষয়গুলো জানা থাকলে আপনি যেকোনো জটিল CI/CD Pipeline সহজেই পড়তে ও তৈরি করতে পারবেন!
