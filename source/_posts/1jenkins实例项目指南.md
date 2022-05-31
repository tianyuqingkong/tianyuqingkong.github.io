title: 1 jenkins在GVP_doc项目中的实际使用
date: 2020/5/16
categories:
- jenkins
---
## jenkins在项目中的实际使用

  在项目中有大体来说三种部署方式，第一种通过推送代码到特定分支完成自动部署线上环境加上本地部署环境，第二种是手动一键式部署线上环境，第三种是手动一键式部署本地环境。下面有这三种方式的简单说明。

<!--more-->
  这三种部署方式触发时都可以登录[jenkins网站](http://10.201.81.83:8098/)查看其过程情况，目前有一个账号后面会增加。
  账号：chenkuan 密码：xxxxxxx

  * 推送代码自动部署： 当代码推送到master分支时，`线上环境`和`本地部署环境`会自动部署如果部署成功或者失败会有邮件发给推送代码的人。部署成功具体表现是线上环境的网站内容发生了变化，已经本地部署环境的gitlab库tar包发生了替换。下面是注意事项。
      * 如果只想部署一个环境可以在下图对应代码注释，如下面就是注释了自动本地部署。
    ![choice](1jenkins实例项目指南/choice.png)  

      * 代码推送特定分支发生自动部署， 这个特定分支是可以配置的目前是配置的master，如果想改变有下面步骤：  
      登录到jenkins网站 -> auto_online_deploy ->auto_online_GVPdocs -> Configure -> Buid Triggers -> Advanced -> Allowed branches -> Filter branches by name -> Include
      ![config](./1jenkins实例项目指南/config.png)  


  * 手动一键式部署线上环境，这个是一键点击完成将特定分支的代码（目前设置是master）部署到线上环境，点击完毕后看下面的job跑完就是发布成功。部署成功具体表现是线上环境的网站内容发生了变化。这里需要注意的是要有两个步骤一个是确认发布的分支是否是你想要发布的分支，另一个就是点击按钮。  
    * 确认分支  
    登录到jenkins网站 -> manual_online_deploy -> manual_online_GVPdocs -> Configure -> Source Code Management -> Branches to build
    ![vertifyConfig](1jenkins实例项目指南/vertifyConfig.png)  

    * 点击立即构建  
    登录到jenkins网站 -> manual_online_deploy -> manual_online_GVPdocs -> 点击立即构建  
    ![trigger](1jenkins实例项目指南/trigger.png)  

  * 手动一键式本地部署，和上面的手动一键式部署线上环境只是名字的不同由manual_online_GVPdocs变为manual_local_GVP_docs