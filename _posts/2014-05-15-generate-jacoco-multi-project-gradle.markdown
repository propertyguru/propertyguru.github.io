---
layout: post
title:  "Generate Jacoco coverage for a multi-project Gradle setup"
date:   2014-05-15 02:08:34
summary: "Configuring Gradle build system to combine Jacoco and Android plugins for unit test coverage generation for multi-project setup"
author: Ha Duy Trung
author_link: //twitter.com/hadt
tags: android jacoco coverage robolectric gradle testing
---

An example of how to configure Jacoco task in your `build.gradle` to generate coverage
report for a multi-project Gradle setup. The script will iterate through a set of
dependencies that is manually specified and merge all `.class` and source files into
final Jacoco report.

{% highlight groovy %}
apply plugin: 'jacoco'

// get class dirs for project dependencies
FileTree getJacocoClassDirs(List projects) {
    def classDirs = fileTree(dir: "${buildDir}/classes/debug", exclude: '**/R*.class')
    projects.each {
        def projBuildDir = project(it).buildDir
        classDirs += fileTree(dir: "${projBuildDir}/classes/release", exclude: '**/R*.class')
    }
    return classDirs
}

// get source dirs for project dependencies
FileCollection getJacocoSrcDirs(List projects) {
    Set srcDirs = android.sourceSets.main.java.srcDirs
    projects.each {
        def projDir = project(it).projectDir
        srcDirs.add("${projDir}/src") // assume that android main sourceSets is here
    }
    return files(srcDirs)
}

// generate coverage report for this project and all its project dependencies
task jacocoTestReport(type: JacocoReport, dependsOn: test) {
    reports {
        xml.enabled false
        csv.enabled false
        html.destination "${buildDir}/reports/coverage"
    }

    // TODO: automatically get project dependencies recursively
    def dependencies = [] // your gradle project dependencies go here
    classDirectories = getJacocoClassDirs(dependencies)
    sourceDirectories = getJacocoSrcDirs(dependencies)
    executionData = files("${buildDir}/jacoco/testDebug.exec")
}
{% endhighlight %}
