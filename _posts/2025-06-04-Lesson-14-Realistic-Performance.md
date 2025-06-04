---
title: "Lesson 14 - Realistic Performance Tests"
date: 2025-06-04
---
In the [previous lesson](/public-website/2025/06/03/Lesson-13-Performance-Anxiety.html) we created an artificial performance test with an arbitrary target just to see the application fail.

Now we need to create a more realistic test, representing a real user journey and gathering performance data that we can act on.

So far we have just hit a single endpoint - this is not often what we expect users to do. Our users navigate through a web application which can hit multiple endpoints as pages are constructed.

What we need is a combination of a user journey - such as those we mapped when we were writing functional tests - with the ability to simultaneously place load on the system.

k6 provides this ability through its [browser module](https://grafana.com/docs/k6/latest/using-k6-browser/?src=k6io&pg=get&plcmt=selfmanaged-box10-cta1).

# k6 browser
The k6 browser module allows us to write functional tests in a Playwright-ish language but rather than just perform the actions, it gives us insights into the performance of the page.

e.g. we navigate to the AtSea shop homepage, check that the image for "Docker for Developers" is showing and gather timing data:
```javascript
// browser-test-atsea-shop_landing_page.js
import { browser } from 'k6/browser';
import { check } from 'https://jslib.k6.io/k6-utils/1.5.0/index.js';;

export const options = {
  scenarios: {
    ui: {
      executor: 'shared-iterations',
      options: {
        browser: {
          type: 'chromium',
        },
      },
    },
  },
  thresholds: {
    checks: ['rate==1.0'],
  },
};

export default async function () {
  const context = await browser.newContext();
  const page = await context.newPage();

  try {
    await page.goto('http://localhost:8080/');

    await check(page.locator('//div[.="Docker for Developers"]//..//img'), {
      header: async (tileTitle) => (await tileTitle.isVisible()),
    });
  } finally {
    await page.close();
  }
}
```

As with the previous endpoint tests, we supply an `options` object to set up the configuration for the test and a `function` (async in this case) to describe the circumstances of the test.

We have added a `check` that the image is showing and specified in the `options` that this check should pass 100% of the time.

When we run the test, we get a lot of output:
```
         /\      Grafana   /‾‾/  
    /\  /  \     |\  __   /  /   
   /  \/    \    | |/ /  /   ‾‾\ 
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/ 

     execution: local
        script: browser-test-atsea-shop_landing_page.js
        output: -

     scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
              * ui: 1 iterations shared among 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)



  █ THRESHOLDS 

    checks
    ✓ 'rate==1.0' rate=100.00%


  █ TOTAL RESULTS 

    checks_total.......................: 1       0.982804/s
    checks_succeeded...................: 100.00% 1 out of 1
    checks_failed......................: 0.00%   0 out of 1

    ✓ header

    EXECUTION
    iteration_duration...................................: avg=708.07ms min=708.07ms med=708.07ms max=708.07ms p(90)=708.07ms p(95)=708.07ms
    iterations...........................................: 1      0.982804/s
    vus..................................................: 1      min=1       max=1
    vus_max..............................................: 1      min=1       max=1

    NETWORK
    data_received........................................: 0 B    0 B/s
    data_sent............................................: 0 B    0 B/s

    BROWSER
    browser_data_received................................: 2.3 MB 2.3 MB/s
    browser_data_sent....................................: 4.4 kB 4.4 kB/s
    browser_http_req_duration............................: avg=29.73ms  min=3.44ms   med=20.36ms  max=99.45ms  p(90)=64.92ms  p(95)=77.75ms 
    browser_http_req_failed..............................: 5.88%  1 out of 17

    WEB_VITALS
    browser_web_vital_cls................................: avg=0.288931 min=0.288931 med=0.288931 max=0.288931 p(90)=0.288931 p(95)=0.288931
    browser_web_vital_fcp................................: avg=352ms    min=352ms    med=352ms    max=352ms    p(90)=352ms    p(95)=352ms   
    browser_web_vital_lcp................................: avg=436ms    min=436ms    med=436ms    max=436ms    p(90)=436ms    p(95)=436ms   
    browser_web_vital_ttfb...............................: avg=6.89ms   min=6.89ms   med=6.89ms   max=6.89ms   p(90)=6.89ms   p(95)=6.89ms  




running (00m01.0s), 0/1 VUs, 1 complete and 0 interrupted iterations
ui   ✓ [======================================] 1 VUs  00m01.0s/10m0s  1/1 shared iters
```

These [browser metrics](https://grafana.com/docs/k6/latest/using-k6-browser/metrics/#browser-metrics-1) give us valuable information about how the page is performing.

Not surprisingly as everything is running locally, our latency `browser_web_vital_ttfb` is below 7ms.

There is one reported instance of `browser_http_req_failed`. 

If we re-run the test with the debug flag: `K6_BROWSER_DEBUG=true k6 run browser-test-atsea-shop_landing_page.js` then we can search for the failed response: 

`DEBU[0000] Failed to load resource: the server responded with a status of 404 ()  browser_source=network line_number=0 source=browser url="http://localhost:8080/favicon.ico"`

This just means we don't have the icon for the browser tab and we can ignore it for now (or fix it).

# Adding user load
We can change the options to run the test with 10 simultaneous users for 30s:
```javascript
export const options = {
  scenarios: {
    ui: {
      executor: 'constant-vus',
      vus: 10,
      duration: '30s',
      options: {
        browser: {
          type: 'chromium',
        },
      },
    },
  },
  thresholds: {
    checks: ['rate==1.0'],
  },
};
```
And this already makes a difference to the test results:
```
    EXECUTION
    iteration_duration...................................: avg=1.59s    min=997.29ms med=1.62s    max=1.91s    p(90)=1.77s    p(95)=1.8s  

    WEB_VITALS
    browser_web_vital_cls................................: avg=0.311018 min=0.288931 med=0.288931 max=0.370292 p(90)=0.370292 p(95)=0.370292
    browser_web_vital_fcp................................: avg=732.5ms  min=380ms    med=740ms    max=1s       p(90)=874.4ms  p(95)=885.2ms 
    browser_web_vital_lcp................................: avg=964.06ms min=532ms    med=988ms    max=1.22s    p(90)=1.11s    p(95)=1.15s   
    browser_web_vital_ttfb...............................: avg=19.94ms  min=8.4ms    med=16.69ms  max=59.79ms  p(90)=33.55ms  p(95)=41.09ms
```
