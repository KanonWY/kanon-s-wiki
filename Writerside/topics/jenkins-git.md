# jenkins_git

## 1、Jenkins 简单拉取代码
```Groovy

pipeline {
    agent {
        docker {
            image 'ubuntu:20.04'
        }
    }
    environment {
        GIT_CRE_ID = 'xxx'
        GIT_URL = 'xxx'
    }

    stages {
        stage('checkout') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                            branches: [[name: 'master']],
                            doGenerateSubmoduleConfigurations: false,  // false dont pull the submodule
                            extensions: [], submoduleCfg: [],
                            userRemoteConfigs:
                                [[
                                    credentialsId: "${GIT_CRE_ID}",
                                    url: "${GIT_URL}"
                                ]]
                            ])
                }
                script {
                    sh'''
                    ls -al
                    git log
                    '''
                }
            }
        }
    }
}
```
doGenerateSubmoduleConfigurations 为 true 则会拉取该仓库的所有子库，false 则不会拉取所有的子库。

## 2、Jenkins 代码拉取控制
GitSCM 配置 extensions 和 submoduleCfg 可以进行更加细致的拉取配置
```Groovy

```