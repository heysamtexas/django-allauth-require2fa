# Django Require 2FA

[![PyPI version](https://badge.fury.io/py/django-require2fa.svg)](https://badge.fury.io/py/django-require2fa)
[![Test](https://github.com/your-org/django-require2fa/actions/workflows/test.yml/badge.svg)](https://github.com/your-org/django-require2fa/actions/workflows/test.yml)
[![codecov](https://codecov.io/gh/your-org/django-require2fa/branch/main/graph/badge.svg)](https://codecov.io/gh/your-org/django-require2fa)
[![Python versions](https://img.shields.io/pypi/pyversions/django-require2fa.svg)](https://pypi.org/project/django-require2fa/)
[![Django versions](https://img.shields.io/pypi/djversions/django-require2fa.svg)](https://pypi.org/project/django-require2fa/)

A production-ready Django package that enforces Two-Factor Authentication (2FA) across your entire application.

## Why This Exists

### The Django-Allauth Gap

Django-allauth provides excellent 2FA functionality, but **intentionally does not include** site-wide 2FA enforcement. This decision was made explicit in:

- **[PR #3710](https://github.com/pennersr/django-allauth/pull/3710)** - A middleware approach was proposed but **rejected** by the maintainer
- **[Issue #3649](https://github.com/pennersr/django-allauth/issues/3649)** - Community discussed various enforcement strategies, issue was **closed without implementation**

The django-allauth maintainer's position:
> "leave such functionality for individual projects to implement"

### The Enterprise Need

Many organizations need to:
- Enforce 2FA for **all users** (not optional)
- Configure 2FA requirements **at runtime** (not hardcoded)
- Support **SaaS/multi-tenant** scenarios with organization-level policies
- Maintain **audit trails** of security configuration changes

Since django-allauth won't provide this, there's a clear market need for a standalone solution.

## Our Solution

### What We Built

This package provides what the **rejected django-allauth PR attempted**, but with significant improvements:

| Feature | Rejected PR #3710 | Our Implementation |
|---------|------------------|-------------------|
| URL Matching | String prefix matching (vulnerable) | Proper Django URL resolution |
| Configuration | Hardcoded settings | Runtime admin configuration |
| Testing | Basic tests | 15 comprehensive security tests |
| Security | Known vulnerabilities | Production-hardened |
| Admin Protection | Exempt admin login | Proper 2FA for admin access |

### Key Security Features

- **Vulnerability Protection**: Fixed Issue #173 path traversal attacks
- **URL Resolution**: Uses Django's proper URL resolution instead of dangerous string matching
- **Configuration Validation**: Prevents dangerous Django settings misconfigurations
- **Comprehensive Testing**: 15 security tests covering edge cases, malformed URLs, and regression scenarios
- **Admin Security**: Removed admin login exemption (admins now require 2FA)

## Installation

Install from PyPI:

```bash
pip install django-require2fa
```

Or with uv (recommended):

```bash
uv add django-require2fa
```

## Quick Start

1. Add `require2fa` to your `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # ...
    'django.contrib.admin',
    'allauth',
    'allauth.account',
    'allauth.mfa',
    'solo',
    'require2fa',  # Add this
    # ...
]
```

2. Add the middleware to your `MIDDLEWARE` setting:

```python
MIDDLEWARE = [
    # ... other middleware
    'require2fa.middleware.Require2FAMiddleware',  # Add this
]
```

3. Run migrations:

```bash
python manage.py migrate require2fa
```

4. Configure in Django Admin:
   - Go to **Django Admin** → **Two-Factor Authentication Configuration**
   - Check **"Require Two-Factor Authentication"**
   - Changes take effect immediately (no restart required)

## How It Works

1. **Authenticated users** without 2FA are redirected to `/accounts/2fa/`
2. **Exempt URLs** (login, logout, 2FA setup) remain accessible
3. **Static/media files** are automatically detected and exempted
4. **Admin access** requires 2FA verification (security improvement)

## Configuration

### Runtime Configuration

Visit Django Admin → **Two-Factor Authentication Configuration**:
- **Require Two-Factor Authentication**: Toggle 2FA enforcement site-wide
- Changes take effect immediately (no server restart required)

### Architecture

- **Django-Solo Pattern**: Runtime configuration via admin interface
- **Middleware Approach**: Site-wide enforcement without code changes
- **Allauth Integration**: Works seamlessly with django-allauth's MFA system
- **Production Ready**: Data migrations, backward compatibility, zero downtime

## Requirements

- **Python**: 3.12+
- **Django**: 4.2+
- **django-allauth**: 0.57.0+
- **django-solo**: 2.0.0+

## Testing

This package includes 15 comprehensive security tests covering:
- URL resolution edge cases
- Malformed URL handling
- Static file exemptions
- Admin protection
- Configuration security
- Regression tests for known vulnerabilities

```bash
# Run tests
python manage.py test require2fa

# Or with pytest
pytest require2fa/tests/
```

## Security

This middleware is **security-critical**. Key protections include:

1. **Path Traversal Protection**: Fixed vulnerability where `/accounts/../admin/` could bypass 2FA
2. **URL Resolution**: Uses Django's URL dispatcher, not string matching
3. **Configuration Validation**: Prevents dangerous Django settings that would bypass security
4. **Comprehensive Testing**: 15 security-focused tests ensure no regressions
5. **Security Logging**: Uses dedicated `security.2fa` logger for audit trails

## Contributing

Contributions are welcome! Please ensure:

1. **Security first** - any middleware changes need security review
2. **Comprehensive testing** - maintain the 15-test security suite
3. **Backward compatibility** - consider migration paths
4. **Code quality** - use pre-commit hooks (`pre-commit install`)

### Development Setup

```bash
# Clone the repository
git clone https://github.com/your-org/django-require2fa.git
cd django-require2fa

# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync --dev

# Install pre-commit hooks
uv run pre-commit install

# Run tests
uv run pytest

# Run code quality checks
uv run ruff check .
uv run ruff format .
uv run bandit -r require2fa/
```

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

---

**Note**: This package fills the gap left by django-allauth's intentional decision not to include site-wide 2FA enforcement. It provides enterprise-ready security features that many organizations require.