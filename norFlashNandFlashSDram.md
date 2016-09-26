S3C2440的启动时读取的第一条指令是在0x00上，分为成nand flash和nor flash上启动。
 
  + nand flash：适合大容量数据存储，类似硬盘；
  
  + nor flash：适合小容量的程序或数据存储，类似小硬盘；
  
  + sdram：主要用于程序执行时的程序存储、执行或计算，类似内存。
  
  + Nor flash的有自己的地址线和数据线，可以采用类似于memory的随机访问方式，在nor flash上可以直接运行程序，所以nor flash可以直接用来做boot，采用nor flash启动的时候会把地址映射到0x00上。

  + Nand flash是IO设备，数据、地址、控制线都是共用的，需要软件区控制读取时序，所以不能像nor flash、内存一样随机访问，不能EIP（片上运行），因此不能直接作为boot。

  + NANDFlash启动：<p>
    NANDFlash控制器自动把nandflash存储器的前4K载到Steppingstone（内部SRAM缓冲器），并把0x00000000S设置为内部SRAM的起始地址，cpu从内部SRAM的0x00000000开始启动，这个过程不需要程序干涉。
（cpu会自动从NAND flash中读取前4KB的数据放置在片内SRAM里（s3c2440是soc），同时把这段片内SRAM映射到nGCS0片选的空间（即0x00000000）。cpu是从0x00000000开始执行，也就是NAND flash里的前4KB内容。因为NAND FLASH连地址线都没有，不能直接把NAND映射到0x00000000，只好使用片内SRAM做一个载体。通过这个载体把nandflash中大代码复制到RAM(一般是SDRAM)中去执行）。 
程序员要完成的工作是把最核心的代码放在nandflash的前4K中。4K代码要完成S3C2440的核心配置以及启动代码（U-boot）的剩余部分拷贝到SDRAM中。
这4K的启动代码需要将NANDFlash中的内容复制到SDRAM中执行。NANDFlash的前4K空间放启动代码，SDRAM速度较快，用来执行主程序的代码。ARM一般从ROM或Flash启动完成初始化，然后将应用程序拷贝到RAM，然后跳到RAM执行。
  + NORflash启动：<p>
    支持XIP即代码直接在NOR Flash上执行，无需复制到内存中。这是由于NORFlash的接口与RAM完全相同，可随机访问任意地址数据。NORflash速度快，数据不易失，可作为存储并执行起到代码和应用程序的存储器，norflash可像内存一样读操作，但擦初和写操作效率很低，远不及内存，一般先在代码的开始部分使用汇编指令初始化外接的的内存部件（外存SDRAM），最后跳到外存中继续执行。对于小程序一般把它烧到NANDflash中，借助cpu内部RAM（SRAM）直接云行。
nor flash被映射到0x00000000地址（就是nGCS0，这里就不需要片内SRAM来辅助了，所以片内SRAM的起始地址还是0x40000000）. 然后cpu从0x00000000开始执行（也就是在Norfalsh中执行）。

  + NORflash速度快，数据不易失，可作为存储并执行起到代码和应用程序的存储器，norflash可像内存一样读操作，但擦初和写操作效率很低，价格很昂贵。SDRAM和nandflash的价格比较适中。根据这些特点，一些人产生了这样一种想法：外部nandflash中执行启动代码，SDRAM中执行主程序。NANDFlash控制器自动把nandflash存储器的前4K载到Steppingstone（内部SRAM缓冲器），并把0x00000000S设置为内部SRAM的起始地址，cpu从内部SRAM的0x00000000开始启动，这个过程不需要程序干涉。这4K的启动代码需要将NANDFlash中的内容复制到SDRAM中执行。NANDFlash的前4K空间放启动代码，SDRAM速度较快，用来执行主程序的代码。ARM一般从ROM或Flash启动完成初始化，然后将应用程序拷贝到RAM，然后跳到RAM执行。
