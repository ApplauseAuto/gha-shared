# ApplauseAuto Github Actions
Repository for common Github Actions used across the ApplauseAuto organization.

> Available actions:
>* mvn_java_setup
>* allure_reporting
>* allure_email

## mvn_java_setup
This action installs maven and java on your host machine.

### Usage

#### Inputs

* `java-version`: specifies the java version to install (`temurin` distribution)
  * default: `17`
* `mvn-version`: specifies the maven version to install
  * default: `3.9.2`
* `use-mvn-cache`: specifies usage of maven cache, when available
  * default: `false`

#### Example usage

```yaml
name: Your Workflow Name

on: push

jobs:
  your-job-name:
    runs-on: [self-hosted, opsvpc-customer, small]

  steps:
    - uses: actions/checkout@v3
    - uses: ApplauseAuto/gha-shared/.github/actions/mvn_java_setup@v1
      with:
        java-version: 21
        mvn-version: 3.9.2
        use-mvn-cache: 'true'

    - name: TestNG SDK tests
      env:
        apiKey: ${{ secrets.YOUR_SDK_API_KEY }}
      run: |
        mvn --no-transfer-progress test -Dexecution=clientSide -DdownloadResults=never \
        -DsuiteFile=some-Test.xml \
        -DdriverConfig=some-driver.json \
        -DproductId=123
```
---

## allure_reporting
This action performs Allure reporting tasks. 

It generates reports from TestNG results, manages history and publishes to Github Pages

### Requirements
The `pages-branch` (default: `gh-pages`) must be precreated and marked as the github-pages branch for your repository.

To make this branch default github-pages for your repository:
* Go to your repository on github.com.
* Click repository `Settings`
* Click `Pages` subsection from left-side menu
* In `GitHub Pages visibility` section, mark as `Private`. This restricts the pages to users that have access to the repository.
* In `Build and deployment` section, mark `source: Deploy from a branch` and `branch: gh-pages / (root)`

### Usage

#### Inputs

* `allure-history`: The relative path to the folder, that will be published to Github Pages.
  * default: `allure-history`
* `allure-history-size`: the number of Allure report histories to maintain
  * default: `60`
* `allure-reports`: The relative path to the directory where Allure will write the generated report.
  * *default*: `allure-report`
* `allure-results`: The relative path to the directory where Allure will keep report artifacts.
  * *default*: `target/allure-results`
* `pages-branch`: The Github Pages branch (and relative path/directory) to publish 
  * default: `gh-pages`
* `add-summary`: Adds Github Action Summary Step output
  * default: `true`
* `github-token`: The github token used for publishing to Github Pages.
  * required
  * Note: This is **NOT** a Personal Access Token.  This token is created automatically and is used for your workflow.  See example for usage.

#### Example usage

```yaml
name: Your Workflow Name

on: push

jobs:
  your-job-name:
    runs-on: [self-hosted, opsvpc-customer, small]

  steps:
    - uses: actions/checkout@v3
    - uses: ApplauseAuto/gha-shared/.github/actions/mvn_java_setup@v0.0.2

    - name: TestNG SDK tests
      env:
        apiKey: ${{ secrets.YOUR_SDK_API_KEY }}
      run: |
        mvn --no-transfer-progress test -Dexecution=clientSide -DdownloadResults=never \
        -DsuiteFile=some-Test.xml \
        -DdriverConfig=some-driver.json \
        -DproductId=123

    - uses: ApplauseAuto/gha-shared/.github/actions/allure_reporting@v0.0.2
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
```

## allure_email
This action sends email containing Allure report hyperlink when using the repository github-pages hosting.

### Requirements
The repository should be configured for github-pages hosting (recommended using the action `allure_reporting`)

This action only works when using Applause self-hosted runners.

### Usage

#### Inputs

* `recipients`: CSV list of email recipients
  * default: `allure-history`
  * notes: empty string will skip email sending
* `subject`: add custom email subject
  * default: "repo-owner/repo-name Allure report date:time
    * example: `ApplauseAuto/gha-trial Allure report 14/12/23T21:21`

#### Example usage

```yaml
name: Your Workflow Name

on: push

jobs:
  your-job-name:
    runs-on: [self-hosted, opsvpc-customer, small]

  steps:
    - uses: actions/checkout@v3
    - uses: ApplauseAuto/gha-shared/.github/actions/mvn_java_setup@v0.0.4

    - name: TestNG SDK tests
      env:
        apiKey: ${{ secrets.YOUR_SDK_API_KEY }}
      run: |
        mvn --no-transfer-progress test -Dexecution=clientSide -DdownloadResults=never \
        -DsuiteFile=some-Test.xml \
        -DdriverConfig=some-driver.json \
        -DproductId=123

    - uses: ApplauseAuto/gha-shared/.github/actions/allure_reporting@v0.0.4
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}

  # configuring this as a separate job, 
  # depending on previous job eg: [needs: your-job-name]
  # prevents duplicate emails when using matrix-driven tests, ensuring it only runs once at the end
  email-job:
    runs-on: [self-hosted, opsvpc-customer, small]
    needs: your-job-name
  steps:
    - uses: ApplauseAuto/gha-shared/.github/actions/allure_email@v0.0.4
      with:
        recipients: user1@applause.com, user2@applause.com
```
