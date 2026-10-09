# 🔮 GitHub Actions: Expressions & Contexts

GitHub Actions-এর নিজস্ব **Expression ও Context** সিস্টেম রয়েছে যা Workflow ফাইলের ভেতর ডায়নামিক ভ্যালু ব্যবহার করতে দেয়। এগুলো না জানলে Workflow Debug করা কঠিন হয়ে যায়।

---

## ১. Expression সিনট্যাক্স

সকল Expression `${{ }}` এর ভেতরে লেখতে হয়:

```yaml
- name: Print branch name
  run: echo "Branch is ${{ github.ref_name }}"

- name: Deploy only on main
  if: ${{ github.ref == 'refs/heads/main' }}
  run: ./deploy.sh
```

---

## ২. সবচেয়ে গুরুত্বপূর্ণ Contexts

### 🔵 `github` Context — Repository ও Event সম্পর্কিত তথ্য

| Expression | মানে কী? | উদাহরণ মান |
|:---|:---|:---|
| `github.ref` | Full branch/tag reference | `refs/heads/main` |
| `github.ref_name` | শুধু Branch বা Tag এর নাম | `main` |
| `github.event_name` | Trigger event এর নাম | `push`, `pull_request` |
| `github.actor` | কে কোড Push/Trigger করেছে | `nayon` |
| `github.repository` | Repo এর পূর্ণ নাম | `nayon/my-laravel-app` |
| `github.sha` | Current Commit এর SHA | `a1b2c3d...` |
| `github.run_number` | কততম বার এই Workflow রান হচ্ছে | `42` |

**উদাহরণ:**
```yaml
- name: Show commit info
  run: |
    echo "Actor: ${{ github.actor }}"
    echo "Branch: ${{ github.ref_name }}"
    echo "Commit: ${{ github.sha }}"
```

---

### 🟢 `runner` Context — Runner Machine সম্পর্কিত

| Expression | মানে কী? | উদাহরণ মান |
|:---|:---|:---|
| `runner.os` | Runner এর Operating System | `Linux`, `Windows`, `macOS` |
| `runner.arch` | CPU Architecture | `X64`, `ARM64` |
| `runner.temp` | Temp directory path | `/tmp` |

**উদাহরণ:**
```yaml
- name: Show runner info
  run: echo "Running on ${{ runner.os }} (${{ runner.arch }})"
```

---

### 🟡 `env` Context — Environment Variables

```yaml
env:
  APP_NAME: "My Laravel App"

jobs:
  test:
    steps:
      - name: Use env variable
        run: echo "App name is ${{ env.APP_NAME }}"
        # অথবা সরাসরি Shell Variable হিসেবে: echo "$APP_NAME"
```

---

### 🔴 `secrets` Context — GitHub Secrets

```yaml
- name: Deploy to server
  run: |
    echo "${{ secrets.SSH_PRIVATE_KEY }}" > private_key.pem
    ssh -i private_key.pem ${{ secrets.SSH_USERNAME }}@${{ secrets.SSH_HOST }} "cd /app && git pull"
```

> ⚠️ Secrets এর মান কখনো Log-এ দেখা যায় না। GitHub স্বয়ংক্রিয়ভাবে `***` দিয়ে mask করে।

---

### 🟠 `matrix` Context — Matrix Build তথ্য

```yaml
strategy:
  matrix:
    php: ['8.2', '8.3']
    os: [ubuntu-latest, windows-latest]

steps:
  - name: Setup PHP ${{ matrix.php }} on ${{ matrix.os }}
    uses: shivammathur/setup-php@v2
    with:
      php-version: ${{ matrix.php }}
```

---

### 🟣 `steps` Context — আগের Step এর Output ব্যবহার

```yaml
steps:
  - name: Get version
    id: get-version                          # Step এর ID
    run: echo "version=1.0.0" >> $GITHUB_OUTPUT

  - name: Use version
    run: echo "Deploying version ${{ steps.get-version.outputs.version }}"
```

---

## ৩. Built-in Functions (বিল্ট-ইন ফাংশন)

| ফাংশন | কাজ | উদাহরণ |
|:---|:---|:---|
| `contains(str, val)` | String এ কিছু আছে কিনা | `contains(github.ref, 'main')` |
| `startsWith(str, val)` | String কি দিয়ে শুরু | `startsWith(github.ref, 'refs/tags/')` |
| `endsWith(str, val)` | String কি দিয়ে শেষ | `endsWith(github.actor, 'bot')` |
| `format(str, val...)` | String format করা | `format('Hello {0}!', github.actor)` |
| `join(arr, sep)` | Array জোড়া লাগানো | `join(matrix.php, ', ')` |
| `toJSON(val)` | JSON string এ রূপান্তর | `toJSON(github.event)` |
| `fromJSON(str)` | JSON string parse করা | `fromJSON(steps.data.outputs.json)` |
| `hashFiles(path)` | File এর hash বের করা | `hashFiles('**/composer.lock')` |

**উদাহরণ (Deploy শুধু main বা release ব্রাঞ্চে):**
```yaml
- name: Deploy
  if: contains(fromJSON('["main", "release"]'), github.ref_name)
  run: ./deploy.sh
```

---

## ৪. Status Check Functions (`if` Condition এ)

```yaml
steps:
  - name: Run tests
    id: tests
    run: php artisan test

  - name: Notify on failure
    if: failure()          # আগের step fail করলে চলবে
    run: echo "Tests failed!"

  - name: Cleanup
    if: always()           # সফল বা ব্যর্থ যাই হোক, সবসময় চলবে
    run: rm -rf temp/

  - name: Upload artifact
    if: success()          # সব কিছু সফল হলে চলবে (default)
    run: echo "Uploading..."
```

| Function | কখন `true` হয়? |
|:---|:---|
| `success()` | সব আগের Step সফল হলে (Default) |
| `failure()` | যেকোনো Step ব্যর্থ হলে |
| `always()` | সর্বদা (fail হলেও) |
| `cancelled()` | Workflow বাতিল হলে |
