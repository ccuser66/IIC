本文主要介绍OLED屏显和汉字点阵编码原理，并在此基础上，使用STM32F103的SPI接口、AHT20温度传感器和OLED屏显，实现显示学号姓名、显示温湿度、长字符滑动显示。

#### 目录 

 *  [一、任务要求][Link 1]
 *  [二、OLED屏显][OLED]
 *  *  [2.1、OLED屏显简介][2.1_OLED]
    *  [2.2、OLED分类][2.2_OLED]
    *  [2.3、OLED的优势][2.3_OLED]
    *  [2.4、OLED屏显的特性][2.4_OLED]
    *  [2.5、电路图以及接法][2.5]
 *  [三、汉字点阵简介][Link 2]
 *  *  [3.1、点阵概念][3.1]
    *  [3.2、汉字编码][3.2]
    *  [3.3、点阵字库存储][3.3]
    *  [3.4、利用汉字机内码获取汉字][3.4]
 *  [四、SPI通信协议][SPI]
 *  *  [4.1、SPI通讯协议][4.1_SPI]
    *  [4.2、传输过程][4.2]
    *  [4.3、SPI特性][4.3_SPI]
    *  [4.4、时钟][4.4]
    *  [4.5、SPI模式][4.5_SPI]
    *  [4.6、多从机模式][4.6]
    *  [4.7、优缺点][4.7]
    *  [4.8、与UART进行对比][4.8_UART]
 *  [五、显示姓名学号][Link 3]
 *  *  [5.1、文字取模][5.1]
    *  [5.2、代码编写][5.2]
    *  [5.3、硬件连接][5.3]
    *  [5.4、编译烧录][5.4]
    *  [5.5、实现效果][5.5]
    *  [5.6、完整代码][5.6]
 *  [六、显示温湿度][Link 4]
 *  *  [6.1、文字取模][6.1]
    *  [6.2、代码编写][6.2]
    *  [6.3、硬件连接][6.3]
    *  [6.4、编译烧录][6.4]
    *  [6.5、实现效果][6.5]
    *  [6.6、完整代码][6.6]
 *  [七、滑动显示长字符][Link 5]
 *  *  [7.1、文字取模][7.1]
    *  [7.2、代码编写][7.2]
    *  [7.3、硬件连接][7.3]
    *  [7.4、编译烧录][7.4]
    *  [7.5、实现效果][7.5]
    *  [7.6、完整代码][7.6]
 *  [八、总结][Link 6]

## 一、任务要求 

理解OLED屏显和汉字点阵编码原理，使用STM32F103的SPI或IIC接口实现以下功能：  
（1）显示自己的学号和姓名；  
（2）显示AHT20的温度和湿度；  
（3）上下或左右的滑动显示长字符，比如“Hello，欢迎来到重庆交通大学物联网205实训室！”或者一段歌词或诗词(最好使用硬件刷屏模式)

## 二、OLED屏显 

### 2.1、OLED屏显简介 

OLED即有机发光二极管（Organic Light-Emitting Diode），又称为有机电激光显示（Organic Electroluminesence Display， OELD）。OLED 由于同时具备自发光，不需背光源、对比度高、厚度薄、视角广、反应速度快、可用于挠曲性面板、使用温度范围广、构造及制程较简单等优异之特性，被认为是下一代的平面显示器新兴应用技术。

LCD 都需要背光，而 OLED 不需要，因为它是自发光的。这样同样的显示，OLED 效果要来得好一些。以目前的技术，OLED 的尺寸还难以大型化，但是分辨率确可以做到很高。

我们使用的是 ALINETEK 的 OLED 显示模块，该模块有以下特点：

 *  模块有单色和双色两种可选，单色为纯蓝色，而双色则为黄蓝双色。
 *  尺寸小，显示尺寸为 0.96 寸，而模块的尺寸仅为 27mmx26mm 大小。
 *  高分辨率，该模块的分辨率为128x64。
 *  多种接口方式，该模块提供了总共 5 种接口包括：6800、8080 两种并行接口方式、3线或 4 线的穿行 SPI 接口方式、IIC 接口方式（只需要 2 根线就可以控制 OLED 了）。
 *  不需要高压，直接接 3.3V 就可以工作了。

注意：该模块不和 5.0V 接口兼容，使用的时候不要直接接到 5V 上去，否则可能烧坏模块  
在这里插入图片描述


### 2.2、OLED分类 

从器件结构上进行分类：  
1、 单层器件  
单层器件也就是在器件的正、负极之间接入一层可以发光的有机层，其结构为衬底/ITO/发光层/阴极。在这种结构中由于电子、空穴注入、传输不平衡，导致器件效率、亮度都较低，器件稳定性差  
2、双层器件  
双层器件是在单层器件的基础上,在发光层两侧加入空穴传输层（HTL）或电子传输层（ETL），克服了单层器件载流子注入不平衡的问题，改善了器件的电压-电流特性，提高了器件的发光效率  
3、三层器件  
三层器件结构是应用最广泛的一种结构，其结构为衬底/ITO/HTL/发光层/ETL/阴极。这种结构的优点是使激子被局限在发光层中，进而提高器件的效率  
4、多层结构  
多层结构的性能是比较好的一种结构，其能够很好的发挥各个层面的作用。发光层也可以由多层结构组成，由于各发机层之间相互独立，可以分别优化。因此，这种结构能充分发挥各有机层的作用，极大地提高了器件设计的灵活性

从驱动方式上进行分类：  
1、一种是主动式，一种是被动式  
2、主动式的一般为有源驱动，被动式的为无源驱动。在实际的应用过程中，有源驱动主要是用于高分辨率的产品，而无源驱动主要应用在显示器尺寸比较小的显示器中

从材料上进行分类：  
1、可根据有机物的种类划分，一种为小分子，另一种是高分子  
2、这两种器件的主要差别在制作工艺上，小分子器件主要采用的是真空热蒸发工艺，高分子器件采用的是旋转涂覆或者是喷涂印刷工艺

### 2.3、OLED的优势 

 *  1、相较于LED或LCD的晶体层，OLED的有机塑料层更薄、更轻而且更富于柔韧性
 *  2、OLED的发光层比较轻：因此它的基层可使用富于柔韧性的材料，而不会使用刚性材料。OLED基层为塑料材质，而LED和LCD则使用玻璃基层
 *  3、OLED比LED更亮：OLED有机层要比LED中与之对应的无机晶体层薄很多，因而OLED的导电层和发射层可以采用多层结构。此外，LED和LCD需要用玻璃作为支撑物，而玻璃会吸收一部分光线。OLED则无需使用玻璃
 *  4、OLED并不需要采用LCD中的逆光系统：LCD工作时会选择性地阻挡某些逆光区域，从而让图像显现出来，而OLED则是靠自身发光。因为OLED不需逆光系统，所以它们的耗电量小于LCD(LCD所耗电量中的大部分用于逆光系统)。这一点对于靠电池供电的设备(例如移动电话)来说，尤其重要
 *  5、OLED制造起来更加容易，还可制成较大的尺寸：OLED为塑胶材质，因此可以将其制作成大面积薄片状，而想要使用如此之多的晶体并把它们铺平，则要困难得多
 *  6、OLED的视野范围很广，可达170度左右：而LCD工作时要阻挡光线，因而在某些角度上存在天然的观测障碍。OLED自身能够发光，所以视域范围也要宽很多

### 2.4、OLED屏显的特性 

OLED技术之所以能够获得广泛的应用，在于其与其它技术相比，具有以下优点：

 *  功耗低
 *  响应速度快
 *  较宽的视角
 *  能实现高分辨率显示
 *  宽温度特性
 *  OLED能够实现软屏
 *  OLED成品的质量比较轻

### 2.5、电路图以及接法 

电路图：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ee55d5091db34ae2b6ebf4b2cf47755e.png)


在下面实验中，将会采用IIC_OLED  
参考厂家给出的Demo程序： [0.96寸IIC\_OLED模块配套资料包](http://www.lcdwiki.com/res/MC096GX/0.96inch-OLED-MC096GX-V1.0.zip)


引脚介绍：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2eb7d3e623cf4cdfad0e3914a83420a5.png)


 

[0.96寸IIC OLED显示屏相关介绍可参考链接 ](http://www.lcdwiki.com/zh/0.96inch_OLED_Module_MC096GX)
## 三、汉字点阵简介 

### 3.1、点阵概念 

我们用之前的方法一个IO口只能控制一个led，如果需要用更少的IO口控制更多的led怎么办？于是，就有了点阵。  
例如：8X8点阵共由64个发光二极管组成，且每个发光二极管是放置在行线和列线的交叉点上，当对应的某一行置1电平，某一列置0电平，则相应的二极管就亮；如要将第一个点点亮，则1脚接高电平a脚接低电平，则第一个点就亮了。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48a16a601c98ac8b57b5d297a17ac3cc.png)

实物图  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c3e1dcdff6af479abcebabe6c9c73852.png)


借助取模软件，即可将我们所需要的文字或字母，以点阵的形式呈现出来。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4c472308b00244d7be1eec53cac60a8b.png)


我们知道英文字母数量比较少，我们只要用一个字节（8位）就足以表达。但是汉字非常多。要怎么表达呢？

前人采用的一个方法就是把ASCII码的高128位作为汉字的内码，低128位仍然作为英文字母的内码，然后用两个字节来表示一个汉字。通过这个内码，我们可以获取汉字的字模信息。然后再根据这些字模的信息，把相应的汉字显示出来。

### 3.2、汉字编码 

区位码  
点阵字库其实就是按照汉字内码的顺序，把汉字的字模信息存起来。16×16的点阵字库有94区，每个区有94个汉字的字模。这样总的有94×94个汉字。

在国标 GD2312—80 中规定，所有的国标汉字及符号分配在一个 94 行、94 列的方阵中，方阵的每一行称为一个“区”，编号为 01 区到 94 区，每一列称为一个“位”，编号为01 位到 94 位，方阵中的每一个汉字和符号所在的区号和位号组合在一起形成的四个阿拉伯数字就是它们的“区位码”。

区位码的前两位是它的区号，后两位是它的位号。用区位码就可以唯一地确定一个汉字或符号。

机内码  
汉字的机内码是指在计算机中表示一个汉字的编码。机内码与区位码稍有区别。

汉字区位码的区码和位码的取值均在 1-94 之间，如直接用区位码作为机内码，就会与基本 ASCII 码混淆。为了避免机内码与基本 ASCII 码的冲突，需要避开基本 ASCII 码中的控制码(00H~1FH)，还需与基本 ASCII 码中的字符相区别。

因此：先在区码和位码分别加上 20H，在此基础上再加 80H。  
(此处“H”表示前两位数字为十六进制数)

经过这些处理，用机内码表示一个汉字需要占两个字节，分别称为高位字节和低位字节，这两位字节的机内码按如下规则表示：

> `高位字节 = 区码 + 20H + 80H(或区码 + A0H)`  
> `低位字节 = 位码 + 20H + 80H(或位码 + AOH)`

由于汉字的区码与位码的取值范围的十六进制数均为 01H-5EH(即十进制的 01-94)，所以汉字的高位字节与低位字节的取值范围则为 A1H-FEH(即十进制的 161-254)。

例如，汉字“啊”的区位码为 1601，区码和位码分别用十六进制表示即为 1001H，它的机内码的高位字节为 B0H，低位字节为 A1H，机内码就是 B0A1H。

### 3.3、点阵字库存储 

在汉字的点阵字库中，每个字节的每个位都代表一个汉字的一个点，每个汉字都是由一个矩形的点阵组成，0 代表没有，1 代表有点，将 0 和 1 分别用不同颜色画出，就形成了一个汉字。

字库根据字节所表示点的不同有分为 横向矩阵 和 纵向矩阵，目前多数的字库都是横向矩阵的存储方式(用得最多的应该是早期 UCDOS 字库)，纵向矩阵一般是因为有某些液晶是采用纵向扫描显示法，为了提高显示速度，于是便把字库矩阵做成纵向。

之后所描述的都是指横向矩阵字库。

16\*16点阵字库  
对于 1616 的矩阵来说，它所需要的位数共是 1616＝256 个位，每个字为 8 位，因此，每个汉字都需要用 256/8=32 个字节来表示。

每两个字节代表一行的 16 个点，共需要 16 行，显示汉字时，只需一次性读取 32 个字节，并将每两个字节为一行打印出来，即可形成一个汉字。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3041ee823ee94366a99bf2f4d9457f17.png)


1414与1212点阵字库  
对于 1414 和 1212 的字库，理论上计算，它们所需要的点阵分别为(1414/8)=25, (1212/8)=18 个字节，但是，如果按这种方式来存储，那么取点阵和显示时， 由于它们每一行都不是 8 的整位数 ，因此，就会涉到点阵的计算处理问题，会增加程序的复杂度，降低程序的效率。

为解决这个问题，有些点阵字库会将 1414 和 1212 的字库按 1614和 1612 来存储，即，每行还是按两个字节来存储，但是 1414 的字库，每两个字节的最后两位是没有使用，1212 的字节，每两字节的最后 4 位是没有使用，这个根据不同的字库会有不同的处理方式，所以在使用字库时要注意这个问题。

### 3.4、利用汉字机内码获取汉字 

汉字的区位码和机内码的关系如下：

> 机内码高位字节 = 区码 + 20H + 80H(或区码 + A0H)  
> 机内码低位字节 = 位码 + 20H + 80H(或位码 + AOH)

反过来说，我们也可以根据机内码来获得区位码：

> 区码 = 机内码高位字节 - A0H  
> 位码 = 机内码低位字节 - AOH

这样，算出区位码之后，我们就可以用它在汉字库里面寻址找字模了。具体方式：  
`该汉字的偏移地址 =（区码-1）×94×一个字占用的字节数 + 位码×一个字占用的字节数`

## 四、SPI通信协议 


### IIC 简介 

IIC(Inter－Integrated Circuit)总线是一种由NXP（原PHILIPS）公司开发的两线式串行总线，用于连接微控制器及其外围设备。多用于主控制器和从器件间的主从通信，在小数据量场合使用，传输距离短，任意时刻只能有一个主机等特性。

在 CPU 与被控 IC 之间、IC 与 IC 之间进行双向传送，高速 IIC 总线一般可达 400kbps 以上。

PS： 这里要注意IIC是为了与低速设备通信而发明的，所以IIC的传输速率比不上SPI  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/21f4f147f8f34cc193aea5cfbf49d50e.png)


#### ＩＩＣ的物理层 

IIC一共有只有两个总线： 一条是双向的串行数据线ＳＤＡ，一条是串行时钟线ＳＣＬ

 *  SDA(Serial data)是数据线，D代表Data也就是数据，Send Data 也就是用来传输数据的
 *  SCL(Serial clock line)是时钟线，C代表Clock 也就是时钟 也就是控制数据发送的时序的

所有接到I2C总线设备上的串行数据SDA都接到总线的SDA上，各设备的时钟线SCL接到总线的SCL上。I2C总线上的每个设备都自己一个唯一的地址，来确保不同设备之间访问的准确性。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-fxNfPPim-1586571859509)(C:\Users\48013\AppData\Roaming\Typora\typora-user-images\image-20200331212342296.png)]

IIC主要特点：

通常我们为了方便把IIC设备分为主设备和从设备，基本上谁控制时钟线（即控制SCL的电平高低变换）谁就是主设备。\*\*

 *  IIC主设备功能：主要产生时钟，产生起始信号和停止信号
 *  IIC从设备功能：可编程的IIC地址检测，停止位检测
 *  IIC的一个优点是它支持多主控(multimastering)， 其中任何一个能够进行发送和接收的设备都可以成为主总线。一个主控能够控制信号的传输和时钟频率。当然，在任何时间点上只能有一个主控。
 *  支持不同速率的通讯速度，标准速度(最高速度100kHZ),快速（最高400kHZ）
 *  SCL和SDA都需要接上拉电阻 (大小由速度和容性负载决定一般在3.3K-10K之间) 保证数据的稳定性，减少干扰。
 *  IIC是半双工，而不是全双工 ，同一时间只可以单向通信
 *  为了避免总线信号的混乱，要求各设备连接到总线的输出端时必须是漏极开路（OD）输出或集电极开路（OC）输出。这一点在等下我们会讲解

#### IIC的高阻态 

漏极开路（Open Drain）即高阻状态，适用于输入/输出，其可独立输入/输出低电平和高阻状态，若需要产生高电平，则需使用外部上拉电阻

高阻状态：高阻状态是三态门电路的一种状态。逻辑门的输出除有高、低电平两种状态外，还有第三种状态——高阻状态的门电路。电路分析时高阻态可做开路理解。

我们知道IIC的所有设备是接在一根总线上的，那么我们进行通信的时候往往只是几个设备进行通信，那么这时候其余的空闲设备可能会受到总线干扰，或者干扰到总线，怎么办呢？

为了避免总线信号的混乱，IIC的空闲状态只能有外部上拉， 而此时空闲设备被拉到了高阻态，也就是相当于断路， 整个IIC总线只有开启了的设备才会正常进行通信，而不会干扰到其他设备。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9b56c229523f488a921b44dbc0ebfa2a.png)


IIC器件地址： 每一个IIC器件都有一个器件地址，有的器件地址在出厂时地址就设定好了，用户不可以更改,比如OV7670的地址为0x42。有的器件例如EEPROM，前四个地址已经确定为1010，后三个地址是由硬件链接确定的，所以一IIC总线最多能连8个EEPROM芯片。

#### IIC物理层总结： 

I2C 总线在物理连接上非常简单，分别由SDA(串行数据线)和SCL(串行时钟线)及上拉电阻组成。通信原理是通过对SCL和SDA线高低电平时序的控制，来产生I2C总线协议所需要的信号进行数据的传递。在总线空闲状态时，SCL和SDA被上拉电阻Rp拉高，使SDA和SCL线都保持高电平。

I2C通信方式为半双工，只有一根SDA线，同一时间只可以单向通信，485也为半双工，SPI和uart通信为全双工。

主机和从机的概念：

主机就是负责整个系统的任务协调与分配，从机一般是通过接收主机的指令从而完成某些特定的任务，主机和从机之间通过总线连接，进行数据通讯。

 *  发布主要命令的称为主机
 *  接受命令的称为从机

半双工和全双工：

#### ＩＩＣ的协议层 

I2C 总线在传送数据过程中共有三种类型信号， 它们分别是：开始信号、结束信号和应答信号。

 *  开始信号：SCL 为高电平时，SDA 由高电平向低电平跳变，开始传送数据。
 *  结束信号：SCL 为高电平时，SDA 由低电平向高电平跳变，结束传送数据。
 *  应答信号：接收数据的 IC 在接收到 8bit 数据后，向发送数据的 IC 发出特定的低电平脉冲，表示已收到数据。CPU 向受控单元发出一个信号后，等待受控单元发出一个应答信号，CPU 接收到应答信号后，根据实际情况作出是否继续传递信号的判断。若未收到应答信号，由判断为受控单元出现故障。

这些信号中，起始信号是必需的，结束信号和应答信号，都可以不要。

#### IIC 总线时序图 

![img](https://i-blog.csdnimg.cn/blog_migrate/9b9cbd73dfa4308f734dad39e146e437.png)

下面我们来详细的介绍下IIC的通信协议流程：

#### 初始(空闲)状态 

因为IIC的 SCL 和SDA 都需要接上拉电阻，保证空闲状态的稳定性

所以IIC总线在空闲状态下SCL 和SDA都保持高电平

代码：

```java
void IIC_init()       //IIC初始化

{
            
   
     
     

       SCL=1; //首先把时钟线拉高

       delay_us(4);//延时函数

       SDA=1; //在SCL为高的情况下把SDA拉高

       delay_us(4); //延时函数

}
```

#### 开始信号： 

SCL保持高电平，SDA由高电平变为低电平后，延时(>4.7us)，SCL变为低电平。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2251edfc24034fc8b7fa75638b5e8d58.png)


代码表示：

```java
//产生IIC起始信号
//1.先拉高SDA，再拉高SCL，空闲状态
//2.拉低SDA
void IIC_Start()         //启动信号

{
            
   
     
     
       SDA=1; //确保SDA线为高电平
       delay_us(5);
       SCL=1;  //确保SCL高电平
       delay_us(5);
       SDA=0; //在SCL为高时拉低SDA线，即为起始信号
       delay_us(5);
       SCL=0;   //钳住I2C总线，准备发送或接收数据 
    
}
```

#### 停止信号 

停止信号：SCL保持高电平。SDA由低电平变为高电平。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/840aad851d454c4e8298a01df29ef628.png)


```java
//产生IIC停止信号
//1.先拉低SDA，再拉低SCL
//2.拉高SCL
//3.拉高SDA
//4.停止接收数据
void IIC_Stop(void)
{
            
   
     
     

	IIC_SCL=0;
	IIC_SDA=0;    //STOP:当SCL高时，数据由低变高
 	delay_us(4);
	IIC_SCL=1; 
	IIC_SDA=1;    //发送I2C总线结束信号
	delay_us(4);							   	
}
```

在起始条件产生后，总线处于忙状态，由本次数据传输的主从设备独占，其他I2C器件无法访问总线；而在停止条件产生后，本次数据传输的主从设备将释放总线，总线再次处于空闲状态。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/36ab6390bb8b446982c6a2a3963355b3.png)


#### 数据有效性 

IIC信号在数据传输过程中，当SCL=1高电平时，数据线SDA必须保持稳定状态，不允许有电平跳变，只有在时钟线上的信号为低电平期间，数据线上的高电平或低电平状态才允许变化。

SCL=1时 数据线SDA的任何电平变换会看做是总线的起始信号或者停止信号。

也就是在IIC传输数据的过程中，SCL时钟线会频繁的转换电平，以保证数据的传输

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/30ace8c044314b9d89d37f005b51a683.png)


![img](https://i-blog.csdnimg.cn/blog_migrate/ce180799ae948bbff3ebc01000b75530.png)

#### 应答信号 

每当主机向从机发送完一个字节的数据，主机总是需要等待从机给出一个应答信号，以确认从机是否成功接收到了数据，

应答信号：主机SCL拉高，读取从机SDA的电平，为低电平表示产生应答

 *  应答信号为低电平时，规定为有效应答位（ACK，简称应答位），表示接收器已经成功地接收了该字节；
 *  应答信号为高电平时，规定为非应答位（NACK），一般表示接收器接收该字节没有成功。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a3094193912d4e13a6afa3f5770374ef.png)


\*\*每发送一个字节（8个bit）\*\*在一个字节传输的8个时钟后的第九个时钟期间，接收器接收数据后必须回一个ACK应答信号给发送器，这样才能进行数据传输。

应答出现在每一次主机完成8个数据位传输后紧跟着的时钟周期，低电平0表示应答，1表示非应答，

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/619f27ebdbdb4025bd938e9945797f2e.png)


```java
//主机产生应答信号ACK
//1.先拉低SCL，再拉低SDA
//2.拉高SCL
//3.拉低SCL## 标题
void I2C_Ack(void)
{
            
   
     
     
   IIC_SCL=0;   //先拉低SCL，使得SDA数据可以发生改变
   IIC_SDA=0;   
   delay_us(2);
   IIC_SCL=1;
   delay_us(5);
   IIC_SCL=0;
}


//主机不产生应答信号NACK
//1.先拉低SCL，再拉高SDA
//2.拉高SCL
//3.拉低SCL
void I2C_NAck(void)
{
            
   
     
     
   IIC_SCL=0;   //先拉低SCL，使得SDA数据可以发生改变
   IIC_SDA=1;   //拉高SDA，不产生应答信号
   delay_us(2);
   IIC_SCL=1;
   delay_us(5);
   IIC_SCL=0;
}
```

等待应答信号：

```java
//等待应答信号到来
//返回值：1，接收应答失败
//        0，接收应答成功
char IIC_Wait_Ack(void)
{
            
   
     
     
	u8 ucErrTime=0;
	 
	IIC_SDA=1;delay_us(1);	   
	IIC_SCL=1;delay_us(1);	 
	while(IIC_SDA)
	{
            
   
     
     
		ucErrTime++;
		if(ucErrTime>250)
		{
            
   
     
     
			IIC_Stop();
			return 1;
		}
	}
	IIC_SCL=0;//时钟输出0 	   
	return 0;  
}
```

#### IIC数据传送 

#### 数据传送格式 

SDA线上的数据在SCL时钟“高”期间必须是稳定的，只有当SCL线上的时钟信号为低时，数据线上的“高”或“低”状态才可以改变。输出到SDA线上的每个字节必须是8位，数据传送时，先传送最高位（MSB），每一个被传送的字节后面都必须跟随一位应答位（即一帧共有9位）。

当一个字节按数据位从高位到低位的顺序传输完后，紧接着从设备将拉低SDA线，回传给主设备一个应答位ACK， 此时才认为一个字节真正的被传输完成 ，如果一段时间内没有收到从机的应答信号，则自动认为从机已正确接收到数据。

![img](https://i-blog.csdnimg.cn/blog_migrate/4e038c8f4975b8b5bb8caed9276698c4.png)

IIC写数据：

![img](https://i-blog.csdnimg.cn/blog_migrate/3a4f9d6e2338d40010882ee1d6cbc2e6.png)

多数从设备的地址为7位或者10位，一般都用七位。  
八位设备地址=7位从机地址+读/写地址，

再给地址添加一个方向位位用来表示接下来数据传输的方向，

 *  0表示主设备向从设备(write)写数据，
 *  1表示主设备向从设备(read)读数据

IIC的每一帧数据由9bit组成，

如果是发送数据，则包含 8bit数据+1bit ACK,

如果是设备地址数据，则8bit包含7bit设备地址 1bit方向  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0448e5ff960f42acb01107119f46f6e5.png)


> 在起始信号后必须传送一个从机的地址(7位) 1~7位为7位接收器件地址，第8位为读写位，用“0”表示主机发送数据(W)，“1”表示主机接收数据 （R）, 第9位为ACK应答位，紧接着的为第一个数据字节，然后是一位应答位，后面继续第2个数据字节。

IIC发送一个字节数据：

```java
//IIC发送一个字节
//返回从机有无应答
//1，有应答
//0，无应答            

//IIC_SCL=0;
//在SCL上升沿时准备好数据，进行传送数据时，拉高拉低SDA，因为传输一个字节，一个SCL脉冲里传输一个位。
//数据传输过程中，数据传输保持稳定（在SCL高电平期间，SDA一直保持稳定，没有跳变）
//只有当SCL被拉低后，SDA才能被改变
//总结：在SCL为高电平期间，发送数据，发送8次数据，数据为1,SDA被拉高，数据为0，SDA被拉低。
//传输期间保持传输稳定，所以数据线仅可以在时钟SCL为低电平时改变。
void IIC_Send_Byte(u8 txd)
{
            
   
     
                             
    u8 t;   
    SDA_OUT();         
    IIC_SCL=0;//拉低时钟开始数据传输
    for(t=0;t<8;t++)
    {
            
   
     
                   
        //IIC_SDA=txd&0x80;   //获取最高位
        //获取数据的最高位，然后数据左移一位
        //如果某位为1，则SDA为1，否则相反
        if(txd&0x80)
            IIC_SDA=1;
        else
            IIC_SDA=0;
        txd<<=1;       
        delay_us(2);   
        IIC_SCL=1;
        delay_us(2); 
        IIC_SCL=0;    
        delay_us(2);
    }     
}       
或者：
        //IIC_SDA=txd&0x80;   //获取最高位
        //获取数据的最高位，然后右移7位,假设为 1000 0000 右移7位为 0000 0001 
        // 假设为 0000 0000 右移7位为 0000 0000 
        //如果某位为1，则SDA为1，否则相反
        IIC_SDA=((txd&0x80)>>7);
        txd<<=1;
```

IIC读取一个字节数据：

```java
//读1个字节，ack=1时，发送ACK，ack=0，发送nACK   
u8 IIC_Read_Byte(unsigned char ack)
{
            
   
     
     
	unsigned char i,receive=0;
	SDA_IN();        //SDA设置为输入
    for(i=0;i<8;i++ )
	{
            
   
     
     
        IIC_SCL=0; 
        delay_us(2);
		IIC_SCL=1;
        receive<<=1;
        if(READ_SDA)receive++;   
		delay_us(1); 
    }					 
    if (!ack)
        IIC_NAck();        //发送nACK
    else
        IIC_Ack();         //发送ACK   
    return receive;
}
```

#### IIC发送数据 

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/78724cee7b8ff727a4a77a967eedd6f7.png)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/04ecd28cf12f430f9a7296adcc1b4e68.png)


Start: IIC开始信号，表示开始传输。  
DEVICE\_ADDRESS:: 从设备地址,就是7位从机地址  
R/W： W(write)为写，R(read)为读  
ACK： 应答信号  
WORD\_ADDRESS ： 从机中对应的寄存器地址 比方说访问 OLED中的 某个寄存器  
DATA: 发送的数据  
STOP: 停止信号。结束IIC

主机要向从机写数据时：

> 1.  主机首先产生START信号
> 2.  然后紧跟着发送一个从机地址，这个地址共有7位，紧接着的第8位是数据方 向位(R/W)，0表示主机发送数据(写)，1表示主机接收数据(读)
> 3.  主机发送地址时，总线上的每个从机都将这7位地址码与自己的地址进行比较，若相同，则认为自己正在被主机寻址，根据R/T位将自己确定为发送器和接收器
> 4.  这时候主机等待从机的应答信号(A)
> 5.  当主机收到应答信号时，发送要访问从机的那个地址， 继续等待从机的应答信号
> 6.  当主机收到应答信号时，发送N个字节的数据，继续等待从机的N次应答信号，
> 7.  主机产生停止信号，结束传送过程。

#### IIC读数据： 

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bee190f471354f289601fcfccef23b51.png)
  
主机要从从机读数据时

> 1.  主机首先产生START信号
> 2.  然后紧跟着发送一个从机地址，注意此时该地址的第8位为0，表明是向从机写命令，
> 3.  这时候主机等待从机的应答信号(ACK)
> 4.  当主机收到应答信号时，发送要访问的地址，继续等待从机的应答信号，
> 5.  当主机收到应答信号后，主机要改变通信模式(主机将由发送变为接收，从机将由接收变为发送)所以主机重新发送一个开始start信号，然后紧跟着发送一个从机地址，注意此时该地址的第8位为1，表明将主机设 置成接收模式开始读取数据，
> 6.  这时候主机等待从机的应答信号，当主机收到应答信号时，就可以接收1个字节的数据，当接收完成后，主机发送非应答信号，表示不在接收数据
> 7.  主机进而产生停止信号，结束传送过程。



## 五、显示姓名学号 

### 5.1、文字取模 

 *  利用取模软件将需要显示的文字用十六进制表示出来  
    链接：[https://pan.baidu.com/s/1YbUiGlyfxHi\_h2B\_rqogBg?pwd=2022 ][https_pan.baidu.com_s_1YbUiGlyfxHi_h2B_rqogBg_pwd_2022]  
    提取码：2022
 *  打开软件，进行初始设置  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a48f4eccfb5644f7904dee35d76df79b.png)

 *  点击选项 按照图片设置
         ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d0144a629c6545589fde0221829b2e98.png)

 *  在文字输入区输入目标文字，并 点击生成字模
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ebbda3a2158c4b95914958bfec739494.png)



    

### 5.2、代码编写 

 *  解压开始下载的文件，打开该文件  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8afff93861a74aef8331f210904a232b.png)

 *  打开 1-Demo->Demo\_STM32->0.96inch\_OLED\_Demo\_STM32F103RCT6\_Software\_4-wire\>PROJECT  
      
    
    
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/81de6bac34d891c9fe12acf00ccd86df.png)
 *  双击打开工程  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a8173ef3920b864ea4bba365d280124.png)
 *  修改main.c文件里的函数

```java
int main(void)
  {	u8 t;
		delay_init();	    	 //延时函数初始化	  
		NVIC_Configuration(); 	 //设置NVIC中断分组2:2位抢占优先级，2位响应优先级 	LED_Init();			     //LED端口初始化
		OLED_Init();			//初始化OLED  
		OLED_Clear(0)  	; 
		
	
		t=' ';

	while(1) 
	{		
		OLED_Clear(0);
		OLED_ShowCHinese(6,0,0);//人
	OLED_ShowCHinese(24,0,1);//民
//		OLED_ShowCHinese(42,0,2);//万
//			OLED_ShowCHinese(60,0,3);//岁
			
		OLED_ShowString(6,3,"632207030420",16);
		   delay_ms(50000);	 
	}
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3d5f425f99c5420782e8f598dafe267c.png)


 *  找到工程项目中 oledfont.h 文件中的char Hzk[][32]数组，在里面添加用字模提取器输出的点阵码
 ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6f2283a1312b47548f60b605668639ad.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a5105a9676174a27a53e14a1be2257c5.png)


```java
char Hzk[][32]={

{ 0x00,0x00,0x00,0x00,0x00,0x00,0xC0,0x3F,0xC0,0x00,0x00,0x00,0x00,0x00,0x00,0x00},
{ 0x80,0x40,0x20,0x10,0x0C,0x03,0x00,0x00,0x00,0x03,0x0C,0x10,0x20,0x40,0x80,0x00},/*"人",0*/
/* (16 X 16 , 宋体 )*/

{ 0x00,0x00,0xFE,0x22,0x22,0x22,0x22,0x22,0xE2,0x22,0x22,0x22,0x3E,0x00,0x00,0x00},
{ 0x00,0x00,0xFF,0x41,0x21,0x11,0x01,0x01,0x03,0x0D,0x11,0x21,0x41,0xF1,0x00,0x00},/*"民",1*/
/* (16 X 16 , 宋体 )*/

{ 0x04,0x04,0x04,0x04,0x04,0xFC,0x44,0x44,0x44,0x44,0x44,0xC4,0x04,0x04,0x04,0x00},
{ 0x80,0x40,0x20,0x18,0x06,0x01,0x00,0x00,0x40,0x80,0x40,0x3F,0x00,0x00,0x00,0x00},/*"万",2*/
/* (16 X 16 , 宋体 )*/

{ 0x00,0x00,0x1E,0x10,0x10,0x90,0xF0,0x9F,0x90,0x90,0x90,0x90,0x1E,0x00,0x00,0x00},
{ 0x80,0x80,0x84,0x42,0x41,0x42,0x24,0x28,0x10,0x08,0x04,0x03,0x00,0x00,0x00,0x00},/*"岁",3*/


```






### 5.3、硬件连接 

供电模块连接方法：

<table> 
 <thead> 
  <tr> 
   <th>USB转TTL</th> 
   <th>STM32F103C8T6</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>GND</td> 
   <td>G</td> 
  </tr> 
  <tr> 
   <td>3V3</td> 
   <td>3V3</td> 
  </tr> 
  <tr> 
   <td>RXD</td> 
   <td>PA9</td> 
  </tr> 
  <tr> 
   <td>TXD</td> 
   <td>PA10</td> 
  </tr> 
 </tbody> 
</table>


注意将核心板上的BOOT0设置为1，BOOT1设置为0

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75eb85671e610260ff3e6ffb114639cf.jpeg)  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/990cffa0abff20123f56fc8cd6cebbec.png)




### 5.4、编译烧录 

 *  编译  
 * ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3844c8d890774507b3e7bad3ec35d1b6.png)

    
 *  烧录  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3221c6b1f3fc4ad18da435e2bcd0d6a0.png)


### 5.5、实现效果 

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0a4b4a67bc3f4399809f644e4a5cc86c.jpeg)




## 六、显示温湿度 

### 6.1、文字取模 

 *  在文字输入区输入 温湿度显示，按下 ctrl+enter，点击 C51格式 取模，得到点阵图  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3de20917c0b9f1dae003e2efbeaab57a.png)

### 6.2、代码编写 

 *  在 USER->gui.c 下面找到项目中 oledfont.h ，在 cfont16\[ \] 内增添所需文字点阵

```java
const typFNT_GB16 cfont16[] = 
{
            
   
     
     
  "温",0x00,0x00,0x23,0xF8,0x12,0x08,0x12,0x08,0x83,0xF8,0x42,0x08,0x42,0x08,0x13,0xF8,
0x10,0x00,0x27,0xFC,0xE4,0xA4,0x24,0xA4,0x24,0xA4,0x24,0xA4,0x2F,0xFE,0x00,0x00,
	"湿",0x00,0x00,0x27,0xF8,0x14,0x08,0x14,0x08,0x87,0xF8,0x44,0x08,0x44,0x08,0x17,0xF8,
0x11,0x20,0x21,0x20,0xE9,0x24,0x25,0x28,0x23,0x30,0x21,0x20,0x2F,0xFE,0x00,0x00,
	"度",0x01,0x00,0x00,0x80,0x3F,0xFE,0x22,0x20,0x22,0x20,0x3F,0xFC,0x22,0x20,0x22,0x20,
0x23,0xE0,0x20,0x00,0x2F,0xF0,0x24,0x10,0x42,0x20,0x41,0xC0,0x86,0x30,0x38,0x0E,
	"显",0x00,0x00,0x1F,0xF0,0x10,0x10,0x10,0x10,0x1F,0xF0,0x10,0x10,0x10,0x10,0x1F,0xF0,
0x04,0x40,0x44,0x44,0x24,0x44,0x14,0x48,0x14,0x50,0x04,0x40,0xFF,0xFE,0x00,0x00,
	"示",0x00,0x00,0x3F,0xF8,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0xFF,0xFE,0x01,0x00,
0x01,0x00,0x11,0x10,0x11,0x08,0x21,0x04,0x41,0x02,0x81,0x02,0x05,0x00,0x02,0x00,
};
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b293f16956ef495ab663e9394477a5dd.png)


 *  在 bsp\_i2c.c 中重新写入函数 read\_AHT20

```java
void read_AHT20(void)
{
            
   
     
     
	uint8_t   i;
	for(i=0; i<6; i++)
	{
            
   
     
     
		readByte[i]=0;
	}

	//-------------
	I2C_Start();

	I2C_WriteByte(0x71);
	ack_status = Receive_ACK();
	readByte[0]= I2C_ReadByte();
	Send_ACK();

	readByte[1]= I2C_ReadByte();
	Send_ACK();

	readByte[2]= I2C_ReadByte();
	Send_ACK();

	readByte[3]= I2C_ReadByte();
	Send_ACK();

	readByte[4]= I2C_ReadByte();
	Send_ACK();

	readByte[5]= I2C_ReadByte();
	SendNot_Ack();
	//Send_ACK();

	I2C_Stop();

	//--------------
	if( (readByte[0] & 0x68) == 0x08 )
	{
            
   
     
     
		H1 = readByte[1];
		H1 = (H1<<8) | readByte[2];
		H1 = (H1<<8) | readByte[3];
		H1 = H1>>4;

		H1 = (H1*1000)/1024/1024;

		T1 = readByte[3];
		T1 = T1 & 0x0000000F;
		T1 = (T1<<8) | readByte[4];
		T1 = (T1<<8) | readByte[5];

		T1 = (T1*2000)/1024/1024 - 500;

		AHT20_OutData[0] = (H1>>8) & 0x000000FF;
		AHT20_OutData[1] = H1 & 0x000000FF;

		AHT20_OutData[2] = (T1>>8) & 0x000000FF;
		AHT20_OutData[3] = T1 & 0x000000FF;
	}
	else
	{
            
   
     
     
		AHT20_OutData[0] = 0xFF;
		AHT20_OutData[1] = 0xFF;

		AHT20_OutData[2] = 0xFF;
		AHT20_OutData[3] = 0xFF;
		printf("lyy");

	}
	t=T1/10;
	t1=T1%10;
	a=(float)(t+t1*0.1);
	h=H1/10;
	h1=H1%10;
	b=(float)(h+h1*0.1);
	sprintf(strTemp,"%.1f",a);   //调用Sprintf函数把DHT11的温度数据格式化到字符串数组变量strTemp中  
    sprintf(strHumi,"%.1f",b);    //调用Sprintf函数把DHT11的湿度数据格式化到字符串数组变量strHumi中  
	GUI_ShowCHinese(16,00,16,"温湿度显示",1);
	GUI_ShowCHinese(16,20,16,"温度",1);
	GUI_ShowString(53,20,strTemp,16,1);
	GUI_ShowCHinese(16,38,16,"湿度",1);
	GUI_ShowString(53,38,strHumi,16,1);
	delay_ms(1500);		
	delay_ms(1500);
	
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2b8e73c4333b4d7eb94d107f1e718cca.png)


 *  改写 main.c 的主函数

```java
int main(void)
{
            
   
     
     	
	delay_init();	    	       //延时函数初始化    	  
	uart_init(115200);	 
	IIC_Init();
		  
	NVIC_Configuration(); 	   //设置NVIC中断分组2:2位抢占优先级，2位响应优先级 	
	OLED_Init();			         //初始化OLED  
	OLED_Clear(0); 
	while(1)
	{
            
   
     
     
		//printf("温度湿度显示");
		read_AHT20_once();
		OLED_Clear(0); 
		delay_ms(1500);
  }
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5900f6cd8eda4128bc87d2c905feaadc.png)


### 6.3、硬件连接 

供电模块连接方法：

<table> 
 <thead> 
  <tr> 
   <th>USB转TTL</th> 
   <th>STM32F103C8T6</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>GND</td> 
   <td>G</td> 
  </tr> 
  <tr> 
   <td>3V3</td> 
   <td>3V3</td> 
  </tr> 
  <tr> 
   <td>RXD</td> 
   <td>PA9</td> 
  </tr> 
  <tr> 
   <td>TXD</td> 
   <td>PA10</td> 
  </tr> 
 </tbody> 
</table>


注意将核心板上的BOOT0设置为1，BOOT1设置为0

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1e1735d1d22a4c55aa9c0d5aea350702.png)
  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4a810bc64b7144ff9df9f5f57239bd4f.png)


AHT20传感器模块连接方法：

<table> 
 <thead> 
  <tr> 
   <th>STM32F103C8T6</th> 
   <th>AHT20传感器</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>3.3V</td> 
   <td>1号脚（VDD）</td> 
  </tr> 
  <tr> 
   <td>PB7</td> 
   <td>2号脚（SDA）</td> 
  </tr> 
  <tr> 
   <td>GND</td> 
   <td>3号脚（GND）</td> 
  </tr> 
  <tr> 
   <td>PB6</td> 
   <td>4号脚（SCL）</td> 
  </tr> 
 </tbody> 
</table>


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9c7d9ffb98f242a4ac01ef02da9e0b0b.png)
  
  




### 6.4、编译烧录 

 *  编译  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/915d3320e9bd45a3a5620720da91e8c5.png)

 *  烧录  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/cafcea53e60d4adea6ea6494c2f47323.png)


### 6.5、实现效果 


[video(video-5q4nmwzJ-1734179032810)(type-csdn)(url-https://live.csdn.net/v/embed/438900)(image-https://i-blog.csdnimg.cn/img_convert/1d3916d7281d153a70d4e98c10dcf627.jpeg)(title-温湿度)]


## 七、滑动显示长字符 

### 7.1、文字取模 

 *  在文字输入区输入 满堂花醉三千客，一剑寒霜十四州，点击生成字模，得到点阵图  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/82e01f9bd0824b799ed26b028fdfb9f9.png)


### 7.2、代码编写 

 *  =找到 oledfont.h 文件，在char Hzk[][32]内存储生成的汉字点阵

```java
const typFNT_GB16 cfont16[] = 
{
            
   
     
     
{0x10, 0x60, 0x02, 0x8C, 0x00, 0x24, 0x24, 0x2F, 0xE4, 0x24, 0x24, 0xE4, 0x2F, 0x24, 0x24, 0x00,},
{0x04, 0x04, 0x7E, 0x01, 0x00, 0xFF, 0x11, 0x09, 0x07, 0x19, 0x09, 0x07, 0x49, 0x91, 0x7F, 0x00,},/*满",0*/
/* (16 X 16 , 宋体 )*/

{0x20, 0x18, 0x08, 0xEA, 0x2C, 0x28, 0x28, 0x2F, 0x28, 0x28, 0x2C, 0xEA, 0x08, 0x28, 0x18, 0x00,},
{0x40, 0x40, 0x48, 0x49, 0x49, 0x49, 0x49, 0x7F, 0x49, 0x49, 0x49, 0x49, 0x48, 0x40, 0x40, 0x00,},/*堂",1*/
/* (16 X 16 , 宋体 )*/

{0x04, 0x04, 0x04, 0x84, 0x6F, 0x04, 0x04, 0x04, 0xE4, 0x04, 0x8F, 0x44, 0x24, 0x04, 0x04, 0x00,},
{0x04, 0x02, 0x01, 0xFF, 0x00, 0x10, 0x08, 0x04, 0x3F, 0x41, 0x40, 0x40, 0x40, 0x40, 0x78, 0x00,},/*花",2*/
/* (16 X 16 , 宋体 )*/

{0xF2, 0x12, 0xFE, 0x12, 0xFE, 0x12, 0xF2, 0x44, 0x34, 0x45, 0x86, 0x44, 0x34, 0x44, 0x84, 0x00,},
{0xFF, 0x4A, 0x49, 0x48, 0x49, 0x49, 0xFF, 0x04, 0x04, 0x04, 0xFE, 0x04, 0x04, 0x04, 0x04, 0x00,},/*醉",3*/
/* (16 X 16 , 宋体 )*/

{0x00, 0x04, 0x84, 0x84, 0x84, 0x84, 0x84, 0x84, 0x84, 0x84, 0x84, 0x84, 0x84, 0x04, 0x00, 0x00,},
{0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x00,},/*三",4*/
/* (16 X 16 , 宋体 )*/

{0x80, 0x80, 0x84, 0x84, 0x84, 0x84, 0x84, 0xFC, 0x82, 0x82, 0x82, 0x83, 0x82, 0x80, 0x80, 0x00,},
{0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xFF, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,},/*千",5*/
/* (16 X 16 , 宋体 )*/

{0x10, 0x0C, 0x84, 0x44, 0xB4, 0xA4, 0x25, 0x26, 0x24, 0xA4, 0x64, 0x24, 0x04, 0x14, 0x0C, 0x00,},
{0x04, 0x04, 0x04, 0xFA, 0x4A, 0x4A, 0x49, 0x49, 0x49, 0x4A, 0x4A, 0xFA, 0x04, 0x04, 0x04, 0x00,},/*客",6*/
/* (16 X 16 , 宋体 )*/

{0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x00,},
{0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,},/*一",7*/
/* (16 X 16 , 宋体 )*/

{0x40, 0x20, 0x50, 0x4C, 0x43, 0x44, 0x48, 0x10, 0x20, 0x00, 0xF8, 0x00, 0x00, 0xFF, 0x00, 0x00,},
{0x00, 0x44, 0xD8, 0x41, 0x46, 0x20, 0x38, 0x27, 0x20, 0x00, 0x0F, 0x40, 0x80, 0x7F, 0x00, 0x00,},/*剑",8*/
/* (16 X 16 , 宋体 )*/

{0x10, 0x0C, 0x44, 0x54, 0x54, 0xFC, 0x55, 0x56, 0x54, 0xFC, 0x54, 0x54, 0x44, 0x14, 0x0C, 0x00,},
{0x11, 0x11, 0x09, 0x05, 0x03, 0x21, 0x25, 0x45, 0x49, 0x91, 0x03, 0x05, 0x09, 0x11, 0x11, 0x00,},/*寒",9*/
/* (16 X 16 , 宋体 )*/

{0x10, 0x0C, 0x05, 0x55, 0x55, 0x55, 0x05, 0x7F, 0x05, 0x55, 0x55, 0x55, 0x05, 0x14, 0x0C, 0x00,},
{0x20, 0x12, 0x0A, 0x06, 0xFF, 0x0A, 0x12, 0x00, 0xFF, 0x55, 0x55, 0x55, 0x55, 0xFF, 0x00, 0x00,},/*霜",10*/
/* (16 X 16 , 宋体 )*/

{0x40, 0x40, 0x40, 0x40, 0x40, 0x40, 0x40, 0xFF, 0x40, 0x40, 0x40, 0x40, 0x40, 0x40, 0x40, 0x00,},
{0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xFF, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,},/*十",11*/
/* (16 X 16 , 宋体 )*/

{0x00, 0xFC, 0x04, 0x04, 0x04, 0xFC, 0x04, 0x04, 0x04, 0xFC, 0x04, 0x04, 0x04, 0xFC, 0x00, 0x00,},
{0x00, 0x7F, 0x28, 0x24, 0x23, 0x20, 0x20, 0x20, 0x20, 0x21, 0x22, 0x22, 0x22, 0x7F, 0x00, 0x00,},/*四",12*/
/* (16 X 16 , 宋体 )*/

{0x00, 0xE0, 0x00, 0xFF, 0x00, 0x20, 0xC0, 0x00, 0xFE, 0x00, 0x20, 0xC0, 0x00, 0xFF, 0x00, 0x00,},
{0x81, 0x40, 0x30, 0x0F, 0x00, 0x00, 0x00, 0x00, 0x3F, 0x00, 0x00, 0x00, 0x00, 0xFF, 0x00, 0x00,},/*州",1327*/
/* (16 X 16 , 宋体 )*/
```







 *  修改 main.c 文件的主函数，添加相应的 OLED 滚动代码；

```java
int main(void)
 {	//u8 t;
//		delay_init();	    	 //延时函数初始化	  
//		NVIC_Configuration(); 	 //设置NVIC中断分组2:2位抢占优先级，2位响应优先级 	LED_Init();			     //LED端口初始化
//		OLED_Init();			//初始化OLED  
//		OLED_Clear(0)  	; 
//		
//	
//		t=' ';
//			delay_init();	    	       //延时函数初始化	  
	NVIC_Configuration(); 	   //设置NVIC中断分组2:2位抢占优先级，2位响应优先级 	
	OLED_Init();			         //初始化OLED  
	OLED_Clear(0);             //清屏（全黑）
	OLED_WR_Byte(0x2E,OLED_CMD);        //关闭滚动
  OLED_WR_Byte(0x27,OLED_CMD);        //水平向左或者右滚动 26/27
  OLED_WR_Byte(0x00,OLED_CMD);        //虚拟字节
	OLED_WR_Byte(0x00,OLED_CMD);        //起始页 0
	OLED_WR_Byte(0x07,OLED_CMD);        //滚动时间间隔
	OLED_WR_Byte(0x07,OLED_CMD);        //终止页 7
	OLED_WR_Byte(0x00,OLED_CMD);        //虚拟字节
	OLED_WR_Byte(0xFF,OLED_CMD);        //虚拟字节
		OLED_ShowCHinese(6,0,1);//满 
		OLED_ShowCHinese(24,0,2);//堂 
		OLED_ShowCHinese(42,0,3);//花 
		OLED_ShowCHinese(60,0,4);//醉
		OLED_ShowCHinese(78,0,5);//三 
		OLED_ShowCHinese(96,0,6);//千
		OLED_ShowCHinese(114,0,7);//客 
		OLED_ShowCHinese(6,3,8);//一 
		OLED_ShowCHinese(24,3,9);//剑
		OLED_ShowCHinese(42,3,10);//寒
		OLED_ShowCHinese(60,3,11);//霜
		OLED_ShowCHinese(78,3,12);//十
		OLED_ShowCHinese(96,3,13);//四
		OLED_ShowCHinese(114,3,14);//州
	OLED_WR_Byte(0x2F,OLED_CMD);        //开启滚动
	while(1) 
	{


    }
 }
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/38a14519dfa244b482bd6b4f72e9fae7.png)


### 7.3、硬件连接 

供电模块连接方法：

<table> 
 <thead> 
  <tr> 
   <th>USB转TTL</th> 
   <th>STM32F103C8T6</th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>GND</td> 
   <td>G</td> 
  </tr> 
  <tr> 
   <td>3V3</td> 
   <td>3V3</td> 
  </tr> 
  <tr> 
   <td>RXD</td> 
   <td>PA9</td> 
  </tr> 
  <tr> 
   <td>TXD</td> 
   <td>PA10</td> 
  </tr> 
 </tbody> 
</table>


注意将核心板上的BOOT0设置为1，BOOT1设置为0

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4f2d633479934045a7c0b45ed61b59c8.png)
  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/10822b4dd8014f548281e2f30b200f4f.png)


 
  


### 7.4、编译烧录 

 *  编译  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7324c6b4252a44c8b01dc9ed1b70c0ee.png)

 *  烧录  
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/272c904c9b6f4a2fad8abecaaa4d69c1.png)


### 7.5、实现效果 


[video(video-1BUYsvKh-1734179737685)(type-csdn)(url-https://live.csdn.net/v/embed/438901)(image-https://img-home.csdnimg.cn/images/20230724024159.png?origin_url=https%3A%2F%2Fv-blog.csdnimg.cn%2Fasset%2F7f3e246a189957cea0b7d22390926e2c%2Fcover%2FCover0.jpg&pos_id=img-PYtmzK64-1734193442458))(title-滚动)]



## 八、总结 


文章围绕 OLED 屏显与汉字点阵编码原理展开，深入且系统地介绍了相关知识与应用实践。OLED 屏显部分，不仅阐述其自发光、对比度高、厚度薄等显著优势，还详细讲解了多种分类方式以及各类接口特点，为后续应用提供理论基础。汉字点阵编码原理中，清晰地说明了汉字通过区位码与机内码转换在字库中的存储与寻址方式，让复杂的汉字编码变得易于理解。在实践应用方面，基于 STM32F103 与 AHT20 温度传感器等，展示了显示学号姓名、温湿度及长字符滑动显示的实现过程，从文字取模、代码编写、硬件连接到最终的编译烧录与效果呈现，每一步骤都详尽细致，为电子技术爱好者和相关专业学习者提供了极具价值的参考与学习范例，有助于深入掌握该领域知识与技能并应用于实际项目。

参考列表：  
1.[SPI协议详解（图文并茂+超详细）][SPI 1]  
2.[SPI原理超详细讲解—值得一看][SPI 2]  
3.[点阵汉字的字模读取与显示][Link 7]  
4.[基于STM32的0.96寸OLED显示屏显示数据][STM32_0.96_OLED]  
5.[STM32+OLED屏显应用实例][STM32_OLED]


[Link 1]: #_4
[OLED]: #OLED_10
[2.1_OLED]: #21OLED_11
[2.2_OLED]: #22OLED_27
[2.3_OLED]: #23OLED_47
[2.4_OLED]: #24OLED_54
[2.5]: #25_64
[Link 2]: #_80
[3.1]: #31_81
[3.2]: #32_96
[3.3]: #33_120
[3.4]: #34_140
[SPI]: #SPI_154
[4.1_SPI]: #41SPI_155
[4.2]: #42_168
[4.3_SPI]: #43SPI_184
[4.4]: #44_208
[4.5_SPI]: #45SPI_244
[4.6]: #46_277
[4.7]: #47_308
[4.8_UART]: #48UART_324
[Link 3]: #_339
[5.1]: #51_340
[5.2]: #52_358
[5.3]: #53_415
[5.4]: #54_434
[5.5]: #55_441
[5.6]: #56_444
[Link 4]: #_452
[6.1]: #61_453
[6.2]: #62_457
[6.3]: #63_590
[6.4]: #64_622
[6.5]: #65_629
[6.6]: #66_632
[Link 5]: #_640
[7.1]: #71_641
[7.2]: #72_645
[7.3]: #73_732
[7.4]: #74_751
[7.5]: #75_758
[7.6]: #76_763
[Link 6]: #_770
[0.96_SPI_OLED]: http://www.lcdwiki.com/res/Program/OLED/0.96inch/SPI_SSD1306_MSP096X_V1.0/0.96inch_SPI_OLED_Module_SSD1306_MSP096X_V1.0.zip
[https_pan.baidu.com_s_1kRSY2vH5Ycm527dE5tnvGw_pwd_2022]: https://pan.baidu.com/s/1kRSY2vH5Ycm527dE5tnvGw?pwd=2022
[http_www.lcdwiki.com_zh_0.96inch_SPI_OLED_Module]: http://www.lcdwiki.com/zh/0.96inch_SPI_OLED_Module
[https_pan.baidu.com_s_1YbUiGlyfxHi_h2B_rqogBg_pwd_2022]: https://pan.baidu.com/s/1YbUiGlyfxHi_h2B_rqogBg?pwd=2022
[https_pan.baidu.com_s_1-bktJppbVh54TvyERdEg0A_pwd_2022]: https://pan.baidu.com/s/1-bktJppbVh54TvyERdEg0A?pwd=2022
[https_gitee.com_xuwenjie10086_qrs_tree_master_OLED_E5_B1_8F_E5_B9_95_E6_98_BE_E7_A4_BA_E5_90_8D_E5_AD_97_E5_AD_A6_E5_8F_B7]: https://gitee.com/xuwenjie10086/qrs/tree/master/OLED%E5%B1%8F%E5%B9%95%E6%98%BE%E7%A4%BA%E5%90%8D%E5%AD%97%E5%AD%A6%E5%8F%B7
[https_pan.baidu.com_s_1cAklk_5GPxIzaUmWQ4dxrA_pwd_2022]: https://pan.baidu.com/s/1cAklk_5GPxIzaUmWQ4dxrA?pwd=2022
[https_gitee.com_xuwenjie10086_qrs_tree_master_OLED_E5_B1_8F_E5_B9_95_E6_98_BE_E7_A4_BA_E6_B8_A9_E6_B9_BF_E5_BA_A6]: https://gitee.com/xuwenjie10086/qrs/tree/master/OLED%E5%B1%8F%E5%B9%95%E6%98%BE%E7%A4%BA%E6%B8%A9%E6%B9%BF%E5%BA%A6
[https_pan.baidu.com_s_13rgixlMhU1Rt9q8LrU5-YA_pwd_2022]: https://pan.baidu.com/s/13rgixlMhU1Rt9q8LrU5-YA?pwd=2022
[https_gitee.com_xuwenjie10086_qrs_tree_master_OLED_E5_B1_8F_E5_B9_95_E6_BB_91_E5_8A_A8_E6_98_BE_E7_A4_BA_E9_95_BF_E5_AD_97_E7_AC_A6_E4_B8_B2]: https://gitee.com/xuwenjie10086/qrs/tree/master/OLED%E5%B1%8F%E5%B9%95%E6%BB%91%E5%8A%A8%E6%98%BE%E7%A4%BA%E9%95%BF%E5%AD%97%E7%AC%A6%E4%B8%B2
[SPI 1]: https://blog.csdn.net/u010632165/article/details/109460814?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166755074216782391843808%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166755074216782391843808&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-109460814-null-null.142%5Ev63%5Econtrol,201%5Ev3%5Eadd_ask,213%5Ev1%5Et3_esquery_v2&utm_term=SPI&spm=1018.2226.3001.4187
[SPI 2]: https://blog.csdn.net/as480133937/article/details/105764119?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166755074216782391843808%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166755074216782391843808&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-105764119-null-null.142%5Ev63%5Econtrol,201%5Ev3%5Eadd_ask,213%5Ev1%5Et3_esquery_v2&utm_term=SPI&spm=1018.2226.3001.4187
[Link 7]: https://blog.csdn.net/qq_46467126/article/details/121313820?spm=1001.2014.3001.5502
[STM32_0.96_OLED]: https://blog.csdn.net/qq_43279579/article/details/111414037
[STM32_OLED]: https://blog.csdn.net/qq_46467126/article/details/121439142?spm=1001.2014.3001.5502
