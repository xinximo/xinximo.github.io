---
title: Python项目基于Docker+Jenkins实现CI/CD
date: 2022-02-14 15:35:51
tags: [Docker, Jenkins, CI/CD]
---
# Python项目基于Docker+Jenkins实现CI/CD

## 一、本地Python项目Dockerfile配置

1.因为自动化测试报告allure需要基于Java环境，基础镜像可以直接拉取我的（已配置为Java+Python3.6，参考本人其他博客），Python3.6之后因为基础库升级导致会和某些三方库产生方法冲突不建议使用

```bash
# docker pull xinximo/java8-python3.6
# Author: xinximo
# Version: 2021-10-27

# 注意:Mac本地构建使用这个镜像:docker pull xinximo/java8-python3.6
# linux使用这个镜像:docker pull xinximo/java8-python3.6-linux

FROM xinximo/java8-python3.6-linux

MAINTAINER xinximo "woshiwangxin123@126.com"
COPY requirements.txt ./
RUN apk add --no-cache bash git openssh && \
    python -m pip install --upgrade pip && \
    pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip install --no-cache-dir -r requirements.txt
COPY . .
```
<!--more-->
## 二、Jenkins初次安装时需要安装的配置

1.插件

allure

Build With Parameters 输入框式的参数

Persistent Parameter 下拉框式的参数

Python

groovy

ansiColor

Docker

global variable string parameter

[Build Timeout](https://plugins.jenkins.io/build-timeout)

[Email Extension Plugin](https://plugins.jenkins.io/email-ext)

[Extended Choice Parameter Plug-In](https://plugins.jenkins.io/extended-choice-parameter)

[GitHub Branch Source Plugin](https://plugins.jenkins.io/github-branch-source)

[Docker Pipeline](https://plugins.jenkins.io/docker-workflow)

[Blue Ocean Pipeline Editor](https://plugins.jenkins.io/blueocean-pipeline-editor)

[Job Configuration History](https://plugins.jenkins.io/jobConfigHistory)

2.凭证

（1）需要配置github上的私钥（jenkins-gitlab-ssh-key）
（2）需要配置hub.docker.com登录账号密码（docker-hub-xinximo）

3.Global Tool Configuration配置

（1）Allure Commandline添加allure安装的版本
（2）JDK添加Java8版本

## 三、Jenkins-Pipline配置

1.Jenkins拉取github项目并自动构建为镜像Pipline语法

```bash
pipeline {
    agent any
    options {
        ansiColor('xterm')
        timeout(time: 90, unit: 'MINUTES')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '30')
    }
    parameters{
        string(defaultValue: 'master', description: '代码分支', name: 'CODE_BRANCH', trim: true)
        string(defaultValue: '', description: 'Merge Request IDS, eg. 1,2', name: 'MR_IDS', trim: true)
    }
    environment {
        CODE_REPO = "git@github.com:xinximo/TestPlatform.git"
    }
    stages {
        stage('Prepare code') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM',
                    branches: [[name: params.CODE_BRANCH], [name: '*/merge-requests/*']], 
                    userRemoteConfigs: [[
                        credentialsId: 'jenkins-gitlab-ssh-key', 
                        refspec: '+refs/merge-requests/*/head:refs/remotes/origin/merge-requests/*', 
                        url: CODE_REPO ]]])
                script {
                    sh "git checkout ${params.CODE_BRANCH}"
                    mr_ids = params.MR_IDS.trim().split("(\\s+|\\s*,\\s*)")
                    mr_ids.findAll{ it.length() > 0 }.each { id ->
                        sh "git checkout ${params.CODE_BRANCH}"
                        sh "git merge origin/merge-requests/${id}"
                    }
                }
            }
        }
        stage('build-docker') {
            steps {
                script{    
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-xinximo') {
                        def dockerfile = 'Dockerfile'
                        def customImage = docker.build("xinximo/test-platform:1.0-${env.BUILD_ID}")
                        customImage.push()
                    }
                }
            }
        }
    } 
    // post {
    //     always {
    //         echo 'Send Email'
    //     }
    // }
}
```

2.Jenkins拉取项目镜像并运行Pipline语法

```bash
pipeline {
    agent {
        docker {
            image 'xinximo/test-platform:1.0-20'
            registryUrl 'https://registry.hub.docker.com'
            registryCredentialsId 'docker-hub-xinximo'
            reuseNode true
        }
    }
    options {
        ansiColor('xterm')
        timeout(time: 60, unit: 'MINUTES')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '30')
    }
    parameters{
        string(defaultValue: 'master', description: '代码分支', name: 'CODE_BRANCH', trim: true)
        string(defaultValue: '', description: '项目的代码 Merge Request IDS, eg. 1,2', name: 'MR_IDS', trim: true)
    }
    environment {
        CODE_REPO = "git@github.com:xinximo/TestPlatform.git"
    }
    stages {
        stage('Prepare code') {
            steps {
                cleanWs()
                // git clone 
                // git branch: params.CODE_BRANCH, url: CODE_REPO, credentialsId: 'jenkins-gitlab-ssh-key'
                sh """
                git config --global user.email "woshiwangxin123@126.com";
                git config --global user.name "xinximo";
                """
                checkout([$class: 'GitSCM',
                    branches: [[name: params.CODE_BRANCH], [name: '*/merge-requests/*']], 
                    userRemoteConfigs: [[
                        credentialsId: 'jenkins-gitlab-ssh-key', 
                        refspec: '+refs/merge-requests/*/head:refs/remotes/origin/merge-requests/*', 
                        url: CODE_REPO ]]])
                script {
                    mr_ids = params.MR_IDS.trim().split("(\\s+|\\s*,\\s*)")
                    mr_ids.findAll{ it.length() > 0 }.each { id ->
                        sh "git checkout ${params.CODE_BRANCH}"
                        sh "git merge origin/merge-requests/${id}"
                    }
                sh "pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple some-package"
                }
            }
        }
        stage('Start Run') {
            failFast false
            parallel {
                stage('Run') {
                    steps {
                        // sh "cd backend && python back.py"
                        sh "cd testcase && pytest test_server.py"
                    }
                }
            }
        }
        
    }
    post('Allure_Results') {
        always {
            script{// 集成allure，目录需要和保存的results保持一致，注意此处目录为job工作目录之后的目录，Jenkins会自动将根目录与path进行拼接
                allure includeProperties: false, jdk: 'JDK1.8', report: 'report/report', results: [[path: 'report']]
            }
        }
    }
}
```
