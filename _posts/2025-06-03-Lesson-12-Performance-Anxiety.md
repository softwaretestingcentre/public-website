---
title: "Lesson 12 - Performance Anxiety"
date: 2025-11-04
---
In the previous lessons we have seen how to work with a BDD framework to perform automated testing of functional requirements.

Now we are going to turn to non-functional requirements, specifically performance.

Functional testing answers questions about whether the application does what we expect it to.

Non-functional testing explores whether it does those things _well_ - is it usable, reliable, responsive, secure, etc.

To investigate the performance of the app, we will use [Grafana K6](https://grafana.com/docs/k6/latest/set-up/install-k6/?src=k6io&pg=get&plcmt=selfmanaged-box10-cta1).

# A simple performance test
After doing the installation and setup of K6, we can write our first test:
```javascript
// load-test-atsea-shop.js

import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
    iterations: 10,
};

export default function() {
    http.get("http://localhost:8080/index.html");
    sleep(1);
}
```
This is just going to give us some idea of how quickly the system is going to respond to requests for the home page.

> ❗ This is a completely artificial example given that the application is running in a Docker container on the same machine as K6

We run the test with `k6 run load-test-atsea-shop.js`:
```

         /\      Grafana   /‾‾/  
    /\  /  \     |\  __   /  /   
   /  \/    \    | |/ /  /   ‾‾\ 
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/ 

     execution: local
        script: load-test-atsea-shop.js
        output: -

     scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
              * default: 10 iterations shared among 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)



  █ TOTAL RESULTS 

    HTTP
    http_req_duration.......................................................: avg=2.69ms min=2.52ms med=2.64ms max=2.9ms p(90)=2.9ms p(95)=2.9ms
      { expected_response:true }............................................: avg=2.69ms min=2.52ms med=2.64ms max=2.9ms p(90)=2.9ms p(95)=2.9ms
    http_req_failed.........................................................: 0.00%  0 out of 10
    http_reqs...............................................................: 10     0.996448/s

    EXECUTION
    iteration_duration......................................................: avg=1s     min=1s     med=1s     max=1s    p(90)=1s    p(95)=1s   
    iterations..............................................................: 10     0.996448/s
    vus.....................................................................: 1      min=1       max=1
    vus_max.................................................................: 1      min=1       max=1

    NETWORK
    data_received...........................................................: 8.1 kB 809 B/s
    data_sent...............................................................: 800 B  80 B/s




running (00m10.0s), 0/1 VUs, 10 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m10.0s/10m0s  10/10 shared iters
```

No alarms and no surprises - all the response timings are in single milliseconds, with no dropped requests.

Given that we are only delivering 800 bytes of data 10 times, this is nothing to write home about, but at least we know everything is wired up correctly.

> ℹ️ Note that you want to run this test twice to give the application time to "warm up".

# Pushing harder
Typically what we are looking for in performance testing is a system's response to sudden and/or sustatined load (multiple simultaneous and/or large requests).

We are looking for problems relating to not having the resources to meet the load or to resources leaking over time, leading to performance gradually degrading.

> ❗ Note that a system's inability to handle load beyond a particular point may be due to a component's configuration, rather than a lack of resources.
>
> ❗ The symptoms will be the same but if config is at fault, adding extra resources will not improve performance.

So what we can do with our original test is add Virtual (simultaneous) Users and run the test for a set time:

`k6 run --vus 10 --duration 30s load-test-atsea-shop.js`

```

         /\      Grafana   /‾‾/  
    /\  /  \     |\  __   /  /   
   /  \/    \    | |/ /  /   ‾‾\ 
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/ 

     execution: local
        script: load-test-atsea-shop.js
        output: -

     scenarios: (100.00%) 1 scenario, 10 max VUs, 1m0s max duration (incl. graceful stop):
              * default: 10 looping VUs for 30s (gracefulStop: 30s)



  █ TOTAL RESULTS 

    HTTP
    http_req_duration.......................................................: avg=4.46ms min=2.1ms med=4.27ms max=7.59ms p(90)=5.77ms p(95)=6.28ms
      { expected_response:true }............................................: avg=4.46ms min=2.1ms med=4.27ms max=7.59ms p(90)=5.77ms p(95)=6.28ms
    http_req_failed.........................................................: 0.00%  0 out of 300
    http_reqs...............................................................: 300    9.948455/s

    EXECUTION
    iteration_duration......................................................: avg=1s     min=1s    med=1s     max=1s     p(90)=1s     p(95)=1s    
    iterations..............................................................: 300    9.948455/s
    vus.....................................................................: 10     min=10       max=10
    vus_max.................................................................: 10     min=10       max=10

    NETWORK
    data_received...........................................................: 244 kB 8.1 kB/s
    data_sent...............................................................: 24 kB  796 B/s




running (0m30.2s), 00/10 VUs, 300 complete and 0 interrupted iterations
default ✓ [======================================] 10 VUs  30s
```
Again nothing to worry about, the response times have increased slightly but are still good.

We can see the network traffic increase in System Monitor, but nothing much happens to CPU or Memory:

![image](https://github.com/user-attachments/assets/d13d4351-455a-424f-83a6-8d4befa30a60)

Even with 100 VUs the system doesn't break sweat:
```
  █ TOTAL RESULTS 

    HTTP
    http_req_duration.......................................................: avg=3.76ms min=682.77µs med=1.78ms max=70.66ms p(90)=4.04ms p(95)=8.19ms
      { expected_response:true }............................................: avg=3.76ms min=682.77µs med=1.78ms max=70.66ms p(90)=4.04ms p(95)=8.19ms
    http_req_failed.........................................................: 0.00%  0 out of 3000
    http_reqs...............................................................: 3000   99.517228/s

    EXECUTION
    iteration_duration......................................................: avg=1s     min=1s       med=1s     max=1.07s   p(90)=1s     p(95)=1s    
    iterations..............................................................: 3000   99.517228/s
    vus.....................................................................: 100    min=100       max=100
    vus_max.................................................................: 100    min=100       max=100

    NETWORK
    data_received...........................................................: 2.4 MB 81 kB/s
    data_sent...............................................................: 240 kB 8.0 kB/s
```
Just to hammer the point home, this type of performance test is extremely unlikely to show anything interesting - it is only the request for the initial `index.html` page that is being responded to, not any of the JavaScript, page images, dynamic data, etc. We aren't making any calls to the database or 3rd party systems. 

There's **no reason** for this request to be slow unless we do something catastrophically bad.

# Getting more involved
Now we are going to see what happens if we try to deliberately break the system.

Using RestAssured to create some test Customer data:
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
        for (int i = 3; i < 1000; i++) {
            Customer testCustomer = new Customer(i, "Foobar-" + i);
            given().contentType(ContentType.JSON).body(testCustomer)
                    .when().post("http://localhost:8080/api/customer/")
                    .then().statusCode(HttpStatus.SC_CREATED);
        }
    }
```

We now have 1000 Customer records, which we can use for our test:
```javascript
export default function() {
    http.get("http://localhost:8080/api/customer/");
}
```
And with 200 VUs, we see:
```
         /\      Grafana   /‾‾/  
    /\  /  \     |\  __   /  /   
   /  \/    \    | |/ /  /   ‾‾\ 
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/ 

     execution: local
        script: load-test-atsea-shop.js
        output: -

     scenarios: (100.00%) 1 scenario, 200 max VUs, 1m0s max duration (incl. graceful stop):
              * default: 200 looping VUs for 30s (gracefulStop: 30s)



  █ TOTAL RESULTS 

    HTTP
    http_req_duration.......................................................: avg=71.66ms min=688.17µs med=3.18ms max=1.53s p(90)=119.92ms p(95)=618.45ms
      { expected_response:true }............................................: avg=71.66ms min=688.17µs med=3.18ms max=1.53s p(90)=119.92ms p(95)=618.45ms
    http_req_failed.........................................................: 0.00%  0 out of 83729
    http_reqs...............................................................: 83729  2783.941817/s

    EXECUTION
    iteration_duration......................................................: avg=71.77ms min=752.23µs med=3.29ms max=1.53s p(90)=120.04ms p(95)=618.6ms 
    iterations..............................................................: 83729  2783.941817/s
    vus.....................................................................: 200    min=200        max=200
    vus_max.................................................................: 200    min=200        max=200

    NETWORK
    data_received...........................................................: 20 MB  658 kB/s
    data_sent...............................................................: 6.9 MB 231 kB/s




running (0m30.1s), 000/200 VUs, 83729 complete and 0 interrupted iterations
default ✓ [======================================] 200 VUs  30s
```
Now we are getting the slowest responses into the seconds rather than milliseconds duration.

And some significant CPU activity:

![image](https://github.com/user-attachments/assets/1ebe5d4c-6af4-4cc4-989c-33794b82ac88)

Let's try breaking the system's responsiveness by limiting database connections. In `postgresql.conf`:
```
max_connections = 1
```

```

         /\      Grafana   /‾‾/  
    /\  /  \     |\  __   /  /   
   /  \/    \    | |/ /  /   ‾‾\ 
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/ 

     execution: local
        script: load-test-atsea-shop.js
        output: -

     scenarios: (100.00%) 1 scenario, 200 max VUs, 1m0s max duration (incl. graceful stop):
              * default: 200 looping VUs for 30s (gracefulStop: 30s)



  █ TOTAL RESULTS 

    HTTP
    http_req_duration.......................................................: avg=273.98ms min=8.78ms med=161.69ms max=3.57s p(90)=684.47ms p(95)=924.58ms
      { expected_response:true }............................................: avg=273.98ms min=8.78ms med=161.69ms max=3.57s p(90)=684.47ms p(95)=924.58ms
    http_req_failed.........................................................: 0.00%  0 out of 21966
    http_reqs...............................................................: 21966  726.486755/s

    EXECUTION
    iteration_duration......................................................: avg=274.15ms min=8.93ms med=161.83ms max=3.57s p(90)=684.61ms p(95)=924.73ms
    iterations..............................................................: 21966  726.486755/s
    vus.....................................................................: 200    min=200        max=200
    vus_max.................................................................: 200    min=200        max=200

    NETWORK
    data_received...........................................................: 3.0 GB 99 MB/s
    data_sent...............................................................: 1.8 MB 60 kB/s




running (0m30.2s), 000/200 VUs, 21966 complete and 0 interrupted iterations
default ✓ [======================================] 200 VUs  30s
```

We only get a quarter of the requests handled, with a corresponding increase in the average response times.

CPU is almost maxxed out:

![image](https://github.com/user-attachments/assets/81d89c01-ddd7-411e-a9ff-b8d26d484a26)

There's some chunky network traffic:

![image](https://github.com/user-attachments/assets/c8a40796-ae78-458d-9d14-65fe43b8f908)

Even some memory pressure:

![image](https://github.com/user-attachments/assets/252aad68-6a11-4d98-aa19-ff7fa9d3723d)






