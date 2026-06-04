# Repository Guidelines

## Project Structure & Module Organization

This package provides Laravel Nova fields for Spatie Media Library. PHP code lives in `src/`, with field classes in `src/Fields`, HTTP controllers, requests, and resources in `src/Http`, and config in `src/config/nova-media-library.php`. Nova API routes are in `routes/api.php`. Frontend source lives in `resources/js` and `resources/sass`; Vue components are under `resources/js/components`, including field views in `resources/js/components/fields`. Compiled assets are written to `dist/js` and `dist/css`. Documentation images and GIFs are stored in `docs/`.

## Build, Test, and Development Commands

- `composer install`: installs PHP dependencies for local package work.
- `yarn install`: installs JavaScript dependencies from `yarn.lock`.
- `yarn development`: builds unminified Nova assets with Laravel Mix.
- `yarn watch`: rebuilds frontend assets during development.
- `yarn production`: builds production assets into `dist/`.
- `docker compose run --rm node`: builds production assets with the configured Node image.

Run commands from the repository root. When changing Vue, Sass, or `webpack.mix.js`, commit regenerated `dist/` assets with the source changes.

## Coding Style & Naming Conventions

Follow `.editorconfig`: UTF-8, LF line endings, final newline, spaces, and trimmed trailing whitespace. Use 4-space indentation for PHP and general files, 2 spaces for JavaScript, Vue, and YAML. PHP classes use PSR-4 under `Ebess\AdvancedNovaMediaLibrary` and should match file names. Keep Vue component names PascalCase, for example `SingleMedia.vue`, and use descriptive trait names such as `HandlesExistingMediaTrait`.

## Testing Guidelines

No automated test suite is currently included. Validate PHP changes in a Laravel Nova app that installs this package, covering upload, reorder, delete, existing media, conversions, custom properties, and downloads as relevant. Validate frontend changes with `yarn development` or `yarn production` and inspect the Nova field UI. If adding tests, prefer focused PHPUnit or Pest coverage and document the new command here.

## Commit & Pull Request Guidelines

Recent history uses short, imperative or descriptive subjects such as `Bump dependencies for Laravel 13`, `Refactored UI`, and merge commits from pull requests. Keep commits focused and mention framework compatibility when relevant. Pull requests should include a concise summary, linked issue when applicable, manual test notes, and screenshots or GIFs for UI changes. Note any Laravel, Nova, PHP, or Spatie Media Library version impact.

## Security & Configuration Tips

Do not commit application credentials, Nova license secrets, or local host app configuration. Keep package configuration changes in `src/config/nova-media-library.php` generic and document user-facing options in `readme.md`.
