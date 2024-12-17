# Maven命令

## clean

清除已编译信息（删除工程中的target目录）。

## compile

只编译。 类似javac命令

## package

打包。包含**`编译，打包`**两个功能。

## install

本地安装， 包含编译，打包，安装到本地仓库。

## install和package的区别

- package命令完成了项目编译、单元测试、打包功能，但没有把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库

- install命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库，但没有布署到远程maven私服仓库