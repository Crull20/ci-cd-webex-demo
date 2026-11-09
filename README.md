# CI/CD Jenkins + GitHub + ngrok + Webex (Windows PowerShell)

This project uses the original sample Python calculator (`sample_code.py`), pytest unit tests (`tests/test_sample_code.py`), and the original build/test bash scripts. **Run local commands from Windows PowerShell.** Jenkins is a **Linux container** running under Docker Desktop, so the Jenkinsfile runs the `.sh` scripts with bash inside that container.

## 1. Local test from PowerShell

```powershell
cd "$HOME\ci-cd-webex-demo"
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe sample_code.py
.\.venv\Scripts\python.exe -m pycodestyle sample_code.py --max-line-length=100
.\.venv\Scripts\python.exe -m pytest tests/
```

Expected: **4 passed**. `.venv` is gitignored. `pytest.ini` makes the root `sample_code.py` importable.

## 2. GitHub repository

Create an empty public repository named `ci-cd-webex-demo` on GitHub (no initial README). Run:

```powershell
git init -b main
git add .
git commit -m "Initial CI/CD project setup"
git remote add origin https://github.com/YOUR-USERNAME/ci-cd-webex-demo.git
git push -u origin main
```

## 3. Docker Desktop and Jenkins

Switch Docker Desktop to **Linux containers** and ensure it's running. From this directory:

```powershell
docker build -f .\Dockerfile.jenkins -t jenkins-python-ci .
docker run -d --name jenkins-ci --restart unless-stopped -p 127.0.0.1:8080:8080 -v jenkins_home:/var/jenkins_home jenkins-python-ci
docker ps
docker exec jenkins-ci cat /var/jenkins_home/secrets/initialAdminPassword
```

Open http://localhost:8080 in your **Windows browser**, unlock Jenkins using the displayed password, install suggested plugins plus GitHub and Credentials Binding if needed. Add two **Secret text** credentials: `webex-bot-token` and `webex-room-id`.

Create a Jenkins **Pipeline** job from SCM (Git): public GitHub URL, branch `*/main`, script path `Jenkinsfile`. Enable **GitHub hook trigger for GITScm polling**. Run one manual build to check the configuration before making the final demonstration recording.

## 4. ngrok on Windows

```powershell
ngrok config add-authtoken YOUR_NGROK_AUTHTOKEN
ngrok http 8080
```

Keep the ngrok PowerShell window running. Set your GitHub repository **Settings > Webhooks** Payload URL to `https://YOUR-NGROK-DOMAIN/github-webhook/`, content type `application/json`, event **Just the push event**. Consider configuring a matching webhook secret in both GitHub and Jenkins.

Use GitHub's Webhook **Recent Deliveries** and http://localhost:4040 (ngrok inspect) to verify webhook POSTs. On subsequent `git push`, Jenkins should automatically build, run 4 tests, then send a message using the Webex bot token and room ID.

## 5. Final demonstration

1. Edit `sample_code.py` or add an assertion to `tests/test_sample_code.py` in Windows.
2. `git add ...`, `git commit -m "Update calculator tests"`, `git push origin main`.
3. Show commit timestamp in GitHub UI.
4. Show `POST /github-webhook/` in ngrok PowerShell or inspector.
5. Show Jenkins job building without clicking **Build Now**.
6. Show `4 passed`, `Finished: SUCCESS`, and message in Webex space.

**Never commit tokens**. End the ngrok tunnel after demonstration. The GitHub push webhook is separate from Webex bot posting; no incoming Webex webhook is needed.
