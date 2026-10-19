# Exercise 9: Jenkins Multi-Stage Pipeline for a Python Flask Application

## Objective

Build and test a Flask application, copy it to a mock deployment directory, start it in a Docker container, and test it through a Jenkins multi-stage pipeline.

## Files

- `app.py` — Flask application.
- `requirements.txt` — Flask dependency.
- `test_app.py` — unit test for the home route.
- `Dockerfile` — builds the Python application image.
- `Jenkinsfile` — Jenkins pipeline definition.
- `screenshots/` — add your own screenshots here.

## Important note

The Jenkins container from the previous exercises has Docker CLI access but does not have Python installed directly. This pipeline runs Python and the unit tests inside a Docker image instead of installing Python in Jenkins.

## Add to your GitHub repository

Copy this folder into the root of `https://github.com/RayanHaque/kubernetes-hands-on`.

From PowerShell in the repository root, run:

```powershell
git add exercise-9-python-multistage-pipeline
git commit -m "Add Exercise 9 Python multi-stage Jenkins pipeline"
git pull --rebase origin main
git push origin main
```

If there is nothing to commit, check `git status`. If a pull or push reports a conflict, stop and resolve it. Do not force-push.

## Configure the Jenkins job

1. Open `http://localhost:8080`.
2. Click **New Item**.
3. Enter `Python-MultiStage-Pipeline`.
4. Select **Pipeline**, then click **OK**.
5. Under **Pipeline**, set **Definition** to **Pipeline script from SCM**.
6. Set **SCM** to **Git**.
7. Repository URL: `https://github.com/RayanHaque/kubernetes-hands-on.git`
8. Branch Specifier: `*/main`
9. Script Path: `exercise-9-python-multistage-pipeline/Jenkinsfile`
10. Save and click **Build Now**.

The repository is public, so credentials can normally be left empty.

## Pipeline stages

1. **Build** — builds an image containing Python, Flask, and the application.
2. **Test** — runs the unit test inside the image.
3. **Deploy** — copies application files to a mock deployment directory in the Jenkins workspace.
4. **Run Application** — starts a container named `python-multistage-app` mapped to port `5001`.
5. **Test Application** — runs the unit test inside the running container.

## Expected result

A successful run should end with:

```text
Pipeline completed successfully!
Finished: SUCCESS
```

If the port mapping is accessible from your host, visit `http://localhost:5001` to see:

`Hello, Jenkins Multi-Stage Pipeline!`

Check the application container from PowerShell:

```powershell
docker ps --filter "name=python-multistage-app"
```

## Cleanup

When finished, remove the application container from PowerShell:

```powershell
docker rm -f python-multistage-app
```

Do not remove the Jenkins container or `jenkins_home` volume as part of this cleanup.

## Screenshots

Recommended screenshots:

- Jenkins job configuration showing repository, branch, and Script Path.
- Stage View showing all pipeline stages.
- Console Output showing `Pipeline completed successfully!` and `Finished: SUCCESS`.
- The application response at `http://localhost:5001`, if accessible.

Never commit passwords, personal access tokens, or other credentials.
