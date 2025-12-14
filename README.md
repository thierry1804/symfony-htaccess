# symfony-htaccess

A small collection of Apache .htaccess rules and examples tuned for Symfony applications. This repository provides sensible defaults to serve Symfony projects (especially when the `/` directory is used as the document root), optional secure defaults (HTTPS/HSTS), and common performance & security headers.

This README explains how to use the files, common configuration notes, and troubleshooting tips.

## Contents

- Example `.htaccess` for Symfony front controller (redirects to `index.php`, serves static assets directly)
- Optional snippets for:
  - Redirecting all traffic to HTTPS and enabling HSTS
  - Security headers (CSP, X-Frame-Options, X-Content-Type-Options)
  - Caching/Expires headers for static assets

## Requirements

- Apache 2.4+ (or compatible)
- Enabled modules:
  - `mod_rewrite`
  - `mod_headers` (for security headers)
  - `mod_expires` (for caching headers)
  - `mod_env` (optional, for environment flags)
- `AllowOverride All` (or at least `AllowOverride FileInfo Indexes Limit Options`) for the directory serving your Symfony `public/` folder so `.htaccess` files are respected.

## Installation

1. Place the chosen `.htaccess` file into your Symfony `/` directory (or whichever directory is your document root):
   - For example, copy `htaccess.symfony` (or `public.htaccess`) to `project-root/.htaccess`.

2. Ensure the Apache configuration for your site allows overrides:
   - Example Apache vhost snippet:
     <VirtualHost *:80>
       ServerName example.com
       DocumentRoot /var/www/project
       <Directory /var/www/project>
         AllowOverride All
         Require all granted
       </Directory>
     </VirtualHost>

3. Restart or reload Apache:
   - sudo systemctl restart apache2
   - or: sudo apachectl graceful

## Usage

Below is a minimal recommended `.htaccess` suitable for modern Symfony apps (placed in `/.htaccess`):

```apache
# Enable rewrite engine
<IfModule mod_rewrite.c>
  RewriteEngine On

  # If the requested filename exists as a file or directory, serve it directly.
  RewriteCond %{REQUEST_FILENAME} -f [OR]
  RewriteCond %{REQUEST_FILENAME} -d
  RewriteRule ^ - [L]

  # Otherwise forward request to front controller
  RewriteRule ^ index.php [QSA,L]
</IfModule>

# Deny access to sensitive files
<FilesMatch "(^\.env|composer\.json|composer\.lock|README|LICENSE)">
  Require all denied
</FilesMatch>
```

Security & HTTPS snippet (optional):

```apache
# Redirect to HTTPS
<IfModule mod_rewrite.c>
  RewriteCond %{HTTPS} !=on
  RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</IfModule>

# HSTS (1 year, include subdomains)
<IfModule mod_headers.c>
  Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
</IfModule>
```

Static caching snippet (optional):

```apache
<IfModule mod_expires.c>
  ExpiresActive On
  ExpiresByType image/jpeg "access plus 1 year"
  ExpiresByType image/png  "access plus 1 year"
  ExpiresByType text/css   "access plus 1 month"
  ExpiresByType application/javascript "access plus 1 month"
</IfModule>
```

Adapt and combine snippets to match your security and caching policy.

## Notes & Best Practices

- Preferred setup: configure your virtual host so `DocumentRoot` points to the directory that contains your front controller (`index.php`). This is more secure and avoids relying on `.htaccess` for directory traversal protection.
- If possible, avoid long `.htaccess` files — prefer site-wide Apache config for performance.
- Restrict access to environment and configuration files; never expose `.env` or project metadata.
- Test redirects and rewrites locally before deploying to production.

## Troubleshooting

- 404s for routes that should be handled by Symfony: Check that `mod_rewrite` is enabled and `AllowOverride` is set so `.htaccess` is read.
- 500 Internal Server Error after adding rules: Check Apache error logs (`/var/log/apache2/error.log` or equivalent) for syntax or module-related errors.
- HTTPS redirect loops: Ensure your load balancer or reverse proxy sets `X-Forwarded-Proto` and that Apache has `RequestHeader set X-Forwarded-Proto expr=%{REQUEST_SCHEME}` or handle TLS at the proxy level.

## Contributing

Contributions and improvements are welcome. Typical contributions include:

- Better default rules for newer Symfony versions
- Additional security or performance snippets
- Test cases or documented examples for different Apache setups

Please open a GitHub Issue or submit a Pull Request.

## License

MIT — see the LICENSE file for details.

## Author / Maintainer

- Thierry (GitHub: @thierry1804)

If you need a custom `.htaccess` for a specific Symfony version or hosting environment, open an issue describing your environment (Apache version, Symfony version, whether you use a reverse proxy or load balancer) and I'll help craft a tailored configuration.
