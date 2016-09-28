1. 统计行数

  统计文件行数（单个文件）：

  wc -l file

  例如：

  ```
  homer@ubuntu:~/workspace/android/game$ wc -l LGameAndroid2DActivity.java 
  906 LGameAndroid2DActivity.java
  ```

  统计目录所有文件行数（全部目录）：

  `find . -name *.java | xargs wc -l`

  例如：

  ```
  homer@ubuntu:~/workspace/android$ find . -name *.java | xargs wc -l
  817 ./game/core/LHandler.java
  140 ./game/core/LFlicker.java
  ...
  515 ./game/utils/collection/ArrayMap.java
  162 ./game/utils/CollisionUtils.java
  178 ./game/utils/NumberUtils.java
  68753 total
  ```

  统计目录并按行数排序（按行大小排序）：

  ```
  find . -name *.java | xargs wc -l | sort -n
  homer@ubuntu:~/workspace/android$ find . -name *.java | xargs wc -l | sort -n
  25 ./game/action/sprite/Collidable.java
  26 ./game/core/graphics/component/CollisionQuery.java
  27 ./game/core/graphics/filter/ImageFilter.java
  28 ./game/LMode.java
  ...
  1467 ./game/core/geom/Path2D.java
  1919 ./game/core/graphics/Screen.java
  2417 ./game/core/graphics/device/LGraphics.java
  3050 ./game/core/geom/AffineTransform.java
  68753 total
  ```

  统计目录并按行数排序（按行文件名排序）：

  ```
  find . -name *.java | xargs wc -l | sort -k2
  homer@ubuntu:~/workspace/android$ find . -name *.java | xargs wc -l | sort -k2
  210 ./game/action/ActionControl.java
  116 ./game/action/ActionEvent.java
  34 ./game/action/ActionListener.java
  ....
  178 ./game/utils/NumberUtils.java
  342 ./game/utils/RecordStoreUtils.java
  58 ./game/utils/ScreenUtils.java
  650 ./game/utils/StringUtils.java
  68753 total
  ```

2. Eclipse运行环境

  需要jdk（jre）支持，在终端输入apt-get install openjdk-6-jre

  该环境无法支持最新版本的Eclipse(luna)，所以后来又重新安装了jdk1.8.0_20，具体步骤如下：

  前期准备：去官网下载JDK1.8版本，即jdk-8u20-linux-i586.tar.gz

    1. 解压

    2. 移动目录

      将解压后的整个文件夹复制到/usr/lib/jvm 目录下 ，如果此目录不存在就使用命令 sudo mkdir /usr/lib/jvm 建立该目录。

    3. 配置环境变量

      终端输入 gedit ~/.bashrc

      在文本最后加入以下语句：

      ```
      export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_20
      export JRE_HOME=${JAVA_HOME}/jre
      export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
      export PATH=${JAVA_HOME}/bin:$PATH
      ```

      备注：

      JAVA_HOME就是刚刚操作的文件夹的绝对路径，注意名称是否正确。

    4. 配置默认JDK版本

      ```
      sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_20/bin/java 300
      sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_20/bin/javac 300
      ```

      备注：

      会出现“update-alternatives: using /usr/lib/jvm/jdk1.8.0_20/bin/javac to provide /usr/bin/javac (javac) in auto mode.”提示信息，属于正常现象。

      `sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/jdk1.8.0_20/bin/jar 300`

      备注：

      会出现“update-alternatives: using /usr/lib/jvm/jdk1.8.0_20/bin/jar to provide /usr/bin/jar (jar) in auto mode.”提示信息，属于正常现象。

      检查系统配置：

      `sudo update-alternatives --config java`

      此时会出现如下界面：

      ```
      There are 2 choices for the alternative java (providing /usr/bin/java).
      Selection    Path                                      Priority   Status
      ------------------------------------------------------------
      * 0            /usr/lib/jvm/java-6-openjdk/jre/bin/java   1061      auto mode
      1            /usr/lib/jvm/java-6-openjdk/jre/bin/java   1061      manual mode
      2            /usr/lib/jvm/jdk1.8.0_20/bin/java          300       manual mode
      ```

      备注：

      输入数字2，敲击回车即可，表示指定刚刚安装的程序为系统默认，如下所示：

      `Press enter to keep the current choice[*], or type selection number: 2`

      此时，JDK就已经安装完成了。

    5. 查看JAVA版本

      在控制台输入java -version，随即打印出以下语句：

      ```
      java version "1.8.0_20"
      Java(TM) SE Runtime Environment (build 1.8.0_20-b26)
      Java HotSpot(TM) Client VM (build 25.20-b23, mixed mode)
      ```

      说明安装成功！


3. 安装OpenSSL环境

  备注：一开始我直接从官网下载源码包，编译后安装到Ubuntu中，动态库和头文件都放到了相应的目录下，但使用时会出现error: openssl/ssl.h: No such file or directory的错误，需要再安装一下环境：

  安装openssl（我已经安装，跳过）

  `# sudo apt-get install openssl`


  再安装以下：

  `# sudo apt-get install libssl-dev build-essential zlibc zlib-bin libidn11-dev libidn11`

  备注：以上方法持保留意见，因为后面发现如果换一个Openssl的安装版本就不会报错了，不能证实上述方法是否有用。



4. 部署ssh secure shell环境

  想要直接Windows下控制虚拟机，可以使用上述软件，如果Windows下可以ping通虚拟机，但连接失败，则有可能是虚拟机上服务器端未安装，查看方法如下：

  1. 打开终端先输入如下命令 ：

    `ps -e|grep ssh`

    若只有如下打印如下 ：

    `1425?     00:00:00 ssh-agent`

    那么ssh-server 没有启动；

  2. 再输入如下命令：

    `dpkg -l|grep openssh`

    若只有openssh-client 则说明没有安装服务器包；

    则直接输入如下命令安装即可：

    `sudo apt-get install openssh-serser`
    
    如果想直接通过root用户登录，则
    
    1. passwd
    
    2. nano /etc/ssh/sshd_config
    
    ```
    # Logging
    SyslogFacility AUTH
    LogLevel INFO

    # Authentication:
    LoginGraceTime 120
    #PermitRootLogin prohibit-password
    PermitRootLogin yes

    StrictModes yes

    ```


5. ubuntu中如何将终端快捷方式添加到右键

  `sudo apt-get install nautilus-open-terminal`


6. Debian下升级内核

  1. 先安装好Debian，然后编辑/etc/apt/source.list文件，加入backports相关内容；

  2. 使用aptitude更新源，然后搜索linux-image之类的关键词，找到新的内核包，安装；

  3. 结束后重启系统，使用新内核系统；

  4. 再次使用aptitude，搜索老版本内核，卸载，完成后，使用命令update-grub更新系统即可
