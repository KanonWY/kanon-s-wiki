# jenkins_git

## 1、Jenkins 拉取代码
```Groovy
stage("pull git code") {
    deleteDir()
    checkout()(
}
```