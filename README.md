# Postman API Automation Testing Portfolio

**Live Execution Reports:** https://laibazaheer11.github.io/postman-api-testing/


[![QA API Regression Pipeline](https://github.com/laibazaheer11/postman-api-testing/actions/workflows/newman-runner.yml/badge.svg)](https://github.com/laibazaheer11/postman-api-testing/actions/workflows/newman-runner.yml)

![Postman](https://img.shields.io/badge/Postman-FF6C37?style=flat-square&logo=postman&logoColor=white)
![Newman CLI](https://img.shields.io/badge/Newman-000000?style=flat-square&logo=npm&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)

---

## Overview

Final year IT student portfolio for SQA Intern role. Focused on API automation, not manual.

Built 3 independent Postman collections with end-to-end chaining, auth handling, and negative coverage. All tests run on GitHub Actions using Newman with HTML reports published to GitHub Pages.

| Metric | Count |
| :--- | :--- |
| Total Collections | 3 |
| Total Requests | 26 |
| Total Assertions | 49 |
| Negative Cases | 8 |
| Failed Tests | 0 |
| Avg Response Time | <500ms |

---

## Tools Used

- Postman for test design and chaining
- Newman + htmlextra for CLI execution and reporting
- GitHub Actions for CI, Secrets for token security, Pages for live reports
- JavaScript / Chai for assertions

## Test Suites Breakdown

All collections are in `/collections`.

| Collection | What I Tested | Requests | Assertions | Key Logic I Implemented |
| :--- | :--- | :--- | :--- | :--- |
| **GoRest CRUD & Chaining** | Full user lifecycle: Create -> Get -> Update -> Delete | 8 | 15 | Dynamic email generation to avoid 422 duplicates, chained user_id, cleanup after delete |
| **OpenWeather Chaining** | Geocode to Weather to Forecast to Air Pollution | 6 | 13 | Chained lat/lon from city name, validated lat/lon ranges, looped forecast array for temp/humidity |
| **Simple Books E2E Order Flow** | Status -> Auth -> Find Book -> Order -> Verify -> Delete | 12 | 21 | Generated random client, extracted bearer token, used .find() for available book, verified deletion |

### Negative Coverage (8 cases total)

- GoRest: 422 Duplicate Email, 401 Invalid Token, 404 Invalid User ID
- OpenWeather: 200 Empty array for invalid city, 401 Invalid API Key
- Simple Books: 401 Invalid Token, 400 Invalid Book ID, 400 Missing customer name

---

## CI/CD Pipeline

Built with GitHub Actions: `QA API Regression Pipeline`

Workflow file: `.github/workflows/newman-runner.yml`

- Triggers on push to main and manual dispatch
- Uses Node 22, installs Newman globally
- Runs 3 collections with separate secrets:
    - `GOREST_TOKEN` for GoRest
    - `WEATHER_API_KEY` for OpenWeather
- `continue-on-error: true` so one suite failure doesn't hide others
- Exports 3 HTML reports to `/newman`
- Deploys to GitHub Pages + uploads artifact

Live Reports: https://laibazaheer11.github.io/postman-api-testing/
- `gorest-report.html` - 8 req / 15 assertions
- `weather-report.html` - 6 req / 13 assertions
- `books-report.html` - 12 req / 21 assertions
All runs show 0 failed - see live dashboard for current response times.

**Security:** No tokens in code. All handled via GitHub Secrets and `--env-var`.

## How to Run Locally

```bash
npm install -g newman newman-reporter-htmlextra
newman run collections/gorest-api.postman_collection.json --env-var "token=YOUR_TOKEN" -r cli,htmlextra --reporter-htmlextra-export newman/gorest-report.html
newman run collections/openweather-api.postman_collection.json --env-var "api_key=YOUR_KEY" -r cli,htmlextra --reporter-htmlextra-export newman/weather-report.html
newman run collections/simple-books-api.postman_collection.json -r cli,htmlextra --reporter-htmlextra-export newman/books-report.html
```
