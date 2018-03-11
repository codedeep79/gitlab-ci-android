### Allow a person with a React Native project to:
+ add CI (builds, tests) + CD (publishing to Play Store) with a simple [.gitlab-ci.yml](https://gitlab.com/motorica-org/gitlab-ci-android/blob/master/README.rst) (making use of existing container in Container Registry)
+ Fork container-building repos to have a customised setup (again, with CR)

Know Before: 
+ [Setting up GitLab CI for Android projects](https://about.gitlab.com/2016/11/30/setting-up-gitlab-ci-for-android-projects/)
+ [Learn Way To Builds Docker image](https://github.com/jangrewe/gitlab-ci-android/blob/master/Dockerfile)
+ Makes use of Container Registry



### Intro:
Continuous Integration – a practice of automatically building and testing software on every push or merge (and before every merge) – allows you to quickly catch bugs or breakages; Continuous Delivery goes further with the concept by automatically deploying “good” builds to staging (or, in our case, to Google Play alpha track).

### CI

1. Build debug flavour from clean repo

2. Build release

    2.1 Build locally

    2.1.1 Set up release keys

    2.2 Build on CI

    2.2.1 Set up deploy keys

    2.2.2 CI env vars

    2.2.3 Set up SSH, download keystore

3. Version with git

### CD 

4. Create initial entry for app on Play

5. Set up Play keys 

6. Auto publish

    6.1 Locally

    6.2 From CI

7. Automate filling Play metadata

8. Publish to different tracks depending on branch / tag

### Example: .gitlab-ci.yml. You certainly that completed build `Dockerfile` in Container Registry then to this step .
```
image: registry.gitlab.com/motorica-org/gitlab-ci-react-native-android:master

stages:
  - build

before_script:
  # SSH: set up key and trusted hosts
  # see https://gitlab.com/help/ci/ssh_keys/README.md
  - eval $(ssh-agent -s)
  - ssh-add <(echo "$SSH_PRIVATE_KEY")
  - mkdir -p ~/.ssh
  - '[[ -f /.dockerenv ]] && echo "$SSH_SERVER_HOSTKEYS" > ~/.ssh/known_hosts'
  # Android
  - cd android/

  ## Download keystore
  - git archive --remote=$SECRETS_REPO HEAD $KEYSTORE_RELPATH | tar -x

  ## Flavours

  ### app
  - echo $GPLAY_SERVICE_ACCOUNT > app/gplay_service_account.json

  ## /Flavours

  - export GRADLE_USER_HOME=$(pwd)/.gradle
  - chmod +x ./gradlew

  # /Android
  - cd ..

cache:
  key: ${CI_PROJECT_ID}
  paths:
  - node_modules/
  - .gradle/

build:
  stage: build
  script:
    - yarn install
    - cd android/
    - ./gradlew assembleDebug
    - ./gradlew assembleRelease
    - ./gradlew publishApkRelease
  artifacts:
    paths:
      - android/app/build/outputs/apk/app-debug.apk
      - android/app/build/outputs/apk/app-release.apk

```


