<div align="center">

# 🔴 EX288 Task 1

## Build and Deploy an Application using S2I

![OpenShift](https://img.shields.io/badge/OpenShift-4.18-EE0000?logo=redhatopenshift&logoColor=white)
![Build](https://img.shields.io/badge/Source2Image_Strategy-2496ED?logo=docker&logoColor=white)
![Project](https://img.shields.io/badge/Project-production1-7B42BC)
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
oc login -u developer -p developer https://api.ocp4.example.com:6443
mkdir -p /home/student/ex288/task1/
cd /home/student/ex288/task1/

mkdir -p apps/task1/helloworld
cd apps/task1/helloworld


cat <<EOF > package.json
{
  "name": "nodejs-helloworld",
  "version": "1.0.0",
  "description": "Hello World!",
  "main": "devopswala.js",
  "scripts": {
    "start": "node devopswala.js"
  },
  "author": "Red Hat Training from Devopswala",
  "license": "ASL",
  "dependencies": {
    "express" "^4.20.0"
  }
}
EOF

cat <<EOF > devopswala.js
const express = require('express');
const app = express();

app.get('/', function (req, res) {
  res.send('Hello World!\n');
});

app.listen(8080, function () {
  console.log('Devopswala app listening on port 8080!');
});
EOF

cd /home/student/ex288/task1/

git init -b main
git checkout -b secure-api
git config user.name "Student"
git config user.email "student@ocp4.example.com"
git remote add origin https://developer:d3v3lop3r@git.ocp4.example.com/developer/task1-nodejs-helloworld.git
git add .
git commit -m "Add EX288 practice files"
git push -u origin secure-api
````

---

# Q1 — Build and Deploy `todo-ssr` on OpenShift

You are a developer working on an OpenShift cluster.

- The application must be built and deployed from the source code at:
  
       https://git.ocp4.example.com/developer/task1-nodejs-helloworld.git
- The application source code is located in the subdirectory: `apps/task1/helloworld`
- The Git Branch is **`secure-api`**
- The application must be deployed to the project `production1`
- The deployed application and its resources must be named `todo-ssr`
- The application must be based on the image stream tag `nodejs:18-ubi9`
- The application's dependencies are available at: `http://nexus-infra.apps.ocp4.example.com/repository/npm`
- The Git server requires authentication using the following credentials:
```
Username: developer
Password: d3v3lop3r
```
- The Git credentials must be stored as a secret and made available to the build process.
- The application must be accessible using both:
    - http://todo-ssr-production1.apps.ocp4.example.com
    - https://todo-ssr-production1.apps.ocp4.example.com

### Hint

- The npm dependency repository can be passed to the build environment using **`npm_config_registry`**

---
<details>
<summary><strong>✅ Show the complete solution and explanation</strong></summary>

## 1. Login to OpenShift

Login using the developer account:

```bash
oc login -u developer -p developer https://api.ocp4.example.com:6443
```

---

# 2. Switch to the Required Project

Change to the `production1` project:

```bash
oc project production1
```

---

# 3. Create Git Authentication Secret

Create a secret containing Git credentials:

```bash
oc create secret generic gitlab-secret \
--type=kubernetes.io/basic-auth \
--from-literal=username=developer \
--from-literal=password=d3v3lop3r
```

Verify:

```bash
oc get secrets
```

Expected:

```
gitlab-secret
```

<img width="1842" height="1037" alt="Openshift Documentation" src="https://github.com/user-attachments/assets/b4ef4332-5507-4e54-b141-c2c5fdbc51f5" />




---

# 4. Add Git URL Matching Annotation

Annotate the secret so OpenShift can automatically associate it with the Git repository.

```bash
oc annotate secret gitlab-secret \
"build.openshift.io/source-secret-match-uri-1=https://git.ocp4.example.com/*"
```

Verify:

```bash
oc describe secret gitlab-secret
```

Expected:

```
Annotations:
  build.openshift.io/source-secret-match-uri-1: https://git.ocp4.example.com/*
```

---

# 5. Link Secret to Builder Service Account

The S2I build runs using the `builder` service account.

Link the secret:

```bash
oc secrets link builder gitlab-secret
```

Verify:

```bash
oc describe serviceaccount builder
```

Expected:

```
Mountable secrets:
  builder-dockercfg-xxxxx
  gitlab-secret
```

---

# 6. Create the Application

Create the application using S2I:

```bash
oc new-app \
--name=todo-ssr \
--build-env npm_config_registry=http://nexus-infra.apps.ocp4.example.com/repository/npm \
nodejs:18-ubi9~https://git.ocp4.example.com/developer/task1-nodejs-helloworld.git#secure-api \
--context-dir=apps/task1/helloworld
```

This command creates:

* BuildConfig
* ImageStream
* Deployment
* Service

<img width="1864" height="1037" alt="Openshift Doc for (--build-env) " src="https://github.com/user-attachments/assets/5b13bc76-9420-444b-ae72-0ee322dd9a8a" />

---

# 7. Monitor the Build

Check build status:

```bash
oc get builds
```
### If it failed then....


```yaml
[student@workstation task1-nodejs-helloworld]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                   READY   STATUS   RESTARTS   AGE
pod/todo-ssr-1-build   0/1     Error    0          2m4s

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/todo-ssr   ClusterIP   172.30.212.25   <none>        8080/TCP   2m4s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/todo-ssr   0/1     0            0           2m4s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/todo-ssr-67f7dfd5f8   1         0         0       2m4s

NAME                                      TYPE     FROM   LATEST
buildconfig.build.openshift.io/todo-ssr   Source   Git    1

NAME                                  TYPE     FROM          STATUS                        STARTED         DURATION
build.build.openshift.io/todo-ssr-1   Source   Git@1d78f42   Failed (GenericBuildFailed)   2 minutes ago   21s

NAME                                      IMAGE REPOSITORY                                                                    TAGS   UPDATED
imagestream.image.openshift.io/todo-ssr   default-route-openshift-image-registry.apps.ocp4.example.com/production1/todo-ssr          
```

### Check the logs of pods or build.
```
[student@workstation task1-nodejs-helloworld]$ oc logs pods/todo-ssr-1-build 
Defaulted container "sti-build" out of: sti-build, git-clone (init), manage-dockerfile (init)
Adding cluster TLS certificate authority to trust store
time="2026-09-28T17:01:52Z" level=info msg="Not using native diff for overlay, this may cause degraded performance for building images: kernel has CONFIG_OVERLAY_FS_REDIRECT_DIR enabled"
I0928 17:01:52.300642       1 defaults.go:112] Defaulting to storage driver "overlay" with options [mountopt=metacopy=on].
Caching blobs under "/var/cache/blobs".
Trying to pull image-registry.openshift-image-registry.svc:5000/openshift/nodejs@sha256:5c63b1bcde0f0b7de50cd5bcbbe95c3e091c854bc5140a71893e77110d5f1ccc...
Getting image source signatures
Copying blob sha256:1540db9d0f7617b3e726dcc50dbddda620d399076641b79e067d76e46cc8510e
Copying blob sha256:92efcdccd1058003df257e9cbdf756ff6b10bd276551590536a5a1678e099aaf
Copying blob sha256:32cb216dbd98f1b9558c1726079aa254820ad0ae8c0cae13549574f31ca77d64
Copying config sha256:9006b5d7d3ccaa8205ea30eae79fbb9ade5c1b204b3947219c841ea900c7e3e1
Writing manifest to image destination
Generating dockerfile with builder image image-registry.openshift-image-registry.svc:5000/openshift/nodejs@sha256:5c63b1bcde0f0b7de50cd5bcbbe95c3e091c854bc5140a71893e77110d5f1ccc
Adding transient rw bind mount for /run/secrets/rhsm
STEP 1/9: FROM image-registry.openshift-image-registry.svc:5000/openshift/nodejs@sha256:5c63b1bcde0f0b7de50cd5bcbbe95c3e091c854bc5140a71893e77110d5f1ccc
STEP 2/9: LABEL "io.openshift.build.commit.id"="1d78f42c4cef86c80b7323bc8d537083784d5fa8"       "io.openshift.build.commit.ref"="secure-api"       "io.openshift.build.commit.message"="hi"       "io.openshift.build.image"="image-registry.openshift-image-registry.svc:5000/openshift/nodejs@sha256:5c63b1bcde0f0b7de50cd5bcbbe95c3e091c854bc5140a71893e77110d5f1ccc"       "io.openshift.build.commit.author"="Student User <student@workstation.lab.example.com>"       "io.openshift.build.commit.date"="Mon Sep 28 13:00:59 2026 -0400"
STEP 3/9: ENV OPENSHIFT_BUILD_NAME="todo-ssr-1"     OPENSHIFT_BUILD_NAMESPACE="production1"     OPENSHIFT_BUILD_SOURCE="https://git.ocp4.example.com/developer/task1-nodejs-helloworld.git"     OPENSHIFT_BUILD_COMMIT="1d78f42c4cef86c80b7323bc8d537083784d5fa8"     npm_config_registry="http://nexus-infra.apps.ocp4.example.com/repository/npm"
STEP 4/9: USER root
STEP 5/9: COPY upload/src /tmp/src
STEP 6/9: RUN chown -R 1001:0 /tmp/src
STEP 7/9: USER 1001
STEP 8/9: RUN /usr/libexec/s2i/assemble
---> Installing application source ...
---> Installing all dependencies
npm error code EJSONPARSE
npm error path /opt/app-root/src/package.json.################### This is the error
npm error JSON.parse Unexpected string in JSON at position 271 while parsing near "...s\": {\n    \"express\" \"^4.20.0\"\n  }\n}\n"
npm error JSON.parse Failed to parse JSON data.
npm error JSON.parse Note: package.json must be actual JSON, not just JavaScript.
npm error A complete log of this run can be found in: /opt/app-root/src/.npm/_logs/2026-09-28T17_02_04_240Z-debug-0.log
error: build error: building at STEP "RUN /usr/libexec/s2i/assemble": while running runtime: exit status 1
[student@workstation task1-nodejs-helloworld]$ 
```


### From the above output, one can get to know that issue is with "package.json" file.

```bash
[student@workstation task1-nodejs-helloworld]$ git clone  https://git.ocp4.example.com/developer/task1-nodejs-helloworld.git
Cloning into 'task1-nodejs-helloworld'...
Username for 'https://git.ocp4.example.com': developer
Password for 'https://developer@git.ocp4.example.com': 

[student@workstation task1-nodejs-helloworld]$ cd task1-nodejs-helloworld/

[student@workstation task1-nodejs-helloworld]$ python3 -m json.tool package.json 
Expecting ':' delimiter: line 12 column 15 (char 271)

[student@workstation task1-nodejs-helloworld]$ cat -n package.json 
     1  {
     2    "name": "nodejs-helloworld",
     3    "version": "1.0.0",
     4    "description": "Hello World!",
     5    "main": "devopswala.js",
     6    "scripts": {
     7      "start": "node devopswala.js"
     8    },
     9    "author": "Red Hat Training from Devopswala",
    10    "license": "ASL",
    11    "dependencies": {
    12      "express" "^4.20.0".  ##### >>>>> ":" Colon is missing.
    13    }
    14  }
[student@workstation task1-nodejs-helloworld]$ 

[student@workstation task1-nodejs-helloworld]$ vim package.json 
[student@workstation task1-nodejs-helloworld]$ python3 -m json.tool package.json 
{
    "name": "nodejs-helloworld",
    "version": "1.0.0",
    "description": "Hello World!",
    "main": "devopswala.js",
    "scripts": {
        "start": "node devopswala.js"
    },
    "author": "Red Hat Training from Devopswala",
    "license": "ASL",
    "dependencies": {
        "express": "^4.20.0"
    }
}
[student@workstation task1-nodejs-helloworld]$ 

[student@workstation task1-nodejs-helloworld]$ git add . ; git commit -m "hi" ; git push 

[student@workstation task1-nodejs-helloworld]$ oc start-build bc/todo-ssr --follow 

[student@workstation task1-nodejs-helloworld]$ oc get all
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                            READY   STATUS      RESTARTS   AGE
pod/todo-ssr-1-build            0/1     Error       0          7m42s
pod/todo-ssr-2-build            0/1     Completed   0          32s
pod/todo-ssr-7d7cdfd7f4-mtxk2   1/1     Running     0          7s

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/todo-ssr   ClusterIP   172.30.212.25   <none>        8080/TCP   7m42s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/todo-ssr   1/1     1            1           7m42s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/todo-ssr-67f7dfd5f8   0         0         0       7m42s
replicaset.apps/todo-ssr-7d7cdfd7f4   1         1         1       7s

NAME                                      TYPE     FROM   LATEST
buildconfig.build.openshift.io/todo-ssr   Source   Git    2

NAME                                  TYPE     FROM          STATUS                        STARTED          DURATION
build.build.openshift.io/todo-ssr-1   Source   Git@1d78f42   Failed (GenericBuildFailed)   7 minutes ago    21s
build.build.openshift.io/todo-ssr-2   Source   Git@90a702c   Complete                      32 seconds ago   27s

NAME                                      IMAGE REPOSITORY                                                                    TAGS     UPDATED
imagestream.image.openshift.io/todo-ssr   default-route-openshift-image-registry.apps.ocp4.example.com/production1/todo-ssr   latest   7 seconds ago


[student@workstation task1-nodejs-helloworld]$ oc create route edge --service=todo-ssr --hostname=todo-ssr-production1.apps.ocp4.example.com --insecure-policy=Allow
route.route.openshift.io/todo-ssr created

[student@workstation task1-nodejs-helloworld]$ curl https://todo-ssr-production1.apps.ocp4.example.com
Hello World!
[student@workstation task1-nodejs-helloworld]$ curl http://todo-ssr-production1.apps.ocp4.example.com
Hello World!
[student@workstation task1-nodejs-helloworld]$
```

# Direct commands.

```bash
oc create secret generic gitlab-secret --type=kubernetes.io/basic-auth --from-literal=username=developer --from-literal=password=d3v3lop3r
oc annotate secret gitlab-secret "build.openshift.io/source-secret-match-uri-1=https://git.ocp4.example.com/*"
oc secrets link builder gitlab-secret
oc new-app --name=todo-ssr --build-env npm_config_registry=http://nexus-infra.apps.ocp4.example.com/repository/npm nodejs:18-ubi9~https://git.ocp4.example.com/developer/task1-nodejs-helloworld.git 
oc get pods
oc get all
oc logs pods/todo-ssr-1-build 
git clone  https://git.ocp4.example.com/developer/task1-nodejs-helloworld.git
ls -ltr
cd task1-nodejs-helloworld/
python3 -m json.tool package.json 
cat -n package.json 
vim package.json 
python3 -m json.tool package.json 
git add . ; git commit -m "hi" ; git push 
oc start-build bc/todo-ssr --follow 
oc get all
oc create route edge --service=todo-ssr --hostname=todo-ssr-production1.apps.ocp4.example.com --insecure-policy=Allow
curl https://todo-ssr-production1.apps.ocp4.example.com
curl http://todo-ssr-production1.apps.ocp4.example.com
```




Here is **Part 2/3**. Continue copying this immediately after:

```markdown
<details>
<summary><strong>✅ Show the complete solution and explanation</strong></summary>
```

---

```markdown
# 🚀 Solution

---

# 1. Login to OpenShift

Login using the developer account:

```bash
oc login -u developer -p developer https://api.ocp4.example.com:6443
```

---

# 2. Switch to the Required Project

Change to the `production1` project:

```bash
oc project production1
```

---

# 🔐 3. Create Git Authentication Secret

The Git repository requires authentication. Create a secret containing the Git credentials.

```bash
oc create secret generic gitlab-secret \
--type=kubernetes.io/basic-auth \
--from-literal=username=developer \
--from-literal=password=d3v3lop3r
```

---

## Verify Secret Creation

```bash
oc get secrets
```

Expected:

```text
gitlab-secret
```

---

# 🔗 4. Add Git URL Matching Annotation

OpenShift uses the annotation below to automatically associate the secret with the matching Git repository URL.

```bash
oc annotate secret gitlab-secret \
"build.openshift.io/source-secret-match-uri-1=https://git.ocp4.example.com/*"
```

---

## Verify Annotation

```bash
oc describe secret gitlab-secret
```

Expected:

```text
Annotations:
  build.openshift.io/source-secret-match-uri-1: https://git.ocp4.example.com/*
```

---

# 🔑 5. Link Secret to Builder Service Account

The S2I build process runs using the `builder` service account.

Link the Git secret:

```bash
oc secrets link builder gitlab-secret
```

---

## Verify Builder Service Account

```bash
oc describe serviceaccount builder
```

Expected:

```text
Mountable secrets:
  builder-dockercfg-xxxxx
  gitlab-secret
```

---

# 🚀 6. Create the Application

Create the application using Source-to-Image (S2I).

```bash
oc new-app \
--name=todo-ssr \
--build-env npm_config_registry=http://nexus-infra.apps.ocp4.example.com/repository/npm \
nodejs:18-ubi9~https://git.ocp4.example.com/developer/task1-nodejs-helloworld.git#secure-api \
--context-dir=apps/task1/helloworld
```

---

## Application Resources Created

The above command creates:

| Resource | Purpose |
|---|---|
| BuildConfig | Defines the source build process |
| ImageStream | Stores generated container images |
| Deployment | Runs application pods |
| Service | Provides internal application access |

---

# 📊 7. Monitor the Build

Check the build status:

```bash
oc get builds
```

Example:

```text
NAME          TYPE     FROM       STATUS
todo-ssr-1    Source   Git        Running
```

---

Check all application resources:

```bash
oc get all
```

---

# 🛠️ 8. Troubleshooting Build Failure

If the build fails:

Check the build pod logs:

```bash
oc logs pods/todo-ssr-1-build
```

Look for application build errors.

Common build problems include:

- Invalid application files
- Dependency installation failures
- Incorrect Git source configuration
- Incorrect build environment variables

---

# 🔍 9. Verify Build Logs

Example S2I build flow:

```text
Installing application source ...

Installing all dependencies

Building application image

Successfully built image
```

A successful build should complete with:

```text
Build completed successfully
```

---


## How to clear the lab?
```
oc delete  all -l app=todo-ssr
```
