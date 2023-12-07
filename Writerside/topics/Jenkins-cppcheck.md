# Jenkins_cppcheck

## 在 jenkins 中使用 cppcheck

### 1、安装 cppcheck
需要在 jenkins 的 master 节点和 slave 节点上全部安装 cppcheck
```Bash
sudo apt-get install cppcheck
```
在 jenkins 插件市场上安装 cppcheck 插件

### 2、生成编译数据库
在 CMake 的工程项目中，使用 CMAKE_EXPORT_COMPILE_COMMANDS 选项开启。
```Bash
cmake -S . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```
### 3、使用 cppcheck 生成 xml 结果报告
```Bash
cppcheck --enable=all --xml --xml-version=2  --project=./build/compile_commands.json 2>cppcheck.xml
```

### 3、在 jenkins 中使用  Warnings Next Generation Plugin. 插件
声明式流水线示例
```Groovy
pipeline {
    agent any
    environment {
        GIT_CRE_ID = '432a750f-7c6c-4da4-b61a-0569e66517e8'
        GIT_URL = 'https://gitee.com/kanon-wy/test_jenkins.git'
    }
    options {
        // 支持彩色输出
        ansiColor('xterm')
        // 禁用默认的代码检出行为
        skipDefaultCheckout(true)
        // 在控制台输出的每个步骤前添加时间戳
        timestamps()
    }
    stages {
        stage('checkout') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                            branches: [[name: 'master']],
                            userRemoteConfigs:
                                [[
                                    credentialsId: "${GIT_CRE_ID}",
                                    url: "${GIT_URL}"
                                ]],
                            doGenerateSubmoduleConfigurations: false,
                            extensions: [],
                            submoduleCfg: []
                    ])
                }
            }
        }
        stage('cppcheck') {
            steps {
                script {
                    sh'''#! /bin/bash
                        ls -alh
                        git log
                        cmake -S . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
                        cppcheck --enable=all --xml --xml-version=2  --project=./build/compile_commands.json 2>cppcheck.xml
                    '''
                }
            }
        }
        stage('publish result') {
            steps {
                recordIssues(tools: [cppCheck(pattern: 'cppcheck.xml')])
            }
        }
    }
}
```
**recordIssues(tools: [cppCheck(pattern: 'cppcheck.xml')]) 用于指定 cppcheck.xml 的位置。**

## Ref
[cppcheck example](https://cloud.tencent.com/developer/article/1643682)    
[warnings-ng-plugin 插件文档](https://github.com/jenkinsci/warnings-ng-plugin/blob/main/SUPPORTED-FORMATS.md)