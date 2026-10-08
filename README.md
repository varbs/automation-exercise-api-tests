# Automation Exercise API Tests (Postman)

A Postman collection that tests the public practice API from [Automation Exercise](https://automationexercise.com/api_list). It covers all 14 endpoints with test scripts that check response codes, messages and data, and it documents behaviors I found along the way.

**Author:** Varbie Sumido, Software QA Specialist

## What is covered

| # | Request | Method | Endpoint | Expected result |
| --- | --- | --- | --- | --- |
| 1 | Get all products | GET | `/productsList` | 200, list of products |
| 2 | POST to products list | POST | `/productsList` | 405, method not supported |
| 3 | Get all brands | GET | `/brandsList` | 200, list of brands |
| 4 | PUT to brands list | PUT | `/brandsList` | 405, method not supported |
| 5 | Search product | POST | `/searchProduct` | 200, matching products |
| 6 | Search without parameter | POST | `/searchProduct` | 400, parameter missing |
| 7 | Verify login, valid | POST | `/verifyLogin` | 200, "User exists!" |
| 8 | Verify login without email | POST | `/verifyLogin` | 400, parameter missing |
| 9 | DELETE on verify login | DELETE | `/verifyLogin` | 405, method not supported |
| 10 | Verify login, invalid | POST | `/verifyLogin` | 404, "User not found!" |
| 11 | Create account | POST | `/createAccount` | 201, "User created!" |
| 12 | Delete account | DELETE | `/deleteAccount` | 200, "Account deleted!" |
| 13 | Update account | PUT | `/updateAccount` | 200, "User updated!" |
| 14 | Get user by email | GET | `/getUserDetailByEmail` | 200, user details |

The base URL for every request is `https://automationexercise.com/api`.

## What the tests check

- The `responseCode` and `message` fields in the JSON response body
- That lists such as products and brands are not empty and each item has the expected fields
- That created data is saved correctly, by reading the account back with a second request
- Wrong methods and missing parameters, to confirm the API refuses them with a clear message
- Response time

## Getting started

### Requirements

- [Postman](https://www.postman.com/downloads/) (the free plan is enough)

### Setup

1. Import the collection file from the `postman/` folder into Postman.
2. Create an environment (for example `AutoEx`) with these variables:

   | Variable | Value |
   | --- | --- |
   | `baseUrl` | `https://automationexercise.com/api` |
   | `email` | leave blank, a script fills it in |
   | `password` | any made-up password, for example `Test@12345` |

3. Select the environment in the dropdown at the top right of Postman.
4. On the collection's **Scripts > Pre-request** tab, add this script. It creates a unique fake email only if one does not exist yet:

   ```javascript
   if (!pm.environment.get("email")) {
     pm.environment.set("email", `tester${Date.now()}@example.com`);
   }
   ```

5. On the create account request only, add this to **Pre-request** so that each new account gets a new email:

   ```javascript
   pm.environment.set("email", `tester${Date.now()}@example.com`);
   ```

### Running the account flow

Run the requests in this order, because each one depends on the one before:

1. API 11: Create account
2. API 14: Get account by email
3. API 7: Verify login
4. API 13: Update account
5. API 14: Get account again, to confirm the change
6. API 12: Delete account
7. API 7: Verify login again, which should now say the user is not found

Request bodies for `createAccount`, `updateAccount`, `deleteAccount` and `verifyLogin` use `x-www-form-urlencoded`.

## Findings

Things I noticed while testing, worth knowing before you rely on this API:

- **Errors return HTTP 200.** The error code is only inside the JSON body. For example, updating an account that does not exist returns HTTP status 200 with `"responseCode": 404` in the body. A test that checks only the HTTP status would pass on a failed request, so these tests check `responseCode` in the body.
- **JSON bodies are not accepted for account requests.** Sending the create account fields as raw JSON returns `400` with "name parameter is missing", so the API appears to read form fields only.
- **Response field names differ from request names.** `birth_date` comes back as `birth_day`, `firstname` as `first_name`, and `lastname` as `last_name`.
- **`birth_month` can come back empty.** In one run, the month I sent was not returned when I fetched the account. This needs more testing to confirm.

## Notes

- All data in this project is made up. No real credentials, tokens or personal details are included.
- The API is a shared public practice site, so it may be slow, change, or reset without notice.
- Each flow ends by deleting the test account, to keep the site tidy.
- The test scripts were written from the API list page and checked against real responses where I could. Field names and messages may need adjusting if the API changes.

## Next steps

- [ ] Add positive and negative test cases for create account
- [ ] Repeat the checks as Playwright API tests, using data-driven test cases
- [ ] Add a short report of the bugs and quirks found

## Source

API list and expected responses: [automationexercise.com/api_list](https://automationexercise.com/api_list)
