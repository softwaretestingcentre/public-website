---
title: "Side Quest 2 - deploying a docker container to Azure"
date: 2025-06-05
---
For the lessons on security testing, I wanted to use [OWASP's Juice Shop](https://github.com/juice-shop/juice-shop) web app running on Azure.

After setting up a free Azure account, I created an App Service `stc-owasp-juice` and deployed the containerised app:

`az webapp create --resource-group <my group> --plan <My plan> --name stc-owasp-juice --deployment-container-image bkimminich/juice-shop`

Deployment took about 5 minutes and port 3000 was recognised and translated to 80:
```
2025_06_05_lw1sdlwk0002JH_docker.log:
2025-06-05T13:36:56.3206509Z Container start method called.
2025-06-05T13:36:56.3517246Z Establishing network.
2025-06-05T13:36:56.3524568Z Pulling image: docker.io/bkimminich/juice-shop:latest.
2025-06-05T13:39:45.6028815Z Image docker.io/bkimminich/juice-shop:latest is pulled from registry docker.io
2025-06-05T13:39:45.6226775Z Container is starting.
2025-06-05T13:39:45.6303734Z Establishing user namespace if not established already.
2025-06-05T13:39:45.6677355Z Establishing network if not established already.
2025-06-05T13:39:45.6750832Z Mounting volumes.
2025-06-05T13:39:45.7230376Z Nested mountpoint volatile/logs
2025-06-05T13:39:45.7588733Z Nested mountpoint
2025-06-05T13:39:46.0199704Z Creating container.
2025-06-05T13:39:46.0207263Z Creating pipes for streaming container io.
2025-06-05T13:39:46.0282149Z Creating stdout named pipe at /podr/container/pipe/50b1370cc836_stc-owasp-juice/stdout_d24aecb26cba4e6a8b4436dfd880f88e.
2025-06-05T13:39:46.0305172Z Successfully created stdout named pipe at: /podr/container/pipe/50b1370cc836_stc-owasp-juice/stdout_d24aecb26cba4e6a8b4436dfd880f88e.
2025-06-05T13:39:46.0404499Z Opening named pipe /podr/container/pipe/50b1370cc836_stc-owasp-juice/stdout_d24aecb26cba4e6a8b4436dfd880f88e for reading in non-blocking mode.
2025-06-05T13:39:46.0483908Z Successfully opened named pipe: /podr/container/pipe/50b1370cc836_stc-owasp-juice/stdout_d24aecb26cba4e6a8b4436dfd880f88e.
2025-06-05T13:39:46.0496980Z Successfully removed non-blocking flag from /podr/container/pipe/50b1370cc836_stc-owasp-juice/stdout_d24aecb26cba4e6a8b4436dfd880f88e.
2025-06-05T13:39:46.0582650Z Creating stderr named pipe at /podr/container/pipe/50b1370cc836_stc-owasp-juice/stderr_c1f21e7b21d440d2a94cd9b9917f3881.
2025-06-05T13:39:46.0593142Z Successfully created stderr named pipe at: /podr/container/pipe/50b1370cc836_stc-owasp-juice/stderr_c1f21e7b21d440d2a94cd9b9917f3881.
2025-06-05T13:39:46.0687176Z Opening named pipe /podr/container/pipe/50b1370cc836_stc-owasp-juice/stderr_c1f21e7b21d440d2a94cd9b9917f3881 for reading in non-blocking mode.
2025-06-05T13:39:46.0777086Z Successfully opened named pipe: /podr/container/pipe/50b1370cc836_stc-owasp-juice/stderr_c1f21e7b21d440d2a94cd9b9917f3881.
2025-06-05T13:39:46.0980832Z Successfully removed non-blocking flag from /podr/container/pipe/50b1370cc836_stc-owasp-juice/stderr_c1f21e7b21d440d2a94cd9b9917f3881.
2025-06-05T13:39:48.0374199Z Container image exposes ports: 3000. Resolved main port to 3000.
2025-06-05T13:39:48.0384434Z Setting value of PORT variable to 3000
2025-06-05T13:39:48.0785188Z Creating container with image: docker.io/bkimminich/juice-shop:latest from registry: docker.io and fully qualified image name: docker.io/bkimminich/juice-shop:latest
2025-06-05T13:39:50.6419929Z Starting container: 50b1370cc836_stc-owasp-juice.
2025-06-05T13:39:51.5081598Z Starting watchers and probes.
2025-06-05T13:39:51.5673664Z Starting metrics collection.
2025-06-05T13:39:51.5997947Z Container is running.
2025-06-05T13:39:53.3973354Z Container start method finished after 177046 ms.
2025-06-05T13:41:12.3749281Z Site startup probe succeeded after 76.6326902 seconds.
2025-06-05T13:41:14.7200042Z Site started.
```

The app is running at `https://stc-owasp-juice-dnebatcgf2ddf4cr.uksouth-01.azurewebsites.net/#/`

![image](https://github.com/user-attachments/assets/1385a000-e991-4d2c-b50c-a75b0296aa82)
