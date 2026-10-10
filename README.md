# Postman API Automation Testing Portfolio

[![QA API Regression Pipeline](https://github.com/laibazaheer11/postman-api-testing/actions/workflows/newman-runner.yml/badge.svg)](https://github.com/laibazaheer11/postman-api-testing/actions/workflows/newman-runner.yml)

![Postman](https://img.shields.io/badge/Postman-FF6C37?style=flat-square&logo=postman&logoColor=white)
![Newman CLI](https://img.shields.io/badge/Newman-000000?style=flat-square&logo=npm&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)


This repository contains my portfolio of automated API test suites. It demonstrates end-to-end integration testing, data chaining between endpoints, and automatic schema validation. 

To simulate a real-world QA workflow, all tests are automated to run in the cloud using GitHub Actions and the Newman command-line runner whenever changes are made.

---

## 🛠️ Tools & Technologies Used
* **Testing Tool:** Postman (Web Browser Edition)
* **Runner Tool:** Newman CLI (Used to run Postman collections via text commands)
* **Automation Platform:** GitHub Actions (Cloud server environment running on `ubuntu-latest`)
* **Assertion Scripting:** JavaScript (Using Postman's built-in Chai assertion library)

---

## 📂 Project Structure & Test Cases Breakdown

All test suites are cleanly organized inside the `/collections` folder:

### 1. GoRest API Chaining Suite (`/collections/gorest-api.postman_collection.json`)
* **Goal:** Tests a complete user data lifecycle (Create, Read, Update, Delete) and passes data from one request to the next.
* **Testing Logic:** To prevent data errors caused by creating duplicate users, I used a pre-request script to generate a brand new username and email automatically before every run:
```javascript
var username = "user" + Math.floor(Math.random() * 10000);
var useremail = "test" + Date.now() + "@example.com";
pm.collectionVariables.set("user_email", useremail);
pm.collectionVariables.set("user_name", username);
```
* **Test Flow Steps:**
  1. `POST /users` -> Creates a user, checks that the response is `201 Created`, extracts the unique user ID from the response body, and saves it.
  2. `GET /users/{id}` -> Fetches the user using the saved ID, and checks that the returned name and email match our saved variables exactly.
  3. `PUT /users/{id}` -> Updates the user's gender and status, ensuring the server response time stays under 3000ms.
  4. `DELETE /users/{id}` -> Deletes the user record and verifies that the server returns a successful `204 No Content` status code.

### 2. OpenWeather Map API Suite (`/collections/openweather-api.postman_collection.json`)
* **Goal:** Tests data types, object properties, value ranges, and lists of data items.
* **Testing Logic:** Verifies that geographical coordinates returned by the API are actual numbers and fall inside valid real-world ranges (Latitude between -90 and 90, Longitude between -180 and 180):
```javascript
pm.test("latitude and longitude are valid numbers in range", () => {
    pm.expect(res.lat).to.be.a("number");
    pm.expect(res.lat).to.be.at.least(-90).and.at.most(90);
});
```
* **Test Flow Steps:**
  1. `GET Geocode direct` -> Converts a text city name into latitude and longitude numbers, checking that the returned array is not empty.
  2. `GET Current Weather` -> Uses the saved coordinates to fetch the weather, confirming mandatory fields like `main`, `weather`, and `name` exist, and checking that the humidity percentage is between 0 and 100.
  3. `GET 5-Day Forecast` -> Uses a `forEach` loop to look inside the weather forecast array, making sure every single timestamp entry contains valid temperature and date numbers.
  4. `GET Air Pollution` -> Validates the Air Quality Index (AQI) value to ensure it falls strictly between categories 1 and 5.

### 3. Simple Books API Suite (`/collections/simple-books-api.postman_collection.json`)
* **Goal:** Tests token-based bearer authentication, order matching logic, and final variable cleanup.
* **Testing Logic:** Uses a JavaScript array function (`.find()`) to dynamically search through the list of books to find one that is marked as available (`available === true`) instead of using a hardcoded book number:
```javascript
let availableBook = data.find(book => book.available === true);
pm.expect(availableBook).to.not.be.undefined;
pm.collectionVariables.set("book_id", availableBook.id);
```
* **Test Flow Steps:**
  1. `GET /status` -> Verifies the API server status returns an "OK" message.
  2. `POST /api-clients` -> Simulates creating an account to generate a secure API access token, verifying it is a valid text string.
  3. `GET /books` -> Checks the book list and saves the ID of an available book.
  4. `POST /orders` -> Places a book order using the generated bearer access token for authorization.
  5. `GET /orders/{id}` -> Fetches the placed order to confirm the details match our order request.
  6. `PATCH /orders/{id}` -> Updates the customer's name on the order and checks for a `204` successful status code.
  7. `DELETE /orders/{id}` -> Deletes the book order.
  8. `Verify Deletion` -> Tries to fetch the deleted order again to confirm the API returns a proper `404 Not Found` error.
  9. `Variable Cleanup` -> Automatically unsets and clears out all temporary variables from Postman memory so the next test run starts fresh.

---

## 📋 Summary of Automated Test Types Covered

* **Status Code Validations:** Asserting HTTP response codes like `200 OK`, `201 Created`, `204 No Content`, and `404 Not Found`.
* **Response Time Audits:** Monitoring API latency to check that responses come back within acceptable limits (e.g., below 3000ms).
* **Data Type & Schema Validations:** Confirming that field values match their expected formats (e.g., ID must be a `number`, email text must contain an `@` symbol).
* **Dynamic Chaining:** Capturing values from an API response body and passing them directly into the parameters of the next API request.
* **Array & Loop Verifications:** Iterating through large JSON response arrays using loops to check that every item contains valid mandatory data properties.
* **Authentication Security Checks:** Validating endpoints that require token headers, and ensuring deleted or unauthenticated access results in proper error handling.

---

## ⚙️ How the Automated CI/CD Execution Works

The automated testing process is handled step-by-step by GitHub Actions inside the configuration file `.github/workflows/newman-runner.yml`:

1. **Start System:** GitHub spins up a clean, temporary virtual Linux computer whenever code changes are saved.
2. **Setup Dependencies:** It downloads Node.js software tools and installs the command-line interface tool (`newman`) onto the runner.
3. **Secret Security Management:** The workflow securely pulls private authentication tokens from the repository's encrypted settings, keeping them safe from public view.
4. **Execution Run:** It sequentially triggers Newman commands to execute all three collections against the live test servers, outputting clear pass/fail results directly inside the GitHub Actions tab.
