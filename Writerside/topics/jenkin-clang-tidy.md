# jenkins-clang-tidy

## 使用 jenkins 来集成 clang-tidy
### 1、安装 clang-tidy
```Bash
sudo apt-get install clang-tidy
```

### 2、指定 clang-tidy 的选项
在项目根目录下创建 .clang-tidy 文件，然后填写下列检查选项
```Bash
Checks: >
  -abseil-no-namespace,
  bugprone*,
  # Sadly narrowing conversions is too noisy
  -bugprone-narrowing-conversions,
  -bugprone-easily-swappable-parameters,
  -bugprone-branch-clone,
  -bugprone-implicit-widening-of-multiplication-result,
  boost-use-to-string,
  performance*,
  cert*,
  -cert-err58-cpp,
  # Doesn't work with abseil flags
  clang-analyzer*,
  google-*,
  -google-runtime-int,
  -google-readability-*,
  -google-build-using-namespace,
  misc-definitions-in-headers,
  misc-misleading*,
  misc-misplaced-const,
  misc-new-delete-overloads,
  misc-non-copyable-objects,
  misc-redundant-expression,
  misc-static-assert,
  misc-throw-by-value-catch-by-reference,
  misc-unconventional-assign-operator,
  misc-uniqueptr-reset-release,
  misc-unused-alias-decls,
  misc-unused-using-decls,
  modernize-deprecated-headers,
  modernize-macro-to-enum,
  modernize-make-shared,
  modernize-make-unique,
  modernize-pass-by-value,
  modernize-raw-string-literal,
  modernize-redundant-void-arg,
  modernize-replace-disallow-copy-and-assign-macro,
  modernize-return-braced-init-list,
  modernize-shrink-to-fit,
  modernize-unary-static-assert,
  modernize-use-emplace,
  modernize-use-equals-delete,
  modernize-use-noexcept,
  modernize-use-transparent-functors,
  modernize-use-uncaught-exceptions,
  modernize-use-using,
  readability-avoid-const-params-in-decls,
  readability-const-return-type,
  readability-container-contains,
  readability-container-size-empty,
  readability-delete-null-pointer,
  readability-duplicate-include,
  readability-function-size,
  readability-identifier-naming,
  readability-inconsistent-declaration-parameter-name,
  readability-make-member-function-const,
  readability-misplaced-array-index,
  readability-named-parameter,
  readability-non-const-parameter,
  readability-redundant-access-specifiers,
  readability-redundant-control-flow,
  readability-redundant-declaration,
  readability-redundant-function-ptr-dereference,
  readability-redundant-member-init,
  readability-redundant-preprocessor,
  readability-redundant-smartptr-get,
  readability-redundant-string-cstr,
  readability-redundant-string-init,
  readability-simplify-subscript-expr,
  readability-static-definition-in-anonymous-namespace,
  readability-string-compare,
  readability-suspicious-call-argument,
  readability-uniqueptr-delete-release,
  readability-use-anyofallof


# Disabled because they're currently too disruptive, but one day might be nice to have:
# modernize-use-nullptr,
# modernize-use-equals-default,
# readability-qualified-auto,
```

### 3、clang-tidy 生成 txt 报告
- CMake 工程指定 CMAKE  
```Bash
cmake -S . -B build -DDCMAKE_EXPORT_COMPILE_COMMANDS=ON
clang-tidy -p=build src/main.cpp > clang-tidy.txt
clang-tidy -p=build cpp源码 > clang-tidy.txt
```

### 3、使用 jenkins 插件查看 clang-tidy 问题
```Groovy
pipeline {
    agent any
    environment {
        GIT_CRE_ID = '432a750f-7c6c-4da4-b61a-0569e66517e8'
        GIT_URL = 'https://gitee.com/kanon-wy/test_jenkins.git'
        CLANG_CHECK = '-*,misc-*,modernize-*'
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
                        echo "---> ${CLANG_CHECK}"
                        clang-tidy -p=build src/main.cpp > clang-tidy.txt
                    '''
                }
            }
        }
        stage('publish clang-tidy result') {
            steps {
                recordIssues(tools: [clangTidy(pattern: 'clang-tidy.txt')])
            }
        }
    }
}
```
使用 recordIssues 指定 生成的报告路径即可。