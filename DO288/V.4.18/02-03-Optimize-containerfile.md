<div align="center">

# 🔴 EX288 Task 2

## Build and Deploy an Application using Containerfile

![OpenShift](https://img.shields.io/badge/OpenShift-4.18-EE0000?logo=redhatopenshift&logoColor=white)
![Build](https://img.shields.io/badge/Containerfile-2496ED?logo=docker&logoColor=white)
![Project](https://img.shields.io/badge/Project-production2-7B42BC)
![Guide](https://img.shields.io/badge/Guide-Student_Ready-2EA44F)

</div>

---

## 🧪 How to Prepare the Lab?

Run these commands on the workstation as the `student` user. They download the
practice repository, initialize it as a Git repository, and push it to the lab
Git server.

> [!NOTE]
> Use these preparation commands on a fresh lab environment. Do not change the
> application files before attempting the task.

```bash
lab start deploy-introduction
oc login -u admin -p redhatocp https://api.ocp4.example.com:6443
mkdir -p /home/student/ex288/task2/apps/task2/python-webserver

cd /home/student/ex288/task2/apps/task2/python-webserver


cat <<EOF > Dockerfile
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest

LABEL org.opencontainers.image.title="Devopswala Task2 Web Server"
LABEL org.opencontainers.image.description="Simple Python HTTP Server running on OpenShift"
LABEL org.opencontainers.image.version="2.0"
LABEL org.opencontainers.image.vendor="Devopswala.com Training"
LABEL org.opencontainers.image.author="training@devopswala.com"

USER root

# Install Python and clean package cache
RUN microdnf install -y python3 && \
    microdnf clean all && \
    mkdir -p /devopswala

# Create custom application page
RUN echo '🚀 EX288 Task2 Application Running on OpenShiftCreated by: Devopswala.com Training' > /devopswala/index.html


ENV DOCROOT=/devopswala
ENV APPLICATION="EX288-Task2"

EXPOSE 8080

# OpenShift compatible non-root user
USER 1001

WORKDIR /devopswala

CMD ["python3", "-m", "http.server", "8080"]
EOF


mkdir src
cat <<EOF > src/index.html
<!DOCTYPE html>
<html>
<head>
  <title>2nd Image overriding</title>
</head>
<body>
  <h1>Task 2 Python Webserver </h1>
  <p>If you see this webpage then "COPY" worked correctly!</p>
</body>
</html>
EOF


cd /home/student/ex288/task2/


git init -b main
git checkout -b lab-pythonv3
git config user.name "Student"
git config user.email "student@ocp4.example.com"
git remote add origin https://developer:d3v3lop3r@git.ocp4.example.com/developer/task2-build.git

git add .
git commit -m "Add EX288 practice files for Task2"
git push -u origin lab-pythonv3
rm -rf /home/student/ex288/task2/apps/
```


## Question : 

## Question

Your task is to optimize the Containerfile available at:
	`https://git.ocp4.example.com/developer/task2-build.git` on branch **`lab-pythonv3`** 
- The Containerfile is located under: `apps/task2/python-webserver` 

- The final solution must satisfy the following requirements:

	- Deploy an application in the **`production2`** project and make it accessible through:

	`http://task2-webserver-production2.apps.ocp4.example.com`


	- The generated container image must support **image inheritance**, allowing it to be used as a parent image for child images.
	- Child images must be able to override the default application content by providing their own files from the `src/` directory.
	- The optimized Containerfile must produce an image that:
	  - Contains no more than **10 image layers**
	  - Has a final image size of less than or equal to **256 MiB**
	- The application must be successfully built, deployed, and accessible using the created container image.


### How to clear the lab ?
```
oc delete project production2
rm -rf rm -rf /home/student/ex288/task2/
https://git.ocp4.example.com/developer/task2-build/edit#js-project-advanced-settings
developer/task2-build
```
