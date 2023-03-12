# Mybatis笔记
### Mybatis官网
[https://mybatis.org/mybatis-3/zh/index.html]("https://mybatis.org/mybatis-3/zh/index.html")
### maven的pom.xml文件依赖配置
    <!--mysql数据库依赖-->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>
    <!--mongodb数据库依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
    <!--数据库连接JDBC依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jdbc</artifactId>
    </dependency>
    <!--mybatis依赖-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.12</version>
    </dependency>
###  MyBatis 系统的核心配置（resources目录下创建mybatis-config.xml文件）
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>

        <!-- 给层级结构的包配置别名，不区分大小写,默认的别名就是对应的类名，详见：mybatis文档 -->
        <typeAliases>
            <package name="包路径：例如，com.example.demo" >
        </typeAliases>

        <environments default="development">
        <!--可以配置多个environment标签，通过default进行切换-->
            <environment id="development">
                <!-- 事务类型配置 -->
                <transactionManager type="JDBC"/>
                <dataSource type="POOLED">
                    <!--数据库连接信息-->
                    <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://ip:端口/数据库?useSSL=false"/>
                    <property name="username" value="用户名"/>
                    <property name="password" value="连接密码"/>
                </dataSource>
            </environment>
        </environments>

        <mappers>
            <!--加载sql映射文件路径-->
            <mapper resource="UserMapper.xml"/>
        </mappers>

    </configuration>
### Mapper文件编写 （resources目录下创建实体类对应的mapper文件，例如UserMapper.xml文件）
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <!--namespace:名称空间-->
    <mapper namespace="UserMapper">
        <!-- statement -->
        <!--id:sql语句的唯一标识 resultType:返回的数据类型-->
        <select id="selectAll" resultType="entity.users">
            select * from users;
        </select>
    </mapper>
### 构建 SqlSessionFactory 的实例（在controller类中使用时构建）并使用
    // 1. 加载mybatis的核心配置文件，获取SqlSessionFactory
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    // 2. 获取Sqlsession对象，用它来执行sql
    SqlSession sqlSession = sqlSessionFactory.openSession();
    // 3. 执行sql
    List<users> users = sqlSession.selectList("UserMapper.selectAll");
    users.forEach(System.out::println);
    // 4. 释放资源
    sqlSession.close();
### Mapper代理开发（解决原生方式中的硬编码，简化后期执行SQL）
- Mapper代理方式规则
    1. 定义与SQL映射文件同名的Mapper接口，并且将Mapper接口和SQL映射文件放置在同一目录下
    2. 设置SQL映射文件的namespace属性为Mapper接口全限定名
    3. 在Mapper接口中定义方法，方法名就是SQL映射文件中sql语句的id，并保持参数类型和返回值类型一致  
        resources目录下创建包mapper，将UserMapper.xml移动到包内  
        源码同级目录下创建包mapper,创建接口文件UserMapper.java
    4. 编码：
        ```
        // 1. 加载mybatis的核心配置文件，获取SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        // 2. 获取Sqlsession对象，用它来执行sql 传递参数true后自动提交事务
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        // 3. 获取UserMapper接口的代理对象
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        // 4. 代理形式 执行sql
        List<users> users = userMapper.selectAll();
        users.forEach(System.out::println);
        // 如果没有开启事务，执行update类事务语句，需要执行sqlSession.commit()手动提交事务
        // 5. 释放资源
        sqlSession.close();
        ```
    5. Mapper代理开发模式下 mybatis-config.xml文件mapper扫描加载方式  
        ```<mapper resource="mapper/UserMapper.xml"/>```
        替换为
        ```<package name="mapper.UserMapper"/>```
### Mapper接口文件使用注解开发（不推荐：简单功能可使用注解开发，复杂功能建议用xml文件编写）
- 可使用注解
    1. 查询：@Select({"select vale from table"})
    2. 添加：@Insert({"insert into (values) from (values)"})
    3. 修改：@Update({"update set keys = value from table "})
    4. 删除：@Delete({""})
### Mybatis核心配置文件（mybatis-config.xml文件）需要按配置顺序进行配置，详细配置见官方文档
    configuration（配置）
        properties（属性）
        settings（设置）
        typeAliases（类型别名）
        typeHandlers（类型处理器）
        objectFactory（对象工厂）
        plugins（插件）
        environments（环境配置）
        environment（环境变量）
        transactionManager（事务管理器）
        dataSource（数据源）
        databaseIdProvider（数据库厂商标识）
        mappers（映射器）
### MybatisX插件（IDEA中安装）
- 主要功能
    1. xml和接口方法之间跳转
    2. 根据接口方法生成statement
### 数据库字段名与实体类属性名不一致Mybatis解决方法
1. 方法一：在编写xml文件查询语句过程中使用as关键字将查询字段映射为实体类属性名（不推荐）
    ```
    <select id="selectAll" resultType="brand">
        select id, brand_name as brandName, company_name as companyName, ordered, description, status from tb_brand;
    </select>
    ```
2. 方法二：在xml文件中中定义sql片段（不推荐）
    ```
    <sql id="brand_column">
        id, brand_name as brandName, company_name as companyName, ordered, description, status
    </sql>
    <select id="selectAll" resultType="brand">
        select <include refid="brand_column" /> from tb_brand;
    </select>
    ```
3. 方法三：使用resultMap做映射（推荐）
    ```
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName">
        <result column="company_name" property="companyName">
    </ resultMap>
    <select id="selectAll" resultMap="brandResultMap">
        select * from tb_brand;
    </select>
    ```
4. 方法四：使用Mybatis-Plus查询（只适合做单表查询）
### Mybatis传参
#### 一、参数占位符：
1. #{}：会将其替换为 ?,为了防止SQL注入（推荐）
2. ${}：直接将参数拼接到sql语句中,会存在SQL注入风险（不推荐）
3. 使用时机：
    - 参数传递的时候都使用：#{}
    - 表名或者列名不固定的情况下使用：${} 会有SQL注入风险
#### 二、使用方法：
- 第一步：接口方法定义形参```List<brand> selectById(Integer id)```
- 第二步：xml文件中使用 ```#{id}```接受参数
- 小细节1：xml标签中的参数类型属性parameterType一般都省略不写
- 小细节2：特殊字符处理
    1. 方式1：使用转义字符，比如：< 符号可以用 &lt 表示
    2. 方式2：使用CDATA区，例如：< 使用 ```<![CDATA[ < ]]>```包裹
### Mybatis条件查询（重点）
#### 一、多条件查询
1. 编写接口方法：Mapper接口
    - 参数：所有查询条件
    - 结果：```List<Brand>```
2. 编写SQL语句：xml文件中的select语句
    ```
    // Mapper.xml代码示例
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName">
        <result column="company_name" property="companyName">
    </resultMap>
    <select id="selectByCondition" resultMapp="brandResultMap">
        select *
        from tb_brand
        where status = #{status}
            and company_name like #{companyName}    // 这里注意参数尽量使用实体类属性名
            and brand_name like #{brandName};
    </select>
    ```
3. 参数预处理
    ```
    String companyName = "%" + companyName + "%"
    String brandName = "%" + brandName + "%"
    ```
4. 使用@Param注解参数传递映射关系：@Param("status") => #{status}（散装多参数传递）
    ```
    // Mapper接口代码示例
    List<Brand> selectByCondition(@Param("status")Integer status,@Param("companyName")String companyName,@Param("brandName")String brandName)
    ```
5. 使用对象传参：对象中的属性名称与条件字段名称相同，自动调用对象get方法获取参数
    ```
    // Mapper接口代码示例
    List<Brand> selectByCondition(Brand brand)
    ```
6. 使用Map集合传参：Map集合key的名称与条件字段名称相同
    ```
    // Mapper接口代码示例 map={status=status,companyName=companyName,brandName=brandName}
    List<Brand> selectByCondition(Map map)
    ```
#### 一、Mybatis多条件动态查询（动态SQL）
1. if使用动态条件查询
    ```
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName">
        <result column="company_name" property="companyName">
    </resultMap>
    <select id="selectByCondition" resultMapp="brandResultMap">
        select *
        from tb_brand
        where 
            <if test="status != null">
                status = #{status}
            </if>
            <if test="companyName != null and companyName != '' ">
                // 这里有一个bug,当status为null时会报错
                and company_name like #{companyName}
            </if>
            <if test="brandName != null and brandName != '' ">
                and brand_name like #{brandName};
            </if>
    </select>
    ```
    - 上方代码bug解决方案：
        1. 使用恒等式：1 = 1（可以使用，不太建议）
            ```
            <resultMap id="brandResultMap" type="brand">
                <result column="brand_name" property="brandName">
                <result column="company_name" property="companyName">
            </resultMap>
            <select id="selectByCondition" resultMapp="brandResultMap">
                select *
                from tb_brand
                where 1=1   // 在这里添加一个恒等式
                    <if test="status != null">
                        and status = #{status}  // 这里添加and
                    </if>
                    <if test="companyName != null and companyName != '' ">
                        and company_name like #{companyName}
                    </if>
                    <if test="brandName != null and brandName != '' ">
                        and brand_name like #{brandName};
                    </if>
            </select>
            ```
        2. 使用Mybatis的where标签（推荐）
            ```
            <resultMap id="brandResultMap" type="brand">
                <result column="brand_name" property="brandName">
                <result column="company_name" property="companyName">
            </resultMap>
            <select id="selectByCondition" resultMapp="brandResultMap">
                select *
                from tb_brand
                <where> // where关键字替换为where标签可修复bug
                    <if test="status != null">
                        status = #{status}
                    </if>
                    <if test="companyName != null and companyName != '' ">
                        and company_name like #{companyName}
                    </if>
                    <if test="brandName != null and brandName != '' ">
                        and brand_name like #{brandName}
                    </if>
                </where>
            </select>
            ```
#### 一、Mybatis单条件动态查询（动态SQL）
- 从多个条件中选择一个进行查询  
choose:类似switch  
when:类似case  
otherwise:类似default  
- 示例代码：
    ```
    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName">
        <result column="company_name" property="companyName">
    </resultMap>
    <select id="selectByCondition" resultMapp="brandResultMap">
        select *
        from tb_brand
        <where>
            <choose> <!-- 相当于switch -->
            <when test="status != null"> <!-- 相当于case -->
                status = #{status}
            </when>
            <when test="companyName != null and companyName != '' ">
                company_name like #{companyName}
            </when>
            <when test="brandName != null and brandName != '' ">
                brand_name like #{brandName}
            </when>
            // 使用了where可以不用使用otherwise，使用where关键字otherwise才有用
            <otherwise> <!-- 相当于default -->
                1=1
            </otherwise>
        </where>
    </select>
    ```
### Mybatis添加操作
#### 一、基础添加操作
1. Mapper接口方法:```void add(Brand brand)```
2. statement语句
    ```
    <insert id="add">
        insert into tb_brand (brand_name,company_name,ordered,description,status)
        values (#{brandName},#{companyName},#{ordered},#{description},#{status});
    </insert>
    ```
#### 一、添加数据后主键返回
1. Mapper接口方法:```Brand add(Brand brand)```
2. statement语句
    ```
    // useGeneratedKeys开启主键返回true keyProperty绑定返回属性名（实体对象属性名）
    <insert id="add" useGeneratedKeys="true" keyProperty="id">
        insert into tb_brand (brand_name,company_name,ordered,description,status)
        values (#{brandName},#{companyName},#{ordered},#{description},#{status});
    </insert>
    ```
### Mybatis修改操作
#### 一、修改全部字段
1. Mapper接口方法
    ```
    Integer update(Brand brand)
    ```
2. statement语句
    ```
    <update id="update">
        update tb_brand
        set
            brand_name = #{brandName},
            company_name = #{companyName},
            ordered = #{ordered},
            description = #{description},
            status = #{status}
        where id = #{id};
    </update>
    ```
#### 一、动态修改部分字段
1. Mapper接口方法
    ```
    Integer update(Brand brand)
    ```
2. statement语句
    ```
    <update id="update">
        update tb_brand
        <set>
            <if test="brandName != null and brandName != '' ">
                brand_name = #{brandName},
            </if>
            <if test="companyName != null and companyName != '' ">
                company_name = #{companyName},
            </if>
            <if test="ordered != null">
                ordered = #{ordered},
            </if>
            <if test="description != null and description != '' ">
                description = #{description},
            </if>
            <if test="status != null">
                status = #{status}
            </if>
        </set>
        where id = #{id};
    </update>
    ```
### Mybatis删除操作
#### 一、删除单个元组数据
1. Mapper接口方法
    ```
    void deleteById(Integer id)
    ```
2. statement语句
    ```
    <delete id="deleteById">
        delete from tb_brand where id = #{id};
    </delete>
    ```
#### 二、删除多个元组数据
1. Mapper接口方法
    ```
    void deleteByIds(@Param("ids")Integer[] ids)
    ```
2. statement语句批量删除，使用foreach标签循环获取id，separator：分隔符，open：开始字符，close：结束字符
    ```
    <delete id="deleteByIds">
        delete from tb_brand 
        where id = in
        <foreach collection="ids" item="id" separator="," open="(" close=")">
            #{id}
        </foreach>
    </del>
    ```
### Mybatis接口参数如何封装传递到xml的statement中（重点）
- 单个参数：  
    1. POJO类型:直接使用，属性名 和 参数占位符名称一致
    2. Map集合：直接使用，键名 和 参数占位符名称一致
    3. Collection:封装为Map集合，可以使用@Param注解替换Map集合中默认arg键名
        ```
        map.put("arg0",collection集合);
        map.put("collection",collection集合);
        ```
    4. List:封装为Map集合，可以使用@Param注解替换Map集合中默认arg键名
        ```
        map.put("arg0",list集合);
        map.put("collection",list集合);
        map.put("list",list集合);
        ```
    5. Array:封装为Map集合，可以使用@Param注解替换Map集合中默认arg键名
        ```
        map.put("arg0",数组);
        map.put("array",数组);
        ```
    6. 其他类型：直接使用，任意占位符名称都可接受到参数```User select(int id);```
- 多个参数：自动封装为Map集合，可以使用@Param注解替换Map集合中默认arg键名
    1. 默认键名 参数封装(不使用@Param注解时)
        ```
        User select(String username, String password);
        map.put("arg0",username);
        map.put("param0",username);
        map.put("arg1",password);
        map.put("param1",password);
        ```
    2. 使用@Param封装参数时
        ```
        User select(@Param("name")String username, String password);
        map.put("name",username);
        map.put("param0",username);
        map.put("arg1",password);
        map.put("param1",password);
        ```