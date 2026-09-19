# Contributing to Opsora SRE

Thank you for your interest in contributing to **Opsora SRE**! We welcome contributions from developers, SREs, designers, and technical writers worldwide.

Whether you're fixing a bug, adding new telemetry probes, refining the mobile UI, or polishing documentation, this guide will help you get started.

---

## Code of Conduct

All contributors and maintainers are expected to adhere to our [Code of Conduct](CODE_OF_CONDUCT.md). Please treat everyone with respect, kindness, and empathy.

---

## How Can You Contribute?

- **Report Bugs**: Submit reproducible issues using our [Bug Report Template](.github/ISSUE_TEMPLATE/bug_report.md).
- **Propose Features**: Suggest new capabilities, dashboards, or integrations via our [Feature Request Template](.github/ISSUE_TEMPLATE/feature_request.md).
- **Submit Pull Requests**: Fix bugs, add automated tests, or implement roadmap items.
- **Improve Documentation**: Clarify architectural guides, API specs, and runbooks.
- **Improve Mobile Experience**: Enhance the Flutter app for Android, iOS, or desktop.

---

## Development Workflow

### Prerequisites
- **PHP 8.2+** with extensions (`pdo`, `pdo_pgsql`, `pdo_mysql`, `mbstring`, `bcmath`, `gd`, `zip`)
- **Composer 2.x**
- **Node.js 18+** & **npm**
- **Flutter 3.24+** (for mobile development)
- **Git**

### Setting Up the Web Backend
1. **Fork and Clone**:
   ```bash
   git clone https://github.com/YOUR-USERNAME/opsora-saas.git
   cd opsora-saas
   ```
2. **Install Dependencies**:
   ```bash
   composer install
   npm install && npm run build
   ```
3. **Environment Setup**:
   ```bash
   cp .env.example .env
   php artisan key:generate
   ```
4. **Database & Migrations**:
   ```bash
   # Touch SQLite file or configure MySQL / PostgreSQL in .env
   touch database/database.sqlite
   php artisan migrate --seed
   ```
5. **Start Dev Server**:
   ```bash
   php artisan serve
   ```

### Setting Up the Mobile App
1. **Navigate to the Flutter project**:
   ```bash
   cd npontu_sre_mobile
   flutter pub get
   ```
2. **Run Tests**:
   ```bash
   flutter test
   ```
3. **Run on Desired Target**:
   ```bash
   flutter run -d chrome
   # or
   flutter run -d windows
   ```

---

## Coding Standards & Quality Gates

Before opening a pull request, please ensure your changes pass all quality checks:

1. **Code Style (Pint - PSR-12)**:
   ```bash
   ./vendor/bin/pint --test
   # Fix any issues automatically:
   ./vendor/bin/pint
   ```
2. **Backend Unit & Feature Tests (Pest)**:
   ```bash
   ./vendor/bin/pest
   ```
3. **Flutter Static Analysis**:
   ```bash
   cd npontu_sre_mobile
   flutter analyze
   ```

---

## Architectural Principles

Please adhere to these key architectural conventions when writing code:

- **Thin Controllers**: Controllers should only validate input via Form Requests, delegate business logic to Action or Service classes, and return responses.
- **Action Classes (`app/Actions/`)**: Keep business operations single-responsibility and testable in isolation.
- **Form Requests**: All request validation must reside in dedicated Form Request classes.
- **Policies**: All authorization checks must use Laravel Policies.
- **Audit Trails**: Any user-facing state mutation (creation, update, status change, deletion) must log an audit trail entry via `AuditService`.
- **Tenant Isolation**: All tenant-scoped models must apply `TenantScope` and assign `workspace_id`.

---

## Commit Guidelines

We use **Conventional Commits**:
- `feat: add real-time webhook dispatcher`
- `fix: resolve race condition in shift handover sign-off`
- `docs: update API documentation for v1.3.0`
- `test: add edge-case coverage for token revocation`
- `refactor: extract telemetry calculation into dedicated pipeline`
- `chore: update dependencies and build scripts`

---

## Submitting Pull Requests

1. Create a feature branch (`git checkout -b feat/your-feature-name`).
2. Make your commits following conventional commit messages.
3. Run test suites locally (`./vendor/bin/pest` & `flutter test`).
4. Push your branch to your fork (`git push origin feat/your-feature-name`).
5. Open a Pull Request against the `main` branch with a clear description of your changes.

Thank you for helping make Opsora SRE the premier open-source reliability platform!
