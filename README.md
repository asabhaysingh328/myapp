# myapp

**Jenkins pipeline script is a classic CI/CD flow: it pulls code from GitHub, builds a Docker image, pushes it to Docker Hub, and then deploys it into Kubernetes (Minikube). GitHub’s role here is to host  application source code (`myapp` repo), which Jenkins clones at the start of every run. Docker Hub is used as the container registry, and Kubernetes is the runtime environment where the app is deployed and scaled.**

---

## 🔹 Step-by-Step Explanation of  Script

### 1. **Environment Variables**
```groovy
def KUBECONFIG   = "$HOME/.kube/config"
def REPO_URL     = "https://github.com/asabhaysingh328/myapp.git"
def APP_NAME     = "myapp"
def NAMESPACE    = "cicd-demo"
def DOCKER_IMAGE = "asabhaysing328/asabhaysing328:latest"
```
- Defines paths and names used throughout the pipeline.
- `REPO_URL` points to your GitHub repo containing the app code.
- `DOCKER_IMAGE` is the image name/tag in Docker Hub.

---

### 2. **Clone Repository**
```groovy
git branch: 'main', url: REPO_URL
```
- Jenkins pulls the latest code from GitHub (`main` branch).
- This ensures every build uses the newest source.

---

### 3. **Create Namespace**
```groovy
kubectl get ns cicd-demo || kubectl create namespace cicd-demo
```
- Checks if Kubernetes namespace `cicd-demo` exists.
- Creates it if missing, isolating your app’s resources.

---

### 4. **Build Docker Image**
```groovy
docker build -t myapp:latest .
```
- Builds a Docker image from the repo’s `Dockerfile`.
- Tags it as `myapp:latest`.

---

### 5. **Push to Docker Hub**
```groovy
withCredentials([usernamePassword(credentialsId: 'docker', ...)]) {
    docker login
    docker tag myapp:latest asabhaysingh328/asabhaysingh328:latest
    docker push asabhaysingh328/asabhaysingh328:latest
}
```
- Logs into Docker Hub using Jenkins credentials.
- Tags the image with your Docker Hub repo name.
- Pushes the image so Kubernetes can pull it later.

---

### 6. **Deploy Application**
```groovy
kubectl create deployment myapp --image=DOCKER_IMAGE -n cicd-demo
kubectl expose deployment myapp --port=80 --target-port=80 -n cicd-demo
```
- Creates a Kubernetes Deployment using the pushed image.
- Exposes it as a Service on port 80.

---

### 7. **Verification & Scaling**
- **Verify Pods:** `kubectl get pods`
- **Scale:** `kubectl scale deployment myapp --replicas=10`
- **Rolling Restart:** `kubectl rollout restart deployment/myapp`
- **Rollout Status:** `kubectl rollout status deployment/myapp`
- **Health Check:** `kubectl get deploy myapp`
- **Logs:** `kubectl logs -l app=myapp`
- **Final Verification:** `kubectl get pods`

These stages confirm the app is running, scale it to 10 pods, restart it safely, and check logs/health.

---

## 🔹 GitHub’s Role Here
- **Hosts your source code** (`myapp` repo).
- Jenkins pulls from GitHub at the start of every pipeline run.
- Any commit to `main` can trigger a new build/deploy cycle.

Your repo: **asabhaysingh328/myapp [(github.com in Bing)](https://www.bing.com/search?q="https%3A%2F%2Fgithub.com%2Fasabhaysingh328%2Fmyapp")** is a fork of another sample app, and it contains the Dockerfile and app code needed for this pipeline.

---

## 🔹 Summary
- **GitHub** → Source code repository.  
- **Jenkins** → CI/CD orchestrator.  
- **Docker Hub** → Container registry.  
- **Kubernetes (Minikube)** → Deployment environment.  

---
## Symbols Description for Jenkins File 

Got it, Abhay — let’s decode the highlighted symbols in your Jenkins pipeline script:

---

## 🔹 Triple Single Quotes (`''' ... '''`)
- In Groovy (and Jenkins pipeline), triple quotes are used for **multi‑line strings**.
- Example:
  ```groovy
  sh '''
  kubectl get ns cicd-demo || kubectl create namespace cicd-demo
  '''
  ```
- This lets you write shell commands across multiple lines without worrying about escaping characters.
- It’s especially useful when you want to run several commands in one `sh` block.

---

## 🔹 Double Pipe (`||`)
- This is a **logical OR operator** in shell scripting.
- Meaning: run the second command **only if the first one fails**.
- Example:
  ```bash
  kubectl get ns cicd-demo || kubectl create namespace cicd-demo
  ```
  - If the namespace exists → `kubectl get ns cicd-demo` succeeds, so the second command is skipped.
  - If the namespace doesn’t exist → the first command fails, so `kubectl create namespace cicd-demo` runs.

---

## 🔹 Backslash (`\`)
- Used in shell scripts to **continue a command onto the next line**.
- Example:
  ```bash
  kubectl get deploy nginx-demo -n cicd-demo || \
  kubectl create deployment nginx-demo1 --image=nginx -n cicd-demo
  ```
- Here, the backslash tells the shell: “this command continues on the next line.”
- It’s just for readability — you could write it all on one line, but splitting makes it easier to read.

---

## 🔹 Double Dash (`--`)
- In `kubectl` commands, `--` introduces **flags/options**.
- Example:
  ```bash
  kubectl create deployment nginx-demo1 --image=nginx -n cicd-demo
  ```
  - `--image=nginx` → specifies which container image to use.
  - `-n cicd-demo` → shorthand for `--namespace cicd-demo`.

---

✅ So in summary:
- `''' ... '''` → multi‑line string in Groovy.  
- `||` → run second command if first fails.  
- `\` → line continuation.  
- `--` → flag/option for a command.  

---


