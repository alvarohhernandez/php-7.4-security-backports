# PHP 7.4 Security Backports

Reproducible Docker builds of **PHP 7.4.33 Apache** with selected minimal upstream security backports applied to the End-of-Life PHP 7.4 branch.

> [!WARNING]
> PHP 7.4 is End-of-Life and no longer receives general security support.
>
> These images **do not make PHP 7.4 generally secure or supported**. They apply specific upstream security backports to PHP 7.4.33 as an emergency mitigation for legacy applications that cannot immediately migrate to a supported PHP release.
>
> Migration to a currently supported PHP version is strongly recommended.

## Available image

```text
ghcr.io/alvarohhernandez/php-7.4-security-backports:7.4.33-apache
```

The image provides PHP 7.4.33 running as an Apache module (`mod_php`), maintaining compatibility with the historical official `php:7.4.33-apache` image.

## Included security backports

The current build includes the upstream PHP security commit:

```text
712e60f9edfd03f0960fc4fc3c2c375f42b90aa2
```

Commit:

```text
Avoid state corruption via bailout
```

It provides the minimal PHP 7.4 mitigation for:

- CVE-2026-109618 / GHSA-w8hw-w9fx-j79w
- CVE-2026-109619 / GHSA-6wmf-4rv5-p3rp

The backport modifies 17 PHP source files and contains both textual and binary Git patch data.

## Provenance

The build is based on the historical official Docker PHP definition:

```text
Repository: docker-library/php
Commit:     b9f17156020c3aef71df681b27684533529347a7
Path:       7.4/bullseye/apache/Dockerfile
```

PHP source:

```text
Version: 7.4.33

Official PHP tag:
php-7.4.33

Tag commit:
dca8d5565b947270a29f232bc54efd9df3b92b94
```

Official PHP source archive:

```text
php-7.4.33.tar.xz

SHA256:
924846abf93bc613815c55dd3f5809377813ac62a9ec4eb3778675b82a27b927
```

The original Docker build verifies both the SHA-256 checksum and PHP source GPG signature before compilation.

## Security patch integrity

The security patch was generated directly from upstream `php/php-src` commit:

```text
712e60f9edfd03f0960fc4fc3c2c375f42b90aa2
```

using:

```bash
git format-patch \
  -1 \
  --stdout \
  --binary \
  712e60f9edfd03f0960fc4fc3c2c375f42b90aa2 \
  > php-7.4.33-CVE-2026-109618-109619.patch
```

Expected patch SHA-256:

```text
c4a4e6638b895c94d9c6bd6401b4e93780111f3405d98ee570db2325f9be23ea
```

The Docker build verifies this checksum before applying the patch.

## Base operating system

The historical PHP 7.4 Apache image is based on Debian 11 Bullseye.

Because Bullseye is archived, this build uses the official Debian archive repositories:

```text
deb http://archive.debian.org/debian bullseye main
deb http://archive.debian.org/debian bullseye-updates main
deb http://archive.debian.org/debian-security bullseye-security main
```

This preserves compatibility with the historical PHP 7.4.33 Apache build environment.

## Build

Clone the repository and build the image:

```bash
docker build \
  --pull \
  --no-cache \
  -t php-7.4-security-backports:7.4.33-apache \
  .
```

Verify PHP:

```bash
docker run --rm \
  php-7.4-security-backports:7.4.33-apache \
  php -v
```

Expected PHP version:

```text
PHP 7.4.33
```

Verify the Apache PHP module:

```bash
docker run --rm \
  php-7.4-security-backports:7.4.33-apache \
  sh -c 'apache2ctl -M 2>&1 | grep -i php'
```

Expected:

```text
php7_module (shared)
```

## Backport verification

One observable change introduced by the upstream security backport is the `max_multipart_body_parts` INI directive.

The original PHP 7.4.33 image does not contain it:

```bash
docker run --rm php:7.4.33-apache \
  php -r 'var_dump(ini_get("max_multipart_body_parts"));'
```

Expected:

```text
bool(false)
```

The patched build contains the directive:

```bash
docker run --rm \
  ghcr.io/alvarohhernandez/php-7.4-security-backports:7.4.33-apache \
  php -r 'var_dump(ini_get("max_multipart_body_parts"));'
```

Expected:

```text
string(2) "-1"
```

This provides an observable runtime indication that the backported source is present in the compiled PHP binary.

## Compatibility verification

The build has been compared with the historical official:

```text
php:7.4.33-apache
```

The following were verified to be identical:

- PHP module list (`php -m`)
- PHP `Configure Command`
- PHP 7.4.33 NTS execution model
- Apache `mod_php` integration

The intention is to minimize behavioral differences from the historical official image while applying the selected security backports.

## Usage

```dockerfile
FROM ghcr.io/alvarohhernandez/php-7.4-security-backports:7.4.33-apache
```

For production deployments, pinning the image by digest is recommended instead of relying only on a mutable tag.

## Scope

This project is intentionally narrow.

It is **not**:

- a maintained fork of PHP 7.4;
- a replacement for upgrading PHP;
- a guarantee that all known PHP 7.4 vulnerabilities have been fixed;
- an extended security support distribution.

It provides reproducible emergency backports for explicitly documented vulnerabilities where upstream PHP changes can be applied to PHP 7.4.33 with minimal compatibility impact.

## License

PHP itself is distributed under the PHP License.

Files derived from the official Docker PHP images retain their applicable upstream licensing.

Any project-specific documentation and supporting files in this repository are provided under the license included in this repository.

## Credits

PHP security fixes are authored and maintained by the PHP project and their respective contributors.

The Docker build definition is derived from the official `docker-library/php` project.

This repository packages those upstream changes into a reproducible PHP 7.4.33 Apache build for legacy migration scenarios.
