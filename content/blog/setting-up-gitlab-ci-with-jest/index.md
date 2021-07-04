---
title: Setting up Jest tests and coverage in GitLab CI
date: "2021-07-04T22:12:03.284Z"
description: "Configuring Jest Tests in GitLab CI"
---

### 1. Add GitLab CI configuration file in the root

In the root of your project, add `.gitlab-ci.yml` with the configuration below.

```yaml
image: node:latest

stages:
  - build
  - test

build:
  stage: build
  script:
    - yarn
  cache:
    paths:
      - node_modules/
  artifacts:
    expire_in: 1 days
    when: on_success
    paths:
      - node_modules/

test:
  stage: test
  coverage: /All files[^|]*\|[^|]*\s+([\d\.]+)/
  dependencies:
    - build
  script:
    - yarn test:ci
  artifacts:
    when: always
    reports:
      junit:
        - junit.xml
      cobertura: coverage/cobertura-coverage.xml
```
Note that We have cached the `node_modules/` in build stage to make them available for subsequent jobs without having to download them again.

Pushing this to GitLab will automatically trigger the CI build. But before that, we'll add the required packages/configuration so that the build passes.

### 2. Install `jest-junit` package for reporting of tests
Next, we'll configure jest-junit, which will generate JUnit report format XML file (`junit.xml`) in the project root. GitLab will parse this XML format and then these reports can be viewed inside the pipelines details page, and also in the reports panel in Merge Requests. See the [GitLab Unit test reports](https://docs.gitlab.com/ee/ci/unit_test_reports.html#how-it-works) docs for more details.
```shell
npm install --save-dev jest-junit
```
OR
```shell
yarn add --dev jest-junit
```

### 3. Add `coverageReporters` and other required config to `jest.config.js`
From the GitLab Docs -
> Collecting the coverage information is done via GitLab CI/CD’s artifacts reports feature. You can specify one or more coverage reports to collect, including wildcard paths. GitLab then takes the coverage information in all the files and combines it together.
>
> For the coverage analysis to work, you have to provide a properly formatted Cobertura XML report to artifacts:reports:cobertura.

So we need to add Cobertura coverage reporter in `jest.config.js` for test coverage in GitLab Merge Requests
```js
module.exports = {
  "collectCoverageFrom": ["src/**/*.js", "!**/node_modules/**"],
  "coverageReporters": ["html", "text", "text-summary", "cobertura"],
  "testMatch": ["**/*.test.js"]
}
```
Adding `cobertura` to `coverageReporters` will generate `cobertura-coverage.xml` inside `/coverage/` folder created by Jest, and will be parsed by GitLab.

### 4. Add CI test run script to `package.json` with jest-junit reporter

```json
{
  "scripts": {
    "test:ci": "jest --config ./jest.config.js --collectCoverage --coverageDirectory=\"./coverage\" --ci --reporters=default --reporters=jest-junit --watchAll=false"
  }
}
```
This script is used in the `test` stage in the `.gitlab-ci.yaml` file we created in step 1.

### 5. Modify GitLab Project CI/CD settings for test coverage parsing

Go to Project > Settings > CI/CD > General pipelines > Test coverage parsing
Add the following RegEx -

`Lines\s*:\s*(\d+.?\d*)%`

This regular expression is used to find test coverage output in the job log. This coverage % can be viewed on Project > CI/CD > Jobs. We can also configure Badges on Project Overview page to show coverage % (see next step).

Finally, push all the changes to GitLab and you should see your pipeline up and running.
Also in the subsequent Merge Requests, you should see the number of tests, failing tests (if any) and failure reason, and test coverage information infiles.

### 6. Show Pipeline and Coverage Badge on the Project Overview page
We can add Badges to the overview page of GitLab projects to display useful information such as pipeline status, current release version, test coverage percentage etc. Below is how we can configure and add Badges -

[From GitLab Docs:](https://docs.gitlab.com/ee/user/project/badges.html#project-badges)
>
> To add a new badge to a project:
> 1. Navigate to your project’s Settings > General > Badges.
>2. Under “Link”, enter the URL that the badges should point to and under “Badge image URL” the URL of the image that should be displayed.
> 3. Submit the badge by clicking the Add badge button.
> ### Example project badge: Pipeline Status
>
> A common project badge presents the GitLab CI pipeline status.
>
> To add this badge to a project:
>
> 1. Navigate to your project’s Settings > General > Badges.
> 2. Under Name, enter Pipeline Status.
> 3. Under Link, enter the following URL: https://gitlab.com/%{project_path}/-/commits/%{default_branch}
> 4. Under Badge image URL, enter the following URL: https://gitlab.com/%{project_path}/badges/%{default_branch}/pipeline.svg
> 5. Submit the badge by clicking the Add badge button.

In the similar way, we can add a `coverage` badge to project. Just replace `pipeline.svg` with `coverage.svg` in step 4 above.


### 7. Publish Jest Coverage Report to GitLab pages

We can publish our Jest coverage report (`.html`) to GitLab pages to view detailed Jest coverage report on a GitLab Pages URL.

To publish - modify `.gitlab-ci.yml` to add deploy stage for publishing the coverage report HTML to GitLab pages
```diff
image: node:latest

stages:
  - build
  - test
+ - deploy

build:
  stage: build
  script:
    - yarn
  cache:
    paths:
      - node_modules/
  artifacts:
    expire_in: 1 days
    when: on_success
    paths:
      - node_modules/

test:
  stage: test
  coverage: /All files[^|]*\|[^|]*\s+([\d\.]+)/
  dependencies:
    - build
  script:
    - yarn test:ci
+ cache:
+   paths:
+     - coverage/
  artifacts:
+   paths:
+     - coverage/
    when: always
    reports:
      junit:
        - junit.xml
      cobertura: coverage/cobertura-coverage.xml

+ pages:
+  stage: deploy
+  dependencies:
+    - test
+  script:
+    - mkdir .public
+    - cp -r coverage/* .public
+    - mv .public public
+  artifacts:
+    paths:
+      - public
+  only:
+    - master
```

After pushing the changes, when the deploy step is successful in pipeline, We can access the Jest coverage report page using the URL mentioned in `Project > Settings > Pages`.

Each time the deploy job runs, a new coverage report will be published to the GitLab pages URL. Note that we have published coverage report to Pages only for `master` branch, because we don't want all branch commits to publish coverage report.

## References
1. [GitLab CI/CD process overview - GitLab Docs](https://docs.gitlab.com/ee/ci/quick_start/#cicd-process-overview)
2. [Unit test reports - GitLab Docs](https://docs.gitlab.com/ee/ci/unit_test_reports.html)
3. [Test Coverage Visualization - GitLab Docs](https://docs.gitlab.com/ee/user/project/merge_requests/test_coverage_visualization.html)
4. [How to display code coverage of a Vue project in Gitlab](https://javascript.plainenglish.io/how-to-display-code-coverage-of-a-vue-project-in-gitlab-9d1510a3e794)