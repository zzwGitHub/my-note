- # MyBatis从入门到精通

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

  - 接口定义的放回只类型必须和XML中配置的 resulType 一致，或者与 resultMap 中的 type 一致。真正决定返回类型的是 XML！

  - 接口的返回值，可以自行判断是  SysUser 还是 List\<SysUser>，总之XML 中没有这中区分。但是！如果返回值是多个。而接口中是单个。会抛出异常 ： TooManyResyltsException

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

  - 在 sql 语句中，如果把列名 别名为 user.userName 那么 resultType 指向实体类中的 user 属性(user是另外一个实体类)，就能直接被赋值了。这种方法很方便！

  ### insert

  - sql 可以写成

    `insert into sys_user(create_time) values (#{createTime, jdbcType= TIMESTAMP})`

    这是因为数据库区分 date、time、datetime类型，而Java中一般都使用 Date。为了对应准确，最好手动指定一下

  - 由于默认的 `sqlSessionFactory.openSession()`是不自动提交的，因此不手动执行 commit 是不会提交到数据库的。当然也可以执行 `sqlSession.rollback()`进行回滚

  - mybatis可以做出 **增加完成后，为原对象赋值的操作**，当然最常见的就是填充实体类的ID了。做法如下

    ```xml
    <insert id="insert2" useGeneratedKeys="true" keyProperty="id">
    	insert into sys_user(user_name, user_password, user_email,
    	user_info, head_img, create_time)
    	values(
    	#{userName}, #{userPassword}, #{userEmail}, #{userInfo}, #{headImg, jdbcType=BLOB}, #{createTime, jdbcType=TIMESTAMP} )
    </insert>
    ```

    语句中必然是没有ID的；以及加入了两个标签，如果需要操作原实体类的多个类，需要研究 keyColumn 属性值

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

  # 第三章 注解

  - @Select 、@Inser、@Update、@Delete ，很简单，如果用到了，再看吧

  - 是不推荐使用注解的，因为需要重新编译代码呀

  - 对于返回值对应关系呢，可以：

    - sql 取别名和java类对应上
    - mapUnderscoreToCamelCase 配置为true

  - 对于返回值，还是向 resultMap 上想，企图将sql的返回值做成java类。

    使用 @Results 注解。它在 3.3.0 版之后，就能**公用**了(@ResultMap)，居然也可以公用 xml 中配置的.....

  - 还有 @PrivilegeMapper 

  # 第四章 动态 SQL