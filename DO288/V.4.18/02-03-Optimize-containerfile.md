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

Your task is to optimize the Dockerfile available at:
	`https://git.ocp4.example.com/developer/task2-build.git` on branch **`lab-pythonv3`** 
- The Containerfile is located under: `apps/task2/python-webserver` 

- The final solution must satisfy the following requirements:

	- Deploy an application in the **`production2`** project and make it accessible through:

	`http://task2-webserver-production2.apps.ocp4.example.com`


	- The generated Docker image must support **image inheritance**, allowing it to be used as a parent image for child images.
	- Child images must be able to override the default application content by providing their own files from the `src/` directory.
	- The optimized Dockerfile must produce an image that:
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













---

# ✅ Solution

## Step 1: Create Project

Create the OpenShift project where the application will be deployed.

```bash
oc new-project production2
```

Verify:

```bash
oc project
```

Expected:

```
Using project "production2"
```

---

# Step 2: Clone Application Repository

Clone the Git repository containing the Dockerfile.

```bash
git clone https://git.ocp4.example.com/developer/task2-build.git

cd task2-build/apps/task2/python-webserver/
```

Verify files:

```bash
ls -l
```

Expected:

```
Dockerfile
src/
```

---

# Step 3: Optimize Dockerfile

The goal is:

- Image size <= 256 MiB
- Maximum 7 layers
- Support parent-child image inheritance

The optimized Dockerfile:

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest

LABEL org.opencontainers.image.title="Devopswala Task2 Web Server" \
      org.opencontainers.image.description="Reusable Python HTTP Server Parent Image" \
      org.opencontainers.image.version="2.0" \
      org.opencontainers.image.vendor="Devopswala.com Training"


USER root


RUN microdnf install -y python3 && \
    microdnf clean all && \
    mkdir -p /devopswala && \
    echo "Default Parent Image Content" > /devopswala/index.html


ENV DOCROOT=/devopswala \
    APPLICATION="EX288-Task2"


EXPOSE 8080


USER 1001


WORKDIR /devopswala


# Child images will execute this instruction
# when they inherit this image.
ONBUILD COPY src/ /devopswala/


CMD ["python3", "-m", "http.server", "8080"]
```

---

# Understanding ONBUILD COPY

`ONBUILD` stores an instruction inside the parent image.

The instruction does **not execute while building the parent image**.

It executes when another image uses:

```dockerfile
FROM parent-image
```

Example:

```
Parent Image
     |
     |
     +---- ONBUILD COPY src/
                  |
                  |
                  v
          Child Image Build
```

Memory:

```
COPY = Execute now

ONBUILD COPY = Execute when child image is created
```

---

# Step 4: Build and Validate Parent Image Locally

Build the image:

```bash
podman build --format docker -t test-image .
```

Check image:

```bash
podman image ls
```

Example:

```
REPOSITORY          TAG       SIZE
test-image          latest    <256MB
```

---

## Verify Image Layers

Requirement:

```
Maximum 7 layers
```

Check:

```bash
podman inspect test-image \
--format '{{len .RootFS.Layers}}'
```

Example:

```
5
```

The result must be:

```
<= 7
```

---

## Verify Image History

```bash
podman history test-image:latest
```

Review:

- RUN instructions
- Layer count
- Image optimization
```

# Part 2/3

```markdown
# Step 5: Commit Optimized Dockerfile to Git

After modifying the Dockerfile, push the changes to the Git repository.

Check status:

```bash
git status
```

Add changes:

```bash
git add .
```

Commit:

```bash
git commit -m "Optimize Dockerfile with ONBUILD COPY support"
```

Push to branch:

```bash
git push origin lab-pythonv3
```

Verify:

```bash
git log --oneline
```

---

# Step 6: Configure Git Authentication Secret

OpenShift requires Git credentials to clone the private repository.

Create a secret:

```bash
oc create secret generic git-secret \
--type=kubernetes.io/basic-auth \
--from-literal=username=developer \
--from-literal=password=d3v3lop3r
```

Verify:

```bash
oc get secret git-secret
```

---

## Add Git Secret Annotation

The annotation allows OpenShift builds to automatically select this secret for matching Git repositories.

```bash
oc annotate secret git-secret \
build.openshift.io/source-secret-match-uri=https://git.ocp4.example.com/*
```

Verify:

```bash
oc describe secret git-secret
```

Expected:

```text
Annotations:
  build.openshift.io/source-secret-match-uri=https://git.ocp4.example.com/*
```

---

## Link Secret with Builder Service Account

The builder service account must have access to the Git secret.

```bash
oc secrets link builder git-secret
```

Verify:

```bash
oc describe sa builder
```

Expected:

```text
Mountable secrets:
  git-secret
```

---

# Step 7: Create Parent Image Build

The first build creates the reusable parent image.

The parent image contains:

- Python runtime
- HTTP server
- Default application content
- ONBUILD trigger

Create the build configuration:

```bash
oc new-build \
--name=parent-image \
--strategy=docker \
--source-secret=git-secret \
https://developer:d3v3lop3r@git.ocp4.example.com/developer/task2-build.git#lab-pythonv3 \
--context-dir=apps/task2/python-webserver
```

> Note:
> The Git credentials are included in the URL because `oc new-build` validates the source repository before creating the BuildConfig. The `source-secret` is still used by the build process.

---

# Step 8: Start Parent Image Build

Start the build:

```bash
oc start-build parent-image --follow
```

Monitor:

```bash
oc get builds
```

Expected:

```text
NAME              TYPE       STATUS
parent-image-1    Docker     Complete
```

---

# Step 9: Verify Parent Image

Check ImageStream:

```bash
oc get imagestream
```

Expected:

```text
NAME
parent-image
```

Inspect:

```bash
oc describe imagestream parent-image
```

You should see the generated image reference.

---

# Step 10: Deploy Parent Image (Validation)

Create deployment:

```bash
oc new-app parent-image:latest
```

Verify:

```bash
oc get pods
```

Expected:

```text
NAME                              READY   STATUS
parent-image-xxxxxx               1/1     Running
```

---

Create service route:

```bash
oc expose service parent-image \
--hostname parent-image-production2.apps.ocp4.example.com
```

Test:

```bash
curl http://parent-image-production2.apps.ocp4.example.com
```

Expected output:

```text
Default Parent Image Content
```

---

# Step 11: Create Child Image

The purpose of this step is to prove that the parent image can be reused.

The child image uses:

```dockerfile
FROM parent-image
```

The child image does not install Python again.

It only provides new application content.

---

Create a child Dockerfile:

```bash
cat <<EOF > Dockerfile
FROM image-registry.openshift-image-registry.svc:5000/production2/parent-image
EOF
```

The parent image contains:

```dockerfile
ONBUILD COPY src/ /devopswala/
```

Therefore during the child image build:

```text
Child Build
    |
    |
    v
FROM parent-image
    |
    |
    v
ONBUILD COPY executes
    |
    |
    v
src/index.html replaces default content
```

---

# Step 12: Add Child Application Content

Create:

```bash
mkdir -p src
```

Create child content:

```bash
cat <<EOF > src/index.html
<!DOCTYPE html>
<html>
<head>
<title>Child Image Override</title>
</head>

<body>

<h1>This content is from the child image</h1>

<p>
The ONBUILD COPY instruction successfully replaced the parent content.
</p>

</body>
</html>
EOF
```

---

# Step 13: Commit Child Image Changes

```bash
git add .
```

Commit:

```bash
git commit -m "Create child image using parent image inheritance"
```

Push:

```bash
git push origin lab-pythonv3
```

---

# Step 14: Create Child Image Build

Create the final application image:

```bash
oc new-build \
--name=task2-webserver \
--strategy=docker \
--source-secret=git-secret \
https://developer:d3v3lop3r@git.ocp4.example.com/developer/task2-build.git#lab-pythonv3 \
--context-dir=apps/task2/python-webserver
```

```

# Part 3/3

```markdown id="94n5n6"
# Step 15: Start Child Image Build

Start the Docker build:

```bash
oc start-build task2-webserver --follow
```

Monitor the build:

```bash
oc get builds
```

Expected:

```text
NAME                  TYPE       STATUS
task2-webserver-1     Docker     Complete
```

---

# Step 16: Verify Child Image Deployment

Deploy the final application image:

```bash
oc new-app task2-webserver:latest
```

Verify resources:

```bash
oc get all
```

Expected resources:

```text
pod/task2-webserver-xxxxx

deployment.apps/task2-webserver

service/task2-webserver
```

Check pod status:

```bash
oc get pods
```

Expected:

```text
NAME                           READY   STATUS
task2-webserver-xxxxx          1/1     Running
```

---

# Step 17: Create Application Route

Expose the application using the required hostname:

```bash
oc expose service task2-webserver \
--hostname task2-webserver-production2.apps.ocp4.example.com
```

Verify route:

```bash
oc get route
```

Expected:

```text
NAME                HOST/PORT
task2-webserver     task2-webserver-production2.apps.ocp4.example.com
```

---

# Step 18: Test Application

Access the application:

```bash
curl http://task2-webserver-production2.apps.ocp4.example.com
```

Expected output:

```html
<!DOCTYPE html>
<html>

<h1>This content is from the child image</h1>

<p>
The ONBUILD COPY instruction successfully replaced the parent content.
</p>

</html>
```

This confirms:

✅ Parent image was created  
✅ Child image inherited from parent image  
✅ ONBUILD COPY executed successfully  
✅ Child content replaced parent default content  

---

# Step 19: Validate Final Image Requirements

## Check Image Size

```bash
oc describe imagestream task2-webserver
```

Example:

```text
Image Size: 60 MB
```

Requirement:

```
<= 256 MiB
```

---

## Check Image Layers

The image must contain:

```
<= 7 layers
```

Using Podman:

```bash
podman pull \
image-registry.openshift-image-registry.svc:5000/production2/task2-webserver:latest
```

Check:

```bash
podman inspect task2-webserver \
--format '{{len .RootFS.Layers}}'
```

Expected:

```text
7
```

or less.

---

# Step 20: Troubleshooting Guide

## Problem: Build cannot clone Git repository

Error:

```text
failed to fetch requested repository
```

Check:

```bash
oc describe secret git-secret
```

Verify:

```text
Type: kubernetes.io/basic-auth
```

Data should contain:

```text
username
password
```

---

## Problem: ONBUILD COPY is not executing

Check the parent image:

```bash
podman inspect parent-image | grep -i onbuild
```

The Dockerfile must contain:

```dockerfile
ONBUILD COPY src/ /devopswala/
```

Do not use:

```dockerfile
#ONBUILD COPY
```

because it is commented.

---

## Problem: ImagePullBackOff

Check image reference:

```bash
oc describe pod <pod-name>
```

Common mistake:

```bash
oc create deployment app --image=localhost/image
```

OpenShift nodes cannot access workstation localhost images.

Use:

```bash
image-registry.openshift-image-registry.svc:5000/<project>/<image>
```

or use ImageStreams:

```bash
oc new-app image-name:latest
```

---

## Problem: ONBUILD ignored during Podman build

You may see:

```text
ONBUILD is not supported for OCI image format
```

Build using Docker format:

```bash
podman build \
--format docker \
-t parent-image .
```

---

# EX288 Task 2 Final Validation Checklist

| Requirement | Validation |
|---|---|
| Application deployed in production2 | `oc project` |
| Git repository used | `oc describe bc` |
| Dockerfile optimized | `podman history` |
| Image size <=256 MiB | `oc describe is` |
| Layers <=7 | `podman inspect` |
| Parent image created | `oc get is parent-image` |
| Child image inheritance | `FROM parent-image` |
| ONBUILD COPY works | Child page displayed |
| Application accessible | `curl route-url` |

---

# Cleanup

Remove application resources:

```bash
oc delete all -l app=task2-webserver
```

Remove images:

```bash
oc delete imagestream parent-image
oc delete imagestream task2-webserver
```

Remove project (optional):

```bash
oc delete project production2
```

---

# Key EX288 Memory Notes

## Dockerfile inheritance

Remember:

```
Parent Image
     |
     |
     v
ONBUILD COPY
     |
     |
     v
Child Image provides files
```

---

## COPY vs ONBUILD COPY

| Instruction | Execution Time |
|---|---|
| COPY | During current image build |
| ONBUILD COPY | During child image build |

Memory shortcut:

```
COPY = My files now

ONBUILD COPY = My child files later
```

---

## Complete Task Flow

```
Git Repository
      |
      |
      v
Optimize Dockerfile
      |
      |
      v
Build Parent Image
      |
      |
      v
Child Image FROM Parent
      |
      |
      v
ONBUILD COPY executes
      |
      |
      v
Deploy Application
      |
      |
      v
Expose Route
```

The student has successfully completed an EX288-style container image inheritance and deployment workflow.
```
