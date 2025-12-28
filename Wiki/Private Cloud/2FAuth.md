# 🔑 2FAuth

:::warning
Dieses Image verwende ich nicht mehr, stattdessen nutze ich die in Bitwarden integrierte OTP Funktionalität

:::

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/9447b5c1-db38-4aaa-8140-aefa322365d8/2fauth_screenshots_dark.png " =1513x640")


2Fauth ist ein One Time Password (OTP) Generator und ersetzt Client-seitige 2-Faktor Authentifizierungs Applikationen. Es lässt sich zusätzlich als Browser Add-On und auch als PWA installieren.

# Installation

## compose.yml

```yaml
services:
  2fauth:
    image: 2fauth/2fauth:latest
    container_name: 2fauth
    volumes:
      - ./data:/2fauth
    networks:
      - caddy
    restart: unless-stopped
    env_file: ./.env
networks:
  caddy:
    external: true   
```

## .env

```yaml
APP_NAME=2FAuth
APP_ENV=local
APP_TIMEZONE=Europe/Berlin
APP_DEBUG=false
SITE_OWNER=DEINEMAIL
# The encryption key for  our database and sessions. Keep this very secure.
# If you generate a new one all existing data must be considered LOST.
# Change it to a string of exactly 32 chars or use command `php artisan key:generate` to generate it
APP_KEY=DEINKEY
# This variable must match your installation's external address.
# Webauthn won't work otherwise.
APP_URL=DEINEURL
# If you want to serve js assets from a CDN (like https://cdn.example.com),
# uncomment the following line and set this var with the CDN url.
# Otherwise, let this line commented.
# - ASSET_URL=http://localhost
#
# Turn this to true if you want your app to react like a demo.
# The Demo mode reset the app content every hours and set a generic demo user.
IS_DEMO_APP=false
# The log channel defines where your log entries go to.
# 'daily' is the default logging mode giving you 7 daily rotated log files in /storage/logs/.
# Also available are 'errorlog', 'syslog', 'stderr', 'papertrail', 'slack' and a 'stack' channel
# to combine multiple channels into a single one.
LOG_CHANNEL=daily
# Log level. You can set this from least severe to most severe:
# debug, info, notice, warning, error, critical, alert, emergency
# If you set it to debug your logs will grow large, and fast. If you set it to emergency probably
# nothing will get logged, ever.
LOG_LEVEL=notice
# Database config (can only be sqlite)
DB_DATABASE="/srv/database/database.sqlite"
# If you're looking for performance improvements, you could install memcached.
CACHE_DRIVER=file
SESSION_DRIVER=file
# Mail settings
# Refer your email provider documentation to configure your mail settings
# Set a value for every available setting to avoid issue
MAIL_MAILER=log
MAIL_HOST=
MAIL_PORT=
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_NAME=2 Factor Authenticatoin
MAIL_FROM_ADDRESS=
# SSL peer verification.
# Set this to false to disable the SSL certificate validation.
# WARNING
# Disabling peer verification can result in a major security flaw.
# Change it only if you know what you're doing.
MAIL_VERIFY_SSL_PEER=true
# API settings
# The maximum number of API calls in a minute from the same IP.
# Once reached, all requests from this IP will be rejected until the minute has elapsed.
# Set to null to disable the API throttling.
THROTTLE_API=60
# Authentication settings
# The number of times per minute a user can fail to log in before being locked out.
# Once reached, all login attempts will be rejected until the minute has elapsed.
# This setting applies to both email/password and webauthn login attemps.
LOGIN_THROTTLE=5
# The default authentication guard
# Supported:
#   'web-guard' : The Laravel built-in auth system (default if nulled)
#   'reverse-proxy-guard' : When 2FAuth is deployed behind a reverse-proxy that handle authentication
# WARNING
# When using 'reverse-proxy-guard' 2FAuth only look for the dedicated headers and skip all other built-in
# authentication checks. That means your proxy is fully responsible of the authentication process, 2FAuth will
# trust him as long as headers are presents.
AUTHENTICATION_GUARD=web-guard
# Authentication log retention time, in days.
# Log entries older than that are automatically deleted.
AUTHENTICATION_LOG_RETENTION=365
# Name of the HTTP headers sent by the reverse proxy that identifies the authenticated user at proxy level.
# Check your proxy documentation to find out how these headers are named (i.e 'REMOTE_USER', 'REMOTE_EMAIL', etc...)
# (only relevant when AUTHENTICATION_GUARD is set to 'reverse-proxy-guard')
AUTH_PROXY_HEADER_FOR_USER=null
AUTH_PROXY_HEADER_FOR_EMAIL=null
# Custom logout URL to open when using an auth proxy.
PROXY_LOGOUT_URL=null
# WebAuthn settings
# Relying Party name, aka the name of the application. If blank, defaults to APP_NAME. Do not set to null.
WEBAUTHN_NAME=2FAuth
# Relying Party ID, should equal the site domain (i.e 2fauth.example.com).
# If null, the device will fill it internally (recommended)
# See https://webauthn-doc.spomky-labs.com/prerequisites/the-relying-party#how-to-determine-the-relying-party-id
WEBAUTHN_ID=null
# Use this setting to control how user verification behave during the
# WebAuthn authentication flow.
#
# Most authenticators and smartphones will ask the user to actively verify
# themselves for log in. For example, through a touch plus pin code,
# password entry, or biometric recognition (e.g., presenting a fingerprint).
# The intent is to distinguish one user from any other.
#
# Supported:
#   'required': Will ALWAYS ask for user verification
#   'preferred' (default) : Will ask for user verification IF POSSIBLE
#   'discouraged' : Will NOT ask for user verification (for example, to minimize disruption to the user interaction flow)
WEBAUTHN_USER_VERIFICATION=preferred
#### SSO settings (for Socialite) ####
# Uncomment and complete lines for the OAuth providers you want to enable.
# - OPENID_AUTHORIZE_URL=
# - OPENID_TOKEN_URL=
# - OPENID_USERINFO_URL=
# - OPENID_CLIENT_ID=
# - OPENID_CLIENT_SECRET=
# - OPENID_HTTP_VERIFY_SSL_PEER=true
# Can also be the path to a custom certificate on disk, i.e
# - OPENID_HTTP_VERIFY_SSL_PEER=/path/to/cert.pem
#
# - GITHUB_CLIENT_ID=
# - GITHUB_CLIENT_SECRET=
# Use this setting to declare trusted proxied.
# Supported:
#   '*': to trust any proxy
#   A comma separated IP list: The list of proxies IP to trust
TRUSTED_PROXIES=null
# Proxy for outgoing requests like new releases detection or logo fetching.
# You can provide a proxy URL that contains a scheme, username, and password.
# For example, "http://username:password@192.168.16.1:10".
PROXY_FOR_OUTGOING_REQUESTS=null
# Set this to true to enable Content-Security-Policy (CSP).
# CSP helps to prevent or minimize the risk of certain types of security threats.
# This is mainly used as a defense against cross-site scripting (XSS) attacks, in which
# an attacker is able to inject malicious code into the web app
CONTENT_SECURITY_POLICY=false
# Leave the following configuration vars as is.
# Unless you like to tinker and know what you're doing.
BROADCAST_DRIVER=log
QUEUE_DRIVER=sync
SESSION_LIFETIME=120
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1
VITE_PUSHER_APP_KEY=null
VITE_PUSHER_APP_CLUSTER=null
MIX_ENV=local
TZ=Europe/Berlin
```


## Reverse Proxy

Für Caddy:

```yaml
2fa.handtrixxx.com {
	reverse_proxy 2fauth:8000
}
```

# Alternativen

* [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=de)
* …

# Quellen / Links

* <https://docs.2fauth.app>
* <https://github.com/Bubka/2FAuth-WebExtension>