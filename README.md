# Laravel boomkit — Base Image

![Docker Pulls](https://img.shields.io/docker/pulls/vanaboom/laravel-boomkit)
![GitHub last commit](https://img.shields.io/github/last-commit/vanaboom/laravel-boomkit)
![License](https://img.shields.io/badge/license-MIT-green)

A **production-ready Laravel Base Docker Image** built on **PHP 8.4 (Debian Bookworm)**.
Pre-installed with essential PHP extensions, PECL modules, MSSQL tools, Node.js, Composer, and Laravel Echo Server — everything you need to start a Laravel project without manual setup.

This image works best when paired with [Laravel Toolbox](https://github.com/vanaboom/laravel-toolbox) — a complementary package that provides extra Laravel tooling and optimizations. Configuring both together in your Laravel application delivers the most efficient development and deployment experience.

---

## 🚀 Features

* **PHP 8.4 CLI** (Debian Bookworm)
* Pre-installed **Composer** (latest version)
* Common PHP extensions:

    * `bcmath`, `gd`, `gmp`, `intl`, `ldap`, `mbstring`, `opcache`, `pcntl`, `pdo_pgsql`, `pgsql`, `sockets`, `zip`
* PECL extensions:

    * `apcu`, `pdo_sqlsrv`, `redis`, `sqlsrv`, `yaml`
* **MSSQL Client Tools** (`mssql-tools18`, `unixodbc-dev`)
* **Node.js & NPM**
* **Ioncube Loader** pre-installed
* **Laravel Echo Server** globally installed
* Configured **timezone** (`Asia/Tehran` by default)
* Optional **APT mirror** and **NPM registry** override

---

## 📦 Available Tags

| Tag      | Description                                          |
| -------- | ---------------------------------------------------- |
| `base`   | Full image with all dependencies & tools for Laravel |
| `latest` | Alias for the latest stable `base` version           |

---

## 🛠 Usage

### 1. Using as a Base Image

```dockerfile
FROM vanaboom/laravel-boomkit:base

WORKDIR /var/www/html
COPY . .

RUN composer install --no-dev --optimize-autoloader
```

### 2. Recommended: Pair with Laravel Toolbox

1. Install [Laravel Toolbox](https://github.com/vanaboom/laravel-toolbox) in your Laravel project:

   ```bash
   composer require vanaboom/laravel-toolbox
   ```
2. Publish and configure the package according to its documentation.

Using `laravel-boomkit` + `laravel-toolbox` provides:

* Pre-configured Docker environment
* Extra Laravel CLI tools
* Streamlined development workflow

---

## ⚙️ Build Arguments

| ARG / ENV                    | Default Value                           | Description                              |
| ---------------------------- | --------------------------------------- | ---------------------------------------- |
| `USE_CUSTOM_APT_MIRROR`      | `false`                                 | Set to `true` to use a custom APT mirror |
| `CUSTOM_APT_MIRROR`          | `http://deb.debian.org/debian`          | Custom Debian APT mirror URL             |
| `CUSTOM_APT_SECURITY_MIRROR` | `http://deb.debian.org/debian-security` | Custom Debian security mirror URL        |
| `NPM_REGISTRY`               | `https://registry.npmjs.org`            | NPM registry URL                         |

**Example:**

```bash
docker build \
  --build-arg USE_CUSTOM_APT_MIRROR=true \
  --build-arg CUSTOM_APT_MIRROR=https://mirror.example.com/apt/debian \
  --build-arg CUSTOM_APT_SECURITY_MIRROR=https://mirror.example.com/apt/debian/security \
  --build-arg NPM_REGISTRY=https://registry.example.com/ \
  -t vanaboom/laravel-boomkit:base \
  ./base
```

---

## 🔧 Build Locally (Optional)

```bash
git clone https://github.com/vanaboom/laravel-boomkit.git
cd laravel-boomkit/base
docker build -t vanaboom/laravel-boomkit:base .
```

---

## 📥 Pull the Image

```bash
docker pull vanaboom/laravel-boomkit:latest
```

---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## ❤️ Contributing

Pull requests are welcome!
If you find a bug or have a feature request, open an [issue](https://github.com/vanaboom/laravel-boomkit/issues).

---

**Maintainer:** [Vanaboom](https://github.com/vanaboom)
