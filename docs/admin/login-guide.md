# Platform Administrator Authentication & Login Guide

> **Audience:** DevOps Engineers, Platform Administrators, Security Auditors  
> **Environment:** Local Development & Staging

## Overview

Platform administrators authenticate using the standard Opsora secure authentication pipeline. Upon successful credential verification, the system inspects the user's `platform_role` column. If the user possesses an authorized platform role, they are granted access to `/admin/platform` and `/api/v1/platform/*`.

---

## Platform Role Hierarchy & Access Scope

Opsora SRE provides granular control plane separation for operational integrity:

| Platform Role | Enum Value | Access Scope & Responsibilities |
|---|---|---|
| **Root Super Admin** | `super_admin` | Unrestricted control plane access, tenant provisioning, system feature flags, kill switches |
| **Billing Admin** | `billing_admin` | Commercial subscriptions, SaaS pricing tiers, revenue reporting, plan upgrades |
| **Security Admin** | `security_admin` | SIEM events, suspicious activity alerts, operator suspensions, cryptographic key rotation |
| **Compliance Auditor** | `auditor` | Read-only regulatory audit trail access, compliance export verification |

---

## Provisioning Initial Administrative Accounts

In production and staging environments, administrative accounts are provisioned via:

1. **Self-Service Registration**: Sign up through `/register` to create an initial workspace and owner account.
2. **Platform Promotion (CLI)**: Designate any registered user as a Super Administrator using Laravel Artisan:
   ```bash
   php artisan tinker --execute="\$user = App\Models\User::where('email', 'admin@your-organization.com')->firstOrFail(); \$user->update(['role' => 'admin', 'platform_role' => App\Enums\PlatformRole::SuperAdmin->value]);"
   ```

---

## Login Flow

1. Navigate to `/login` or `/admin`.
2. Entering `/admin` as an unauthenticated visitor redirects to `/login`.
3. If an authenticated user without a `platform_role` navigates to `/admin`, the application returns an HTTP `403 Forbidden` response and logs a `security_events` warning record.
4. If an authorized administrator logs in, navigating to `/admin` automatically redirects to `/admin/platform`.

---

## Password Reset & Emergency Recovery

Administrators can trigger secure password reset links via `/forgot-password` (sent via configured SMTP/SES mailer), or reset credentials directly via Artisan CLI:

```bash
php artisan tinker --execute="\$user = App\Models\User::where('email', 'admin@your-organization.com')->firstOrFail(); \$user->update(['password' => Hash::make('YourNewSecurePassphrase2026!')]);"
```
