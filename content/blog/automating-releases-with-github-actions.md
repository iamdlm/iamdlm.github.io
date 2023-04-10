---
title: "Automating Releases With Github Actions"
date: 2023-04-08T11:08:45+01:00
draft: false
---
# Automating Releases With Github Actions
As a software developer, managing releases and keeping track of release notes can be a time-consuming task. However, with the help of GitHub Actions, you can automate this process to streamline your workflow and ensure accurate and up-to-date release notes. In this blog post, we will dive into a specific GitHub Action which automates the process of writing releases notes and creating a release with the generated release notes for closed milestones in a GitHub repository.

To set up the "release" action in your own GitHub repository, you can create a new workflow file in the .github/workflows directory with the following content:

{{< highlight yaml >}}
name: release

on: 
  milestone:
    types: [closed]

jobs:
  create-release-notes:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    - name: Generate Release Notes
      uses: Decathlon/release-notes-generator-action@v3.1.6
      env:
        GITHUB_TOKEN: ${{ secrets.USER_TOKEN }}
    - name: Create Release
      env:
        GITHUB_TOKEN: ${{ secrets.USER_TOKEN }}
      run: |
        # Get milestone number from event payload
        milestone_number=$(jq --raw-output .milestone.number "$GITHUB_EVENT_PATH")
        # Get milestone title from event payload
        milestone_title=$(jq --raw-output .milestone.title "$GITHUB_EVENT_PATH")
        # Get release notes from previous step
        release_notes=$(cat ${{ github.workspace }}/release_file.md)
        # Create release using GitHub API
        release_data=$(echo '{}' | jq --arg tag_name "$milestone_title" --arg name "$milestone_title" --arg body "$release_notes" '. + { tag_name: $tag_name, name: $name, body: $body }')
        response=$(curl --silent --request POST --header "Authorization: token $GITHUB_TOKEN" --header "Accept: application/vnd.github.v3+json" --url "https://api.github.com/repos/${{ github.repository }}/releases" --data "$release_data")
        # Log response
        echo "$response"
{{< /highlight >}}

The "release" action is triggered by the closing of a milestone in the repository, as specified by the following configuration:
{{< highlight yaml >}}
on:
  milestone:
    types: [closed]
{{< /highlight >}}

This means that whenever a milestone is closed in the repository, the "release" action will be triggered.

The "release" action consists of a single job called "create-release-notes", which runs on the latest version of Ubuntu. The job is comprised of three steps:

1. Checkout code: This step uses the `actions/checkout@v3` action to check out the repository's code to the workflow's workspace.

2. Generate Release Notes: This step uses the `Decathlon/release-notes-generator-action@v3.1.6` action to generate release notes for the closed milestone. Each time a milestone is closed, this GitHub action scans its attached issues and pull request, and automatically generates a release note with [Spring.io changelog generator](https://github.com/spring-io/github-changelog-generator) tool. The `GITHUB_TOKEN` is passed as an environment variable `USER_TOKEN` to authenticate the API requests made by this action. 

3. Create Release: This step creates a GitHub release using the GitHub API. It extracts information from the closed milestone event payload, such as the milestone number and title, and the release notes generated in the previous step. It then constructs a JSON payload and makes a POST request to the GitHub API with the release data. The response from the API is logged for reference.

Here's a breakdown of the key components in the "create-release-notes" job:

- `jq`: This is a lightweight and flexible command-line JSON processor that is used to extract data from the JSON payload of the closed milestone event. It is used to extract the milestone number, title, and release notes from the event payload.

- `curl`: This is a widely-used command-line tool for making HTTP requests. It is used to make a POST request to the GitHub API to create a release with the release data constructed from the extracted milestone information and release notes.

- GitHub API: The GitHub API is a powerful REST API that allows you to interact with GitHub repositories programmatically. In this case, it is used to create a release with the extracted milestone information and release notes.

- Secrets: The `USER_TOKEN` environment variable is used instead of the repository's `GITHUB_TOKEN` to authenticate the API requests made by this step. This is because events triggered by the `GITHUB_TOKEN` do not create a new workflow run, which could result in an infinite loop of workflow runs. By using a Personal Access Token (PAT) as `USER_TOKEN`, you can avoid this issue and ensure that the GitHub release is created successfully. More information about this in [Triggering a workflow from a workflow](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow).

With this "release" action in place, you can ensure that every time a milestone is closed in your GitHub repository, the corresponding release notes are automatically generated and a GitHub release is created with the release notes, saving you time and effort in managing your software releases.

Give it a try and see how it can benefit your development workflow.
