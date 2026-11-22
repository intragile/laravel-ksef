# Laravel KSeF / Polish e-invoicing

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)

[![Install](https://github.com/intragile/laravel-ksef/actions/workflows/ci.yml/badge.svg)](https://github.com/intragile/laravel-ksef/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/packagist/v/intragile/laravel-ksef.svg)](https://packagist.org/packages/intragile/laravel-ksef)
[![PHP](https://img.shields.io/packagist/php-v/intragile/laravel-ksef.svg)](https://packagist.org/packages/intragile/laravel-ksef)
[![License](https://img.shields.io/packagist/l/intragile/laravel-ksef.svg)](LICENSE)

Polish **KSeF** (*Krajowy System e-Faktur*) e-invoicing for Laravel: send FA(3) invoices to KSeF 2.0 online, offline and in batches, receive them on a schedule, enrol KSeF certificates and print KOD I and KOD II QR codes.

> **Not yet released.** This package needs `intragile/laravel-einvoicing` 1.2, which is not released yet. Until then `composer require intragile/laravel-ksef` cannot be resolved.

This is a thin metapackage: it has no code of its own and pulls in [`intragile/laravel-einvoicing`](https://github.com/intragile/laravel-einvoicing) and the engine [`intragile/php-einvoicing`](https://github.com/intragile/php-einvoicing), so that you can find the Polish use case by its name. The full functionality is documented in the **Polish KSeF** section of the [laravel-einvoicing README](https://github.com/intragile/laravel-einvoicing#polish-ksef).

[Magyar nyelvű leírás → README.hu.md](README.hu.md)

## What the Polish flow covers

- **Sending:** queued FA(3) sending to KSeF 2.0 online and in offline mode, with the KSeF number and the official receipt (UPO) stored on the outbound row. Batch sessions are available on the core client (`EInvoicing::ksef()->transport()->sendBatch()`), outside the queue and without a stored row or receipt.
- **Receiving:** inbox polling within KSeF's read limits, recorded as inbound documents.
- **Offline mode:** offline deadlines, and an event when one is about to expire.
- **Certificates:** enrolling KSeF Authentication and Offline certificates from the command line; the private key is written with mode 0600 and never printed.
- **QR codes:** the KOD I and KOD II verification links and their QR codes.

## Requirements

- PHP 8.3+ with `dom`, `libxml`, `mbstring` and `openssl` (and `zip` for batch sessions on the core client)
- Laravel 12 or 13
- Optional: `chillerlan/php-qrcode` ^5 to render the QR codes (`composer require chillerlan/php-qrcode`)

## Quick start

Once released:

```bash
composer require intragile/laravel-ksef
php artisan vendor:publish --tag=einvoicing-config   # config/einvoicing.php
php artisan migrate                                  # outbound, inbound and KSeF cursor tables
php artisan einvoicing:install-rules                 # the Peppol rules the doctor checks
php artisan einvoicing:doctor
```

Environment keys (they fill `einvoicing.ksef`):

```dotenv
KSEF_ENVIRONMENT=test                   # test (TE), demo or production
KSEF_NIP=5265877635                     # the context the client authenticates for
KSEF_CERTIFICATE_PATH=/run/secrets/ksef-seal.crt.pem
KSEF_PRIVATE_KEY_PATH=/run/secrets/ksef-seal.key.pem
# KSEF_PRIVATE_KEY_PASSPHRASE=...       # for an encrypted key
# KSEF_TOKEN=...                        # instead of a certificate: a KSeF token
# KSEF_OFFLINE_CERTIFICATE_PATH=...     # KSeF Offline certificate, signs KOD II
# KSEF_OFFLINE_PRIVATE_KEY_PATH=...
```

Enrol a KSeF Offline certificate (needs certificate credentials; the key is written, never printed):

```bash
php artisan einvoicing:ksef-certificate
```

Schedule the inbox poll and the offline deadline check:

```php
// routes/console.php
use Illuminate\Support\Facades\Schedule;

Schedule::command('einvoicing:poll-inbound --transport=ksef')->everyFifteenMinutes()->withoutOverlapping();
Schedule::command('einvoicing:ksef-deadlines')->hourly();
```

Start with `KSEF_ENVIRONMENT=test`: invoices are sent for real only when the environment is `production`. Sending, certificates, the poll guard and the events are described in the [Polish KSeF section](https://github.com/intragile/laravel-einvoicing#polish-ksef); the FA(3) model and the KSeF client in the [Poland country notes](https://github.com/intragile/php-einvoicing/blob/main/docs/countries/POLAND.md).

## How it differs

- **[`n1ebieski/ksef-php-client`](https://packagist.org/packages/n1ebieski/ksef-php-client)** is a general PHP client for the KSeF API. This package is a Laravel integration: a queue, stored rows, events, scheduled polling and artisan commands around the engine's own client.
- **[`beziworld/laravel-ksef`](https://packagist.org/packages/beziworld/laravel-ksef)** is a Laravel package for KSeF that authenticates with a token only. This package also authenticates with KSeF certificates and enrols them.

Both pull in different code; this metapackage does not depend on either.

## What it pulls in

`intragile/laravel-einvoicing` (the Laravel bridge, MIT), which brings the engine `intragile/php-einvoicing` (MIT) and the rule packages `intragile/einvoicing-rules-en16931` (EUPL-1.2), `intragile/einvoicing-rules-xrechnung` (Apache-2.0) and `intragile/einvoicing-rules-facturx` (Apache-2.0). `intragile/php-einvoicing` is required directly as well.

The metapackage itself is MIT-licensed and ships no code; each pulled package keeps its own licence.

## Full documentation

Full documentation: [laravel-einvoicing](https://github.com/intragile/laravel-einvoicing#polish-ksef) and the [Poland country notes](https://github.com/intragile/php-einvoicing/blob/main/docs/countries/POLAND.md).

## License

MIT. See [LICENSE](LICENSE).

---

Developed and maintained by [IntrAgile Kft.](https://intragile.eu)
