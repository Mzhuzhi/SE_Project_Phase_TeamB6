# 1. Introduction to Testing

- Software testing is the process of evaluating software to identify defects, bugs, and unexpected behaviour. It involves running parts of the system under controlled conditions and comparing the actual output to the expected output, with the goal of finding mistakes before they reach end users.

- Testing is essential in software development because it improves the reliability of the system (the software keeps working as intended over time), the correctness of its features (every component does what it is supposed to do), and its maintainability (developers can change the code with confidence that nothing important has been silently broken). For a platform like UniGate, where students submit personal data and university staff make decisions based on that data, testing is not optional, instead it is a core part of building software that can be trusted.

# 2. Purpose of Testing
- Testing serves two core purposes in software development. The first is to identify defects early, before they reach users or become expensive to fix. Catching a bug at the testing stage costs significantly less time and effort than discovering it after deployment, especially on a platform like UniGate where a broken submission flow or a security gap could directly affect students' applications.

- The second purpose is to verify that software components behave as intended under both expected and unexpected conditions. A component that handles only the standard path is not finished as it must also reject invalid inputs, refuse unauthorised actions, and respond predictably when something goes wrong. Tests prove that these behaviours actually work as intended.

# 3. Focus on Testing a Single Component

For this report, we tested two critical components of the UniGate platform: the Student Application Submission endpoint and the Staff Program Creation endpoint. These two were chosen because they cover both ends of the platform’s most important workflow:the moment a student commits their application, and the moment a university staff member adds a new program to the catalogue.Both components were also chosen because they have clear inputs and outputs at the API layer, which makes them ideal candidates for integration testing. The tests in this report exercise these endpoints through real HTTP requests and verify the responses against expected behaviour.

# 4.Preparing Test Cases

The test cases were designed to cover three categories of input: 
- valid inputs (where everything is correct and the request should succeed),
- invalid inputs (where something is wrong, such as a missing field, a security violation, or a broken business rule),
- boundary conditions (edge values that sit right at the limit of what is allowed, such as a GPA of exactly 4 or a graduation year of 1950).
For each case, the expected behaviour was determined from the system requirements document and the validation rules implemented in the route handlers.

# 4.1 Test Case: Application Submission 

| Test ID | Scenario | Input | Expected Result |
|---|---|---|---|
| TC01 | Valid Bachelor application | All required fields filled, valid GPA in 0–4 range | 201 Created — application stored, status "pending" |
| TC02 | Missing required field | high_school_name omitted | 400 Bad Request: validation error |
| TC03 | Invalid graduation year (boundary) | graduation_year = 1800 (below 1950 floor) | 400 Bad Request: out of range |
| TC04 | Invalid GPA (boundary) | gpa = 5 (above 4 ceiling) | 400 Bad Request: out of range |
| TC05 | Master without bachelor fields | Submitted to Master programme without bachelor_school_name | 400 Bad Request: bachelor fields required |
| TC06 | Valid Master application | All required fields including bachelor degree info | 201 Created : application stored |
| TC07 | One-per-university rule | Second application by same student to a different programme at the same university | 409 Conflict : duplicate university |
| TC08 | Unauthenticated request | No auth cookie attached | 401 Unauthorized |





# 4.2 Test Case: Staff Program Creation 

| Test ID | Scenario | Input | Expected Result |
|---|---|---|---|
| TC01 | Valid programme | All bilingual fields and a valid required_document_types array | 201 Created :program stored with bilingual content |
| TC02 | Cross-university security | Staff sends a university_id matching a different university | 201 Created : university_id ignored, programme created at staff's own university |
| TC03 | Missing Albanian title | title_sq is empty | 400 Bad Request : bilingual validation |
| TC04 | Invalid degree type | degree_type = "Doctorate" (not allowed) | 400 Bad Request |
| TC05 | Negative tuition fee | tuition_fee = -100 | 400 Bad Request |
| TC06 | Invalid document type key | required_document_types contains "fake_document_type" | 400 Bad Request |
| TC07 | Unauthenticated request | No auth cookie | 401 Unauthorized |
| TC08 | Wrong role | Authenticated as student attempting staff action | 403 Forbidden |
| TC09 | Empty document requirements | required_document_types = [] | 201 Created : zero required documents allowed |




# 5. Writing Test Code


The tests were written using Jest as the test runner and assertion library, with Supertest to send simulated HTTP requests directly to the Express application without spinning up a live server. This combination is the standard for testing Node.js / Express APIs.
The tests run against a dedicated test database (unigate_test), separate from the development database, so test data never contaminates real records. A setup.js helper resets the schema and seeds a known minimal dataset before each test, ensuring every test starts from a clean, predictable state.

- # TC07 : the one-per-university rule

  ```javascript
test('TC07: blocks second application to the same university', async () => {
  const cookie = await loginAsStudent();

  // First application succeeds
  await request(app)
    .post('/api/applications')
    .set('Cookie', cookie)
    .send({
      program_id: seedData.bachelor1.id,
      date_of_birth: '2005-03-15',
      nationality: 'Albanian',
      phone: '+355691234567',
      address: 'Tirana, Albania',
      high_school_name: 'Sami Frasheri',
      high_school_country: 'Albania',
      graduation_year: 2023,
      gpa: 3.8,
    });

  // Second application to a different programme at the same
  // university should be blocked
  const res = await request(app)
    .post('/api/applications')
    .set('Cookie', cookie)
    .send({
      program_id: seedData.masterProgram.id,
      bachelor_school_name: 'University of Tirana',
      bachelor_school_country: 'Albania',
      bachelor_graduation_year: 2022,
      bachelor_gpa: 3.5,
    });

  expect(res.status).toBe(409);
});
```



  
  - # TC02 : the security boundary



This test checks a security rule: a staff member should only be able to create programmes at their own university, even if they try to cheat.

```javascript
test('TC02: ignores university_id sent in request body', async () => {
  const cookie = await loginAsTiranaStaff();
  const payload = validProgrammePayload();
  payload.university_id = seedData.epoka.id; // attempt forge

  const res = await request(app)
    .post('/api/staff/programs')
    .set('Cookie', cookie)
    .send(payload);

  expect(res.status).toBe(201);
  // The created programme must belong to Tirana, NOT Epoka
  expect(res.body.data.university_id).toBe(seedData.tirana.id);
});
```

- # TC05 : Master applications require bachelor fields


This test checks that the system correctly rejects a Master's application when the bachelor's degree fields are missing. Since a Master's programme requires proof of a previous degree, the server must ask for extra information that a Bachelor's application does not need. In this test, a student submits an application for a Master's programme but leaves out the bachelor's degree details on purpose. The expected result is a 400 error, confirming that the server catches the missing fields and blocks the incomplete application.

```javascript
test('TC05: rejects Master application missing bachelor fields', async () => {
  const cookie = await loginAsStudent();

  const res = await request(app)
    .post('/api/applications')
    .set('Cookie', cookie)
    .send({
      program_id: seedData.masterProgram.id,
      date_of_birth: '2000-03-15',
      nationality: 'Albanian',
      phone: '+355691234567',
      address: 'Tirana, Albania',
      high_school_name: 'Sami Frasheri',
      high_school_country: 'Albania',
      graduation_year: 2018,
      gpa: 3.8,
      // bachelor_school_name, bachelor_school_country,
      // bachelor_graduation_year — all intentionally omitted
    });

  expect(res.status).toBe(400);
});
```
The full test suites contain 17 tests across two files: 8 for the application submission endpoint and 9 for the staff program creation endpoint.

# 6. Running Tests
The tests are executed with the command npm test in the backend directory. Jest automatically finds and runs all test files matching the pattern tests/**/*.test.js. The tests run one at a time rather than simultaneously, because they all share the same test database and running them at the same time could cause conflicts.

Pass (✓): the assertion succeeded; the system behaved as expected.
Fail (✗):an assertion failed. The output shows exactly which assertion, what was expected, and what was received. This is the most useful debugging information testing provides.
Error :the test code itself crashed before reaching an assertion, for example due to a database connection failure.

The first test runs revealed a subtle inconsistency in the codebase: the application routes return status: 'ok', while the staff program routes return status: 'success'. The project specification defines 'ok' as the correct value, but this was not applied consistently across all routes. This was caught immediately by the test assertions and recorded as a finding for future cleanup.All 17 tests passed in approximately 9 seconds against the test database, covering both happy-path scenarios and a wide range of failure modes for the two components.

- Final Test Execution Result
<img width="974" height="204" alt="image" src="https://github.com/user-attachments/assets/9c387f30-2378-4697-b38b-3c0df10e483f" />










<img width="974" height="201" alt="image" src="https://github.com/user-attachments/assets/b55185c9-c7fc-42a3-b4cb-df878c65d5a5" />


# 7. Test Coverage
Test coverage measures how much of the code is actually checked by the tests. Good coverage means that the most important behaviours, rules, and failure cases have been tested at least once. For UniGate, the goal was to make sure every important decision in the code such as whether a field is valid or whether a user has permission had at least one test checking both the passing and failing case.

# 7.1 What is covered

Application Submission:

- A valid Bachelor and a valid Master application both succeed
- Missing required fields are caught and rejected
- GPA and graduation year are checked to be within the allowed range
- Master applications are rejected if the bachelor degree fields are missing
- A student cannot apply twice to the same university
- Requests without a login are rejected

Staff Program Creation:

- A valid program with all required information is created successfully
- A staff member cannot create a program at a different university, even if they try to change the university in the request
- A program cannot be created if the Albanian title is missing
- Invalid degree types and negative tuition fees are rejected
- Unrecognised document types are rejected
- Requests without a login are rejected
- A student cannot perform staff actions — they get a 403 error
- A program with no required documents is allowed

# 7.2 What is not yet covered

- Update and delete endpoints for both resources are not directly tested in this report. The same testing pattern would apply.
- File upload validation for document submission and program cover images is not covered, partly because file upload tests require additional Supertest configuration for multipart bodies.
- Frontend behaviour is out of scope for these tests as they verify backend correctness only. UI-level testing would be a separate effort using a tool like Vitest with React Testing Library.
- End-to-end flows (a real browser navigating through the application) would be covered by tools like Playwright or Cypress in a more mature testing strategy.


# 7.3 What testing revealed

-  An inconsistency in API response shape was discovered. Some routes return status: ‘ok’ and others status: ‘success’. The project specification calls for ‘ok’. This inconsistency was previously invisible because clients did not check the field, but it would cause confusion for any future developer reading the code or building external integrations.

- The TC02 security test confirmed that a staff member cannot create a programme at a university they do not belong to, even if they try to change the university in the request. This test now acts as a permanent check — if someone accidentally removes that protection in the future, the test will catch it immediately. This is one of the most valuable things testing provides: not just finding bugs today, but making sure the same bugs cannot come back tomorrow.

# 7.4 Conclusion
Testing is not a one-time task. It is a habit that keeps the codebase healthy as the project grows.The setup built for this report: Jest, Supertest, and the test database, makes it easy to keep adding new tests as the platform grows and changes.The tests written for this report already caught a real inconsistency in the codebase and confirmed that a critical security rule is working correctly. As UniGate continues to develop, the testing setup put in place here makes it easy to keep adding new tests and catching problems early.















