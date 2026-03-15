# WikiHub — Complete Client Setup Guide

This guide covers everything from purchasing WikiHub to having it fully running on your server.
It is written for the **site owner** — the person who purchases and administers the WikiHub installation.

---

## Table of Contents

1. [Purchase WikiHub](#1-purchase-wikihub)
2. [Receive Your License Credentials](#2-receive-your-license-credentials)
3. [Server Requirements](#3-server-requirements)
4. [Download WikiHub](#4-download-wikihub)
5. [Upload Files to Your Server](#5-upload-files-to-your-server)
6. [Run the Web Installer](#6-run-the-web-installer)
7. [First Login as Owner](#7-first-login-as-owner)
8. [Initial Configuration](#8-initial-configuration)
9. [Invite Your Team](#9-invite-your-team)
10. [Set Up Google Drive Sync (Optional)](#10-set-up-google-drive-sync-optional)
11. [Ongoing Administration](#11-ongoing-administration)
12. [Server Migration](#12-server-migration)
13. [Troubleshooting](#13-troubleshooting)

---

## 1. Purchase WikiHub

### Step 1 — Visit the purchase page

Go to **https://wpfoss.ke/wikihub/** and click **Get Lifetime Access Now**.

You will be taken to **https://wpfoss.ke/wikihub/buy.php**.

> The page shows two plans — **Cloud** (hosted by WPFoss, monthly) and **Self-Hosted** (your server, one-time). This guide covers the **Self-Hosted** option.

### Step 2 — Choose your payment method

The purchase form has two payment tabs:

---

#### Option A — Pay by Card (Stripe)

1. Click the **Card / Stripe** tab
2. Enter the email address where you want to receive your license key
3. Click **Buy WikiHub →**
4. You are redirected to Stripe's secure checkout — fill in your card details and click **Pay**

> Stripe handles all card processing. WikiHub never sees your card details.

---

#### Option B — Pay by M-Pesa

1. Click the **M-Pesa** tab
2. Enter your **M-Pesa phone number** (format: `0712 345 678`)
3. Enter your **email address** — your license key will be sent here
4. Click **Pay KES 5,000 via M-Pesa →**
5. You will receive an **STK Push** on your phone — a pop-up asking you to enter your M-Pesa PIN
6. Enter your PIN to confirm payment
7. The page polls automatically and redirects to the confirmation page once payment is confirmed

> If you do not receive the STK Push within 30 seconds, check that your phone number is correct and try again. Ensure you have sufficient M-Pesa balance before initiating.

---

### Step 3 — Purchase confirmation

After successful payment you will be redirected to:
**https://wpfoss.ke/wikihub/purchase-success.php**

This page confirms your payment was received and explains that your license email is on its way.

---

## 2. Receive Your License Credentials

Within a few minutes of payment, check the inbox of the email you provided. The email:

- **From:** `noreply@wpfoss.com`
- **Subject:** something like "Your WikiHub License"

The email contains two critical credentials:

| Credential | Example | Purpose |
|---|---|---|
| **License Key** | `C1B6DE-39A6E3-8A2B1D-...` | Identifies your purchase — entered during install |
| **License Token** | `key/abc123...` | Authenticates your key — entered during install |

**Save both credentials in a secure location** (password manager recommended).
You will need them during installation and if you ever migrate to a new server.

> **Did not receive the email?** Check your spam/junk folder first. If it is not there, visit
> **https://wpfoss.ke/wikihub/resend.php**, enter your purchase email, and click **Resend License Email**.
> If still no email, contact **hello@wpfoss.com**.

---

## 3. Server Requirements

WikiHub is a self-hosted application. You need a web server with:

| Requirement | Minimum |
|---|---|
| PHP | 8.0 or higher |
| MySQL / MariaDB | 5.7+ / 10.3+ |
| Apache | 2.4+ with `mod_rewrite` and `mod_headers` |
| HTTPS | SSL certificate required (Let's Encrypt is free) |
| PHP extensions | `pdo`, `pdo_mysql`, `openssl`, `curl`, `json`, `mbstring` |
| Disk space | 50 MB minimum |
| Hostinger | Works on shared hosting (confirmed on hPanel) |

> **Important:** WikiHub must be installed at a domain with a valid SSL certificate (`https://`).
> The license fingerprint is tied to your `HTTP_HOST` and `SERVER_ADDR` — plain `http://` installs will work but migration later may require admin assistance.

---

## 4. Download WikiHub

The purchase confirmation email contains a download link pointing to:
**https://github.com/wpfosshq/wikihub-releases/releases/latest**

On the releases page, download **`wikihub-install.zip`** — this is the all-in-one first-install bundle.

| File | When you need it |
|---|---|
| **`wikihub-install.zip`** | **First-time install** — contains everything (React, PHP, database schema, installer) |
| `php-server.zip` | Used automatically by the dashboard updater — you never download this manually |
| `react-build.zip` | Used automatically by the dashboard updater — you never download this manually |
| `migrations.sql` | Used automatically by the dashboard updater — you never download this manually |

---

## 5. Upload Files to Your Server

### Using Hostinger hPanel File Manager

1. Log in to [hpanel.hostinger.com](https://hpanel.hostinger.com)
2. Go to **Hosting** → your hosting plan → **File Manager**
3. Navigate to **`public_html/`** (or a subdirectory if installing in a subfolder, e.g. `public_html/wiki/`)
4. Click **Upload** → select `wikihub-install.zip`
5. After upload, right-click the ZIP → **Extract** → extract to the current folder (`public_html/`)
6. Delete `wikihub-install.zip` from the server after extraction (optional but tidy)

> **Subdirectory install:** If you want WikiHub at `https://yourdomain.com/wiki/`, navigate to `public_html/wiki/` before uploading and extracting.

### Verify the file structure

After extraction, your `public_html/` should look like:

```
public_html/
├── index.html
├── .htaccess
├── static/
│   ├── css/
│   └── js/
├── database/
│   └── schema.sql
└── api/
    ├── index.php       ← PHP router
    ├── install.php     ← web installer
    ├── .htaccess
    ├── src/
    └── vendor/
```

If you see a nested folder (e.g. `public_html/wikihub-install/`), move the contents up one level.

### Create the MySQL database

Before running the installer, create an empty MySQL database:

**In Hostinger hPanel:**
1. Go to **Databases** → **MySQL Databases**
2. Click **Create a New MySQL Database**
3. Enter a name (e.g. `wikihub_db`)
4. Create a database user with a strong password
5. Assign the user to the database with **All Privileges**
6. Note down: database name, username, password, and hostname (usually `localhost`)

---

## 6. Run the Web Installer

Open your browser and visit:

```
https://yourdomain.com/api/install.php
```

The installer runs in four steps:

### Step 0A — Legal Agreements

Three checkboxes appear. **All three must be checked** before you can proceed:

- ☐ I agree to the **Terms of Service** (link opens in new tab)
- ☐ I agree to the **Privacy Policy** (link opens in new tab)
- ☐ I agree to the **WikiHub Source-Available License** (link opens in new tab)

Read each document and check all three boxes. Then click **Continue to License Validation**.

### Step 0B — License Validation

Enter the credentials from your purchase email:

| Field | Value |
|---|---|
| **License Key** | The key from your email (format: `XXXXXX-XXXXXX-...`) |
| **License Token** | The token from your email (starts with `key/`) |

Click **Validate License**.

The installer contacts the WikiHub relay server, validates your credentials against the Keygen license database, and fingerprints your server.

- **Success:** A green "License validated" message appears. Click **Continue to Setup**.
- **Failure:** An error message is shown. Double-check you copied both credentials exactly (no extra spaces).

### Step 1 — Database Configuration

| Field | What to enter |
|---|---|
| **Database Host** | `localhost` (on Hostinger; use `127.0.0.1` only if `localhost` fails) |
| **Database Name** | The database name you created (e.g. `wikihub_db`) |
| **Database User** | The database username |
| **Database Password** | The database password |

Click **Test Connection** to verify before proceeding.

### Step 2 — Application Settings

| Field | What to enter |
|---|---|
| **App URL** | Your full domain URL with no trailing slash (e.g. `https://wiki.yourdomain.com`) |
| **JWT Secret** | Click **Generate** to create a random secret (or enter your own 32+ character string) |
| **Organisation Name** | Your company or team name (shown in the app header) |

### Step 3 — Email (SMTP)

WikiHub uses SMTP to send password reset emails and invite emails to new users.

| Field | What to enter |
|---|---|
| **SMTP Host** | Your mail server host (e.g. `smtp.gmail.com`, `smtp.zoho.com`, `smtp.hostinger.com`) |
| **SMTP Port** | `587` (TLS) or `465` (SSL) — use `587` for Gmail |
| **SMTP Username** | Your email address |
| **SMTP Password** | Your email password or app-specific password |
| **From Email** | The address that WikiHub emails will show as coming from |
| **From Name** | Display name for outgoing emails (e.g. `WikiHub` or your org name) |

> **Gmail:** You must enable 2FA on your Google account and create an App Password at
> `myaccount.google.com/apppasswords`. Use the 16-character App Password as `SMTP Password`.
> Using your regular Gmail password will not work.

### Step 4 — Owner Account

Create the first (owner) account:

| Field | What to enter |
|---|---|
| **First Name** | Your first name |
| **Last Name** | Your last name |
| **Email Address** | Your email (used for login) |
| **Password** | Strong password (min 8 characters) |

Click **Complete Setup**.

### What happens during setup

The installer:
1. Tests the database connection
2. Runs `schema.sql` to create all tables
3. Creates the organisation record
4. Creates your owner account with `ROLE_OWNER`
5. Writes `api/.env` with all configuration values
6. Records the baseline migration version
7. **Self-deletes** — `install.php` is permanently removed from the server

### After installation

You will see a "Setup Complete" screen. Click **Open WikiHub** or navigate to:

```
https://yourdomain.com
```

> **Pre-seeded categories:** Your WikiHub comes with a set of default categories already created — **Onboarding** (with starter links: Mission Statement, Vision, About Us), Operations, Clients, Marketing, Finance, HR & Admin, Templates & Assets, and Compliance & Legal. You can rename, delete, or add to these at any time.

---

## 7. First Login as Owner

1. Navigate to `https://yourdomain.com`
2. Enter the email and password you created in Step 4 of the installer
3. Click **Sign In**

You are now logged in as the **Owner** — the highest-privilege account.

> If you enabled 2FA later (see [Initial Configuration](#8-initial-configuration)), you will also need to enter a 6-digit code from your authenticator app.

---

## 8. Initial Configuration

After first login, take a few minutes to configure your WikiHub instance.

### Organisation Settings

Go to **Settings** → **Organisation** (owner-only tab).

Here you can:
- Update your **Organisation Name**
- Upload a custom **Logo** (displayed in the header and export files)
- Set your **Timezone**

### Two-Factor Authentication (Highly Recommended)

Go to **Settings** → **Security** → **Two-Factor Authentication**.

1. Click **Enable 2FA**
2. A QR code appears — scan it with any TOTP app:
   - **Google Authenticator** (Android/iOS)
   - **Authy** (Android/iOS/Desktop)
   - **Microsoft Authenticator**
   - Any RFC 6238 compatible app
3. Enter the 6-digit code from the app to confirm setup
4. **Save your backup key** (shown below the QR) — this can recover your account if you lose your phone

Future logins will require your password + the 6-digit code.

### SMTP Email Verification

After setup, send yourself a test email:
1. Go to **Settings** → **Email**
2. Click **Send Test Email**
3. Check your inbox — you should receive a test message within a minute

If the test email fails, review your SMTP credentials in Settings or re-run the installer's email step.

### Application Branding

Go to **Settings** → **General**:
- **App Name** — shown in the browser title and emails
- **Support Email** — used in system-generated emails as the reply-to address

---

## 9. Invite Your Team

WikiHub uses invite-only registration — users cannot sign up themselves.

### Create custom roles (optional)

Before inviting, you can create roles that match your organisation's structure:

1. Go to **User Management** (left sidebar, admin users only)
2. Click **Manage Roles** or the roles section
3. Click **+ New Role** → enter a name (e.g. `Marketing`, `Engineering`, `HR`)
4. Save

Three built-in roles always exist:
- **Owner** — you (only one); full access + exclusive Google Drive connect
- **Admin** — full access to all categories and user management (no Drive connect)
- **User** — standard access; can view public categories and any restricted ones they are granted access to

Custom roles are organisation-scoped and can be assigned any combination of category access.

### Invite a user

1. Go to **User Management** → **Invite User**
2. Enter their **email address** and **first/last name**
3. Select a **role** from the dropdown
4. Click **Send Invite**

WikiHub sends an email to the user with a secure setup link (valid for 48 hours).
The user clicks the link, sets their password, and can immediately log in.

> If the invite email fails, the system shows a fallback setup URL you can copy and send manually.

### Manage existing users

From **User Management** you can:
- View all users with their roles and status
- Resend invite emails to users who haven't accepted
- Reset a user's password (sends them a reset email)
- Suspend or remove users
- Change a user's role

---

## 10. Set Up Google Drive Sync (Optional)

WikiHub can mirror your link library to Google Drive — every category becomes a folder, every link becomes a Drive shortcut or text file.

> Drive sync requires the **Owner** role and is configured once per installation.

### Prerequisites

1. A Google account with Google Drive access
2. The WikiHub Google OAuth credentials must be activated for your domain by the relay admin
   - Your domain is automatically registered when you complete the installer
   - The relay admin (WPFoss team) approves it — typically within 1 business day

### Connect Google Drive

1. Go to **Settings** → **Google Drive** (owner-only section)
2. Click **Connect Google Drive**
3. A Google sign-in popup appears — sign in with the Google account that owns your Drive
4. Grant the requested permissions (Drive, Sheets, Docs — scoped to your own files only)
5. After authorization, you are returned to Settings

### Initial sync

1. Click **Start Drive Sync** (in Settings → Google Drive)
2. WikiHub creates a root folder `WikiHub - [Your Org Name]` in your Google Drive
3. Every category is created as a subfolder
4. Every link is created as a Drive shortcut (for Drive URLs) or `.txt` file (for external URLs)

This initial sync is additive — it will never delete anything from Drive.

### Ongoing sync

After setup, Drive stays in sync automatically:
- Creating a category → Drive subfolder is created
- Creating a link → Drive shortcut/file is created
- Renaming/moving → reflected in Drive

Changes made in Drive (adding files, folders) are also detected by the Apps Script polling trigger and can create corresponding entries in WikiHub.

---

## 11. Ongoing Administration

### Keeping WikiHub up to date

When a new version is available, a green banner appears on the Dashboard:
**"WikiHub v1.x.x is available"**

1. Click the banner or go to **Settings** → **Application** → **Check for Updates**
2. Review the release notes
3. Click **Update Now**
4. The updater applies migrations, PHP, and React in sequence — takes about 60 seconds
5. The page reloads automatically when complete

> Updates are applied in-place — your data, settings, and `.env` are never touched.

### Monitoring license validity

Your license is automatically re-validated once every 24 hours in the background.
If validation fails:
- Days 1–3: continues to work, error logged only
- Day 4+: all API routes return HTTP 402 and a red banner appears

Common causes of license failure:
- Server IP address changed (fingerprint mismatch) — contact WPFoss to deactivate the old machine
- License suspended or revoked by the admin — contact WPFoss support

### Announcements

As owner, you can post announcements visible to all users on the Dashboard:

1. Go to **Settings** → **Announcements** (owner-only)
2. Click **New Announcement** → enter title + body
3. Click **Publish**

Users see the announcement as a dismissable banner on their Dashboard. Dismissed announcements do not reappear.

### Category visibility control

Each category has a **visibility** setting:
- **Public** — visible to all users in the org
- **Restricted** — only visible to roles explicitly granted access
- **Draft** — hidden from all users (only admin/owner can see it)

To change: hover over a category card → click the visibility toggle icon.

For restricted categories, click **Manage Access** to choose which roles can see it.

### Activity log

Go to **Settings** → **Activity Log** (admin/owner only) to review an immutable audit trail of all significant actions:
- User logins
- Category and link changes
- User management actions
- Settings changes

---

## 12. Server Migration

If you need to move WikiHub to a new server or domain:

### Step 1 — Export your data

From the old server's WikiHub:
1. Go to **Settings** → **Export / Import**
2. Click **Export** → download the JSON backup

### Step 2 — Set up the new server

Follow sections 3–6 of this guide on the new server.
When you reach the **License Validation** step, the install will fail because your license is bound to the old server's fingerprint.

### Step 3 — Request machine deactivation

Contact **hello@wpfoss.com** with:
- Your license key
- The old server's domain/IP
- The new server's domain

The relay admin will:
1. Deactivate your old machine in the Keygen dashboard
2. Send you a `reregister.php` file with migration instructions

### Step 4 — Register the new machine

1. Upload `reregister.php` to your new server's `api/` directory
2. Open `https://newdomain.com/api/reregister.php` in your browser
3. Click **Register This Server**
4. Your license is now bound to the new server

### Step 5 — Import your data

From the new server's WikiHub:
1. Go to **Settings** → **Export / Import**
2. Click **Import** → select the JSON backup from Step 1
3. All categories and links are restored

---

## 13. Troubleshooting

### Cannot connect to database during install

- Verify the database name, username, and password exactly
- On Hostinger, use `localhost` (not `127.0.0.1`)
- Ensure the database user has been granted all privileges on the database
- Check the database is on the same server as the web hosting (shared hosting restriction)

### License validation fails during install

- Check that you copied the License Key and License Token exactly as received — no leading/trailing spaces
- Verify the License Token starts with `key/`
- Ensure your server has outbound internet access (most shared hosting does)
- Contact **hello@wpfoss.com** with your license key if the problem persists

### Emails not being sent

- Verify your SMTP credentials in **Settings** → **Email** and try the test email button
- Gmail: ensure you are using a 16-character App Password, not your regular Gmail password
- Hostinger: try using Hostinger's own SMTP relay (`smtp.hostinger.com`, port 587)
- Check the email is not in the recipient's spam folder

### "Access denied" or 403 errors after upload

- Ensure file permissions are correct — PHP files should be `644`, directories `755`
- Hostinger File Manager → right-click file → **Change Permissions**

### White screen / 500 error

- Check PHP error logs (Hostinger hPanel → **PHP Configuration** → **Error Logs**)
- Ensure all files extracted correctly — `api/vendor/` must be present
- Verify PHP version is 8.0 or higher (hPanel → **PHP Version**)

### Install.php shows after previous run

`install.php` self-deletes on successful completion. If it still shows, the previous run may have failed partway through. Re-run it — it is safe to run multiple times before the `.env` is written.

### License error banner appears in WikiHub

Go to **Settings** → **License** to see the last validation result. If the fingerprint has changed (common after server IP changes), contact **hello@wpfoss.com** to deactivate the old machine and re-register.

---

*For additional support: **hello@wpfoss.com***
*WikiHub documentation: **https://wpfoss.ke/wikihub/***
