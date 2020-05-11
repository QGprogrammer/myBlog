#  阿里巴巴Java开发手册第二版

### 一、编码规约

##### （一）命名风格

* 【强制】代码中的命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束。 反例：_name / __name / $name / name_ / name$ / name__ 

* 【强制】代码中的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式。 说明：正确的英文拼写和语法可以让阅读者易于理解，避免歧义。注意，纯拼音命名方式更要避免采用。 正例：renminbi / alibaba / taobao / youku / hangzhou 等国际通用的名称，可视同英文。 反例：DaZhePromotion [打折] / getPingfenByName() [评分] / int 某变量 = 3 

* 【强制】类名使用UpperCamelCase风格，但以下情形例外：***DO / BO / DTO / VO / AO / PO / UID***等。 正例：JavaServerlessPlatform / UserDO / XmlService / TcpUdpDeal / TaPromotion 反例：javaserverlessplatform / UserDo / XMLService / TCPUDPDeal / TAPromotion 

* 【强制】方法名、参数名、成员变量、局部变量都统一使用lowerCamelCase风格，必须遵 从驼峰形式。 正例： localValue / getHttpMessage() / inputUserId

* 【强制】常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字 长。 正例：MAX_STOCK_COUNT / CACHE_EXPIRED_TIME 反例：MAX_COUNT / EXPIRED_TIME

* 【强制】抽象类命名使用***Abstract***或***Base***开头；异常类命名使用***Exception*** 结尾；测试类 命名以它要测试的类的名称开始，以***Test***结尾。

* 【强制】类型与中括号紧挨相连来表示数组。 正例：定义整形数组 int[] arrayDemo; 反例：在main 参数中，使用 String args[]来定义。 

* 【强制】POJO类中布尔类型变量都**不要加is**前缀，否则部分框架解析会引起序列化错误。 说明：在本文 MySQL 规约中的建表约定第一条，表达是与否的值采用 is_xxx 的命名方式，所以，需要在 <resultMap>设置从 is_xxx到 xxx的映射关系。 反例：定义为基本数据类型 Boolean isDeleted 的属性，它的方法也是 isDeleted()，RPC框架在反向解 析的时候，“误以为”对应的属性名称是 deleted，导致属性获取不到，进而抛出异常。

  >idea自动生成setter和Lombok自动生成setter，无论是Boolean还是boolean都是setXXX(); 所以在映射字段时候只会映射到XXX而不是isXXX，若是自己修改set方法，即可正常取值。
  >
  >Lombok插件会对包装类的Boolean做优化，他的setter方法可以拿到isXXX，boolean不行。
  >
  >建议还是不加is。

* 【强制】包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使 用单数形式，但是类名如果有复数含义，类名可以使用复数形式。 正例：应用工具类包名为 com.alibaba.ai.util、类名为 MessageUtils（此规则参考 spring 的框架结构） 

