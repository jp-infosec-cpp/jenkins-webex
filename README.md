# Jenkins Webex CI/CD lab

This uses your professor's calculator and four pytest unit tests. The only added code is the short Jenkins pipeline and Docker setup. No extra Python apps, Flask, Nginx, or build scripts.

## 1. Start Jenkins

Use Docker Desktop on your Mac (or Docker in your Ubuntu VM). Open a terminal **in this folder**:

```sh
docker compose up -d --build
docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Open http://localhost:8080 and enter that password. Install **suggested plugins**, and check that the **Pipeline**, **Git**, **GitHub**, and **Credentials Binding** plugins are installed. Create your Jenkins account.

## 2. Put these files on GitHub

Make a new, empty **public** GitHub repository (e.g. `jenkins-webex-ci`). Then run in this folder:

```sh
git init -b main
git add .
git commit -m "First commit"
git remote add origin https://github.com/YOUR_USERNAME/jenkins-webex-ci.git
git push -u origin main
```

Replace `YOUR_USERNAME` with your GitHub username. Sign in to GitHub if prompted. **Do not push secrets or tokens.** A public repository makes Jenkins checkout easier for this assignment.

## 3. Create a Webex bot

1. Go to https://developer.webex.com/create/docs/bots and create a bot. Copy its **bot access token** somewhere private.
2. In Webex, create a space named `Jenkins`, and **add the bot to the space**.
3. Find the space's **roomId** using `GET https://webexapis.com/v1/rooms` with the bot token (via the Webex API documentation's Try It interface). Choose the `id` for your new space, not its title.

In **Jenkins > Manage Jenkins > Credentials > System > Global credentials**, add two **Secret text** credentials:

| ID | Secret |
| --- | --- |
| `webex-token` | Your Webex **bot** access token |
| `webex-room` | Your Webex space **roomId** |

## 4. Create the Jenkins job

1. **New Item** > name: `python-ci` > choose **Pipeline**.
2. Under **Pipeline**, select **Pipeline script from SCM**.
3. **SCM: Git**; URL: `https://github.com/YOUR_USERNAME/jenkins-webex-ci.git`; **Branch:** `*/main`; **Script Path:** `Jenkinsfile`.
4. Save and click **Build Now** *one time*. The Console Output should show **4 passed**, `Webex message sent`, and `Finished: SUCCESS`.

That first manual build loads the pipeline trigger. Do not click Build Now during your graded recording.

## 5. Start ngrok

Install ngrok (https://ngrok.com/download), register it, and in the **same computer running Docker**, run:

```sh
ngrok config add-authtoken YOUR_NGROK_AUTHTOKEN
ngrok http 8080
```

Keep ngrok open and copy its `https://...` address.

## 6. Add GitHub webhook

On GitHub open your repo > **Settings > Webhooks > Add webhook**:

- **Payload URL:** `https://YOUR-NGROK-URL/github-webhook/` (keep the last slash)
- **Content type:** `application/json`
- **Events:** Just the push event
- **Active:** Yes

If ngrok gives you a new URL, update the webhook on GitHub. Keep Jenkins authentication enabled; ngrok temporarily exposes your Jenkins login page to the internet.

## 7. Record the demonstration (80 points)

Change only a print message at the bottom of `sample_code.py` (not the math functions). Then:

```sh
git add sample_code.py
git commit -m "Change calculator message"
git push origin main
```

During the screen recording show:

1. GitHub shows the new commit timestamp (**10 points**).
2. ngrok shows the incoming webhook POST (**20 points**).
3. Jenkins starts a build **automatically** (**20 points**).
4. Jenkins Console Output shows **4 passed** and **SUCCESS** (**20 points**).
5. Webex space receives `Jenkins build #...: SUCCESS` (**10 points**).

When finished, stop ngrok with **Ctrl+C** and run `docker compose down`. Do not use `down -v` unless you want to erase Jenkins settings.
