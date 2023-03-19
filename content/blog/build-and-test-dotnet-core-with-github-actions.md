---
title: "Build and Test ASP.NET Core With GitHub Actions"
date: 2023-03-19T18:50:57Z
draft: false
---
# Build and Test ASP.NET Core With GitHub Actions

GitHub Actions is a powerful feature of the GitHub platform that allows developers to automate their software development workflows. 
In this blog post, I will explain a sample GitHub action that builds and runs tests on a .NET Core project. 
This GitHub action can be used as a starting point to create a continuous integration workflow that is triggered on a pull request. Let's get started!

## The GitHub Action:

{{< highlight yaml >}}
name: build

on:
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '7.0.x'
    - name: Restore Packages
      run: dotnet restore
    - name: Build
      run: dotnet build --configuration Release --no-restore
    - name: Test
      run: dotnet test --no-restore --verbosity normal
{{< /highlight >}}

The above code is a GitHub action that builds and runs tests on a dotnet core project. Let's take a closer look at each step.

## Step 1: Name and Trigger

The first step in the action is to define a name and trigger for the action. In this case, the action is named `build` and is triggered when a pull request is made to the `main` branch.

{{< highlight yaml >}}
name: build

on:
  pull_request:
    branches: [ main ]
{{< /highlight >}}

## Step 2: Define the Job

The next step is to define the job that the action will perform. In this case, the job is named `build` and runs on `ubuntu-latest`.

{{< highlight yaml >}}
jobs:
  build:
    runs-on: ubuntu-latest
{{< /highlight >}}

## Step 3: Checkout the Code

The next step is to checkout the code using the `actions/checkout@v3` action. This step ensures that the code is available for building and testing.

{{< highlight yaml >}}
steps:
  - uses: actions/checkout@v3
{{< /highlight >}}

## Step 4: Setup .NET Core

The next step is to set up .NET Core using the `actions/setup-dotnet@v3` action. This step ensures that the correct version of .NET Core is installed and available for building and testing.

{{< highlight yaml >}}
- name: Setup .NET Core
  uses: actions/setup-dotnet@v3
  with:
    dotnet-version: '7.0.x'
{{< /highlight >}}

## Step 5: Restore Packages

The next step is to restore the project packages using the `dotnet restore` command. This step ensures that all project dependencies are available for building and testing.

{{< highlight yaml >}}
- name: Restore Packages
  run: dotnet restore
{{< /highlight >}}

## Step 6: Build the Project

The next step is to build the project using the `dotnet build` command. This step compiles the project code and generates the necessary output files.

{{< highlight yaml >}}
- name: Build
  run: dotnet build --configuration Release --no-restore
{{< /highlight >}}

## Step 7: Run Tests

The final step is to run tests using the `dotnet test` command. This step ensures that the project tests pass and the project is working as expected.

{{< highlight yaml >}}
- name: Test
  run: dotnet test --no-restore --verbosity normal
{{< /highlight >}}

That's it! To further streamline the development process, this GitHub action can be combined with the ability to merge to the main branch only if the build and test results are successful. This ensures that the code that is merged into the main branch is always stable and meets the quality standards of the project.