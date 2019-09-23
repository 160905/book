1.gitlab-ci.yml参数列表

| 值 | 是否必须 | 描述 |
| --- | --- | --- |
| script | 必须 | 定义由Runner执行的shell脚本或命令 |
| extends | 非必须 | 定义此作业将继承的配置条目 |
| images | 非必须 | 需要使用的docker镜像，请查阅该文档 |
| services | 非必须 | 定义所需的docker服务，请查阅该文档 |
| stage | 非必须 | 定义一个工作场景阶段，默认是test |
| type | 非必须 | stage的别名,不赞成使用 |
| variables | 非必须 | 在job级别上定义的变量 |
| only | 非必须 | 定义job所引用的git分支 |
| except | 非必须 | 定义job所不适用的git分支 |
| tags | 非必须 | 定义job所适用的runner，tags为runner标签 |
| allow_failure | 非必须 | 允许任务失败，但是如果失败，将不会改变提交状态 |
| when | 非必须 | 定义了job什么时候执行，可以是on_success、on_failure、always和manual |
| dependenices | 非必须 | 定义了该job依赖哪一个job，如果设置该项，可以通过artifacts设置 |
| artifacts | 非必须 | 工件，在依赖项之间传递的东西，类似cache，但原理与cache不同 |
| catch | 非必须 | 定义需要被缓存的文件、文件夹列表 |
| before_script | 非必须 | 覆盖在作业之前执行的脚本或命令 |
| after_script | 非必须 | 覆盖在作业之后执行的脚本或命令 |
| environment | 非必须 | 定义让job完成部署的环境名称 |
| coverage | 非必须 | 定义job设置代码覆盖率 |
| retry | 非必须 | 定义job失败后的自动重试次数 |
2.gitlab-ci配置示例
``` shell
# docker镜像
image: ruby:2.1
# 依赖的docker服务
services:
  - postgres
# 开始执行脚本前所需执行脚本
before_script:
  - bundle install
# 脚本执行完后的钩子，执行所需脚本
after_script:
  - rm secrets
# 该ci pipeline适合的场景
stages:
  - build
  - test
  - deploy
# 定义的任务1
job1:
  # 场景为构建
  stage: build
  # 所需执行的脚本
  script:
    - execute-script-for-job1
  # 在哪个分支上可用
  only:
    - master
  # 指定哪个ci runner跑该工作
  tags:
    - docker
```
3.生产环境配置示例
``` shell
# general settings for all
.general: &general
  stage: deploy     #定义构建场景为部署，1.init初始化、2.lint代码规范、3.unit_test单元测试、4.build构建、5.deploy部署，若其中任务一个步骤出错，都不会到部署
  only:
    - hotfix/hotfix-conference      #指定分支名为紧急修bug，master主开发分支、feature新功能分支、release发布分支、hotfix紧急修bug分支
  when: manual   #触发条件为手工执行
  tags:
    - cloud         #指定在哪个ci runner工作，云
  image: ip:30050/builder/maven:v1-alpine      #青云maven容器
  script:
    - echo "current branch ****** $CI_COMMIT_REF_NAME ******"
    - echo "deploy start ..."
 
    - /share/script/deploy.sh $CI_JOB_NAME     #执行脚本，此脚本路径是被映射至gitlab runner容器内的路径，参数为CI_JOB_NAME变量
     
    - echo "deploy done."
 
# general settings for dev
.dev: &dev        #开发环境
  <<: *general    #继承general定义的变量，若重新定义将被覆盖
  tags:
    - local     #本地
  image: ip:30050/builder/maven:v1-alpine   #本地maven容器
 
 
# golive deploy setting
.golive: &golive  
  <<: *general  #继承general定义的变量，若再定义将被覆盖
  only:
    - master    #指定分支名
  script:
    - echo "current branch ****** $CI_COMMIT_REF_NAME ******"
    - echo "deploy golive start ..."
 
    - /share/script/golive/deploy.sh $CI_JOB_NAME
     
    - echo "deploy golive done."
 
 
#######定义hotfix/hotfix-conference 分支使用dev使用dev job的配置、test使用 general job的配置  
# deploy discovery
discovery - dev: *dev
discovery - test: *general
 
# deploy services
services - dev: *dev
services - test: *general
 
######定义仅master分支使用golive job定义的配置
backend - staging - node1: *golive
backend - staging - node2: *golive
 
backend - prod - node1: *golive  
backend - prod - node2: *golive
```