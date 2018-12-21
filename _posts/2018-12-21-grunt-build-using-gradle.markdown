---
layout: post
title:  "Run grunt using gradle"
date:   2018-12-21 15:41:34 -0700
categories: grunt gradle build ui oracle jet weblogic war
---
If you have microservices in your project chances are that you are using gradle for your microservices. If you also have ui and ui engineers use grunt, you want to have one build system that can be integrated with jenkins. You can spin off grunt from gradle

Following gradle script takes care of launching and generating war file using gradle

```
buildscript {
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath "com.moowork.gradle:gradle-node-plugin:1.2.0"
    }
}
apply plugin: "com.moowork.grunt"
apply plugin: 'war'

node {
     workDir = file("${project.buildDir}/nodejs")
    npmWorkDir = file("${project.buildDir}/npm")
    nodeModulesDir = file("${project.projectDir}/ui")
}

grunt {
    workDir = file("${project.projectDir}/ui")
    colors = true
    bufferOutput = false
    gruntFile = 'Gruntfile.js'
}


def srcDir = new File(projectDir, "ui")
def targetDir = new File(project.buildDir, "target")
grunt_dist.inputs.dir srcDir
grunt_dist.outputs.dir targetDir


task ui_build(type: GruntTask) {
   if (project.hasProperty('env') && 'prod' == project.property("env")) {
      args = ["build:release"]
   } else {
      args = ["build"]
   }
}

ui_build.dependsOn 'installGrunt'
ui_build.dependsOn 'npmInstall'
project.tasks['war'].dependsOn 'ui_build'

clean.doFirst {
    //web directory contains the output of grunt build
    delete "${project.projectDir}/ui/web"
}

war {
    archiveName = "mywebapp.war"
    enabled = true
    from ("${project.projectDir}/ui/web") {
        include '**/**'
    }
    manifest {
    }
}

```