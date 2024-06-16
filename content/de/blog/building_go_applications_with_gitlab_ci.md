---
author: Eve Kolb
title: Go Anwendunge mit Gitlab CI bauen
date: 2024-04-21
description: Eine einfach zu folgende Anleitung zum automatisierten bauen von Go Anwendungen
tags: ["ci", "gitlab", "go"]
type: blog
thumbnail: https://i.ibb.co/VBkX0m0/734157-A-Golang-Gopher-working-on-a-pipeline-xl-1024-v1-0.png
---

## Intro

Seit kurzem baue ich meine Go Anwendungen automatisiert mit Gitlab CI.
Heute möchte ich teilen wie ich meine Pipelines strukturiere um meine
Applikationen resilienter und sicherer zu machen.

## Die Pipeline

### Die Schritte einer Pipeline

Zuerst sollten wir die einzelnen Schritte definieren die
wir in unserer Pipeline benötigen.
Diese teile ich in Schritte auf die **verifizieren**, **transformieren**
und Schritte die unsere Arbeit veröffentlichen.

#### Verifizieren

- Linting
- Unit Tests / Integration Tests
- SAST (Static Analysis Security Testing)
- Secret Erkennung
- Dependency Scanning

#### Transformieren

- Bauen der Anwendung

### Verification

#### Linter

Der erste Schritt, den wir zur Pipeline hinzufügen ist ein Linter.
Dieser erkennt häufig vorkommende Fehler und Stilbrüche.
Da wir in Go programmieren, nutzen wir hierfür [golangci-lint](https://golangci-lint.run/).

Hier ist der Code für diesen Schritt:

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
This file is then used as a report that Gitlab can consume to show the results
in the GUI.

