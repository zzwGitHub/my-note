# MyBatis从入门到精通

## 第二章

### select 用法

- 默认遵循“下划线转驼峰”

- byte[] 对应的是 BLOB LONGVAREBINARY 以及一些和二进制流有关的字段类型

- java实体类中，不要使用int 要使用 Integer ，因为int默认值是0。xml中判断时，如果判断 ==null 时，总会得到 true。会出现隐藏问题

- 接口方法和xml文件中的id对应，如果在xml中找不到对应的id，会报错

- 映射XML 和  接口的命名规则

  - 只使用 XML 没有接口时，namespace 可以为**任意不重复**的值
  - 标签的 id 属性不能有 英文 的 点 ，并且不能重复
  - 接口中方法是可以重载的，知道吧~~~~而XML的id对应的是方法名，所以**接口中的所有同名方法会对应着XML中的同一个标签**

- **井号大括号**：`#{id}` 这是预编译参数的一种方式

- resultMap 标签参数如下：

  | 属性名      | 是否必填 | 属性值       | 解释                 |
  | ----------- | -------- | ------------ | -------------------- |
  | id          | 必       |              | 标识                 |
  | type        | 必       |              | 欲映射成的java实体类 |
  | extends     |          | 欲继承者的id | 继承自其它resultMap  |
  | autoMapping |          |              |                      |

- resultMap 包含的标签如下：（它们负责构建出JAVA对象哦~）

  | 标签名        | 解释                                                         |
  | ------------- | ------------------------------------------------------------ |
  | constructor   | 通过构造方法生成 JAVA对象。他有idArg 和 arg两个子标签。对应着 id 和 普通结果。 |
  | id            | 负责主键（或唯一值）字段（可以多个）。（提高性能）           |
  | result        | 负责普通结果                                                 |
  | association   |                                                              |
  | collection    |                                                              |
  | discriminator |                                                              |
  | case          |                                                              |

- id 标签 和 result 标签 的属性如下：（它们两个最常用）

  | 属性名      | 解释                                                         |
  | ----------- | ------------------------------------------------------------ |
  | column      | sql 语句查询出来的列名                                       |
  | property    | java 类的属性名，也可以映射对象中的对象，例如 address.street.number。（不区分大小写） |
  | javaType    |                                                              |
  | jdbcType    |                                                              |
  | typeHandler |                                                              |

- 接口定义的返回值类型必须和XML中配置的 resulType 一致，或者与 resultMap 中的 type 一致。真正决定返回类型的是 XML！

- 接口的返回值，可以自行判断是  SysUser 还是 List\<SysUser>，总之XML 中没有这种区分。但是！如果返回值是多个。而接口中是单个。会抛出异常 ： TooManyResyltsException

- 不是非要写 resultMap，如果sql 返回列名就能和实际类**完全对应**上，那么！直接 resultType就成。

- resultMap 中的property 不是对应java类中的域吗.......这个对应，mybatis 是先把两个都变成大写，换言之，**property ，不区分大小写**。

- 很多数据库是不区分大小写的，所以！推荐的命名方式是：`user_name  user_email`这样，这种方式也很常见。同样常见是java 中 使用 驼峰命名法。因此，Mybatis提供一个全局属性 mapUnderscoreToCamelCase 。

  如果把它配为 true （默认是false），mybatis 在把sql 返回列解析为java 类的时候，**就会自动将下划线命和驼峰对应上** ！

  ```xml
  <settings>
      <!-- 其他配置......-->
      <setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
  ```

- 在 sql 语句中，如果把列名 别名为 user.userName （这种带有层级结构的）那么 resultType 指向实体类中的 user 属性(user是另外一个实体类)，就能直接被赋值了。这种方法很方便！

### insert

- sql 可以写成

  `insert into sys_user(create_time) values (#{createTime, jdbcType= TIMESTAMP})`

  这是因为数据库区分 date、time、datetime类型，而Java中一般都使用 Date。为了对应准确，最好手动指定一下

- 由于默认的 `sqlSessionFactory.openSession()`是不自动提交的，因此不手动执行 commit 是不会提交到数据库的。当然也可以执行 `sqlSession.rollback()`进行回滚

- mybatis可以做出 **增加完成后，为原对象赋值** 的操作，当然最常见的就是填充实体类的ID了。做法如下

  ```xml
  <insert id="insert2" useGeneratedKeys="true" keyProperty="id">
  	insert into sys_user(user_name, user_password, user_email,
  	user_info, head_img, create_time)
  	values(
  	#{userName}, #{userPassword}, #{userEmail}, #{userInfo}, #{headImg, jdbcType=BLOB}, #{createTime, jdbcType=TIMESTAMP} )
  </insert>
  ```

  这样写的话：语句中必然是没有ID的；以及加入了两个标签，如果需要操作原实体类的多个类，需要研究 keyColumn 属性值

- 如果数据库没有主键自增这个设定，需要使用 selectKey 标签用到再研究吧

  ```xml
  <selectKey keyColumn="id" resultType="long" keyProperty="id" order="AFTER" >
  	SELECT LAST INSERT ID () 
  </selectKey>
  ```

### update

### delete

### 2.7 多参数的用法

- 如果是一个参数，Mybatis 不管那么多，直接拿来用（em.......）
- 如果是多参数（与参数类型无关，下面也是哦），第一种方案是 使用map（不建议）
- 第二种方案，使用 **顺序** ：Available parameters are [0, 1, param1, param2] 
- 第三种方案，使用@Param 注释
- 如果是嵌套的话，比如入参是个user对象，也可以 @Param("u") User user。然后用的时候#{u.name} 这样~

## 第三章 注解

- @Select 、@Inser、@Update、@Delete ，很简单，如果用到了，再看吧

- 是不推荐使用注解的，因为需要重新编译代码呀

- 对于返回值对应关系呢，可以：

  - sql 取别名和java类对应上
  - mapUnderscoreToCamelCase 配置为true

- 对于返回值，还是向 resultMap 上想，企图将sql的返回值做成java类。

  使用 @Results 注解。它在 3.3.0 版之后，就能**公用**了(@ResultMap)，居然也可以公用 xml 中配置的.....

- 还有 @PrivilegeMapper 

## 第四章 动态 SQL

### 4.1  if  标签

- if 标签必须有的一个属性是 test

- test 中的表达式，称作 OGNL 表达式，它的结果可以使 true 或者 false 。此外，**所有非 0 值都是true**，只有 0 是false。为了不混乱，还是用 是否 吧。

- 判断条件 `property !=null` 或 `property == null`： 适用于任何类型的字段 ，用于判断属性值是否为空。

- 判断条件 `property != ''` 或 `property == ''` ：仅适用于 String 类型的字段，用于判断是否为空字符串。

- **仅利用 if 标签就可以完成较为复杂的 “查询” 代码拼接了**

  - 技巧1：在 where 后跟一个 1=1
  - 技巧2：可以使用 and 和 or ，并且使用小括号处理它们混用的情况

- **仅利用 if 标签实现 “更新”**

  ```xml
  update sys_user
  set
  	<if test="userPassword != null and userPassword != ''">
  		user_password = #{userPassword},
  	</if>
  
  	......
  
  	<if test="createTime != null">
  		create_time = #{cerateTime, jdbcType=TIMESTAMP},
  	</if>
  	id = #{id}
  where id = #{id}
  ```

  **注意：每个if内，结尾有逗号；在所有if结束后加一个id = #{id}**

  十分精髓的技巧！这就保证了**对不同参数个数的兼容性**

- **仅利用 if 标签实现 “新增”**

  - 注意，在列部分加了 if，那么 value 部分也要加 if，不要忘记。

### 4.2  choose 标签

- 如果想实现 if else 效果，需要用到此标签

- 其中至少有一个 when ，可以没有otherwise

  ```xml
  <choose> 
      <when test=" id != null " >
      and id= #{id} 
      </when> 
      <when test=" userName != null and userName !=''">
      and user name = #{userName} 
      </when> 
      <otherwise> 
      and 1 = 2 
      </otherwise> 
  </choose>
  ```

### 4.3  where / set / trim

- where：如果标签内有值，就插入一个where；如果表达式最终结果，开头是and 或者 or，就自动抹去。较上面的例子更干净

- set：如果过标签内元素有值，就插入一个set；如果set最后以逗号结束，就抹去。也是更干净！

  - 但是啊！发现如果 if 导致的结果是set 中空无一物，那也是会报错的，这可想而知（update竟然一个也不更新？！）。所以要考虑这个问题的存在。
  - 结论：还是在 set 内，结尾加个 `id = #{id}` 吧............

- trim：其实上面两个都是这个标签实现的

  ```xml
  <!-- 假装where --> 
  <!-- 注意AND 和 OR 后有空格！！！！！ --> 
  <trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
  </trim>
  ```

  - prefix：当标签内有内容时，给内容加前缀
  - prefixOverrids：当标签内有内容时，会看内容最前是否有此项，有就去掉
  - suffix：加后缀
  - suffixOverrides：去后缀

### 4.4  foreach标签

```xml
...
where id in
<foreach collection="list" open="(" close=")" separator="," item="id" index="i">
	#{id}
</foreach>
...
```

- 属性

  - collection：必填！这是要迭代循环的属性名，与需要循环的那个入参对应
  - item：每次迭代的指代
  - index：在集合数组情况下是 每次的索引。**当迭代是Map时，它就是Key值。**
  - 剩下三个，不写了，都明白

- 详解collection参数

  - 当入参只一个，且是 数组或者 集合时，会默认转成 Map，意味着，我们填写在 collection属性中的值，需要是入参被匹配的Key值。下面说这个Key是怎么被默认上的
  - 如果入参 instanceof Collection，key是 collection。如果入参 instanceof List，key 是 list（因为map里是两条，所以key也是 collection）。如果入参是 数组，key 是 array

- 上面说的是默认的。**推荐使用 @Param 来指定**..........不去用那混乱的默认。补充：当然给方法的入参就直接是一个自己弄好的Map也是可以的。对象也行。这一小部分，主要说的是mybatis是怎么默认。。

- 常用的是 in 场景，当然如果想批量 insert（首先数据库要支持）的话，也是可以的。

- 遍历Map的介绍，正好使用update作为例子：**也用到了 ${}** 

  ```xml
  <update id="test001">
      update sys_user
      set
      <foreach collection="_parameter" item="val" index="key" separator=",">
          ${key} = #{val}
      </foreach>
      where id = #{id}
  </update>
  ```

### 4.5 bind 用法

```xml
<select id="selectLike" resultMap="userMap" >
    <bind name= "bb" value="'%' + string + '%'" />
    select * from sys_user 
    where user_password like #{bb}
</select>
<!-- 或者 -->
<if test="userName != null and userName !=''">
    <bind name= "userNameLike" value="'%' + userName + '%'" />
    and user_name like #{userNameLike} 
</if>
```

- 书中给的案例是不同数据库，对于 concat  函数的支持是不同。所以在sql中用 % 拼接 sql 是有风险的
- 其实这个东西就是拼接而已，拼出一个 OGNL 表达式，以备后用
- 发现参数还是 规矩的@Param指定下吧，不然好乱

### 4.6  多数据库支持

- 就是写完的代码在mysql或orcale上跑时，执行不同的sql。可以直接写两个select标签指定databaseId属性，也可以在if中 用_databaseId 去判断。当然这个 databaseId是需要再xml中预先配置的

  ```xml
  <!-- mybatis-config.xml -->
  <databaseIdProvider type="DB_VENDOR" >
      <property name=" SQL Server " value="sqlserver" />
      <property name="Oracle" value="oracle" />
      <property name="MySQL" value="mysql"/>
  </databaseIdProvider>
  ```

- 这个简单看看就完事了，用到时再说吧

### 4.7  OGNL表达式

- 这东西在动态SQL 、${balabaka} 、#{balabala}  中都会用到，需要关注下哦！

- 写这种表达式的时候，头脑首先做个切换，就好像在写 java 代码一样（或者js代码）

  ```xml
  <select id="testBind" resultMap="userMap" >
      <bind name= "bb" value="'select * from sys_user where user_password like \'%' + string +'%\''" />
      ${bb}
  </select>
  ```

  如上 value = "   " 中就是个OGNL

- 在它其中可以直接使用**变量**，就是也不用加引号啊，不用括号什么的。在式子中直接就能用（比如 @Param 指定的变量）。

- 它其中要是使用到字符串了，需要引号引起来。

- 然后其中可以用 and、or、==、!=、lt(小于)、lte(小于等于)、gt、gte、!

- 也可以调用方法     list.zise()

- 也可以直接获取属性    user.name

- 也可以按索引取值    arr[6]

- 甚至能调用静态方法和静态字段

  ```xml
  <!-- 看看就好了 -->
  <if test= "@tk.mybatis.util.StringUtil@isNotEmpty(userName)">
  	and user_name like concat('%', #{userName}, '%')
  </ if>
  ```

- 或者更加看看就好了的：这样是能运行，并查询成功的。（BaseMapperTest有个静态方法，它返回一个string）

  ```xml
  <!-- xml文件 -->
  <select id="testBind" resultMap="userMap" >
      <bind name= "bb" value="'select * from sys_user where user_password = '" />
      <bind name="pirnt" value="@top.ziw.simp.mapper.BaseMapperTest@tep()"/>
      ${bb} #{pirnt}
  </select>
  ```

  ```java
  //java中的静态方法
  public static String tep() {
      System.out.println("teteetetp");
      return "123131ff";
  }
  ```

- 更加深刻的理解 # 和 $ ：**$仅仅是拿过来，#是加上引号拿过来**。

- 好像是，#是原地留问号（坑），最后运行时，再用这个内容填坑

  突然发现，竟然能如下这样写！这就意味着$中就是个 OGNL表达式了，就这样吧，不多想了

  ```xml
  <select id="testAll" resultMap="userMap" >
      ${"select * from sys_user"}
  </select>
  ```

## 第五章 代码生成器

### 5.1  配置

- 顶层是`generatorConfiguration`，它其中有三个，分别是

  - properties（0个或者1个）：为了引入配置文件，供后文使用
  - classPathEntry（任意个）
  - context（1个或多个）：核心！重点！

- 上述三个，以及下面谈到的众多标签，必须按照行文顺序来加入配置文件中，顺序乱了，则GG

- context 属性

  | 属性值           |                                                              |
  | ---------------- | ------------------------------------------------------------ |
  | defaultModelType | 定义了MBG如何生成实体类。推荐使用 flat ：一个表生成一个实体类 |
  | targetRuntime    | 默认值：MyBatis3           MyBatis3Simple：不会生成Example   |

- context 的子签如下：

  - property（任意个）
  - plugin（任意个）
  - commentGenerator（0或1个）
  - jdbcConnection（1个）
  - javaTypeResolver（0或1个）
  - javaModelGenerator（1个）
  - sqlMapGenerator（0或1个）
  - javaClientGenerator（0或1个）
  - table（1个或多个）

### 5.1.1 **property 标签**

- 首先介绍分割符的概念：如果在数据库中竟然有用`user base info`这样的表，那sql是没法写的，在mysql中需要\`user base info\`  这样来书写，在SQL Server中需要 [user base info]  这样

  | 属性                | 说明                                                   | 值              |
  | ------------------- | ------------------------------------------------------ | --------------- |
  | autoDelimitKeywords | 如果发现了数据库字段有和关键字一样的，就给它加上分隔符 | 推荐配置为 true |
  | beginningDelimiter  | 分割符前缀                                             | `               |
  | endingDelimiter     | 后缀                                                   | `               |
  | javaFileEncoding    | Java 文件编码。默认是使用当前运行环境的编码            |                 |

### 5.1.2 plugin 标签

- 这是配置插件用的，用到较少

### 5.1.3 commentGenerator

- 这标签主要是为了控制生成代码中的注释信息。

- 它有一个属性 type，可以指定用户自己的类，欲自己定义类，需要实现CommentGenerator，并留一个空参数的构造方法。这个应该用不到........看看好了

- type 有个默认实现类，有三个属性

  - suppressAllComments：阻止生成注释，默认false
  - suppressDate：阻止生成注释包含时间戳，默认 false
  - addRemarkComments：注释是否添加数据库表的信息，默认false

- 默认是会记录很多注释的，这注释没有用，因为总变，干脆就别要注释了，注释也是英文的。。。。。

  ```xml
  <commentGenerator>
      <property name="suppressAllComments" value="true"/>
      <!-- <property name="suppressDate" value="true"/>
       <property name="addRemarkComments" value="true"/> -->
  </commentGenerator>
  ```

### 5.1.4 jdbcConnection

- 这标签很关键，必须有且仅有一个，链接数据库用啊！
- 需要配置它的四个属性，略.....

### 5.1.5  javaTypeResolver

- 它用来配置 JDBC 类型和 JAVA 类型如何转换
- 与上面注释那个一样的道理，可以用 type 来指定一个自己的实现类，或者不配置，用它自己的默认实现类
- 默认实现类有一个 forceBigDecimals ，可以用property标签来配置，它控制数字转 成 long 还是 int 还是 BigDecimal

### 5.1.6  javaModelGenerator

- 它控制生成的实体类

- 它有两个属性

  - targetPackage：生成实体类存放的包名（同时会生成包）
  - targetProject：生成在哪里（这个路径需要从项目根路径开始写）

- 它还有很多需要 property 子标签来配置的属性

  | 属性                | 说明                                                         | 值   |
  | ------------------- | ------------------------------------------------------------ | ---- |
  | constructorBased    | 只对 MyBatis3 有效，true 就会使用构造方法入参，false（默认）就使用get set |      |
  | enableSubPackages   | 如果为 true, MBG 会根据 catalog 、schema 来生成子包。默认false |      |
  | immutable（不可变） | 配置为true后，无论constructorBased为何，都会生成构造方法入参，并且不会有get set方法 |      |
  | rootClass           | 为所有类设置基类，如果MBG能加载这个类的话，它就可以判断从而不覆盖和父类完全匹配的属性 |      |
  | trimStrings         | 默认false，如果是true，set 方法就如下，会去掉空格哦！<br />this.username = username ==null? null : username.trim() ; |      |

### 5.1.7  sqlMapGenerator

- 它控制生成的Mapper.xml 

- 它同样有上述两个属性

- 它还有一个需要 property 子标签来配置的属性

  - enableSubPackages：同上

- **注意：只有当下面这个javaClientGenerator配置为需要构建 xml 时，才会配置本标签（这是显而易见的）。**

  **倘若连 javaClientGenerator 都没有配置的话，系统看是否建立了 sqlMapGenerator，如果有 ，则建立实体类和接口；如果无，则只建立实体类。**

### 5.1.8  javaClientGenerator

- 它控制生成的接口

- 它除了那两个属性外，还有另一个必选属性，可以一个可选属性

  - type（必选）：同样可以自己定义，或者使用默认实现。根据前面targetRuntime配置的不同，这里会出现5种默认实现

    | targetRuntime类型 | 默认实现类      | 说明                                             |
    | ----------------- | --------------- | ------------------------------------------------ |
    | MyBatis3          | ANNOTATEDMAPPER | 基于注解，不会有 XML文件                         |
    |                   | MIXEDMAPPER     | 混合方式（将上面方式的 SQL Provider注解换成XML） |
    |                   | XMLMAPPER       | 正常的 XML 方式                                  |
    | MyBatis3Simple    | ANNOTATEDMAPPER | 基于注解，不会有 XML 文件                        |
    |                   | XMLMAPPER       | 正常的 XML 方式                                  |

  - implementationPackage(可选)：它能让接口出现实现类，不知道干嘛用的，不常用，不管了

### 5.1.9 table

- 负责管理，究竟要为哪些表生成代码
- 它的一个必选属性，是 tableName 指定表名
- 只有配置在这标签内的表，才会被生成代码，全部可以这样：`<table tableName="%" />`  
- 它有很多很多可选属性：
  - schema：
  - alias：设置后，查询语句的每个字段名，都会是：设置的值_字段名
  - domainObjectName：设置后，实体类以及接口的命名都是使用这个名字作为基础名
  - enableXXX ：XXX代表欲生成的方法名。设置后，用于控制这个方法是否要被生成，我看了下 大概有10个
  - modelType：用于单独设置 defaultModelType 属性
  - escapeWildcards：是否对SQL通配符进行转义
  - delimitIdentifiers：是否给标识符增加分隔符（默认就行）
  - delimitAllColumns：是否给所有列增加分隔符
- 它还有很多property子签配置的可选属性
  - constructorBased：和javaModelGenerator中一样
  - ignoreQualifiersAtRuntime：生成SQL中的表名将不包含 schema 和 catalog 前缀
  - immutable：和javaModelGenerator中一样
  - modelOnly：是否只生成实体类
  - rootClass：和javaModelGenerator中一样
  - ......
- 它还有4个子标签
  - generatedKey
  - columnRenamingRule
  - columnOverride：
  - ignoreColumn：可以屏蔽不需要生成的列，可以配置多个。它的属性column配置列名，delimitedColumnName用于配置是否区分大小写

### 5.2 ~ 5.3  基本配置和运行

- 首先还是将一个基本的xml给出吧

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE generatorConfiguration 
  	PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" 
  	"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
  <generatorConfiguration>
    <context id="context1" defaultModelType="flat" targetRuntime="MyBatis3Simple">
    	<property name="beginningDelimiter" value="`"/>
    	<property name="endingDelimiter" value="`"/>
        
      <commentGenerator>
        <property name="suppressDate" value="true"/>
      	<property name="addRemarkComments" value="true"/>
      </commentGenerator>
      
      <jdbcConnection connectionURL="jdbc:mysql://172.26.104.56:3306/test2" 
      			driverClass="com.mysql.jdbc.Driver" password="root" userId="root" />
      
      <javaModelGenerator targetPackage="test.model" targetProject="src\main\java" />
      <sqlMapGenerator targetPackage="test.xml" targetProject="src\main\resources" />
      <javaClientGenerator targetPackage="test.dao" targetProject="src\main\java" 
                           type="XMLMAPPER" />
      
      <table tableName="%"></table>
    </context>
  </generatorConfiguration>
  ```

- 第一种方式是很简单的，在 eclipse 市场里下载 MyBatis Generator 插件。然后随便找个地方 写个xml，然后对着xml 右键 Run As 就能搞定

  - 这个方式，似乎路径要带上 eclipse 的项目名

- 第二种方式是java代码（感觉比较稳）

  - 新建一个 maven 项目，pom如下

    ```xml
    <dependencies>
        <!-- 生成器 -->
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.7</version>
        </dependency>
        <!-- 因为需要链接数据库，所以... -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>
    </dependencies>
    ```

  - 在一个main方法下执行如下，即可！

    ```java
    public static void main(String[] args) throws Exception {
        //MBG执行过程中的警告信息
        List<String> warnings = new ArrayList<String>();
        //当生成代码重复时，覆盖原代码
        boolean overwrite = true;
        InputStream is = Generator.class.getClassLoader()
            .getResourceAsStream("generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is); 
        is.close();
        
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = 
            new MyBatisGenerator(config, callback, warnings);
        
        //执行
        myBatisGenerator.generate(null);
        warnings.forEach(System.out::println);
    }
    ```

### 5.4  Example

## 第六章 高级查询

### 6.1  高级映射结果

- 说高级，其实就是指：实体类中包含实体类。。。

