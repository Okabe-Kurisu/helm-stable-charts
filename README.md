# 如何维护helm stable库
`helm stable`库是一个chart表的集合，在`https://kubernetes-charts.storage.googleapis.com/index.yaml`中可以下载到一个`index.yaml`这个文件中包含了所有`helm stable`库中打好的tgz包。

## 工作流程 - 手动

1. ### 下载chart包
    首先，在linux 环境中使用
    ``` shell
    curl https://kubernetes-charts.storage.googleapis.com/index.yaml | grep -o "http.*.tgz" | xargs wget -P ./hubtgz/
    ```
    来下载`index.yaml`中的全部tgz。

2. ### 解压并按照版本合并chart
    将`./hubtgz/`中的chart包解压后，得到一大堆文件夹，大概如同下图所示：
    ```
    ./hubtgz/
        projectA-versionA/
        projectA-versionB/
        projectA-versionC/
        projectB-versionA/
        projectB-versionB/
        projectB-versionC/
        projectB-versionD/
    ```
    需要将这些包合并成这种格式：
    ```
    ./hubtgz/
        projectA/
            versionA/
            versionB/
            versionC/
        projectB/
            versionA/
            versionB/
            versionC/
            versionD/
    ```

3. ### 获取每个项目的图标
    在每个解压好的chart都会有一个叫`chart.yaml`的文件，在这个文件中有可能会有一个叫icon的字段，如果有的话，这个字段就是这个项目的图标的url，下载这个图标，并重命名为icon（后缀名不动）。如果没有的话就上网找一个该项目的图标，也重命名。
    
    然后并将这个图标放在项目根目录下，如下图所示
    ```
    ./hubtgz/
        projectA/
            versionA/
            versionB/
            versionC/
            icon.png
    ```
    但是这个样子的话，在容器云平台中还是会从其`chart.yaml`文件中icon地址去下载logo，所以我们需要将该项目下所有版本的`chart.yaml`中的icon字段修改为本地地址。如
    ```yaml
        icon: file://../icon.png
    ```

4. ### 创建git，并上传到什么地方
    将处理完的全部文件夹用一个更大的文件夹包裹着，然后在当前目录下使用`git init`创建一个git仓，并且将这个git仓上传到某个容器云平台能访问到的地址上。git项目结构可以参考下图：
    ```
    ./charts/
        .git/
        templates/
            projectA/
                versionA/
                versionB/
                icon.png
            projectB/
                versionA/
                icon.png
        README.md 
    ```
5. ### 在容器云平台中添加应用源
   登陆到容器云平台中，在顶部导航栏中找到`APP`或者`应用`一览，点击。然后按照提示添加应用源即可。

## 工作流程 - 脚本
首先下载一个`rancher-tools`，并按照其中的说明进行安装。安装完成后可以使用以下命令完成上述的操作。

   1. 下载chart包
        ```
        python3 main.py --gat
        ```

   2. 解压并按照版本合并chart
        ```
        python3 main.py --fut
        ```

   3. 获取每个项目的图标
        ```
        python3 main.py --gaicon
        ```
        注意，在这一步中会有很多图标无法被下载或者是就没图标，可以手动下载一个并按照手动第三步的操作重命名并放在项目中。然后再运行一次`python3 main.py --gaicon`就可以自动修改其项目下全部yaml文件中icon字段。

        至于没有图标的项目有哪些，可以在脚本路径下的`out/noIconList.txt`文件中看到。

   4. 创建git，并上传到什么地方
        这一步中创建git，以及commit的过程都会在前面的命令中被完成。然后通过
        ```
        python3 main.py --git
        ```
        来提交更改。

   5. 在容器云平台中添加应用源
        这一部还是继续手动添加吧。