# pipelineBase

## 1. Jenkins 环境变量
查看当前[环境变量](http://127.0.0.1:8080/pipeline-syntax/globals#env)
- BUILD_ID: 当前构建ID
- BUILD_NUMBER: 当前构建号
- BUILD_TAG: **jenkins-${JOB_NAME}-${BUILD_NUMBER}**
- BUILD_URL: 本次构建结果的 URL (http://buildserver/jenkins/job/MyJobName/17/)
- JOB_NAME: 本次构建的项目名称
- NODE_NAME: 运行本次构建的节点名称
- WORKSPACE：workspace 的绝对路径

## 2. credentials
许多三方网站和应用可以与Jenkins交互，如Artifact仓库，基于云的存储系统和服务等可以在应用程序中配置 credentials 供Jenkins使用，
通常将访问控制应用于这些 credentials， 以“锁定”Jenkins可用的应用程序功能区域。一旦Jenkins管理员（即管理Jenkins站点的用户） 
在Jenkins中添加/配置这些 credentials，Pipeline项目就可以使用 credentials 与三方应用交互。


## Ref
[pipeline 基本语法](https://www.cnblogs.com/FLY_DREAM/p/17279733.html)  
[部署基础](https://github.com/zhouxinlei828/sparkzxl-framework/tree/master/docs/forward/deploy)  
[官方文档](https://www.jenkins.io/zh/doc/book/pipeline/jenkinsfile/)  
[图文文档](https://mdnice.com/writing/eaf78c6cd2cf44b7aba8cbc6431ecead)  
[jenkins 官方中文文档](https://www.k8stech.net/jenkins-docs/pipelinesyntax/chapter03/)  