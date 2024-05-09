---
author: Eve Kolb
title: Building Go Applications with Gitlab CI
date: 2024-04-21
description: An easy to follow Guide on building Go Applications with Gitlab CI
tags: ["ci", "gitlab", "go"]
type: blog
thumbnail: https://i.ibb.co/VBkX0m0/734157-A-Golang-Gopher-working-on-a-pipeline-xl-1024-v1-0.png
---

## Intro

Recently I've started building and deploying Go Applications with Gitlab CI.
Today I wanna share how I build a Pipeline that aims to make your
code more resilient and safe.

## The Pipeline

### Steps of a Pipeline

First we should identify the steps we need in a Pipeline.
We'll split those into steps that **verify** and steps that **transform**,
and steps that **publish**.

#### Verify

- Linting
- Unit Tests / Integration Tests
- SAST
- Secret Detection
- Dependency Scanning

#### Transform

- Building

### Verification

#### Linter

The first Step we're gonna add is the **Linter**.
Since we're working with go we're gonna use [golangci-lint](https://golangci-lint.run/).
Lets look at the code snippet for this part:

```yaml
lint:
  image: golangci/golangci-lint:v1.57.2-alpine
  stage: lint
  script:
    # Write the code coverage report to gl-code-quality-report.json
    # and print linting issues to stdout in the format: path/to/file:line description
    # remove `--issues-exit-code 0` or set to non-zero to fail the job if linting issues are detected
    - golangci-lint run --issues-exit-code 0 --print-issued-lines=false --out-format code-climate:gl-code-quality-report.json,line-number
  artifacts:
    reports:
      codequality: gl-code-quality-report.json
    paths:
      - gl-code-quality-report.json
```

As you can see we use the official golangci-lint docker image.
Then we call the command for it and tell it to output the results to a file.
This file is then used as a report that Gitlab can consume to show the results in the GUI.

#### Tests

Now we run the tests. Look at the following code snippet:

```yaml
test:
  stage: test
  image: golang:1.22
  needs:
    - build
  script:
    - go install gotest.tools/gotestsum@latest
    - gotestsum --junitfile report.xml --format testname -- -coverprofile=coverage.out -covermode count ./...
    - go get github.com/boumenot/gocover-cobertura
    - go run github.com/boumenot/gocover-cobertura < coverage.out > coverage.xml
    - go tool cover -func=coverage.out
  artifacts:
    when: always
    reports:
      junit: report.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
  coverage: "/\\(statements\\)\\s+\\d+.?\\d+%/"
```

This runs the tests and calculates the coverage.
Both are put into reports and passed to Gitlab for Consumption.

#### SAST (Static Application Security Testing)

This allows us to find some bugs that might affect security.
Same spiel, lets look at the code:

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml

semgrep-sast:
  stage: scan
```

Its a bit different than the previous ones.
Thats because we're using one of the Templates provided by Gitlab.
The only thing we're changing is the stage of the Step.

#### Secret Detection

We don't want our precious Passwords and Token show up in Git, do we?
Thats why we use Secret Detection to Detect them early on.

**Reuse the include statement from the previous step!**

```yaml
include:
  - template: Security/Secret-Detection.gitlab-ci.yml

secret_detection:
  stage: scan
```

Same thing as above, just changing the stage.

#### Dependency Scanning

**This is a Gitlab Ultimate Feature,
the Pipeline won't fail because of this,
but it won't be as useful for you, if you don't have a subscription!**

This is the code:

```yaml
include:
  - template: Security/Dependency-Scanning.gitlab-ci.yml

gemnasium-dependency_scanning:
  stage: scan
```

Same thing, only changing the stage.

### Transformation

Now the final step, let's actually build the program

```yaml
variables:
  OUTPUT_NAME: __bin__/$CI_PROJECT_NAME

build:
  stage: build
  image: golang:1.22
  script:
    - mkdir -p $OUTPUT_NAME
    - go build -o $OUTPUT_NAME ./...
  artifacts:
    paths:
      - $OUTPUT_NAME
```

Set the image to the Golang version you need.
The built binary will now be available as an Artifact.

## Conclusion

Alright. Now we got our Pipeline. Of course it could extended.
For example by building a Docker Image out of your code and publishing it.
But thats a task for another day.

You can find the whole pipeline on my Gitlab in this
[Repo](https://gitlab.com/thelooter/build-go-with-gitlab-ci/-/blob/main/.gitlab-ci.yml?ref_type=heads)

I hope you learned something. If you have any feedback don't hesitate to reach out.
