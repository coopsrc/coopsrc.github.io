---
title: Publish android library to nexus repository
tags:
  - maven
  - nexus-oss
date: 2020-10-16 14:42:45
---


- 配置nexus 账户密码
    - NEXUS_REPOSITORY_URL
    - NEXUS_USERNAME
    - NEXUS_PASSWORD

- 配置library: Example: `implementation 'com.example.utils:utils:1.0.2'`
    - version: `1.0.2`
    - artifactId: `utils`
    - groupId: `com.example.utils`
    - packaging: `aar`
    
- 创建 `nexus_upload.gradle` 脚本

<!-- more -->

```groovy
apply plugin: 'maven'

task androidJavadocs(type: Javadoc) {
    options.encoding = "utf-8"
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
    classifier = 'javadoc'
    from androidJavadocs.destinationDir
}

task androidSourcesJar(type: Jar) {
    classifier = 'sources'
    from android.sourceSets.main.java.srcDirs
}

artifacts {
    archives androidSourcesJar
    archives androidJavadocsJar
}

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: NEXUS_REPOSITORY_URL) {
                authentication(userName: NEXUS_USERNAME, password: NEXUS_PASSWORD)
            }
            pom.project {
                name POM_NAME
                version POM_VERSION
                artifactId POM_ARTIFACTID
                groupId POM_GROUPID
                packaging POM_PACKAGING
                description POM_DESCRIPTION
            }
        }
    }
}
```

- 在library module `build.gradle` 最后引入`apply from: 'nexus_upload.gradle'`