# EX288 Practice Lab - Task66 Python Webserver

### Create Git workspace:

```bash
oc new-project task66
mkdir -p /home/student/git/task66-python-webserver
cd /home/student/git/task66-python-webserver

git init

# 3. Create Python Application

cat <<EOF > app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Welcome to phyton-webserver EX288 Lab"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
EOF

# 4. Create Python Requirements

cat <<EOF > requirements.txt
Flask==3.0.0
EOF


# 5. Commit Application Source

git config user.name "Student"
git config user.email "student@ocp4.example.com"

git add .

git commit -m "Initial Python webserver application"


# 6. Push Application to Git Server


git branch -M main

git remote add origin \
https://developer:d3v3lop3r@git.ocp4.example.com/developer/task66-python-webserver.git

git push -u origin main


# 7. Create External Build Script


mkdir -p /home/student/ex288/task66

cat <<EOF > /home/student/ex288/task66/code66.py
#!/usr/bin/env python3

print("================================")
print("code66.py executed successfully")
print("Post build task completed")
print("================================")
EOF

### Make executable:
chmod +x /home/student/ex288/task66/code66.py

# 8. Create ConfigMap for Build Script

## Since Git is read-only, store the script externally.

oc create configmap code66-script --from-file=/home/student/ex288/task66/code66.py

oc get configmap


# 9. Create Python S2I BuildConfig

oc new-build \
--strategy=source \
--image-stream=python:3.11-ubi9 \
https://developer:d3v3lop3r@git.ocp4.example.com/developer/task66-python-webserver.git \
--name=phyton-webserver

oc new-app phyton-webserver:latest
oc expose svc phyton-webserver

oc get all
````
