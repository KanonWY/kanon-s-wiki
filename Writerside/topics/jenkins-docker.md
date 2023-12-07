# jenkins_docker

## 1、在 jenkins 中使用 docker镜像作为流水线环境
如果要在 jenkins 中使用 docker 作为流水线的环境，需要满足以下的条件：
- 1、安装 docker 支持的插件
- 2、将 jenkins 用户加入到 docker 用户组中
- 3、编写流水线的时候指定 agent 为 docker  
以下是一个简单的流水线的示范：
```Groovy
pipeline {
    agen none
    stages {
        stage('docker test') {
            agent {
                docker {
                    image 'ubuntu:18.04'
                }
            }
            steps {
                script {
                    sh'''#! /bin/bash
                        uname -a
                    '''
                }
            }
        }
    }
}

```