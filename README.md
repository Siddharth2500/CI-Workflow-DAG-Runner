# 🧩 CI Workflow DAG Runner — Local CI/CD Simulator with Caching & Retries

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg?logo=python&logoColor=white)  
![CI/CD](https://img.shields.io/badge/CI/CD-DAG_Workflows-2088FF?logo=githubactions)  
![Caching](https://img.shields.io/badge/Cache-Content_Hash-4CAF50)  
![Artifacts](https://img.shields.io/badge/Artifacts-Per_Job-795548)  

**CI Workflow DAG Runner** is a **Python 3** tool that simulates a lightweight **CI/CD pipeline engine**:  
- Runs jobs in **topological order** using `needs` dependencies  
- Supports **caching** based on command + input file content hash  
- Retries jobs with **exponential backoff**  
- Collects **artifacts per job** into `./artifacts/<jobname>/`  
- Works in **Google Colab, local dev, or Docker**  

---

## 🛠 Tech & Languages

| Layer   | Tech / Format   | Notes                                       |
|---------|-----------------|---------------------------------------------|
| Language| **Python 3.10+**| Stdlib only, no external dependencies       |
| Config  | YAML-like file  | Minimal syntax for jobs, runs, needs, etc.  |
| Caching | SHA-256 hash    | Based on command + inputs                   |
| Artifacts| Filesystem     | Stored under `./artifacts/<job>/...`        |
| CI/CD   | GitHub Actions  | Can be embedded in workflows                |

---

## 📦 Repository Structure

ci-dag-runner/
├─ workflow.yml # Pipeline definition
├─ dag_runner.py # Runner script
├─ artifacts/ # Collected per-job artifacts (created at runtime)
└─ README.md

yaml
Copy code

---

## ▶️ Run in Google Colab

```python
from dag_runner import run_workflow
# Run pipeline
run_workflow("workflow.yml")
📄 Example workflow.yml
yaml
Copy code
jobs:
  - name: build
    runs: |
      echo "building"
      mkdir -p out
      echo "hello" > out/app.txt
    retries: 1
    inputs:
      - src/**
    artifacts:
      - out/**
  - name: test
    needs:
      - build
    runs: |
      echo "testing"
      test -f artifacts/build/app.txt
      grep -q hello artifacts/build/app.txt
    retries: 1
  - name: package
    needs:
      - test
    runs: |
      echo "packaging"
      mkdir -p dist
      tar -czf dist/app.tgz -C artifacts/build app.txt
      ls -l dist
    retries: 1
    artifacts:
      - dist/**
🧪 Example Output
First run:

bash
Copy code
▶️  build (attempt 1)
✅ build succeeded
▶️  test (attempt 1)
✅ test succeeded
▶️  package (attempt 1)
✅ package succeeded
🎉 pipeline complete
Re-run (cache hit):

bash
Copy code
⚡ cache hit: build
⚡ cache hit: test
⚡ cache hit: package
🎉 pipeline complete
🐳 Docker
Build:

bash
Copy code
docker build -t ci-dag-runner:latest .
Run:

bash
Copy code
docker run --rm -v $(pwd):/work -w /work ci-dag-runner:latest \
  python dag_runner.py
☸️ CI/CD Integration Ideas
Add a GitHub Actions job that runs python dag_runner.py for quick local-style pipelines

Publish ./artifacts/** as GitHub Actions artifacts

Extend runner to execute steps in Docker containers for parity with real CI

📊 Use Cases
Prototype pipelines locally without a CI server

Run Google Colab demos of DAG builds, caching, and retries

Teach CI/CD concepts in classrooms, workshops, or interviews

Pre-flight test before pushing to GitHub Actions/Jenkins/GitLab

👤 Author
Siddharth Raut — DevOps / Platform Engineer
