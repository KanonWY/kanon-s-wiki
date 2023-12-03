# JenkinsKeyWord

## triggers
触发器允许 jenkins 使用下面的方法来触发流水线
- cron: 使用 cron 语法来定义何时触发流水线
- pollSCM: 通过使用cron语法，它允许您定义Jenkins何时检查新的源存储库更新。如果检测到更改，则将重新触发流水线。（从Jenkins 2.22开始可用）
- upstream: 将Jenkins任务和阈值条件作为输入。当列表中的任何任务符合阈值条件时，将触发流水线
下面是简单的例子：
```Groovy
pipeline {
    agent any
    triggers {
        cron('0 */4 * * 1-5')
    }
    stages {
        stage('info') {
            steps {
                script {
                    echo "hello world"
                }
            }
        }
    }
}
```
poolSCM
```Groovy
pipeline {
    agent any
    triggers {
        //Query repository weekdays every four hours starting at minute 0
        pollSCM('0 */4 * * 1-5')
    }
    stages {
        ...
    }
}
```
upstream
```Groovy
pipeline {
    agent any
    triggers {
        //当 job1 job2 成功的时候触发
        upstream(upstreamProjects: 'job1, job2', threshold: hudson.model.Result.SUCCESS)
    }
    stages {
        ...
    }
}
```
## POST
指定post{}可以定义一些失败或者成功的状态处理事件
```Groovy
pipeline {
    agent { 
        node {
            label "build"
        }
    }
    // 流水线阶段
    stages {
        // 1. get code
        stage('get code') {
            steps {
                script {
                    println("获取代码")
                }       
            }
        }
        stage('build code') {
            steps {
                script {
                    println("build code")
                }
            }
        }
    }
    post {
        always {
            script {
                println("流水线结束后需要做的事情")
            }
        }
        success {
            script {
                println("流水线成功后需要做的事情")
            }
        }
        failure {
            script {
                println("流水线失败后需要做的事情")
            }
        }
        aborted {
            script {
                println("流水线取消后需要做的事情")
            }
        }
    }

}

```