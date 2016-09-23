环境准备：

操作系统：ubuntu-10.04-desktop-i386版本，新安装，之前没有安装过任何程序。

1. 总体介绍

  ONVIF是一套基于安防产品的标准，采用SOAP协议实现，SOAP是基于XML的简易协议，可使应用程序在HTTP之上进行信息交换。SOAP提供了一种标准的方法，使得运行在不同的操作系统并使用不同的技术和编程语言的应用程序可以互相进行通信。SOAP协议手工实现费时费力，所以出现了GSOAP，它自动将用户定义的本地化的C或C++数据类型转变为符合XML语法的数据结构。

2. GSOAP的下载、部署和测试
    1. 访问GSOAP官网（`http://www.cs.fsu.edu/~engelen/soap.html`），点击Download菜单，进入下载页面，下载2.8.16版本的gsoap压缩包；
    2. 安装gsoap；
        1. 在ubuntu任意目录下解压该压缩包，生成gsoap-2.8目录；
        2. 安装g++编译器；
            sudo apt-get install g++
        3. 安装必要的bison环境；
            sudo apt-get install bison
        4. 安装必要的flex环境；
            sudo apt-get install flex
        5. 安装必要的openssl环境（ubuntu默认已经安装）；
            sudo apt-get install openssl
        6. 安装必要的其它环境；
            sudo apt-get install libssl-dev build-essential zlibc zlib-bin libidn11-dev libidn11
        7. 运行配置GSOAP属性命令；
            ./configure
        8. 运行make命令；
            make
        9. 运行make install命令；
            sudo make install
        到此GSOAP部署完成。
        
    3. 测试gsoap环境（可以跳过）；
    
        测试目的：
        
          测试gsoap环境是否安装成功，工具和相关头文件是否能够成功引用；
            
        测试内容：
        
          创建客户端和服务器端，相互通信采用TCP协议，客户端和服务器端只支持本地运行，服务器端为单线程，绑定本地回环地址负责监听，客户端发送两个整形数字给服务器端，服务器端将其相加后的值返回给客户端。

        测试代码：
        
          1. 编写服务器端源代码；
            
          创建StandaloneAdd.c源文件：
            
          ```
          ————————————————————————————————————
          *****StandaloneAdd.c*****
          #include "soapH.h"
          #include "add.nsmap"


          int main()
          {
              struct soap soap;
              int master, slave;
              soap_init(&soap);
              master = soap_bind(&soap, "127.0.0.1", 18888, 100); 
              if(master < 0)
                  soap_print_fault(&soap, stderr);
              else
              {
                  sprintf(stderr, "Socket connection successful:master socket = %d\n", master);
  
              while(1)
              {
                  slave = soap_accept(&soap);
                  if(slave < 0)
                  {
                      soap_print_fault(&soap, stderr);
                      break;
                  }
                  sprintf(stderr, "accepted connection from IP=%d.%d.%d.%d socket=%d",(soap.ip >> 24)&0xFF, (soap.ip >> 16)&0xFF, (soap.ip >> 8)&0xFF, soap.ip&0xFF, slave);
                  if(soap_serve(&soap) != SOAP_OK)
                      soap_print_fault(&soap, stderr);
                  fprintf(stderr, "request served\n");
                  soap_destroy(&soap);
                  soap_end(&soap);
              }
              }
              soap_done(&soap);
          }            


          int ns__add(struct soap *soap, double a, double b, double *result)
          {
              *result = a + b;
              return SOAP_OK;
          }
          ————————————————————————————————————
          ```

          ```
          创建StandaloneAdd.h源文件：
          ————————————————————————————————————
          *****StandaloneAdd.h*****
          //gsoap ns service name:    add Simple StandaloneAdd service
          //gsoap ns service style:    rpc
          //gsoap ns service enconding:    encoded
          //gsoap ns service namespace:    http://localhost/StandaloneAdd.wsdl
          //gsoap ns service location:    http://localhost:18888
          //gsoap ns schema namesapce:    urn:add


          int ns__add(double a, double b, double *result);
          ```

          注意：
            
          注释内容是有作用的，soapcpp2需要根据这些注释内容生成相关文件。location指的是服务的地址，这里把服务绑定在本机的端口号为18888的端口上。

          2. 使用gsoap工具编译源码；
        
            进入到gsoap安装目录gsoap-2.8/gsoap下，将stdsoap2.c和stdsoap2.h文件复制到刚刚编写的源代码目录下，然后运行以下命令行：
            
            1. $soapcpp2 -c StandaloneAdd.h
            
              -c参数指明这里生成的文件是纯C的格式的文件。
              
            2. $cc -o StandaloneAdd StandaloneAdd.c stdsoap2.c soapC.c soapServer.c
            
              查看当前目录，会发现新生成了很多文件，其中有个可执行文件StandaloneAdd和wsdl服务文件。
              
              如果提示soapcpp2找不到，说明gsoap没有部署成功，需要重新部署。
              
              到目前为止，服务端的程序就已经完成了。
              
          3. 编写客户端源代码；
        
            创建ClientTest.c源文件：
            
            ```
            ————————————————————————————————————
            ***** ClientTest.c*****
            #include "soapH.h"
            #include "add.nsmap"
  
  
            const char server[] = "http://localhost:18888";
  
  
            int main(int argc, char **argv)
            {
                struct soap soap;
                double a, b, result;
                soap_init1(&soap, SOAP_XML_INDENT);
                a = strtod(argv[2], NULL);
                b = strtod(argv[3], NULL);
    
                soap_call_ns__add(&soap, server, "", a, b, &result);
                if(soap.error)
                {
                    soap_print_fault(&soap, stderr);
                    exit(1);
                }
                else
                    printf("result = %g\n", result);
                soap_destroy(&soap);
                soap_end(&soap);
                soap_done(&soap);
                return 0;
    
            }
            ————————————————————————————————————
            ```
            
          4. 使用gsoap工具编译源码；
        
            进入到刚刚编写的源代码目录下，然后运行以下命令行：
            
            1. $cc -o ClientTest ClientTest.c stdsoap2.c soapC.c soapClient.c
            
              这时会在当前目录下生成目标文件ClientTest，这就是可执行文件。
                
          5. 运行服务器端和客户端；
        
            在当前目录下打开两个终端，一个终端输入./StandaloneAdd，即服务器端程序开始监听；另一个终端输入./ClientTest add 32 23，回车后，该终端会打印出：result = 55
