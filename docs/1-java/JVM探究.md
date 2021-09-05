 # JVM探究

- 请你谈谈你对JVM的理解？java8虚拟机和之前的变化更新？
- 什么事OOM，什么是栈溢出StackOverFlowError？怎么分析
- JVM的常用调优参数有哪些？
- 内存快照如何抓取，怎么分析Dump文件？知道吗？
- 谈谈JVM，类加载器你的认识？



1. jvm的位置

![image-20210622230021873](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210622230021873.png)

2. JVM的体系结构

![image-20210622230928559](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210622230928559.png)



![image-20210622231047980](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210622231047980.png)



![image-20210705164326968](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210705164326968.png)

3. 类加载器

作用：加载Class文件 new Student();

![image-20210622233651895](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210622233651895.png)



- 虚拟机自带的加载器
- 启动类（根）加载器
- 扩展类加载器
- 应用程序加载器

```
public class Car {

    public int age;

    public static void main(String[] args) {
        Car car1 = new Car();
        Car car2 = new Car();
        Car car3 = new Car();
        System.out.println(car1.hashCode());
        System.out.println(car2.hashCode());
        System.out.println(car3.hashCode());
        Class<? extends Car> aClass1 = car1.getClass();

        ClassLoader classLoader = aClass1.getClassLoader();
        System.out.println(classLoader);//AppClassLoader 应用程序加载器
        System.out.println(classLoader.getParent());//ExtClassLoader 拓展类加载器 \jre\lib\ext

        System.out.println(classLoader.getParent().getParent());//1、不存在 2、java程序获取不到 rt.jar
    }

}
```



```
package lang;

public class String {

    //双亲委派机制：安全
    //1.APP-->EXC-->BOOT(最终执行)
    //BOOT
    //EXC
    //EXC


   /* public String toString(){
        return "hello";
    }*/

    public static void main(String[] args) {
        String s = new String();
        s.toString();
    }
    /**
     * 1.类加载器收到类加载的请求！
     * 2.将这个请求向上委托给父类加载器去完成，一直向上委托，直到启动类加载器
     * 3.启动加载器检查是否能够加载当前这个类，能加载就结束，使用当前的加载器，否则抛出异常，通知子加载器进行加载
     * 4.重复3步骤
     * Class Not Found
     * null  java调用不到C、C++
     * java=C++——
     *
     */
}
```



5. 百度：双亲委派机制

6. 双亲委派机制

   

7. 沙箱安全机制

   ### 什么是沙箱？

     Java安全模型的核心就是Java沙箱（sandbox），什么是沙箱？沙箱是一个限制程序运行的环境。沙箱机制就是将 Java 代码限定在虚拟机(JVM)特定的运行范围中，并且严格限制代码对本地系统资源访问，通过这样的措施来保证对代码的有效隔离，防止对本地系统造成破坏。沙箱**主要限制系统资源访问**，那系统资源包括什么？——`CPU、内存、文件系统、网络`。不同级别的沙箱对这些资源访问的限制也可以不一样。

     所有的Java程序运行都可以指定沙箱，可以定制安全策略。

   ### java中的安全模型：

     在Java中将执行程序分成本地代码和远程代码两种，本地代码默认视为可信任的，而远程代码则被看作是不受信的。对于授信的本地代码，可以访问一切本地资源。而对于非授信的远程代码在早期的Java实现中，安全依赖于沙箱 (Sandbox) 机制。如下图所示 JDK1.0安全模型
   ![在这里插入图片描述](https://img-blog.csdn.net/20181022102129401?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMzM2NDMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

     但如此严格的安全机制也给程序的功能扩展带来障碍，比如当用户希望远程代码访问本地系统的文件时候，就无法实现。因此在后续的 Java1.1 版本中，针对安全机制做了改进，增加了`安全策略`，允许用户指定代码对本地资源的访问权限。如下图所示 JDK1.1安全模型
   ![在这里插入图片描述](https://img-blog.csdn.net/2018102210231938?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMzM2NDMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

     在 Java1.2 版本中，再次改进了安全机制，增加了`代码签名`。不论本地代码或是远程代码，都会按照用户的安全策略设定，由类加载器加载到虚拟机中权限不同的运行空间，来实现差异化的代码执行权限控制。如下图所示 JDK1.2安全模型
   ![在这里插入图片描述](https://img-blog.csdn.net/20181022102746857?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMzM2NDMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

     当前最新的安全机制实现，则引入了域 (Domain) 的概念。虚拟机会把所有代码加载到不同的系统域和应用域，系统域部分专门负责与关键资源进行交互，而各个应用域部分则通过系统域的部分代理来对各种需要的资源进行访问。虚拟机中不同的受保护域 (Protected Domain)，对应不一样的权限 (Permission)。存在于不同域中的类文件就具有了当前域的全部权限，如下图所示 最新的安全模型(jdk 1.6)
   ![在这里插入图片描述](https://img-blog.csdn.net/20181022103108566?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMzM2NDMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

     以上提到的都是基本的`Java 安全模型概念`，在应用开发中还有一些`关于安全的复杂用法`，其中最常用到的 API 就是 doPrivileged。`doPrivileged 方法能够使一段受信任代码获得更大的权限，甚至比调用它的应用程序还要多，可做到临时访问更多的资源`。有时候这是非常必要的，可以应付一些特殊的应用场景。例如，应用程序可能无法直接访问某些系统资源，但这样的应用程序必须得到这些资源才能够完成功能。

   ### 组成沙箱的基本组件：

   - `字节码校验器`（bytecode verifier）：确保Java类文件遵循Java语言规范。这样可以帮助Java程序实现内存保护。但并不是所有的类文件都会经过字节码校验，比如核心类。

   - ```
     类装载器
     ```

     （class loader）：其中类装载器在3个方面对Java沙箱起作用

     - 它防止恶意代码去干涉善意的代码；
     - 它守护了被信任的类库边界；
     - 它将代码归入保护域，确定了代码可以进行哪些操作。

     虚拟机为不同的类加载器载入的类提供不同的命名空间，命名空间由一系列唯一的名称组成，每一个被装载的类将有一个名字，这个命名空间是由Java虚拟机为每一个类装载器维护的，它们互相之间甚至不可见。

     类装载器采用的机制是双亲委派模式。

   1. 从最内层JVM自带类加载器开始加载，外层恶意同名类得不到加载从而无法使用；
   2. 由于严格通过包来区分了访问域，外层恶意的类通过内置代码也无法获得权限访问到内层类，破坏代码就自然无法生效。

   - `存取控制器`（access controller）：存取控制器可以控制核心API对操作系统的存取权限，而这个控制的策略设定，可以由用户指定。

   - `安全管理器`（security manager）：是核心API和操作系统之间的主要接口。实现权限控制，比存取控制器优先级高。

   - ```
     安全软件包
     ```

     （security package）：java.security下的类和扩展包下的类，允许用户为自己的应用增加新的安全特性，包括：

     - 安全提供者
     - 消息摘要
     - 数字签名
     - 加密
     - 鉴别

   ### 沙箱包含的要素:

   #### 1. 权限

     权限是指允许代码执行的操作。包含三部分：权限类型、权限名和允许的操作。权限类型是实现了权限的Java类名，是必需的。权限名一般就是对哪类资源进行操作的资源定位（比如一个文件名或者通配符、网络主机等），一般基于权限类型来设置，有的比如java.security.AllPermission不需要权限名。允许的操作也和权限类型对应，指定了对目标可以执行的操作行为，比如读、写等。如下面的例子：

   ```
   permission java.security.AllPermission;    //权限类型
   permission java.lang.RuntimePermission "stopThread";    //权限类型+权限名
   permission java.io.FilePermission "/tmp/foo" "read";    //权限类型+权限名+允许的操作
   123
   ```

   标准权限:

   | 说明       | 类型                                | 权限名                                                   | 操作                   | 例子                                                         |
   | ---------- | ----------------------------------- | -------------------------------------------------------- | ---------------------- | ------------------------------------------------------------ |
   | 文件权限   | java.io.FilePermission              | 文件名（平台依赖）                                       | 读、写、删除、执行     | 允许所有问价的读写删除执行：permission java.io.FilePermission “<< ALL FILES>>”, “read,write,delete,execute”;。允许对用户主目录的读：permission java.io.FilePermission “${user.home}/-”, “read”; |
   | 套接字权限 | java.net.SocketPermission           | 主机名:端口                                              | 接收、监听、连接、解析 | 允许实现所有套接字操作：permission java.net.SocketPermission “:1-”, “accept,listen,connect,resolve”;。允许建立到特定网站的连接：permission java.net.SocketPermission “.abc.com:1-”, “connect,resolve”; |
   | 属性权限   | java.util.PropertyPermission        | 需要访问的jvm属性名                                      | 读、写                 | 读标准Java属性：permission java.util.PropertyPermission “java.”, “read”;。在sdo包中创建属性：permission java.util.PropertyPermission “sdo.”, “read,write”; |
   | 运行时权限 | java.lang.RuntimePermission         | 多种权限名[见附录A]                                      | 无                     | 允许代码初始化打印任务：permission java.lang.RuntimePermission “queuePrintJob” |
   | AWT权限    | java.awt.AWTPermission              | 6种权限名[见附录B]                                       | 无                     | 允许代码充分使用robot类：permission java.awt.AWTPermission “createRobot”; permission java.awt.AWTPermission “readDisplayPixels”; |
   | 网络权限   | java.net.NetPermission              | 3种权限名[见附录C]                                       | 无                     | 允许安装流处理器：permission java.net.NetPermission “specifyStreamHandler”;。 |
   | 安全权限   | java.security.SecurityPermission    | 多种权限名[见附录D]                                      | 无                     |                                                              |
   | 序列化权限 | java.io.SerializablePermission      | 2种权限名[见附录E]                                       | 无                     |                                                              |
   | 反射权限   | java.lang.reflect.ReflectPermission | suppressAccessChecks（允许利用反射检查任意类的私有变量） | 无                     |                                                              |
   | 完全权限   | java.security.AllPermission         | 无（拥有执行任何操作的权限）                             | 无                     |                                                              |

   #### 2. 代码源

      代码源是类所在的位置，表示为以URL地址。

   #### 3. 保护域

      保护域用来组合代码源和权限，这是沙箱的基本概念。保护域就在于声明了比如由代码A可以做权限B这样的事情。

   #### 4. 策略文件

     策略文件是控制沙箱的管理要素，一个策略文件包含一个或多个保护域的项。策略文件完成了代码权限的指定任务，策略文件包括全局和用户专属两种。

     为了管理沙箱，策略文件我认为是最重要的内容。JVM可以使用多个策略文件，不过一般两个最常用。一个是全局的：`$JREHOME/lib/security/java.policy`，作用于JVM的所有实例。另一个是用户自己的，可以存储到用户的主目录下。策略文件可以使用jdk自带的policytool工具编辑。

   默认的策略文件我们先参考一下：

   ```text
   // Standard extensions get all permissions by default
   grant codeBase "file:${{java.ext.dirs}}/*" {
           permission java.security.AllPermission;
   };
   // default permissions granted to all domains
   grant {
           // Allows any thread to stop itself using the java.lang.Thread.stop()
           // method that takes no argument.
           // Note that this permission is granted by default only to remain
           // backwards compatible.
           // It is strongly recommended that you either remove this permission
           // from this policy file or further restrict it to code sources
           // that you specify, because Thread.stop() is potentially unsafe.
           // See the API specification of java.lang.Thread.stop() for more
           // information.
           permission java.lang.RuntimePermission "stopThread";
           // allows anyone to listen on dynamic ports
           permission java.net.SocketPermission "localhost:0", "listen";
           // permission for standard RMI registry port
           permission java.net.SocketPermission "localhost:1099", "listen";
           // "standard" properies that can be read by anyone
           permission java.util.PropertyPermission "java.version", "read";
           permission java.util.PropertyPermission "java.vendor", "read";
           permission java.util.PropertyPermission "java.vendor.url", "read";
           permission java.util.PropertyPermission "java.class.version", "read";
           permission java.util.PropertyPermission "os.name", "read";
           permission java.util.PropertyPermission "os.version", "read";
           permission java.util.PropertyPermission "os.arch", "read";
           permission java.util.PropertyPermission "file.separator", "read";
           permission java.util.PropertyPermission "path.separator", "read";
           permission java.util.PropertyPermission "line.separator", "read";
           permission java.util.PropertyPermission "java.specification.version", "read";
           permission java.util.PropertyPermission "java.specification.vendor", "read";
           permission java.util.PropertyPermission "java.specification.name", "read";
           permission java.util.PropertyPermission "java.vm.specification.version", "read";
           permission java.util.PropertyPermission "java.vm.specification.vendor", "read";
           permission java.util.PropertyPermission "java.vm.specification.name", "read";
           permission java.util.PropertyPermission "java.vm.version", "read";
           permission java.util.PropertyPermission "java.vm.vendor", "read";
           permission java.util.PropertyPermission "java.vm.name", "read";
   };
   1234567891011121314151617181920212223242526272829303132333435363738394041
   ```

     策略文件的内容格式就是这样，grant授权允许操作某个权限。这个默认的策略文件就指明了jdk扩展包可以有全部权限，允许代码stop线程，允许监听1099端口(1099号端口，是默认的服务器端RMI监听端口)等等。

     另一个很重要的是参数文件——java.security，这个文件和策略文件在同一个目录下。这个参数文件定义了沙箱的一些参数。比如默认的沙箱文件是这样的（截取部分）：

   ```text
   # The default is to have a single system-wide policy file,
   # and a policy file in the user's home directory.
   policy.url.1=file:${java.home}/lib/security/java.policy
   policy.url.2=file:${user.home}/.java.policy
   # whether or not we expand properties in the policy file
   # if this is set to false, properties (${...}) will not be expanded in policy
   # files.
   policy.expandProperties=true
   # whether or not we allow an extra policy to be passed on the command line
   # with -Djava.security.policy=somefile. Comment out this line to disable
   # this feature.
   policy.allowSystemProperty=true
   123456789101112
   ```

      **`policy.url.\*这个属性指明了使用的策略文件`**，如上文所述，默认的两个位置就在这里配置，用户可以自行更改顺序和存储位置。而policy.allowSystemProperty指明是否允许用户自行通过命令行指定policy文件。

   #### 5. 密钥库

      保存密钥证书的地方。

   ### 默认沙箱

     通过Java命令行启动的Java应用程序，默认不启用沙箱。要想启用沙箱，启动命令需要做如下形式的变更：
       `java -Djava.security.manager <other args>`
     沙箱启动后，安全管理器会使用两个默认的策略文件来确定沙箱启动参数。当然也可以通过命令指定：
       `java -Djava.security.policy=<URL>`
     如果要求启动时只遵循一个策略文件，那么启动参数要加个等号，如下：
       `java -Djava.security.policy==<URL>`

   ### 如何使用

   #### 1. 限制读文件

   这个例子很简单，首先写一个r.txt文件，里面的内容是“abcd”，再写个程序如下读取这个r.txt。

   ```java
   import java.io.File;
   import java.io.FileInputStream;
   import java.io.FileNotFoundException;
   import java.io.IOException;
   import java.io.InputStream;
   public class PolicyTest {
       public static void file() {
           File f = new File("D:\\github\\CDLib\\src\\main\\resources\\security\\r.txt");
           InputStream is;
           try {
               is = new FileInputStream(f);
               byte[] content = new byte[1024];
               while (is.read(content) != -1) {
                   System.out.println(new String(content));
               }
           } catch (FileNotFoundException e) {
               // TODO Auto-generated catch block
               e.printStackTrace();
           } catch (IOException e) {
               // TODO Auto-generated catch block
               e.printStackTrace();
           }
       }
       public static void main(String[] args) {
           // test read file.
           file();
       }
   }
   12345678910111213141516171819202122232425262728
   ```

   发现输出是abcd。

   接下来修改java启动参数，加入-Djava.security.manager，启动了安全沙箱。再运行，输出变成了异常

   ```
   Exception in thread "main" java.security.AccessControlException: access denied ("java.io.FilePermission" "D:\github\CDLib\src\main\resources\security\r.txt" "read") 
   at java.security.AccessControlContext.checkPermission(Unknown Source) 
   at java.security.AccessController.checkPermission(Unknown Source) 
   at java.lang.SecurityManager.checkPermission(Unknown Source) 
   at java.lang.SecurityManager.checkRead(Unknown Source) 
   at java.io.FileInputStream.(Unknown Source) 
   at com.taobao.cd.security.PolicyTest.main(PolicyTest.java:15)
   1234567
   ```

     这里已经提示了，访问被拒绝，说明了沙箱启动，同时也验证了默认沙箱——禁止本地文件访问。

   再来，我们构建一个custom.policy文件如下：

   ```
   grant {
       permission java.io.FilePermission "D:\\github\\CDLib\\src\\main\\resources\\security\\*", "read";
   };
   123
   ```

   这里构建了一条安全策略——允许读取security目录下的文件。

   修改启动命令，添加

   ```
   -Djava.security.policy=D:\\github\\CDLib\\src\\main\\resources\\security\\custom.policy
   1
   ```

   再执行，结果输出了abcd。

     如上例。我们通过自定义policy文件修改了默认沙箱的安全策略，再通过启动参数开启沙箱模式。这样就可以构造我们自己想要的沙箱效果了。

   ### 附录

   A

   | 权限名                | 用途说明                                                     |
   | --------------------- | ------------------------------------------------------------ |
   | accessClassInPackage. | 允许代码访问指定包中的类                                     |
   | accessDeclaredMembers | 允许代码使用反射访问其他类中私有或保护的成员                 |
   | createClassLoader     | 允许代码实例化类加载器                                       |
   | createSecurityManager | 允许代码实例化安全管理器，它将允许程序化的实现对沙箱的控制   |
   | defineClassInPackage. | 允许代码在指定包中定义类                                     |
   | exitVM                | 允许代码关闭整个虚拟机                                       |
   | getClassLoader        | 允许代码访问类加载器以获得某个特定的类                       |
   | getProtectionDomain   | 允许代码访问保护域对象以获得某个特定类                       |
   | loadlibrary.          | 允许代码装载指定类库                                         |
   | modifyThread          | 允许代码调整指定的线程参数                                   |
   | modifyThreadGroup     | 允许代码调整指定的线程组参数                                 |
   | queuePrintJob         | 允许代码初始化一个打印任务                                   |
   | readFileDescriptor    | 允许代码读文件描述符（相应的文件是由其他保护域中的代码打开的） |
   | setContextClassLoader | 允许代码为某线程设置上下文类加载器                           |
   | setFactory            | 允许代码创建套接字工厂                                       |
   | setIO                 | [允许代码重定向System.in](http://xn--system-ox8ir5ngrhe5ufw7emp6bnsq.in/)、System.out或System.err输入输出流 |
   | setSecurityManager    | 允许代码设置安全管理器                                       |
   | stopThread            | 允许代码调用线程类的stop()方法                               |
   | writeFileDescriptor   | 允许代码写文件描述符                                         |

   B

   | 权限名                         | 用途说明                      |
   | ------------------------------ | ----------------------------- |
   | accessClipboard                | 允许访问系统的全局剪贴板      |
   | accessEventQueue               | 允许直接访问事件队列          |
   | createRobot                    | 允许代码创建AWT的Robot类      |
   | listenToAllAWTEvents           | 允许代码直接监听事件分发      |
   | readDisplayPixels              | 允许AWT Robot读显示屏上的像素 |
   | showWindowWithoutWarningBanner | 允许创建无标题栏的窗口        |

   C

   | 权限名                        | 用途说明                      |
   | ----------------------------- | ----------------------------- |
   | specifyStreamHandler          | 允许在URL类中安装新的流处理器 |
   | setDefaultAuthenticator       | 可以安装鉴别类                |
   | requestPassworkAuthentication | 可以完成鉴别                  |

   D

   | 权限名                     | 用途说明                                 |
   | -------------------------- | ---------------------------------------- |
   | addIdentityCertificate     | 为Identity增加一个证书                   |
   | clearProviderProperties.   | 针对指定的提供者，删除所有属性           |
   | createAccessControlContext | 允许创建一个存取控制器的上下文环境       |
   | getDomainCombiner          | 允许撤销保护域                           |
   | getPolicy                  | 检索可以实现沙箱策略的类                 |
   | getProperty.               | 读取指定的安全属性                       |
   | getSignerPrivateKey        | 由Signer对象获取私有密钥                 |
   | insertProvider.            | 将指定的提供者添加到响应的安全提供者组中 |
   | loadProviderProperties.    | 装载指定的提供者的属性                   |
   | printIdentity              | 打印Identity类内容                       |
   | putAllProviderProperties.  | 更新指定的提供者的属性                   |
   | putProviderProperty.       | 为指定的提供者增加一个属性               |
   | removeIdentityCertificate  | 取消Identity对象的证书                   |
   | removeProvider.            | 将指定的提供者从相应的安全提供者组中删除 |
   | removeProviderProperty.    | 删除指定的安全提供者的某个属性           |
   | setIdentityInfo            | 为某个Identity对象设置信息串             |
   | setIdentityPublicKey       | 为某个Identity对象设置公钥               |
   | setPolicy                  | 设置可以实现沙箱策略的类                 |
   | setProperty.               | 设置指定的安全属性                       |
   | setSignerKeyPair           | 在Signer对象中设置密钥对                 |
   | setSystemScope             | 设置系统所用的IdentityScope              |

   E

   | 权限名                       | 用途说明                                                     |
   | ---------------------------- | ------------------------------------------------------------ |
   | enableSubstitution           | 允许实现ObjectInputStream类的enableResolveObject()方法和ObjectOutputStream类的enableReplaceObject()方法 |
   | enableSubclassImplementation | 允许ObjectInputStream和ObjectOutputStream创建子类，子类可以覆盖readObject()和writeObject()方法 |

8. Native

   

9. PC寄存器









5. 方法区

   Mehtod Area 方法区

   方法区是被锁的线程共享，所有字段和方法字节码，一级一些特殊方法，如构造函数、接口代码也在此定义，简单说，所有定义的方法的信息都保存在该区域，此区域属于共享区域。

   静态变量、常量、类信息（构造方法、接口9定义）、运行时的常量池存在方法区中，但是实力变量存在堆内存中，和方法区无关

   static final Class,常量池

   在java基础 创建对像的内存分析

   

6. 栈

   栈：数据结果

   程序=数据结构+算法：持续学习

   程序=框架+业务逻辑：淘汰、吃饭

   先进后出、后进先出。桶

   队列：先进先出（FIFO ：First Input First Output）

   喝多了吐就是栈，吃多了拉就是队列

   为什么main（）先执行，最后结束

   栈：栈内存，主管程序的运行，生命周期和线程同步；

   线程结束，栈内存也就释放，对于栈来说，不存在垃圾回收问题

   一旦线程结束，栈就over！

   栈：8大基本类型+对象应引用+实例的方法

栈运行原理：栈帧



栈满了：StackOverflowError

![image-20210627143633579](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210627143633579.png)

栈+堆+方法区：交互关系

![image-20210627143853850](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210627143853850.png)

画出一个对象实例化的过程



5. 三种JVM

- sun公司  HotSpot(TM) Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)

- BEA JRocket

- IBM J9 VM 

我们学习的是:HotSpot(TM)

## 堆

Heap,一个JVM只有一个堆内存，堆内存的大小是可以调节的。

类加载器读取了类文件后，一般会把什么东西放在堆中？类，方法，常量，变量，保存我们引用类型的真实对象；

堆内存中还要细分为三个区域：

- 新生区（伊甸园区）

- 养老区
- 永久区

![image-20210627181921427](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210627181921427.png)

Gc垃圾回收，主要是在伊甸园区和养老区

假设内存满了，OOM，堆内存不够

![image-20210627182207266](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210627182207266.png)

在JDK8以后，永久存储区改了个名字（元空间）；

**新生区**

- 类：诞生成长的地方，甚至死亡；
- 伊甸园，所有的对象都是在伊甸园区new出来的
- 幸存者区（0、1）





**老年区**



![image-20210627184835855](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210627184835855.png)

真理：经过研究，99%的对象都是临时对象！



**永久区**

这个区域常驻内存的。用来存放jdk自身携带的class对象。interface元数据，存储的是java运行时的一些环境或类型信息-，这个区域不存在垃圾回收！关闭虚拟机就会释放这个区域的内存~

一个启动类，加载了大量的第三方jar包。Tomcat部署了太多的应用，大量动态生成的反射类。不断的被加载。直到内存满，就会出现OOM



![image-20210627190855250](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210627190855250.png)

元空间：逻辑上存在，物理上不存在

在项目中，突然出现了OOM故障，那么该如何排除·研究为什么出错~

- 能够看到代码第几行出错：内存快照分析工具，MAT，JProfiler

- Debug,一行行分析代码！

  MAT，JProfiler作用



- 分析Dump内存文件，快速定位内存泄漏
- 获取堆中的数据
- 获取大的对象
- 。。。。。。









```
package jie;

import java.util.Random;

public class Hello {
    public static void main(String[] args) {
        String str="jiestudy";
        while (true){
            str+=str+new Random().nextInt(888888888)+new Random().nextInt(888888888);
        }
    }
}
```

```
package jie;

public class Demo02 {
    public static void main(String[] args) {
        //返回虚拟机试图使用的最大内存
        long max = Runtime.getRuntime().maxMemory();

        //返回虚拟机的初始化总内存
        long total= Runtime.getRuntime().totalMemory();
        System.out.println("max"+max+"字节\t"+(max/(double)1024/1024)+"MB");
        System.out.println("total"+max+"字节\t"+(total/(double)1024/1024)+"MB");

        //默认情况下：分配的总内存是电脑内存的1/4，而初始化内存1/64

        //max3747086336字节  3573.5MB
        //total3747086336字节    241.5MB
        //Heap
        // PSYoungGen      total 75264K, used 6451K [0x000000076c400000, 0x0000000771800000, 0x00000007c0000000)
        //  eden space 64512K, 10% used [0x000000076c400000,0x000000076ca4ce80,0x0000000770300000)
        //  from space 10752K, 0% used [0x0000000770d80000,0x0000000770d80000,0x0000000771800000)
        //  to   space 10752K, 0% used [0x0000000770300000,0x0000000770300000,0x0000000770d80000)
        // ParOldGen       total 172032K, used 0K [0x00000006c4c00000, 0x00000006cf400000, 0x000000076c400000)
        //  object space 172032K, 0% used [0x00000006c4c00000,0x00000006c4c00000,0x00000006cf400000)
        // Metaspace       used 3234K, capacity 4496K, committed 4864K, reserved 1056768K
        //  class space    used 349K, capacity 388K, committed 512K, reserved 1048576K
        //PSYoungGen      total (75264K +172032K)/1024=241.5MB

    }

    //-Xms1024m -Xmx1024m -XX:+PrintGCDetails
    //OOM:
    //1.常试扩大堆内存看结果
    //2.分析内存，看一下哪个地方出现了问题（）
}
```

```
package jie;

import java.util.ArrayList;


//-Xms设置初始化内存分配大小1/64
//-XX:PrintGCDetails //打印GC垃圾回收
//Dump -XX:+HeapDumpOnOutOfMemoryError //00m DUMP

public class Demo03 {
    byte[] array=new byte[1*1024*1024];
    public static void main(String[] args) {
        ArrayList<Demo03> list = new ArrayList<>();

        int count=0;
        try {

            while (true){
                list.add(new Demo03());
                count=count+1;
            }
        }catch (Error e){
            System.out.println("count"+count);
            e.printStackTrace();
        }

    }
}
```

- jdk1.6之前：永久代，常量池在方法区；
- jdk1.7：永久代，但是慢慢的退化了，去永久代，常量池在堆中；
- jdk1.8之后：无永久代，常量池在元空间







1. 新生区、老年区
2. 永久区
3. 堆内存调优
4. GC：垃圾回收

不能手动回收，只能手动提醒

![image-20210705151319407](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210705151319407.png)

JVM在进行GC时，并不是对着三个区域统一回收，大部分时候，回收都是新生代~

- 新生代

- 幸存区（from ,to）

- 老年区

  ### GC两种类型：轻GC（普通的GC）， 重GC（全局GC）

题目：

- jvm的内存模型和分区~详细到每个区放什么？
- 堆里面的分区有哪些？Eden，from，to，老年区，说好他们的特点
- GC的算法有哪些？标记清除，标记整理（压缩），复制算法，引用计数法，怎么用的？
- 轻GC和重GC分贝在什么时候发生？



**引用计数法：**



![image-20210705152243831](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210705152243831.png)

**复制算法：**

谁空谁是to



![image-20210705153118976](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210705153118976.png)



![image-20210705153348248](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210705153348248.png)



- 好处：没有内存的碎片
- 坏处：浪费了内存空间：多了一半空间永远是空to,假设对象100%存活（极端情况）



复制算法最佳使用场景：对象存活度较低的时候：新生区~





**标记清除法：**



![image-20210705153858178](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210705153858178.png)



- 优点：不需要额外的空间！

- 缺点：两次扫描，严重浪费时间，会产生内存碎片

  标记压缩：

再优化

![image-20210705154231459](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210705154231459.png)



**标记清楚压缩**

先标记清除几次，再进行压缩



1. 常用算法

   

2. JMM

## 总结

内存效率：复制算法>标记清楚算法>标记压缩算法（时间复杂度）

内存整齐度：复制算法=标记压缩算法>标记清除算法

内存利用率：标记压缩算法=标记清除算法>复制算法

思考一个问题：难道没有一个最优的算法吗？

没有,没有最好的算法，只有最合适的算法---->GC：分代收集算法

年轻代：

- 存活率低
- 复制算法



老年代：

- 区域大：存活率
- 标记清除（内存碎片不是太多）+标记压缩混合实现



## JMM（java内存模型）

1.什么是jmm

https://www.jianshu.com/p/8a58d8335270

JMM即Java内存模型(Java memory model)，在JSR133里指出了JMM是用来定义一个**一致的、跨平台**的内存模型，是缓存一致性协议，用来定义数据读写的规则。（遵守，找到这个规则）

JMM定义了线程工作内存和主内存之间的抽象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。





![image-20210705162406106](C:\Users\11544\AppData\Roaming\Typora\typora-user-images\image-20210705162406106.png)

解决共享对象可见性这个问题：volatile关键字



2.它是干嘛的？：官方，博客，对应的视频



3.它如何学习？

https://www.cnblogs.com/null-qige/p/9481900.html

JMM:抽象的概念，理论

## 内存交互操作

 　内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）

- - lock   （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
  - unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
  - read  （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
  - load   （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
  - use   （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
  - assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
  - store  （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
  - write 　（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

　　JMM对这八种指令的使用，制定了如下规则：

- - 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
  - 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
  - 不允许一个线程将没有assign的数据从工作内存同步回主内存
  - 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
  - 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
  - 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
  - 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
  - 对一个变量进行unlock操作之前，必须把此变量同步回主内存

　　JMM对这八种操作规则和对[volatile的一些特殊规则](https://www.cnblogs.com/null-qige/p/8569131.html)就能确定哪里操作是线程安全，哪些操作是线程不安全的了。但是这些规则实在复杂，很难在实践中直接分析。所以一般我们也不会通过上述规则进行分析。更多的时候，使用java的happen-before规则来进行分析。



学习新东西是常态：

