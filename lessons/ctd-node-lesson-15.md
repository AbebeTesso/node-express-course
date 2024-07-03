Any large development project requires automated testing to ensure components work correctly, that they work well together, and that nothing is broken as the application is maintained and as features are added.  The testing also ensures that the program works to specifications.  Manual testing with Postman or other tools is time consuming and error prone.  This lesson gives practice in developing an API and shows how to do automated Express testing.

## Types of Testing

1. Unit Tests: These are very low level tests of an individual method, class, or module.
2. Integration Tests: These verify that the different parts of the application work well together.
3. Functional or System Tests: These verify that the application produces the required output, each of the components working together.

There is also performance testing, acceptance testing, security testing etc.  But for the developer, the following three types of tests are the main ones.  In some cases, the tests are written before the development begins, based on the application spec.  In any case, as software developers, you will be asked to develop tests, and frequently you will be required to provide tests for any code you submit.  By the way, your functional tests for production applications **should include security testing**, to avoid having security holes.  Security tests make sure that the user is authenticated and authorized to access the information provided.

For Node/Express, there are several standard testing tools. Mocha is a testing framework that automates the running of tests.  Chai is an assertion framework, which provides testing specific language extensions that allow you to describe the expected output and to record errors if the results are not as exprected.  You will also need to test the web front end.  Several Node based tools for this purpose are Puppeteer and Cypress.  We will use Puppeteer.  These examine the web pages that are returned, fill in forms, and submit input as a user would.

## Introducing Mocha and Chai

Here is a sample test case that uses Mocha and Chai.  It won't work for the present lesson, because it is for calling an API, not getting a page.

```javascript
describe("Jobs", function () {
  describe("GET /jobs", function () {
  // Test to get all jobs belonging to the logged on user
    it("should get all jobs for the user", async () => {
      // We need to get the values for request and expect here.
      // More on that later.
      const req = request.execute(app)
        .get('/api/v1/jobs')
        .send()
      const res = await req()
      expect(res).to.have.status(200);
      expect(res.body).to.be.a('object');
      expect(res.body.jobs.length).to.equal(3); // or whatever is in your test data
    });  
         ...
```
The Mocha keywords here are "describe" and "it".  These organise the test suite into blocks and document the purpose of each test case.  The test above is not really complete, in that one would want to verify that
the expected data is returned.  And, as written, the test would fail, or at least would return an empty object, because of course, no user is logged in.  So Mocha provides some additional keywords: before, beforeEach, after, and afterEach.  These are for things such as logon to be done before or after a given block, or before or after each test case in the block.

The chai-http words here are, in this case, get, which returns a result or an error.  Of course there are put, post, patch, and delete as well. The get function is implemented in chai-http.  The Chai word we are using is expect, which checks that the result is correct.

Some other things to think about: We have been using a single Mongo database for development. Were you building an actual production application, you would want separate databases for development, test, and production.  Also, for testing, you would want a way to populate the database with sample data, so that it is in a known state at the start of the test.  For this purpose, we'll use an npm package called factory-bot.

## A Short Introduction to Puppeteer

With Puppeteer, you actually load the pages, find the HTML elements on the page, and interact with them, using a browser engine, which is typically Chrome.  You can run Chrome in "headless" mode, but if you do not do this, you can actually watch your test typing values into a browser window. Puppeteer involves a lot of async/await.  Here is the start of a Puppeteer test, where the connection to the browser is made and a
page is retrieved:
```javascript
  describe("Functional Tests with Puppeteer", function () {
    let browser = null;
    let page = null;
    before(async function () {
      this.timeout(5000);
      browser = await puppeteer.launch();
      page = await browser.newPage();
      await page.goto("http://localhost:3000");
    });
    after(async function () {
      this.timeout(5000);
      await browser.close();
      return;
    });
    describe("got to site", function () {
      it("should have completed a connection", async () => {
      });
    });
    describe("people form", function () {
      this.timeout(5000);
      it("should have various elements", async () => {
        this.nameField = await page.waitForSelector("input[name=\"name\"]");
        this.ageField = await page.waitForSelector("input[name=\"age\"]");
        this.addPerson = await page.waitForSelector("button ::-p-text(Add)")
```
In the above code, this.timeout(5000) sets the timeout for that test, the amount of time by which the operation certainly should have succeeded.  We can verify that the page came up with some entry fields, identified by their HTML IDs or other attributes. To find the button, we use some special syntax provided by Puppeteer, to find a button with "Add" in the text.

Then, we can start interacting with the form, as follows:
```javascript
      it("should create a person record given name and age", async () => {
        await this.nameField.type("Fred");
        await this.ageField.type("10");
        await this.addPerson.click();
        await page.waitForNavigation();
        const resultDataDiv = await page.waitForSelector("#result");
        const resultData = await resultDataDiv.evaluate((element) => element.textContent);
        expect(resultData).to.include("A person record was added");
        ...
```
Here you see the code that actually types into entry fields, and then clicks on the submit button.  You get the idea.

## Goals of the Lesson

You are seeing several kinds of integration/functional testing, involving either direct communication with the controllers (Chai), or communication with the web GUI (Puppeteer).  In each case, the tests cause interaction with the running Express server.  In this lesson, you learn to write these tests.