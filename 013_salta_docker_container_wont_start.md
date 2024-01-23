# "Salta": Docker container won't start.

**Description:** There's a "dockerized" Node.js web application in the `/home/admin/app` directory. Create a Docker container so you get a web app on port `:8888` and can curl to it. For the solution to be valid, there should be only one running Docker container.  

**Test:** `curl localhost:8888` returns `Hello World!` from a running container.  


---

### Solution:
#### 1. Reconnaissance on the server
