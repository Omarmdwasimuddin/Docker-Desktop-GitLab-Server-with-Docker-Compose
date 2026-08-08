# Docker Desktop দিয়ে GitLab Server সেটআপ (Docker Compose)

এই ডকুমেন্টে দেখানো হয়েছে কীভাবে Docker Compose ব্যবহার করে লোকাল মেশিনে একটি GitLab Server রান করানো যায়, এবং কীভাবে Volume ব্যবহার করে ডেটা persist করা যায়।

---

## ধাপ ১: প্রজেক্ট ফোল্ডার তৈরি

প্রথমে PowerShell খুলে নিচের কমান্ডগুলো চালাতে হবে:

```bash
mkdir Demo
cd Demo
code .
```

এতে `Demo` নামে একটি ফোল্ডার তৈরি হবে এবং VS Code-এ ওপেন হয়ে যাবে।

---

## ধাপ ২: `docker-compose.yml` ফাইল তৈরি

প্রজেক্ট ফোল্ডারে একটি `docker-compose.yml` ফাইল তৈরি করে নিচের কনফিগারেশন লিখতে হবে:

```yaml
services:
  gitlab-server:
    image: 'gitlab/gitlab-ce'
    container_name: my-gitlab-server
    ports:
      - '8000:80'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['initial_root_password'] = 'w@sIm1997'
        puma['worker_processes'] = 0
```

> **Note:** নতুন Docker Compose ভার্সনে `version` ফিল্ড আর দেওয়ার প্রয়োজন নেই, তাই এখানে বাদ দেওয়া হয়েছে।

### কনফিগারেশন ব্যাখ্যা

| Key | কাজ |
|---|---|
| `image` | কোন Docker image ব্যবহার হবে (এখানে অফিসিয়াল `gitlab-ce` image) |
| `container_name` | কন্টেইনারের নাম নির্ধারণ করে |
| `ports` | হোস্ট মেশিনের `8000` পোর্টকে কন্টেইনারের `80` পোর্টের সাথে ম্যাপ করে |
| `GITLAB_OMNIBUS_CONFIG` | GitLab-এর ইনিশিয়াল কনফিগারেশন (root password, worker process ইত্যাদি) সেট করার জায়গা |

---

## ধাপ ৩: কন্টেইনার রান করা

VS Code টার্মিনালে নিচের কমান্ড দিতে হবে:

```bash
docker compose up
```

---

## ধাপ ৪: ব্রাউজারে GitLab অ্যাক্সেস করা

ব্রাউজারে সার্চ করতে হবে:

```
http://localhost:8000/users/sign_in
```

> **Note:** GitLab পুরোপুরি লোড হতে কিছুটা সময় নিতে পারে (কয়েক মিনিট পর্যন্ত)।

লগইন করতে হবে:
- **Username:** `root`
- **Password:** `w@sIm1997`

লগইনের পর একটি **Setup Options** পেজ আসবে, যেখানে দুইটা অপশন থাকবে:
- **Skip** করে পরে Dashboard থেকে সেটআপ করা যাবে
- এখনই **Setup** করে সরাসরি Dashboard-এ যাওয়া যাবে

---

## ধাপ ৫: কন্টেইনার বন্ধ করা

```bash
docker compose down
```

> ⚠️ **সতর্কতা:** `docker compose down` (অথবা `docker compose stop` করে কন্টেইনার রিমুভ করলে) কন্টেইনারের ভিতরের সব ডেটা মুছে যেতে পারে, কারণ কোনো Volume attach করা নেই। এই সমস্যা সমাধানের জন্য পরের ধাপে Volume যোগ করা হয়েছে।

---

## ধাপ ৬: Volume যোগ করে ডেটা Persist করা

`docker-compose.yml` ফাইলে `volumes` সেকশন যোগ করতে হবে, যাতে কন্টেইনার বন্ধ বা রিমুভ হলেও ডেটা নষ্ট না হয়:

```yaml
services:
  gitlab-server:
    image: 'gitlab/gitlab-ce'
    container_name: my-gitlab-server
    ports:
      - '8000:80'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['initial_root_password'] = 'w@sIm1997'
        puma['worker_processes'] = 0
    volumes:
      - ./gitlab/config:/etc/gitlab
      - ./gitlab/logs:/var/log/gitlab
      - ./gitlab/data:/var/opt/gitlab
```

### Volume ম্যাপিং ব্যাখ্যা

| Host Path | Container Path | কী থাকে |
|---|---|---|
| `./gitlab/config` | `/etc/gitlab` | GitLab-এর কনফিগারেশন ফাইল |
| `./gitlab/logs` | `/var/log/gitlab` | সব ধরনের লগ ফাইল |
| `./gitlab/data` | `/var/opt/gitlab` | মূল ডেটা (repositories, database ইত্যাদি) |

---

## ধাপ ৭: আবার কন্টেইনার রান করা

```bash
docker compose up
```

এই কমান্ড রান করলে প্রজেক্ট ফোল্ডারে `gitlab` নামে একটি নতুন Directory তৈরি হবে, যার ভেতরে `config`, `data`, এবং `logs` — এই তিনটা Sub-directory থাকবে।

![GitLab volume directory structure](https://imgur.com/l75RhmB.png)

---

## ধাপ ৮: আবার ব্রাউজারে অ্যাক্সেস করা

```
http://localhost:8000/users/sign_in
```

> **Note:** এবারও লোড হতে কিছুটা সময় লাগবে।

লগইন করতে হবে:
- **Username:** `root`
- **Password:** `w@sIm1997`

লগইনের পর আগের মতোই **Setup Options** পেজ আসবে — Skip করা যাবে অথবা এখনই Setup করে Dashboard-এ যাওয়া যাবে।

---

## সারমর্ম

- Volume ছাড়া GitLab কন্টেইনার বন্ধ করলে সব ডেটা হারিয়ে যাওয়ার ঝুঁকি থাকে
- `volumes` ব্যবহার করে config, logs, এবং data আলাদাভাবে হোস্ট মেশিনে persist করা যায়
- এতে কন্টেইনার `down` করলেও পরে আবার `up` করলে আগের সব ডেটা, রিপোজিটরি, এবং সেটিংস ফিরে পাওয়া যায়
