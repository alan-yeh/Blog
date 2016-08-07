---
layout: post
date: 2016-05-25
title: Intellij idea 创建maven webapp步骤
tags: Java-Web
---
　　这篇文章记录如何在Intellij idea里创建一个标准maven webapp的步骤。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_create_new_project.png)

　　以我的习惯，我会先建一个空Project，然后再添加Module。因此下一步，选择新建`Empty Project`。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_create_new_empty_project.png)

　　选择保存目录，确认。接着选择新建Module。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_create_new_module.png)

　　在这个界面选择Maven，勾选上`Create from archetype`，选择`org.apache.maven.archetypes:maven-archetype-webapp`。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_select_maven_module.png)

　　输入你的GroupId、ArtifacId，继续Next。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_input_info.png)

　　继续输入你保存的目录，完成后来到这个界面。点击OK。接着Intellij idea就会开始构建项目。构建速度取决于你当前网速和电脑配置。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_module_setting.png)

　　构建完毕后，一般右上角提示`Maven projects need to be imported`，选择`Enable Auto-Import`。

　　当前的项目结构目录如图所示。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_project_structure.png)

　　你会发现当前目录下，并没有Java源文件目录，因此，我们现在还需要新建一个java目录，并在Intellij idea中将它设为源目录。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_new_sources_folder.png)
![](/assets/blog/idea-create-maven-webapp/intellij_idea_current_project_structure.png)

　　打开`Module Setting`。选中`src/main/java`目录，并将它设置为`Sources`目录。点击OK。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_set_sources.png)

　　最后，项目的目录结构如下。

![](/assets/blog/idea-create-maven-webapp/intellij_idea_final_structure.png)

　　Maven Webapp项目创建完毕了，现在可以开始写代码了。

