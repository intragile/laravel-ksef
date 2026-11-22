# Laravel KSeF / lengyel e-számlázás

Fejlesztő és karbantartó: [IntrAgile Kft.](https://intragile.eu)

[![Install](https://github.com/intragile/laravel-ksef/actions/workflows/ci.yml/badge.svg)](https://github.com/intragile/laravel-ksef/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/packagist/v/intragile/laravel-ksef.svg)](https://packagist.org/packages/intragile/laravel-ksef)
[![PHP](https://img.shields.io/packagist/php-v/intragile/laravel-ksef.svg)](https://packagist.org/packages/intragile/laravel-ksef)
[![License](https://img.shields.io/packagist/l/intragile/laravel-ksef.svg)](LICENSE)

A lengyel **KSeF** (*Krajowy System e-Faktur*) e-számlázás Laravelhez: FA(3) számlák küldése a KSeF 2.0-nak online, offline és kötegelve, fogadásuk ütemezetten, KSeF-tanúsítványok igénylése, KOD I és KOD II QR-kódok nyomtatása.

> **Még nincs kiadva.** A csomaghoz az `intragile/laravel-einvoicing` 1.2 kell, amely még nem jelent meg. Addig a `composer require intragile/laravel-ksef` nem oldható fel.

Ez egy vékony metacsomag: nincs saját kódja, csak a [`intragile/laravel-einvoicing`](https://github.com/intragile/laravel-einvoicing) csomagot és az [`intragile/php-einvoicing`](https://github.com/intragile/php-einvoicing) motort húzza be, hogy a lengyel használati eset név szerint megtalálható legyen. A teljes funkcionalitás a [laravel-einvoicing README](https://github.com/intragile/laravel-einvoicing#polish-ksef) **Polish KSeF** szakaszában van dokumentálva.

[English documentation → README.md](README.md)

## Amit a lengyel folyamat lefed

- **Küldés:** sorba állított FA(3) küldés a KSeF 2.0-nak online, offline módban és kötegelt munkamenetben; a KSeF-szám és a hivatalos átvételi igazolás (UPO) a kimenő soron tárolódik.
- **Fogadás:** a postaláda lekérdezése a KSeF olvasási korlátain belül, a számlák bejövő dokumentumként rögzítve.
- **Offline mód:** offline határidők, és esemény, ha egy határidő közeleg.
- **Tanúsítványok:** KSeF Authentication és Offline tanúsítványok igénylése parancssorból; a privát kulcs 0600 móddal íródik ki, és sosem jelenik meg a kimeneten.
- **QR-kódok:** a KOD I és KOD II ellenőrző linkek és QR-kódjaik.

## Követelmények

- PHP 8.3+ a `dom`, `libxml`, `mbstring` és `openssl` kiterjesztéssel (és `zip` a kötegelt munkamenetekhez)
- Laravel 12 vagy 13
- Opcionális: `chillerlan/php-qrcode` ^5 a QR-kódok kirajzolásához (`composer require chillerlan/php-qrcode`)

## Telepítés

A kiadás után:

```bash
composer require intragile/laravel-ksef
php artisan vendor:publish --tag=einvoicing-config   # config/einvoicing.php
php artisan migrate                                  # kimenő, bejövő és KSeF-kurzor táblák
php artisan einvoicing:install-rules                 # a Peppol szabályok, amelyeket a doctor ellenőriz
php artisan einvoicing:doctor
```

Környezeti kulcsok (az `einvoicing.ksef` beállítást töltik ki):

```dotenv
KSEF_ENVIRONMENT=test                   # test (TE), demo vagy production
KSEF_NIP=5265877635                     # a kontextus, amelyre a kliens hitelesít
KSEF_CERTIFICATE_PATH=/run/secrets/ksef-seal.crt.pem
KSEF_PRIVATE_KEY_PATH=/run/secrets/ksef-seal.key.pem
# KSEF_PRIVATE_KEY_PASSPHRASE=...       # titkosított kulcshoz
# KSEF_TOKEN=...                        # tanúsítvány helyett: KSeF-token
# KSEF_OFFLINE_CERTIFICATE_PATH=...     # KSeF Offline tanúsítvány, a KOD II-t írja alá
# KSEF_OFFLINE_PRIVATE_KEY_PATH=...
```

KSeF Offline tanúsítvány igénylése (tanúsítványos hitelesítés kell hozzá; a kulcs kiíródik, de sosem jelenik meg):

```bash
php artisan einvoicing:ksef-certificate
```

A postaláda-lekérdezés és az offline határidő-ellenőrzés ütemezése:

```php
// routes/console.php
use Illuminate\Support\Facades\Schedule;

Schedule::command('einvoicing:poll-inbound --transport=ksef')->everyFifteenMinutes()->withoutOverlapping();
Schedule::command('einvoicing:ksef-deadlines')->hourly();
```

Kezdd a `KSEF_ENVIRONMENT=test` beállítással: a számlák csak `production` környezetben mennek el élesben. A küldést, a tanúsítványokat, a lekérdezési védelmet és az eseményeket a [Polish KSeF szakasz](https://github.com/intragile/laravel-einvoicing#polish-ksef) írja le; az FA(3) modellt és a KSeF-klienst a [lengyel országjegyzet](https://github.com/intragile/php-einvoicing/blob/main/docs/countries/POLAND.md).

## Miben más

- **[`n1ebieski/ksef-php-client`](https://packagist.org/packages/n1ebieski/ksef-php-client)** általános PHP kliens a KSeF API-hoz. Ez a csomag Laravel-integráció: sor, tárolt sorok, események, ütemezett lekérdezés és artisan parancsok a motor saját kliense körül.
- **[`beziworld/laravel-ksef`](https://packagist.org/packages/beziworld/laravel-ksef)** Laravel csomag a KSeF-hez, amely csak tokennel hitelesít. Ez a csomag KSeF-tanúsítvánnyal is hitelesít, és igényli is azokat.

A két csomag más kódot hoz; ez a metacsomag egyiktől sem függ.

## Amit behúz

`intragile/laravel-einvoicing` (a Laravel-integráció, MIT), amely hozza a `intragile/php-einvoicing` motort (MIT) és a szabálycsomagokat: `intragile/einvoicing-rules-en16931` (EUPL-1.2), `intragile/einvoicing-rules-xrechnung` (Apache-2.0) és `intragile/einvoicing-rules-facturx` (Apache-2.0). Az `intragile/php-einvoicing` közvetlenül is követelmény.

A metacsomag maga MIT licencű és nem szállít kódot; minden behúzott csomag a saját licencét tartja meg.

## Teljes dokumentáció

Teljes dokumentáció: [laravel-einvoicing](https://github.com/intragile/laravel-einvoicing#polish-ksef) és a [lengyel országjegyzet](https://github.com/intragile/php-einvoicing/blob/main/docs/countries/POLAND.md).

## Licenc

MIT. Lásd: [LICENSE](LICENSE).

---

Fejlesztő és karbantartó: [IntrAgile Kft.](https://intragile.eu)
