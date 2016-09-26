1. 安装osip4.1.0和exosip4.1.0

  进入安装目录下，执行

  ```
  $> ./configure
  $> make
  $> sudo make install
  ```

  安装成功后的文件都放在/usr/local/lib下，所以需要指定一下，方便今后使用：

  ```
  $> export PATH=$PATH:/usr/local/lib
  $> sudo ldconfig
  ```

2. sip中的摘要认证默认使用MD5算法，我使用openssl开源库的接口实现：

  实例代码如下：

  ```
  #include <openssl/md5.h>
  #include <iostream>
  #include <cstdio>
  #include <iomanip>
  #include <stdlib.h>
  #include <string.h>
  using namespace std;

  int main()
  {
      std::cout<<"Input: "<<std::endl;
      std::string tmp;
      getline(std::cin,tmp);

      std::cout<<"Your input is: "<<tmp<<std::endl;

      MD5_CTX c;
      unsigned char md5[17]={0};

      MD5_Init(&c);

      MD5_Update(&c, tmp.c_str(), tmp.length());

      MD5_Final(md5,&c);

      std::cout<<"MD5 is: "<<std::endl;
      for(int i = 0; i < 16; i++)
          cout << hex << setw(2) << setfill('0') << (int)md5[i];
      cout << endl;

      return 0;
  }
  ```
