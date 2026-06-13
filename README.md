# Laravel boomkit — Base Image

![Docker Pulls](https://img.shields.io/docker/pulls/vanaboom/laravel-boomkit)
![GitHub last commit](https://img.shields.io/github/last-commit/vanaboom/laravel-boomkit)
![License](https://img.shields.io/badge/license-MIT-green)

A **production-ready Laravel Base Docker Image** built on **PHP 8.4** with Debian, Alpine, and FrankenPHP Alpine variants.
Pre-installed with essential PHP extensions, PECL modules, MSSQL tools, Node.js, Composer, and Laravel Echo Server — everything you need to start a Laravel project without manual setup.

This image works best when paired with [Laravel Toolbox](https://github.com/vanaboom/laravel-toolbox) — a complementary package that provides extra Laravel tooling and optimizations. Configuring both together in your Laravel application delivers the most efficient development and deployment experience.

---

## 🚀 Features

* **PHP 8.4** runtime variants (Debian Bookworm, Alpine, and FrankenPHP Alpine)
* Pre-installed **Composer** (latest version)
* Common PHP extensions:

    * `bcmath`, `gd`, `gmp`, `intl`, `ldap`, `mbstring`, `opcache`, `pcntl`, `pdo_mysql`, `pdo_pgsql`, `pgsql`, `sockets`, `zip`
* PECL extensions:

    * `apcu`, `pdo_sqlsrv`, `redis`, `sqlsrv`, `yaml`
* **MSSQL Client Tools** (`mssql-tools18`, `unixodbc-dev`)
* **Node.js & NPM**
* **Ioncube Loader** pre-installed on the CLI variants
* **Laravel Echo Server** globally installed
* Configured **timezone** (`Asia/Tehran` by default)
* Optional **APT/APK mirror**, **NPM registry**, and **Docker registry** override
* Optional **FrankenPHP** Alpine variant for Laravel Octane without changing the existing RoadRunner/PHP CLI variants

---

## 📦 Available Tags

| Tag                  | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| `debian` / `latest`  | Full Debian image with all dependencies & tools for Laravel  |
| `debian-slim`        | Slim Debian image without MSSQL, Node/NPM, or ionCube        |
| `alpine`             | Full Alpine image with Laravel runtime tools                 |
| `alpine-slim`        | Slim Alpine image without MSSQL, Node/NPM, or ionCube        |
| `alpine-frankenphp`  | FrankenPHP/PHP 8.4 Alpine image for Laravel Octane           |

---

## 🛠 Usage

### 1. Using as a Base Image

```dockerfile
FROM vanaboom/laravel-boomkit:debian

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
| `USE_APT_MIRROR_URL`      | `false`                                 | Set to `true` to use a custom APT mirror |
| `APT_MIRROR_URL`          | `http://deb.debian.org/debian`          | Custom Debian APT mirror URL             |
| `APT_SECURITY_MIRROR_URL` | `http://deb.debian.org/debian-security` | Custom Debian security mirror URL        |
| `APK_MIRROR_URL`          | `https://dl-cdn.alpinelinux.org/alpine` | Custom Alpine APK mirror URL             |
| `NPM_MIRROR_URL`               | `https://registry.npmjs.org`            | NPM registry URL                         |
| `DOCKER_REGISTRY`         | `docker.io`                             | Registry prefix or Docker Hub mirror used by variants such as `alpine-frankenphp` |
| `FRANKENPHP_IMAGE`        | `dunglas/frankenphp:1-php8.4-alpine`    | Base FrankenPHP image for `alpine-frankenphp` |

**Example:**

```bash
docker build \
  --build-arg USE_APT_MIRROR_URL=true \
  --build-arg APT_MIRROR_URL=https://mirror.example.com/apt/debian \
  --build-arg APT_SECURITY_MIRROR_URL=https://mirror.example.com/apt/debian/security \
  --build-arg NPM_MIRROR_URL=https://registry.example.com/ \
  -t vanaboom/laravel-boomkit:debian \
  ./debian
```

For Docker Hub mirrors such as a local or regional registry mirror, keep the image path the same and override only `DOCKER_REGISTRY`.

### FrankenPHP variant

```bash
docker build \
  --build-arg DOCKER_REGISTRY=docker.io \
  -t vanaboom/laravel-boomkit:alpine-frankenphp \
  ./alpine-frankenphp
```

This variant is intended for `OCTANE_SERVER=frankenphp`. RoadRunner-based projects can continue using the existing `debian`, `debian-slim`, `alpine`, and `alpine-slim` tags.

---

## 🔧 Build Locally (Optional)

```bash
git clone https://github.com/vanaboom/laravel-boomkit.git
cd laravel-boomkit
docker build -t vanaboom/laravel-boomkit:debian ./debian
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
