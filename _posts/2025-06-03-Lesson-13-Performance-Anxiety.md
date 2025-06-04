---
title: "Lesson 13 - Performance Anxiety"
date: 2025-06-03
---
In the previous lessons we have seen how to work with a BDD framework to perform automated testing of functional requirements.

Now we are going to turn to non-functional requirements, specifically performance.

**Functional** testing answers questions about whether the application does what we expect it to.

**Non-functional** testing explores whether it does those things _well_ - is it usable, reliable, responsive, secure, etc.

To investigate the performance of the app, we will use [Grafana K6](https://grafana.com/docs/k6/latest/set-up/install-k6/?src=k6io&pg=get&plcmt=selfmanaged-box10-cta1).

# A simple performance test
After doing the installation and setup of K6, we can write our first test, hitting the login API endpoint:
```javascript
// load-test-atsea-shop_login.js
import http from 'k6/http';
import { check } from 'k6';

export default function() {
    const auth = JSON.stringify({
        "username": "Foobar-1",
        "password": "password"
    });
    const params = {
        headers: {
        'Content-Type': 'application/json',
        },
    };
    const response = http.post("http://localhost:8080/login/", auth, params);
    check(response, {"status was 200": (r) => r.status == 200});    
}
```
We add a `check()` command to ensure we aren't just getting errors back and run the test with 200 Virtual Users for 10s `k6 run --vus 200 --duration 10s load-test-atsea-shop_login.js`:

Noting that we are already seeing some significant CPU usage as the application hits the database to verify the user:

![image](https://github.com/user-attachments/assets/2bf158a8-dc7c-4e7a-aff5-5817d2bec736)

```
         /\      Grafana   /‾‾/  
    /\  /  \     |\  __   /  /   
   /  \/    \    | |/ /  /   ‾‾\ 
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/ 

     execution: local
        script: load-test-atsea-shop_login.js
        output: -

     scenarios: (100.00%) 1 scenario, 200 max VUs, 40s max duration (incl. graceful stop):
              * default: 200 looping VUs for 10s (gracefulStop: 30s)



  █ TOTAL RESULTS 

    checks_total.......................: 26705   2659.601326/s
    checks_succeeded...................: 100.00% 26705 out of 26705
    checks_failed......................: 0.00%   0 out of 26705

    ✓ status was 200

    HTTP
    http_req_duration.......................................................: avg=74.77ms min=1.31ms med=6.92ms max=1.67s p(90)=198.06ms p(95)=480.41ms
      { expected_response:true }............................................: avg=74.77ms min=1.31ms med=6.92ms max=1.67s p(90)=198.06ms p(95)=480.41ms
    http_req_failed.........................................................: 0.00%  0 out of 26705
    http_reqs...............................................................: 26705  2659.601326/s

    EXECUTION
    iteration_duration......................................................: avg=74.99ms min=1.47ms med=7.19ms max=1.67s p(90)=198.21ms p(95)=480.61ms
    iterations..............................................................: 26705  2659.601326/s
    vus.....................................................................: 200    min=200        max=200
    vus_max.................................................................: 200    min=200        max=200

    NETWORK
    data_received...........................................................: 13 MB  1.3 MB/s
    data_sent...............................................................: 4.6 MB 463 kB/s




running (10.0s), 000/200 VUs, 26705 complete and 0 interrupted iterations
default ✓ [======================================] 200 VUs  10sh2
```
We can see that k6 has produced some stats for us showing the responsiveness of the system and how many requests it was able to serve in 10s.

We can also see that no requests came back with errors `✓ status was 200`.

# What does good performance look like?
Typically what we are looking for in performance testing is a system's response to sudden and/or sustained load (multiple simultaneous users and/or large requests).

We are looking for problems relating to not having the resources to meet the load or to resources leaking over time, leading to performance gradually degrading.

> ❗ Note that a system's inability to handle load beyond a particular point may be due to a component's configuration, rather than a lack of resources.
>
> ❗ The symptoms will be the same but if config is at fault, adding extra resources will not improve performance.

# Setting performance requirements
> ℹ️ We don't simply run performance tests just to see what happens. We need to specify **up front** our expectations for the system.

e.g. 

| Endpoint | Concurrent Users | 90% responses in |
| -------- | ---------------- | ---------------- |
| login | 200 | 400ms |

Given these performance requirements, we can start to add thresholds to our test:
```javascript
export const options = {
  // define thresholds
  thresholds: {
    http_req_failed: ['rate<0.01'], // http errors should be less than 1%
    http_req_duration: ['p(99)<400'], // 99% of requests should be below 0.4s
  },
};
```
Now when we run the test, we can see that we are falling short of one of our threshold expectations:
```
  █ THRESHOLDS 

    http_req_duration
    ✗ 'p(99)<400' p(99)=638.76ms

    http_req_failed
    ✓ 'rate<0.01' rate=0.00%
```
It may be useful at this point at investigate at what load we fail to meet the threshold, so we add some ramping load to the test and ask it to stop when the threshold is hit:
```javascript
export const options = {
  // define thresholds
  thresholds: {
    http_req_failed: ['rate<0.01'], // http errors should be less than 1%
    http_req_duration: [{ threshold: 'p(99)<400', abortOnFail: true }], // 99% of requests should be below 0.4s
  },
  scenarios: {
    // define scenarios
    breaking: {
      executor: 'ramping-vus',
      stages: [
        { duration: '10s', target: 20 },
        { duration: '10s', target: 40 },
        { duration: '10s', target: 80 },
        { duration: '10s', target: 160 },
        { duration: '10s', target: 200 },
      ],
    },
  },
};
```
As the VUs and duration are now controlled by the `options` object, we run the test with the simpler command `k6 run load-test-atsea-shop_login.js`:
```
breaking ✗ [===========================>----------] 127/200 VUs  38.0s/50.0s
ERRO[0038] thresholds on metrics 'http_req_duration' were crossed; at least one has abortOnFail enabled, stopping test prematurely
```
So we failed to serve 127 concurrent requests in the expected time, well short of our expectation of 200.

# Pushing harder
Now we are going to see what happens if we make much larger requests.

Using RestAssured to create 1000 Customer rows in the database:
```java
    public class Customer {
        private int customerId;
        private String name;
        private String address;
        private String email;
        private String phone;
        private String username;
        private String password;
        private String enabled;
        private String role;

        public Customer(int customerId, String name) {
            this.customerId = customerId;
            this.name = name;
            this.address = "144 Townsend Street";
            this.email = "foo@bar.com";
            this.phone = "12345678";
            this.username = name;
            this.password = "password";
            this.enabled = "true";
            this.role = "USER";
        }
    }

    @Test
    public void createCustomerData() {
        for (int i = 1; i < 1001; i++) {
            Customer testCustomer = new Customer(i, "Foobar-" + i);
            given().contentType(ContentType.JSON).body(testCustomer)
                    .when().post("http://localhost:8080/api/customer/")
                    .then().statusCode(HttpStatus.SC_CREATED);
        }
    }
```

Running with the same ramping load and thresholds as before, we hit the `http://localhost:8080/api/customer/` endpoint, which now returns a 136KB JSON object.
```
         /\      Grafana   /‾‾/  
    /\  /  \     |\  __   /  /   
   /  \/    \    | |/ /  /   ‾‾\ 
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/ 

     execution: local
        script: load-test-atsea-shop_customer_list.js
        output: -

     scenarios: (100.00%) 1 scenario, 200 max VUs, 1m20s max duration (incl. graceful stop):
              * breaking: Up to 200 looping VUs for 50s over 5 stages (gracefulRampDown: 30s, gracefulStop: 30s)



  █ THRESHOLDS 

    http_req_duration
    ✗ 'p(99)<400' p(99)=428.3ms

    http_req_failed
    ✓ 'rate<0.01' rate=0.00%


  █ TOTAL RESULTS 

    checks_total.......................: 21825   642.226405/s
    checks_succeeded...................: 100.00% 21825 out of 21825
    checks_failed......................: 0.00%   0 out of 21825

    ✓ status was 200

    HTTP
    http_req_duration.......................................................: avg=61.54ms min=7.02ms med=28.97ms max=1.26s p(90)=148.44ms p(95)=218.91ms
      { expected_response:true }............................................: avg=61.54ms min=7.02ms med=28.97ms max=1.26s p(90)=148.44ms p(95)=218.91ms
    http_req_failed.........................................................: 0.00%  0 out of 21825
    http_reqs...............................................................: 21825  642.226405/s

    EXECUTION
    iteration_duration......................................................: avg=61.77ms min=7.22ms med=29.23ms max=1.26s p(90)=148.65ms p(95)=219.11ms
    iterations..............................................................: 21825  642.226405/s
    vus.....................................................................: 111    min=2          max=111
    vus_max.................................................................: 200    min=200        max=200

    NETWORK
    data_received...........................................................: 3.0 GB 87 MB/s
    data_sent...............................................................: 1.8 MB 54 kB/s




running (0m34.0s), 000/200 VUs, 21825 complete and 111 interrupted iterations
breaking ✗ [========================>-------------] 019/200 VUs  34.0s/50.0s
ERRO[0034] thresholds on metrics 'http_req_duration' were crossed; at least one has abortOnFail enabled, stopping test prematurely
```
And now we struggle to serve 19 concurrent users 😢 so there is work to do.

> ✔️ We now have an easily repeatable test that gives us useful feedback, so we can use it to assess different approaches to increasing the system performance.

There are may ways that we can react to this performance test failure:
> ☹️ just increase the threshold until the test passes more reliably
> 
> 😐 disregard the result because `/customer/` is not a critical customer-facing endpoint
>
> 🥲 realise there is no use case for this endpoint and just delete it

All of these are valid responses depending on the needs of the business
> ℹ️ just because a test fails doesn't mean anyone is obliged to "fix" it - the signal it provides may call for alternative action


> ❗ The last response is especially valid - we shouldn't provide endpoints just because we can - an unauthenticated list of customer details is a **massive** security breach 😱


> ✔️ The second is actually good feedback about the test itself - in the next lesson we will learn how to identify realistic performance tests 
