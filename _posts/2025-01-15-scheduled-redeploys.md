---
layout: post
title: "How to Enable Scheduled Redeploys in Github Actions"
date: "2025-01-15 12:00:00 +0000"
tags: [github]
excerpt: "This post explains how to to setup github actions so that your jekyll static website is rebuild daily, irrespective of push redeploys, enduring that pushed future posts are added to the live site on the right day automatically."
---

I host this blog on github, which is useful because when I add a post scheduled for release in the past (specified in the frontmatter of the .md post file in `date: 2025-01-01 08:00:00 +0000` for example) and then test the site locally with `bundle exec jekyll serve` I can see exactly what the site will look like when I push the changes to github and trigger a rebuild/redeploy of the website. When I write a post scheduled for a time in the future I can also see what the site will look like online with `bundle exec jekyll serve --future` which builds all posts irrespective of the frontmatter `date` tag. Without amending the github actions this causes a small problem. By default github actions only rebuilds/redeploys the site when changes are pushed to github, so if there are several posts with frontmatter `date` in the future, these posts will not be build until a new push ofter their release date. To automatically rebuild/redeploy the site daily we need to add a scheduled redeployments. Luckily, this is straightforward.

Step 1. Create a `schedule-redeploy.yml` configuration file in `.github/workflows`:

```bash
mkdir -p .github/workflows
touch .github/workflows/schedule-redeploy.yml
```

Step 2. Edit the `schedule-redeploy.yml`:

```yml
# name of the workflow, visible on the github Actions tab
name: Scheduled Redeployment

# define the trigger for the workflow
on:
  schedule:
    # cron is a Unix-based utility for scheduling tasks ("cron jobs")
    # Runs every day at 06:00 AM UTC
    - cron: "0 6 * * *"

# define the job and specify the environment (ubuntu-latest) on which the job will run
jobs:
  rebuild:
    runs-on: ubuntu-latest

    # check out the repo code into the workflow environment
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

        # configure git with username and email for committing changes (required by git)
      - name: Configure Git
        run: |
          git config --global user.name "GitHub Actions"
          git config --global user.email "actions@github.com"

        # use the GITHUB_TOKEN to grant permission in the repo settings
        # creates an empty commit (--allow-empty) to avoid modifying the repo's content
        # pushes the empty commit to the repo, triggering a github pages rebuild
      - name: Commit and Push Empty Change
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          git commit --allow-empty -m "Trigger Pages Build"
          git push
```

Step 3. Commit the new `schedule-redeploy.yml` file and push to github:

```bash
git add .github/worklows/schedule-redeploy.yml
git commit -m "added schedule-redeploy.yml for scheduled redeploy"
git push
```

Step 4. If you leave it here, the scheduled redeploy action will fail because by default the workflow settings only allows actions read permissions. You need to set read and write permissions to workflow in the repo Settings->Actions->General section and save the change:

<div style="text-align: center;">
   <img src="{{ site.baseurl }}/assets/posts/github-workflow-permissions.png" style="width: 100%;">
</div>

This will allow the scheduled redeploy to work. You can test this by setting the `schedule-redeploy.yml` `cron` to a few minutes in the future and checking the github Actions tab to check if the action ran successfully. Once this works you can change the `cron` time to whenever you want to action to be performed daily (or monthly or yearly).
