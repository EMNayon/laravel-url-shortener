# 📄 YAML Cheatsheet for GitHub Actions

GitHub Actions Workflow ফাইল লেখার জন্য YAML জানাটা অপরিহার্য। এই চিটশিটটি GitHub Actions-এ সবচেয়ে বেশি ব্যবহৃত YAML প্যাটার্নগুলো কভার করে।

---

## ১. মূল ডেটা টাইপ (Basic Data Types)

```yaml
# String (স্ট্রিং)
name: My Workflow
branch: "main"          # quotes optional, কিন্তু special character থাকলে লাগে

# Integer (সংখ্যা)
timeout-minutes: 10

# Boolean (সত্য/মিথ্যা)
continue-on-error: false
fail-fast: true

# Null
value: ~               # null এর equivalent
```

---

## ২. List / Array (তালিকা)

```yaml
# Block Style (একাধিক লাইনে)
branches:
  - main
  - develop
  - "feature/*"

# Inline / Flow Style (এক লাইনে)
branches: [ "main", "develop" ]

# Nested List
matrix:
  os: [ ubuntu-latest, windows-latest ]
  php: [ '8.2', '8.3' ]
```

---

## ৩. Map / Dictionary (key-value pair)

```yaml
# Block Style
env:
  APP_ENV: testing
  DB_CONNECTION: sqlite
  DB_DATABASE: ":memory:"

# Inline Style
with: { php-version: '8.2', coverage: none }
```

---

## ৪. Multiline Strings (বহুলাইনের টেক্সট)

```yaml
# Literal Block Scalar (|) — প্রতিটি newline সংরক্ষিত হয়
run: |
  echo "Step 1"
  composer install
  php artisan migrate
  php artisan test

# Folded Block Scalar (>) — newline গুলো space এ রূপান্তরিত হয়
description: >
  This is a very long description
  that spans multiple lines
  but becomes a single line.
```

> **টিপস:** GitHub Actions-এ একাধিক Shell কমান্ড রান করতে সবসময় `run: |` ব্যবহার করুন।

---

## ৫. YAML Anchors & Aliases (কোড পুনরাবৃত্তি এড়ানো)

```yaml
# Anchor দিয়ে common config সংরক্ষণ
.default-php-setup: &php-setup
  uses: shivammathur/setup-php@v2
  with:
    php-version: '8.2'
    extensions: mbstring, pdo_sqlite

jobs:
  test:
    steps:
      - name: Setup PHP
        <<: *php-setup           # Alias দিয়ে পুনরায় ব্যবহার

  lint:
    steps:
      - name: Setup PHP
        <<: *php-setup           # আবার ব্যবহার করা হলো
```

---

## ৬. Indentation Rules (ইন্ডেন্টেশন নিয়ম)

```yaml
# ✅ সঠিক — ২ স্পেস indentation
jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - name: Step One
        run: echo "correct"

# ❌ ভুল — Tab ব্যবহার করলে YAML parse error হবে
jobs:
	my-job:            # ← এখানে Tab আছে, Error হবে!
```

---

## ৭. Comments (মন্তব্য)

```yaml
# এটি একটি Single-line Comment

name: My Workflow  # Inline comment এভাবেও লেখা যায়

# Multi-line comment এর জন্য
# প্রতিটি লাইনে # দিতে হয়
# YAML-এ block comment নেই
```

---

## ৮. Special Characters & Quoting (উদ্ধৃতি চিহ্ন)

```yaml
# ✅ Special character থাকলে quotes দিতে হয়
branch: "feature/my-feature"    # / থাকায় quotes লাগে
message: 'It''s working'        # single quote escape করতে '' ব্যবহার

# ✅ Expression এ সবসময় double quotes দিন
run: echo "${{ secrets.API_KEY }}"
if: ${{ github.event_name == 'push' }}
```

---

## ৯. Validation টিপস

```bash
# লোকাল machine-এ YAML ভ্যালিডেট করার জন্য
npx yaml-lint .github/workflows/my-workflow.yml

# অথবা Online tool ব্যবহার করুন:
# https://www.yamllint.com/
# https://rhysd.github.io/actionlint/
```

---

## 🔑 মনে রাখার মতো নিয়মাবলী

| নিয়ম | বিবরণ |
|:---|:---|
| **Tab নয়, Space** | Indentation এ সবসময় Space ব্যবহার করুন। |
| **২ Space** | প্রতিটি Level-এ ২টি করে Space দিন। |
| **Multiline = `|`** | একাধিক কমান্ড চালাতে `run: |` ব্যবহার করুন। |
| **Quote করুন** | Special character (`/`, `:`, `*`, `{`) থাকলে quotes দিন। |
| **`${{ }}`** | GitHub Expressions সবসময় double curly braces এ লিখুন। |
