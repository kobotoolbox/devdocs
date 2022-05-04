---
title: KoboToolbox Configuration
---

This page documents configuration environment variables which may be useful when configuring in a declarative manner. An alternative is to use [kobo-install](github.com/kobotoolbox/kobo-install) for an interactive installation.

# Environment Variables

## KPI

**Required**

- DJANGO_SECRET_KEY - Set to a random 50 character string

**Optional**

- GOOGLE_ANALYTICS_TOKEN
- RAVEN_JS_DSN

### Security Headers

HTTP Strict Transport Security (HSTS). Refer to [Django documentation](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SECURE_HSTS_SECONDS)

- SECURE_HSTS_INCLUDE_SUBDOMAINS
- SECURE_HSTS_PRELOAD
- SECURE_HSTS_SECONDS

Content Security Policy (CSP). Refer to [django-csp](https://django-csp.readthedocs.io/en/latest/)

- CSP_DEFAULT_SRC - Defaults to known safe values for kpi. If you add a third party script, add the domain here.
- CSP_REPORT_URI
- CSP_REPORT_ONLY - Defaults to False

### Storage

KPI supports local storage as well as django-storages supported backends. Refer to [django-storages documentation](https://django-storages.readthedocs.io/en/latest/) for more details.

#### AWS S3

- KPI_DEFAULT_FILE_STORAGE - `storages.backends.s3boto3.S3Boto3Storage`
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- AWS_STORAGE_BUCKET_NAME
- KOBOCAT_DEFAULT_FILE_STORAGE
- KOBOCAT_AWS_STORAGE_BUCKET_NAME

#### Azure Blob

- KPI_DEFAULT_FILE_STORAGE - `storages.backends.azure_storage.AzureStorage`
- AZURE_ACCOUNT_NAME
- AZURE_ACCOUNT_KEY
- AZURE_CONTAINER
- AZURE_URL_EXPIRATION_SECS - Defaults to None
- KOBOCAT_DEFAULT_FILE_STORAGE

### Development only

- FRONTEND_DEV_MODE - Defaults to None. Set to "host" to run webpack outside of Docker while in development.
