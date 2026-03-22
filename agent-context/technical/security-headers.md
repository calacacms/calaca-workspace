# Security Headers Middleware Documentation

## Overview

The `SecurityHeadersMiddleware` adds comprehensive security headers to all HTTP responses to protect against various attacks.

## Features

### Security Headers Added

| Header | Purpose | Value |
|---------|----------|-------|
| X-Content-Type-Options | Prevents MIME type sniffing | `nosniff` |
| X-Frame-Options | Prevents clickjacking | `DENY` |
| X-XSS-Protection | Enables XSS filter in older browsers | `1; mode=block` |
| Content-Security-Policy | Prevents XSS and injection attacks | Configurable |
| Strict-Transport-Security | Enforces HTTPS (HTTPS only) | `max-age=31536000; includeSubDomains` |
| Referrer-Policy | Controls referrer information sent | `strict-origin-when-cross-origin` |
| Permissions-Policy | Controls browser feature access | Restrictive defaults |
| Cross-Origin-Opener-Policy | Controls sharing of browsing context | `same-origin` |
| Cross-Origin-Resource-Policy | Controls which origins can read response | `same-origin` |

## Usage

### Basic Usage

Add the middleware to your application's middleware stack:

```php
use CalacaCMS\Middleware\SecurityHeadersMiddleware;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

// Create middleware instance
$securityMiddleware = new SecurityHeadersMiddleware();

// Wrap your application with middleware
$response = $securityMiddleware->handle($request, function($request) {
    // Your application logic here
    return new Response('Hello World');
});

// Send response with security headers
$response->send();
```

### Integration with Existing Middleware

If you're using AuthMiddleware, apply SecurityHeadersMiddleware last so all responses get headers:

```php
use CalacaCMS\Middleware\AuthMiddleware;
use CalacaCMS\Middleware\SecurityHeadersMiddleware;

$authMiddleware = new AuthMiddleware();
$securityMiddleware = new SecurityHeadersMiddleware();

// Chain middleware (order matters)
$response = $securityMiddleware->handle($request, function($request) use ($authMiddleware) {
    return $authMiddleware->handle($request, function($request) {
        // Route to controller
        return Bootstrap::getController($controller)->$action(...$params);
    });
});
```

## Customization

### Custom CSP Directives

Modify Content-Security-Policy for your needs:

```php
$middleware = new SecurityHeadersMiddleware();

// Allow Google Fonts
$middleware->setCspDirective('font-src', "'self' https://fonts.gstatic.com");

// Allow analytics scripts
$middleware->setCspDirective('script-src', "'self' https://analytics.example.com");

// Allow images from CDN
$middleware->setCspDirective('img-src', "'self' data: https: https://cdn.example.com");
```

### Custom Headers

Add site-specific security headers:

```php
$middleware = new SecurityHeadersMiddleware();
$middleware->addHeader('X-Custom-Security', 'value');
```

## Content-Security-Policy Default Values

The middleware sets these CSP directives by default:

```
default-src 'self';
script-src 'self' 'unsafe-inline' 'unsafe-eval';
style-src 'self' 'unsafe-inline';
img-src 'self' data: https:;
font-src 'self' data:;
connect-src 'self';
media-src 'self';
object-src 'none';
frame-src 'none';
base-uri 'self';
form-action 'self';
frame-ancestors 'none';
```

**Note:** The default CSP allows inline scripts and styles for compatibility. Consider tightening for production.

## Production Considerations

1. **HTTPS Required:** `Strict-Transport-Security` header only applies on HTTPS connections
2. **CSP Reporting:** Consider adding CSP violation reporting for monitoring:
   ```php
   $middleware->setCspDirective('report-uri', '/csp-report');
   ```
3. **Testing:** Use browser developer tools to verify headers are being sent correctly
4. **Preload:** Add your domain to HSTS preload list after testing (https://hstspreload.org/)

## Testing

Verify headers are set correctly:

```bash
curl -I http://localhost:8000/
```

Expected output:
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'; ...
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), ...
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
```

## References

- [OWASP Secure Headers](https://owasp.org/www-project-secure-headers/)
- [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [Content Security Policy Level 3](https://www.w3.org/TR/CSP3/)
