# netflix-clone-on-kubernetes
![Screenshot 2024-03-18 220830](https://github.com/Eric-Kay/netflix-clone-on-kubernetes/assets/126447235/a37b6ccf-17cf-4505-affc-6c6bd399513b)

Step 1 — Launch an Ubuntu(22.04) T2 Large Instance

Step 2 — Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.

Step 3 — Create a TMDB API Key.

Step 4 — Install Prometheus and Grafana On the new Server.

Step 5 — Install the Prometheus Plugin and Integrate it with the Prometheus server.

Step 6 — Email Integration With Jenkins and Plugin setup.

Step 7 — Install Plugins like JDK, Sonarqube Scanner, Nodejs, and OWASP Dependency Check.

Step 8 — Create a Pipeline Project in Jenkins using a Declarative Pipeline

Step 9 — Install OWASP Dependency Check Plugins

Step 10 — Docker Image Build and Push

Step 11 — Deploy the image using Docker

Step 12 — Kubernetes master and slave setup on Ubuntu (20.04)

Step 13 — Access the Netflix app on the Browser.

Step 14 — Terminate the AWS EC2 Instances.

Now, let’s get started and dig deeper into each of these steps:-

STEP1:Launch an Ubuntu(22.04) T2 Large Instance

```bash
docker build -t netflix .
docker run -d --name netflix -p 8081:80 netflix:latest

#to delete
docker stop <containerid>
docker rmi -f netflix
```
