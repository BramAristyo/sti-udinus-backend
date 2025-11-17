# Local Development Setup

* **Clone Repository**

```bash
git clone <repository-url>
cd sti-api
```

* **Install Dependencies**

```bash
composer install
```

* **Environment Setup**

```bash
cp .env.example .env
php artisan key:generate
```

* **Database**

```bash
php artisan migrate
php artisan db:seed
```

* **Storage**

```bash
php artisan storage:link
```

* **Run Development Server**

```bash
php artisan serve
```

* **Run Queue Job**

```bash
php artisan queue:work
```
