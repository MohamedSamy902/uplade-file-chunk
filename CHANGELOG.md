# Changelog

All notable changes to `mohamedsamy902/advanced-file-upload` are documented here.

This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html) and [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [2.0.0] — 2026-06-03

### ⚠ Breaking Changes

- **Minimum PHP raised to `^8.1`** — readonly classes and enums are now used throughout.
- **Minimum Laravel raised to `^10`** — older versions are no longer tested or supported.
- **`intervention/image` upgraded from `^2.7` to `^3.0`** — the v2 `Image::make()` API is gone; all image processing now uses the v3 `ImageManager::gd()->read()` API.
- **`pion/laravel-chunk-upload` is now optional** — moved from `require` to `suggest`. The package no longer hard-depends on it. Chunked uploads are handled natively via `ResumableUploadService`.
- **`FileUploadService::upload()` returns `UploadResult`** (a typed value object) instead of a plain array. `UploadResult` implements `ArrayAccess` for full backward compatibility: existing `$result['path']` code continues to work unchanged.

---

### Added

#### Architecture
- `src/ValueObjects/UploadResult.php` — typed readonly value object returned by every upload. Implements `ArrayAccess` and `JsonSerializable` for backward compatibility.
- `src/ValueObjects/QuotaInfo.php` — readonly value object for quota usage data.
- `src/Services/StorageManager.php` — dedicated service for disk persistence, CDN URL rewriting, and thumbnail storage.
- `src/Services/FileValidator.php` — decoupled validation service with per-type MIME rules.
- `src/Services/ImageProcessor.php` — full rewrite using Intervention/Image v3 (`ImageManager::gd()->read()`). Supports resize, watermark, format conversion (WebP/JPG/PNG), and quality control.
- `src/Services/MimeTypeResolver.php` — centralized MIME-to-extension and MIME-to-type lookup with Symfony MIME fallback.
- `src/Services/UrlDownloader.php` — memory-efficient URL download with retry logic, Content-Type validation, and max-size enforcement.
- `src/Services/QuotaManager.php` — per-user (or per-tenant) storage quota backed by live DB sum queries.
- `src/Services/ResumableUploadService.php` — native resumable/chunked upload engine using an `upload_sessions` database table. No third-party dependency required.
- `src/Security/SsrfValidator.php` — blocks private CIDRs (RFC 1918, RFC 6598, link-local), loopback, AWS EC2 metadata endpoint (`169.254.169.254`), and non-HTTP schemes (`file://`, `ftp://`, `gopher://`, `data:`).
- `src/Models/UploadSession.php` — Eloquent model for resumable upload session state.
- `database/migrations/create_upload_sessions_table.php` — migration for the resumable session store.

#### Contracts (SOLID interfaces)
- `src/Contracts/FileUploadContract.php`
- `src/Contracts/ImageProcessorContract.php`
- `src/Contracts/QuotaManagerContract.php`
- `src/Contracts/SsrfValidatorContract.php`

#### Custom Exceptions
- `src/Exceptions/QuotaExceededException.php`
- `src/Exceptions/SsrfException.php`
- `src/Exceptions/MimeTypeMismatchException.php`
- `src/Exceptions/InvalidFilenameException.php`

#### Tests (140 total — up from 33)
- `tests/Feature/SecurityAttackTest.php` — 26 scenarios: SSRF vectors, disguised files, DoS attempts.
- `tests/Feature/PerformanceTest.php` — single/batch benchmarks and correctness suite.
- `tests/Feature/ErrorHandlingTest.php` — 17 edge cases and exception message coverage.
- `tests/Unit/SsrfValidatorTest.php` — 21 unit tests for the SSRF validator.
- `tests/Unit/UploadResultTest.php` — 15 unit tests for the value object.
- `tests/Unit/ImageProcessorTest.php` — 11 unit tests for the image processor.

#### CI/CD
- `.github/workflows/tests.yml` — GitHub Actions matrix: PHP 8.1–8.4 × Laravel 10–13 with Codecov coverage upload.

#### Documentation
- `README.md` — complete professional rewrite with badges, quick-start, API reference, security section, and architecture overview.
- `UPDATES_AR.md` — full Arabic changelog.

---

### Changed

- `FileUploadService` is now a thin orchestrator — all business logic delegated to dedicated services.
- `handleChunkedReceiver()` type-erased to `object` to remove the hard import of `pion/laravel-chunk-upload` classes.
- `config/file-upload.php` — added `url_upload` section (SSRF domain allowlist, timeout, max size), `chunked.session_ttl_hours`, and `logging` section.
- All services bound to contract interfaces in the service provider, allowing full swap via `AppServiceProvider::bind()`.

---

### Fixed

- Image processing no longer crashes when `intervention/image` v3 is installed — the entire image pipeline was rewritten for the v3 API.
- `ResumableUploadService::writeChunkToDisk()` now uses `copy()` instead of `move()` to preserve the source temporary file, preventing test-environment failures when the same `UploadedFile` instance is reused.
- CDN URL rewriting now correctly applies to thumbnail URLs, not just the primary file URL.

---

### Removed

- `intervention/image ^2.7` hard dependency — replaced by `^3.0`.
- `pion/laravel-chunk-upload` hard dependency — replaced by native `ResumableUploadService`.
- Direct `Image::make()` calls (Intervention v2 API) — replaced by `ImageManager::gd()->read()` (v3 API).

---

## [1.0.0] — 2024-01-01

### Added
- Initial release.
- Chunked uploads via `pion/laravel-chunk-upload`.
- Image processing via `intervention/image ^2.7` (`Image::make()` API).
- Multi-cloud storage: S3 and GCS.
- URL download and storage.
- Quota management.
- CDN URL rewriting.
- Database metadata tracking (`file_uploads` table).
- 33 tests / 61 assertions.

---

[2.0.0]: https://github.com/MohamedSamy902/uplade-file-chunk/compare/v1.0.0...v2.0.0
[1.0.0]: https://github.com/MohamedSamy902/uplade-file-chunk/releases/tag/v1.0.0
