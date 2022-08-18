---
title : helm 錯誤筆記
tags : 學習
---

[toc]

helm 錯誤筆記


## docker file沒有上船
```
 docker build -t registry.oneec-dev.ai/oneec/oneec-consuming-order-status-events-job:dev-0.0.1 .
unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat /var/lib/jenkins/workspace/oneec-consuming-order-status-events-job/Dockerfile: no such file or directory
```


## depolyment.yaml 的repo沒有設定好
```
+ /usr/local/bin/helm upgrade oneec-consuming-recover-data-job helm/oneec-consuming-recover-data-job --namespace oneec --atomic --install --set image.version=dev-0.0.3
Release "oneec-consuming-recover-data-job" has been upgraded. Happy Helming!
NAME: oneec-consuming-recover-data-job
LAST DEPLOYED: Sun Mar 20 22:45:58 2022
NAMESPACE: oneec
STATUS: deployed
REVISION: 2
TEST SUITE: None
+ /usr/local/bin/helm upgrade oneec-call-to-backend-job helm/oneec-call-to-backend-job --namespace oneec --atomic --install --set image.version=dev-0.0.3
Release "oneec-call-to-backend-job" does not exist. Installing it now.
Error: release oneec-call-to-backend-job failed, and has been uninstalled due to atomic being set: timed out waiting for t
```

## yaml 空格沒有空好
```
+ /usr/local/bin/helm upgrade oneec-consuming-recover-data-job helm/oneec-consuming-recover-data-job --namespace oneec --atomic --install --set image.version=dev-0.0.2
Release "oneec-consuming-recover-data-job" does not exist. Installing it now.
Error: YAML parse error on consuming-recover-data-job/templates/deployment.yaml: error converting YAML to JSON: yaml: line 69: did not find expected '-' indicator
```

## 缺少建立springboot.sh檔案 導致無法建docker
```
+ /usr/local/bin/helm upgrade oneec-consuming-action-momo-job helm/oneec-consuming-action-momo-job --namespace oneec --atomic --install --set image.version=dev-0.0.47
Release "oneec-consuming-action-momo-job" does not exist. Installing it now.
Error: release oneec-consuming-action-momo-job failed, and has been uninstalled due to atomic being set: timed out waiting for the condition
```
## Helm 的資料夾不能過長
```

+ /usr/local/bin/helm upgrade oneec-consuming-order-status-events-get-canceled-order-job helm/oneec-consuming-order-status-events-get-canceled-order-job --namespace oneec --atomic --install --set image.version=dev-0.0.1
Error: release name is invalid: oneec-consuming-order-status-events-get-canceled-order-job
```
oneec-consuming-order-status-events-get-canceled-order-job<< 不可以
oneec-order-events-get-canceled-order-job<<可以
類似的錯誤 還有可能是因為有大寫名稱

## 測試沒有過關
```
> Task :lib:test

com.systex.oneec.consuming.action.job.EnumsTest > testDeliveryInfo FAILED
    java.lang.NullPointerException at EnumsTest.java:28

2 tests completed, 1 failed
```

## Helm 的chart中文字要對上
```
Release "oneec-consuming-action-et-mall-job-3" does not exist. Installing it now.
Error: release oneec-consuming-action-et-mall-job-3 failed, and has been uninstalled due to atomic being set: ConfigMap "consuming-action-et-mall-job-ˇ3-configmap" is invalid: metadata.name: Invalid value: "consuming-action-et-mall-job-ˇ3-configmap": a lowercase RFC 1123 subdomain must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character (e.g. 'example.com', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*')
```
## 同一支程式 多個設定檔部屬
![](https://i.imgur.com/vPg8tjr.png)
1. Chart.yaml 的 name 設定跟folder一樣
2. templates中的deployment.yaml
```
 containers:
      - name: {{ .Chart.Name }}
        image: "registry.oneec-dev.ai/oneec/oneec-consuming-order-status-events-job:{{ .Values.image.version }}"
```
需要用指定image的方法 而不能用參數方法



## jenkins 設定
```
pipeline {
    environment {
        projcet = 'oneec-test-job'
        registry = 'registry.oneec-dev.ai/oneec/oneec-test-job'
        registryCredential = '1d9cef99-c720-4826-b835-2584f5e5fcc2'
        gitTag = ''
        dockerImage = ''
        telegramChatId = -647787007
    }
    agent any
    stages {
        stage('Pull Code') {
            steps {
                script {
                    withCredentials([gitUsernamePassword(credentialsId: registryCredential, gitToolName: 'Default')]) {
                        sh """
                            git init
                            git remote add origin https://gitlab.oneec-dev.ai/oneec/${projcet}.git
                            git fetch origin --tags ${env.gitlabBranch}
                            git reset --hard FETCH_HEAD
                        """
                    }
                }
            }
        }
        stage('Check Tag Exist') {
            steps {
                script {
                    gitTag = gitTagName()
                    sh "echo gitTag=${gitTag}"                    
                    if (gitTag == null || gitTag == "null") {
                      currentBuild.result = 'ABORTED'
                      error('tag not exist')
                    }
                }
            }
        }
        stage('Build Jar') {
            steps {
                script {
                    def subdir = "/var/lib/jenkins/workspace/${projcet}"
                    if (gitTag.contains('dev-')) {
                        dir (subdir) {
                            sh """
                                export PATH=$PATH:/opt/gradle-7.3/bin
                                gradle build
                            """
                        }
                    }
                    if (gitTag.contains('ga-')) {
                        dir (subdir) {
                            sh """
                                export PATH=$PATH:/opt/gradle-7.3/bin
                                gradle build
                            """
                        }
                    }
                }
            }                    
        }
        stage('Build Image') {
            when {
              allOf {
                expression { gitTag != null }
                expression { gitTag != "null" }
              }
            }            
            steps {
                script {
                    dockerImage = docker.build registry + ":${gitTag}"
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    def subdir = "/var/lib/jenkins/workspace/${projcet}"
                    if (gitTag.contains('dev-')) {
                        docker.withRegistry( 'https://registry.oneec-dev.ai', registryCredential ) {
                            dockerImage.push()
                            dockerImage.push("dev-latest")
                        }
                        sh """
                            docker rmi -f ${registry}:${gitTag}
                            docker rmi -f ${registry}:dev-latest
                        """
                    }
                    if (gitTag.contains('ga-')) {
                        docker.withRegistry( 'https://registry.oneec-dev.ai', registryCredential ) {
                            dockerImage.push()
                            dockerImage.push("prod-latest")
                        }
                        sh """
                            docker rmi -f ${registry}:${gitTag}
                            docker rmi -f ${registry}:prod-latest
                        """
                    }
                }
            }                    
        }
        stage('Deplay to K8S') {
            steps {
                script {
                    def subdir = "/var/lib/jenkins/workspace/${projcet}"
                    if (gitTag.contains('dev-')) {
                        dir (subdir) {
                            sh """
                               /usr/local/bin/helm upgrade oneec-test-helm-job helm/oneec-test-helm-job --namespace oneec --atomic --install --set image.version=${gitTag}
                               /usr/local/bin/helm upgrade ${projcet} helm/${projcet} --namespace oneec --atomic --install --set image.version=${gitTag}
                            """
                        }
                    }
                    if (gitTag.contains('ga-')) {
                        dir (subdir) {
                            sh """
                                echo "GA_TAG"
                            """
                        }
                    }
                }
            }                    
        }
    }
    post {
        always {
            deleteDir()
            script {
                telegramMsg = "[Jenkins Job] - The <b>${projcet}:${gitTag}</b> was started by GitLab push by <b>${env.gitlabUserName}</b> and got a <b>${currentBuild.result}</b> result."
                sendTelegram(telegramMsg, telegramChatId)
            }
        }
    }
}

/** @return The tag name, or `null` if the current commit isn't a tag. */
String gitTagName() {
    commit = getCommit()
    if (commit) {
        desc = sh(script: "git describe --tags ${commit}", returnStdout: true)?.trim()
        if (isTag(desc)) {
            return desc
        }
    }
    return null
}

String getCommit() {
    return sh(script: 'git rev-parse HEAD', returnStdout: true)?.trim()
}

@NonCPS
boolean isTag(String desc) {
  if(desc.length() > 12){
    match = null

  }else{
    return desc
  }
}

def sendTelegram(message, chat_id) {
    def requestBody = ["text":"$message","chat_id":"$chat_id", "parse_mode":"html"]

    withCredentials([string(credentialsId: 'telegramToken', variable: 'TOKEN')]) {
        response = httpRequest (consoleLogResponseBody: true,
            contentType: 'APPLICATION_JSON_UTF8',
            httpMode: 'POST',
            url: "https://api.telegram.org/bot$TOKEN/sendMessage",
            requestBody: groovy.json.JsonOutput.toJson(requestBody),
            validResponseCodes: '200')
        return response
    }
}
```
 stage('Deplay to K8S')
 中 upgrade的語法設定兩個路徑
 
## 刪除已經註冊的helm檔
```
D:\data\k8s>helm --kubeconfig D:\data\k8s\config uninstall oneec-consuming-recover-data-job -n oneec
release "oneec-consuming-recover-data-job" uninstalled
```
要先去下載helm 程式
然後用指定config的方法去刪除

https://helm.sh/docs/intro/install/

## \r 錯誤
![](https://i.imgur.com/gtqVEQ6.jpg)

原因: windows 的結尾跟linux的不同
因此要用notepad++ 改變字元尾吧(unix格式)
編輯>>換行格式>>UNIX格式

注意 : 此問題無法因git做同步


```
--oneec-call-to-backend-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-call-to-backend-job helm/batch-job/oneec-call-to-backend-job --namespace oneec --atomic --install --reset-values

--oneec-consuming-action-et-mall-job  
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-consuming-action-et-mall-job helm/batch-job/oneec-consuming-action-et-mall-job --namespace oneec --atomic --install --reset-values

--oneec-consuming-action-momo-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-consuming-action-momo-job helm/batch-job/oneec-consuming-action-momo-job --namespace oneec --atomic --install --reset-values

--oneec-consuming-action-shopee-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-consuming-action-shopee-job helm/batch-job/oneec-consuming-action-shopee-job --namespace oneec --atomic --install --reset-values

--oneec-consuming-action-yahoo-mall-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-consuming-action-yahoo-mall-job helm/batch-job/oneec-consuming-action-yahoo-mall-job --namespace oneec --atomic --install --reset-values

--oneec-consuming-order-processing-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-consuming-order-processing-job helm/batch-job/oneec-consuming-order-processing-job --namespace oneec --atomic --install --reset-values

--oneec-consuming-periodic-duty-5-minutes-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-consuming-periodic-duty-5-minutes-job helm/batch-job/oneec-consuming-periodic-duty-5-minutes-job --namespace oneec --atomic --install --reset-values

--oneec-create-recommend-prepare-data-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-create-recommend-prepare-data-job helm/batch-job/oneec-create-recommend-prepare-data-job --namespace oneec --atomic --install --reset-values

--oneec-creating-schedule-create-order-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-creating-schedule-create-order-job helm/batch-job/oneec-creating-schedule-create-order-job --namespace oneec --atomic --install --reset-values

--oneec-creating-schedule-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-creating-schedule-job helm/batch-job/oneec-creating-schedule-job --namespace oneec --atomic --install --reset-values

--oneec-order-events-get-canceled-order-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-order-events-get-canceled-order-job helm/batch-job/oneec-order-events-get-canceled-order-job --namespace oneec --atomic --install --reset-values

--oneec-order-events-get-new-order-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-order-events-get-new-order-job helm/batch-job/oneec-order-events-get-new-order-job --namespace oneec --atomic --install --reset-values

--oneec-order-events-get-new-refund-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-order-events-get-new-refund-job helm/batch-job/oneec-order-events-get-new-refund-job --namespace oneec --atomic --install --reset-values

--oneec-order-events-get-order-process-complete-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-order-events-get-order-process-complete-job helm/batch-job/oneec-order-events-get-order-process-complete-job --namespace oneec --atomic --install --reset-values

--oneec-recommend-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-recommend-job helm/batch-job/oneec-recommend-job --namespace oneec --atomic --install --reset-values

--oneec-sending-localfile-backtoqueue-job
helm --kubeconfig D:\data\k8s\k8s-ga-config.yaml upgrade oneec-sending-localfile-backtoqueue-job helm/batch-job/oneec-sending-localfile-backtoqueue-job --namespace oneec --atomic --install --reset-values


```