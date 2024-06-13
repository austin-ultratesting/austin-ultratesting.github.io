---
layout: post
title: Intercepting a request to help with the automation test flow
categories: [Suvaun App, Cypress, Javascript]
---

One of the earlier problems encountered while testing the Suvaun App was writing up the Benchmark tests intended for their standalone page accessible from the navigation bar. The problem observed during the intial draft of the test was that when the user entered data within the dropdown fields those results would be saved; this prevented a pre-determined state which the tests could run with fresh inputs for the data driven tests.

After some sleuthing and peaking into the developer tools, it was determined that previously entered inputs into the benchmarking page were stored as part of a user's attributes within the response body when API requests are being sent out. This is great, as Cypress has the capacity to intercept API requests in order to manipulate response bodies if API testing is part of the testing framework.

```js
// The intercept call needs to be set up prior to when the user navigates to the intended page when the API calls will trigger

// Send a response to the webpae that clears out existing benchmarking filters for user session

const url = "**/users/userInfo" // Example URL string to catch

cy.intercept(url, (req) => {
    req.continue((res) => {
    res.body.userAttributeValues = [
        /**
        This is where the response body needs to be modified so the data is cleared out.
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