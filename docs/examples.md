# Examples


 - [simple](#simple) - installs NPM dependencies and runs Cypress tests
 - [component-tests](#component-tests) - installs NPM dependencies and runs Cypress component tests
 - [recording](#recording) - Runs all Cypress tests and records them on the Cypress Dashboard
 - [parallel-on-2-machines](#parallel-on-2-machines) - Runs all Cypress tests by load balancing them on two machines
 - [yarn](#yarn) - install dependencies using Yarn
 - [custom-install](#custom-install) - install dependencies using any command
 - [custom-verify](#custom-verify) - use custom command to verify Cypress
 - [custom-cache-key](#custom-cache-key) - apply custom key for npm install (or yarn install) cache
 - [using-node14](#using-node14) - running tests using Node 14
 - [chrome](#chrome) - running tests using Chrome browser
 - [start-server](#start-server) - start server before running tests
 - [wait-for-server-to-respond](#wait-for-server-to-respond) - wait for server to respond before starting tests
 - [build-app](#build-app) - build application after install
 - [groups](#groups) - running several groups of tests
 - [release](#release) - running another job after tests
 - [linux-and-mac](#linux-and-mac) - building using orb on Mac and Linux
 - [custom-executor](#custom-executor) - use custom executor
 - [env-vars](#env-vars) - set additional environment variables
 - [install-private-npm-modules](#install-private-npm-modules) - install private NPM dependencies
 - [store-test-reports](#store-test-reports) - store test reports on Circle
 - [artifacts](#artifacts) - store screenshots and videos on Circle
 - [any-artifacts](#any-artifacts) - store other folders as artifacts on Circle
 - [custom-command](#custom-command) - use a custom command to launch tests
 - [custom-command-prefix](#custom-command-prefix) - prefix default test command
 - [no-workspace](#no-workspace) - faster for a single cypress/run job without saving workspace
 - [private-npm-module](#private-npm-module) - complete NPM module publishing example
 - [custom-directory](#custom-directory) - run commands in a subfolder of a monorepo
 - [custom-cache-and-directory](#custom-cache-and-directory) - use custom cache key in a monorepo situation
 - [print-info](#print-info) - Run "cypress info" command after install
 - [install-extra-tool](#install-extra-tool) - run commands after installing NPM modules but before caching
 - [config](#config) - pass additional config parameter via --config argument
 - [config-file](#config-file) - custom configuration file
 - [env](#env) - Passing values via --env CLI argument
 - [tags](#tags) - tag recorded run
 - [attach-workspace](#attach-workspace) - attaches the workspace assuming previous job has installed it
 - [run-tasks-post-checkout](#run-tasks-post-checkout) - perform steps after code checkout but before installing dependencies
 - [run-on-master-branch](#run-on-master-branch) - run different tests depending on the branch
 - [debug](#debug) - turn on specific DEBUG logs
 - [custom-build-id](#custom-build-id) - Using custom ci-build-id parameter to tie jobs into a logical run

## simple


Runs all Cypress tests without recording results on the Dashboard. Installs dependencies with "npm ci", caches NPM modules and Cypress binary. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run
```

## component-tests


Runs all Cypress component tests without recording results on the Dashboard. Installs dependencies with "npm ci", caches NPM modules and Cypress binary. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1.30.0
workflows:
  build:
    jobs:
      - cypress/run:
          component: true
```

## recording


Runs all Cypress tests and records them on the [Cypress Dashboard](https://on.cypress.io/dashboard-service). Requires `CYPRESS_RECORD_KEY` environment variable to be set. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          record: true
```

## parallel-on-2-machines


Runs all Cypress tests by load balancing them on two machines. Note that first a single job installs NPM dependencies, then 2 jobs run the tests in parallel using Cypress Dashboard [parallelization](https://on.cypress.io/parallelization). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/install
      - cypress/run:
          requires:
            - cypress/install
          record: true
          parallel: true
          parallelism: 2
          group: 2 machines
```

## yarn


Installs NPM dependencies using "yarn install --frozen-lockfile" command, then runs Cypress tests. Caches NPM modules and Cypress binary. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          yarn: true
```

## custom-install


Install dependencies using your own custom command. Related parameter: [custom-verify](#custom-verify) 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          install-command: yarn install --frozen-lockfile
```

## custom-verify


Verify Cypress was installed using custom command. Default command is "npx cypress verify" Related parameter: [custom-install](#custom-install) 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          verify-command: yarn cypress verify
```

## custom-cache-key


Apply custom key for npm install (or yarn install) cache. Useful to tweak caching settings to your liking. Related options [yarn](#yarn). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          yarn: true
          cache-key: 'yarn-packages-{{ arch }}-{{ checksum "yarn.lock" }}'
```

## using-node14


Runs all Cypress tests on Node 14 image by specifying `executor` name. There are [several executors included](https://circleci.com/developer/orbs/orb/cypress-io/cypress#executors) with this orb or you can use your own executor, such as a Docker image from [cypress-docker-images](https://github.com/cypress-io/cypress-docker-images). See [custom-executor](#custom-executor) example. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          executor: cypress/base-14
```

## chrome


Runs tests on Chrome in a [custom-executor](#custom-executor) - a Docker image that includes the Chrome browser, such as a Docker image from [cypress-docker-images](https://github.com/cypress-io/cypress-docker-images). See [`--browser`](https://on.cypress.io/launching-browsers) documentation for available browser arguments. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
executors:
  with-chrome:
    docker:
      - image: 'cypress/browsers:node14.16.0-chrome90-ff88'
workflows:
  build:
    jobs:
      - cypress/run:
          executor: with-chrome
          browser: chrome
```

## start-server


Often we need to start a local webserver before running end-to-end tests. This option runs the given command to starts the server in the background and then runs all Cypress tests. Related option [wait-on](#wait-for-server-to-respond). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          start: npm start
```

## wait-for-server-to-respond


Starts server, waits for it to respond and then runs all Cypress tests. Uses `npx wait-on ...` command under the hood, see [wait-on](https://github.com/jeffbski/wait-on#readme). Note, if you are using Webpack server, it might not respond to the default HTTP OPTIONS request. In that case use `wait-on` to send `HTTP GET` request by using url `wait-on: 'http-get://localhost:....'`. You can also pass `wait-on` config, see [issue #90](https://github.com/cypress-io/circleci-orb/issues/90). Related option [start](#start). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          start: npm start
          wait-on: 'http://localhost:4200'
```

## build-app


Install dependencies and run a custom build command on one machine that builds the web application. Then runs Cypress tests on 3 machines and balances them using [Dashboard parallelization](https://on.cypress.io/parallelization). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/install:
          build: npm run build
      - cypress/run:
          requires:
            - cypress/install
          record: true
          parallel: true
          parallelism: 3
          group: 3x
```

## groups


Runs all tests on 4 machines using Electron browser (default). Also runs some tests using "spec" parameter on Chrome browser. Records both groups on Cypress dashboard. Notice "name" under each "cypress/run" job which will be shown in the Circle workflow UI to tell jobs apart. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/install
      - cypress/run:
          name: 4 machines
          requires:
            - cypress/install
          record: true
          parallel: true
          parallelism: 4
          group: 4 machines
      - cypress/run:
          name: Chrome
          requires:
            - cypress/install
          executor: cypress/browsers-chrome69
          record: true
          parallel: true
          parallelism: 2
          group: smoke tests
          browser: chrome
          spec: cypress/integration/smoke/*
```

## release


If you want to run an entire job after running Cypress tests, you can reuse the workspace from the `cypress/run` job. For example, to run a semantic release script you could follow this example. Note - for simpler steps you can use "post-steps" parameter, see [store-test-reports](#store-test-reports) example. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
jobs:
  release:
    executor: cypress/base-10
    steps:
      - attach_workspace:
          at: ~/
      - run: npm run semantic-release
workflows:
  build:
    jobs:
      - cypress/install
      - cypress/run:
          requires:
            - cypress/install
      - release:
          requires:
            - cypress/run
```

## linux-and-mac


Runs tests on Linux and on Mac via two jobs. Note how the user defines and uses own [custom executor](#custom-executor) "mac". 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
executors:
  mac:
    macos:
      xcode: 10.1.0
workflows:
  build:
    jobs:
      - cypress/run:
          name: Linux test
      - cypress/run:
          name: Mac test
          executor: mac
```

## custom-executor


Use any executor to run the job defined by the orb, such as a custom Docker image name, or a Docker image from [cypress-docker-images](https://github.com/cypress-io/cypress-docker-images). Assumes the executor has all OS dependencies necessary to run Cypress. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
executors:
  with-chrome:
    docker:
      - image: 'cypress/browsers:node14.16.0-chrome90-ff88'
workflows:
  build:
    jobs:
      - cypress/run:
          executor: with-chrome
          browser: chrome
```

## env-vars


Set additional environment variables when running Cypress tests by adding them to the executor. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
executors:
  base10-foo-bar:
    docker:
      - image: 'cypress/base:10'
    environment:
      FOO: foo
      BAR: bar
workflows:
  build:
    jobs:
      - cypress/run:
          executor: base10-foo-bar
```

## install-private-npm-modules


In this example, we write the NPM auth token before running "npm install" command. This allows us to install private NPM modules for example. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/install:
          pre-steps:
            - run: 'echo "//registry.npmjs.org/:_authToken=${NPM_TOKEN}" > ~/.npmrc'
      - cypress/run:
          requires:
            - cypress/install
```

## store-test-reports


Stores test results using post-steps parameter, see https://on.cypress.io/reporters, assumes that reports are saved in folder "cypress/results". Also see [artifacts](#artifacts) parameter. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          post-steps:
            - store_test_results:
                path: cypress/results
```

## artifacts


Stores test screenshots and videos as CircleCI artifacts using "store_artifacts" job option. Note, this setting assumes the default Cypress folders for screenshots and videos. If you store screenshots and videos in custom folders, see [any-artifacts](#any-artifacts) example how to store arbitrary folders. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          store_artifacts: true
```

## any-artifacts


Stores additional folders like "mochawesome-report" or code coverage folder as a CircleCI artifact. Related option [artifacts](#artifacts). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          post-steps:
            - store_artifacts:
                path: mochawesome-report
```

## custom-command


Use your own arbitrary command to launch Cypress tests.

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          command: npx cypress run --record
```

## custom-command-prefix


Use your own arbitrary command to prefix default cypress run test command.

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          command-prefix: npx percy exec --
```

## no-workspace


Faster "cypress/run" job that does not attach workspace, because there are no jobs that follow, so no need to save files. This removes the "Persisting to workspace" step of the job. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          no-workspace: true
```

## private-npm-module


This example shows how to install private NPM modules using NPM_TOKEN environment variable using "pre-steps" parameter. Then it starts the server, waits for it to respond and runs Cypress tests. Then it publishes a new version of the current module using semantic-release script (using the same NPM_TOKEN variable to publish). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          pre-steps:
            - run: 'echo "//registry.npmjs.org/:_authToken=${NPM_TOKEN}" > ~/.npmrc'
          no-workspace: true
          start: npm start
          wait-on: 'http://localhost:3003'
          post-steps:
            - run: npm run semantic-release
```

## custom-directory


Runs all commands in a custom directory, this is useful when using a monorepo where the `package.json` file isn't at the root of the repository (eg. in the `frontend/subfolder`). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          yarn: true
          working_directory: frontend
```

## custom-cache-and-directory


Monorepo users may want to provide a `cache-key` parameter to key the cache with an appropriate checksum. (i.e. `cache-key: 'cache-{{ arch }}-{{ .Branch }}-{{ checksum "frontend/package.json" }}'`) 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/install:
          cache-key: >-
            cache-{{ arch }}-{{ .Branch }}-{{ checksum "frontend/package.json"
            }}
      - cypress/run:
          yarn: true
          cache-key: >-
            cache-{{ arch }}-{{ .Branch }}-{{ checksum "frontend/package.json"
            }}
          working_directory: frontend
```

## print-info


It is useful to print information about the OS and found browsers using the "cypress info" command 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          post-install:
            - run: npx cypress info
```

## install-extra-tool


Sometimes you want to install another tool after installing regular dependencies but before running "cypress verify" and caching NPM modules and Cypress binary. In this example, it installs one more tool "print-env" and runs it. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          post-install:
            - run: npm install -g print-env
            - run: print-env CIRCLE
```

## config


Passing additional Cypress config parameters via --config CLI argument

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          config: 'pageLoadTimeout=100000,watchForFileChanges=false'
```

## config-file


Uses non-default configuration file

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          config-file: staging.json
```

## env


Passing values via --env CLI argument

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          env: 'userName=Joe,apiKey=1234567890'
```

## tags


Pass tags to the dashboard

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          record: true
          tags: 'nightly,staging'
```

## attach-workspace


You may run multiple job after installing dependencies once. In that case every job should attach workspace and require cypress/install. Related parameter [no-workspace](#no-workspace). 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/install
      - cypress/run:
          requires:
            - cypress/install
          attach-workspace: true
```

## run-tasks-post-checkout


Perform commands after checkout, but before install

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          post-checkout:
            - run: echo Scaffolding before installing
```

## run-on-master-branch


Sometimes you want to execute only some tests on the "master" branch. This recipe shows how to set job filters in the workflow. Development tests job runs all the specs on every branch but master. Production tests executes only some specs and runs only on "master" branch. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          name: Development tests
          filters:
            branches:
              ignore:
                - master
      - cypress/run:
          name: Production tests
          spec: cypress/integration/prod/*
          filters:
            branches:
              only:
                - master
```

## debug


To debug Cypress, use DEBUG=... environment variable which works via https://github.com/visionmedia/debug NPM module. For example, to debug Cypress CLI use "cypress:cli", and to debug everything use "cypress*", which could be very verbose. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/run:
          name: Debug with Cypress CLI logs
          debug: 'cypress:cli'
```

## custom-build-id


Runs two jobs splitting the specs in parallel using Cypress Dashboard [parallelization](https://on.cypress.io/parallelization). Uses custom build ID to link the jobs together into a logical run. Notice the environment variable syntax to be expanded at the build time. See example in https://github.com/cypress-io/circleci-orb-parallel-example. 

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/install
      - cypress/run:
          requires:
            - cypress/install
          record: true
          parallel: true
          parallelism: 2
          ci-build-id: 'testing-commit-${CIRCLE_SHA1}'
```

