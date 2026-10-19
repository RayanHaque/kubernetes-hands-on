# Exercise 8: Creating a "Hello World" Jenkins Job

## Objective

Create a simple shell script, store it in the GitHub repository, and configure a Jenkins Freestyle project to execute it.

## 1. Hello World Script

The file `hello-world.sh` contains:

```bash
#!/bin/bash
echo "Hello, Jenkins!"
```

## 2. Add the Exercise to GitHub

This exercise is prepared as a folder inside the existing repository:

`https://github.com/RayanHaque/kubernetes-hands-on.git`

Copy the `exercise-8-hello-world-jenkins` folder into the root of your local `kubernetes-hands-on` repository.

From PowerShell, in the repository root, run:

```powershell
git add exercise-8-hello-world-jenkins
git commit -m "Add Exercise 8 Hello World Jenkins job"
git pull --rebase origin main
git push origin main
```

If Git says there is nothing to commit, check `git status`. If a rebase or push reports a conflict, stop and resolve it before continuing. Do not force-push.

## 3. Create a Jenkins Freestyle Job

Prerequisite: Jenkins must be running and accessible at `http://localhost:8080`.

1. Sign in to Jenkins.
2. Click **New Item**.
3. Enter the job name `HelloWorld`.
4. Select **Freestyle project**, then click **OK**.
5. In **General**, add the description: `Hello World! Jenkins job.`
6. Under **Source Code Management**, select **Git**.
7. Set the Repository URL to:

   `https://github.com/RayanHaque/kubernetes-hands-on.git`

8. Leave credentials empty if the repository is public.
9. Under **Build Steps**, select **Add build step → Execute shell**.
10. Add this command:

```bash
sh exercise-8-hello-world-jenkins/hello-world.sh
```

11. Click **Save**.

## 4. Run the Job

1. Open the `HelloWorld` job.
2. Click **Build Now**.
3. Open the build number in **Build History**.
4. Click **Console Output**.

Expected output includes:

```text
Hello, Jenkins!
Finished: SUCCESS
```

## 5. Verify and Take Screenshots

Recommended screenshots to save in `screenshots/`:

- `jenkins-job-config.png` — the job configuration showing the Git repository and shell build step.
- `jenkins-console-output.png` — the console output containing `Hello, Jenkins!` and `Finished: SUCCESS`.

Do not include passwords, personal access tokens, or other credentials in screenshots or files.

## Conclusion

This exercise demonstrates how to store a shell script in GitHub and use a Jenkins Freestyle project to retrieve the repository and execute the script as a build step.
