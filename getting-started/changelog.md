## 0.2 - 29 September, 2021
### Changelog
- Imports sorted in `sosise-core` (changes just for code beauty)
- `./arisan make:repository` by default creates a repository with an database client. Additionally you could use `-h or --http` flag to create repository prepared for HTTP Requests
- Method `Helper.dd()` improved, it can now accept multiple arguments `Helper.dd('a', 13, {a: 13});`
- Method `Helper.dump()` improved, it can now accept multiple arguments `Helper.dd('a', 13, {a: 13});`
- `./artisan migrate:rollback` Shows which migrations would be rolled back and prompts for a confirmation now, since this operation can be very dangerous.
- Default documenation icon added

### How to upgrade to that version
Just run `npm install sosise-core@latest` or `npm run update-sosise`

## 0.1 - 14 September, 2021
### Changelog
- Added environment param to enable or disable documentation `basic auth`
- `docker-compose.yml` file updated to mount logs
- /docs folder excluded from `editorconfig-checker`
- Database default configuration refactored
- Some other small non breaking changes

### How to upgrade to that version
Just run `npm install sosise-core@latest` or `npm run update-sosise`