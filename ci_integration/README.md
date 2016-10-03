## CI Runner

Continuous integration kurz CI beschreibt eine Software die Automatisch einen Build Prozess ausführt. Je nach Plattform gibt es verschiedene Lösungen.   
Der sogennante Runner führt seine Jobs nach jedem push aus. Je nach Einstellung werden entweder nur Tests durchgeführt oder ein ganzer Build Prozess.

### GitHub
Da GitHub keine Runner zu Verfügung stellt muss man auf externe Anbieter zurückgreifen, dafür wird dort Account mit dem GitHub Account verknüpft.   
 Hier sind die beliebtesten Runnter:
#### [Circle CI](https://circleci.com/) (*circle.yml*)
**Java:**

    machine:
      java:
        version: oraclejdk8
    test:
      override:
        - mvn package
        - cp target/IRCBouncer.jar $CIRCLE_ARTIFACTS/

#### [Travis CI](https://travis-ci.org/) (*.travis.yml*)
**Java:**

    language: java
    jdk: oraclejdk8

    script:
        - mvn clean install -B
        - mkdir Release/
        - cp target/IRCBouncer.jar Release/
        - cp ./*.ini Release/
        - zip -r release.zip Release/
        - ls -la *

    notifications:
        email: false

    deploy:
        provider: releases
        api_key: ${api_key}
        file: "release.zip"
        skip_cleanup: true
        on:
            all_branches: true
            tags: true

**C#:**

    language: csharp
    solution: WindowsBashHere.sln

    script:
        - xbuild "WindowsBashHere.sln"
        - mkdir Release/
        - cp -R WindowsBashHere/bin/Debug/* Release/
        - zip -r release.zip Release

    notifications:
        email: false

    deploy:
        provider: releases
        api_key: ${api_key}
        file: "release.zip"
        skip_cleanup: true
        on:
            all_branches: true
            tags: true

\* _**Notiz:** Da Travis keine Artifacts unterstützt, wird bei diesem Script die *Release.zip* als Release bei GitHub hochgeladen, falls ein Tag mit gepusht wird!_

    git tag <tagname>
    git push origin master --tags

    # force Tag push
    git tag -f <tagname>
    git push origin master -f --tags

### GitLab
#### [GitLab CI](https://gitlab.com/ci/lint) (*.gitlab-ci.yml*)
**Java:**

    image: maven:latest

    build:
      script:
      - mvn clean install -B
      - cp target/IRCBouncer.jar .
      - ls -la *
      artifacts:
        name: "IRCBouncer_${CI_BUILD_REF_NAME}"
        paths:
        - IRCBouncer.jar
        - config.ini
        - accounts.ini

**C#:**

    image: mono:latest

    before_script:
        - nuget restore -NonInteractive

    build:
        script:
        - xbuild "WindowsBashHere.sln"
        - mkdir Release/
        - cp -R WindowsBashHere/bin/Debug/* Release/
        artifacts:
            name: "WindowsBashHere_${CI_BUILD_REF_NAME}"
            paths:
            - Release/


#### Weitere Links:
* Travis-CI https://docs.travis-ci.com/user/getting-started
* Circle-CI https://circleci.com/docs/configuration/
* GitLab-CI https://docs.gitlab.com/ce/ci/quick_start/README.html
