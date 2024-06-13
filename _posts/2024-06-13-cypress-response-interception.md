---
layout: post
title: Intercepting a request to help with the automation test flow
categories: [Suvaun App, Cypress, Javascript]
---

One of the earlier problems encountered while testing the Suvaun App was writing up the Benchmark tests intended for their standalone page accessible from the navigation bar. The problem observed during the intial draft of the test was that when the user entered data within the dropdown fields those results would be saved; this prevented a pre-determined state which the tests could run with fresh inputs for the data driven tests. Other general problems tended to come up with the test scripts running particularly faster than the website able to process changes made. Therefore the need to wait until the website is in a new state for the tests to continue was necessary, and it is generally an anti-programming pattern to hardcode wait values such as ```cy.wait(1000);```.

-----

After some sleuthing and peaking into the developer tools, it was determined that previously entered inputs into the benchmarking page were stored as part of a user's attributes within the response body when API requests are being sent out. This is great, as Cypress has the capacity to intercept API requests in order to manipulate response bodies if API testing is part of the testing framework.

```js
/* The intercept call needs to be set up prior to when the user navigates
   to the intended page when the API calls will trigger

   Send a response to the webpage that clears out existing benchmarking
   filters for user session
*/ 

const url = "**/users/userInfo" // Example URL slug

cy.intercept(url, (req) => {
    req.continue((res) => {
    res.body.userAttributeValues = [
        /**
        This is where the response body needs to be modified so the data
        is cleared out.

        An example of the response body could look like this:
        
        {
            "FILTERS": "{ ... }"
        }
        **/
    ];
    });
});

cy.task("log","Set up complete on intercept call");
```

Now that the intercept is set up in the Cypress Framework, any time that the intercept is called for every data driven test, the test user will navigate to the benchmarking page with the filters cleared out. This achieves the desired result of controlling the flow of the test in a deterministic manner so all tests can be repeated.

-----

Probably the most common control method utilized within the Suvaun tests was waiting on certain API calls to be completed before the website was in a particular controlled state to where the automation could proceed. Thankfully this is perhaps the most routine procedure within Cypress as it utilizes the intercepts and wait calls as shown below:

```js
// Set up the intercept prior to interacting with the webpage
const url = "**/benchmarking/[Example URL Slug]";
cy.intercept(url).as('postComparisonPlan');

// Enter the basic filters for the Benchmarking Page
benchmarkFlow.enterBasicFilters(testItem);
cy.task("log","Set up filters for the benchmarking page!");

/* Apply the basic filters for the Benchmarking Page by clicking
   on "See Results" button
*/
benchmarkFlow.activateFilters();
cy.task("log","Filters activated! Waiting for API requests to finish...");

/* Wait for API requests to finish when applying basic filters.
   Then we can proceed with the rest of the test as needed
*/
cy.wait('@postComparisonPlan', {responseTimeout: 40000}).then(({response}) => {
    expect(response.statusCode).to.eq(200);
});

cy.task("log","API request completed! Filters have been applied to Benchmarking Page");
```

The use of intercepts within Cypress proved to be an effective tool in order to control the test flow in a deterministic manner so tests could be repeatable across different data points.

-----

These are a few practical examples used on the Suvan App project that required the use of intercepts. Such intercepts allowed the test cases to be controlled deterministically no matter what data set were given that drove the tests implemented on the project.