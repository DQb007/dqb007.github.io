---
layout: post
title: "MyBatis 及 MyBatis-Plus的使用"
description: "MyBatis 及 MyBatis-Plus的使用"
categories: [后端技术]
tags: [后端技术, mybatis]
---

#  mybatis plus常用语法专题
> ## 一、mybatis
- ### 1.什么是 MyBatis ？
    ```
    官方定义：
    MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。
    ```
- ### 2.SQL 语句映射的方式
    Mapper XML 文件 定义(****★)
    ```
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="org.mybatis.example.BlogMapper">
      <select id="selectBlog" resultType="Blog">
        select * from Blog where id = #{id}
      </select>
    </mapper>
    ```
    java注解定义
    ```
    package org.mybatis.example;
    public interface BlogMapper {
      @Select("SELECT * FROM blog WHERE id = #{id}")
      Blog selectBlog(int id);
    }
    ```
    对于简单语句来说，注解使代码显得更加简洁，然而 Java 注解对于稍微复杂的语句就会力不从心并且会显得更加混乱。==因此，如果你需要做很复杂的事情，那么最好使用 XML 来映射语句==。
    - [ ] **2.1.Mapper XML 文件**
        ##### select
        查询语句是 MyBatis 中最常用的元素之一，光能把数据存到数据库中价值并不大，如果还能重新取出来才有用，多数应用也都是查询比修改要频繁。对每个插入、更新或删除操作，通常对应多个查询操作。这是 MyBatis 的基本原则之一，也是将焦点和努力放到查询和结果映射的原因。简单查询的 select 元素是非常简单的。
        
        parameterType:参数类型，只能传一个参数，如果有多个参数要封装，如封装成一个类，要写包名加类名，基本数据类型则可以省略
        ```
        <select id="selectPerson" parameterType="int" resultType="hashmap">
          SELECT * FROM PERSON WHERE ID = #{id}
        </select>
        
        <!--常用查询条件-->
        <select id="getByQO" parameterType="com.hand.hcf.app.base.company.dto.CompanyQO" resultType="com.hand.hcf.app.base.company.domain.Company">
            select *
            from sys_company t
            where 1 = 1
            <if test="companyOid != null ">
                and t.company_oid = #{companyOid}
            </if>
            <if test="tenantId != null ">
                and t.tenant_id = #{tenantId}
            </if>
            <if test="setOfBooksId != null">
                and t.set_of_books_id = #{setOfBooksId}
            </if>
            <if test="legalEntityId != null ">
                and t.legal_entity_id = #{legalEntityId}
            </if>
            <if test="parentCompanyId != null ">
                and t.parent_company_id = #{parentCompanyId}
            </if>
            <if test="companyCode != null and companyCode != ''">
                and t.company_code like concat(concat('%',#{companyCode}),'%')
            </if>
            <if test="name != '' and name != null">
                and t.name like concat(concat('%',#{name}),'%')
            </if>
            order by t.company_code
        </select>
        ```
        #####  insert，update 和 delete 语句的示例
        ```
        <insert id="insertAuthor">
          insert into Author (id,username,password,email,bio)
          values (#{id},#{username},#{password},#{email},#{bio})
        </insert>
        <update id="updateAuthor">
          update Author set
            username = #{username},
            password = #{password},
            email = #{email},
            bio = #{bio}
          where id = #{id}
        </update>
        <delete id="deleteAuthor">
          delete from Author where id = #{id}
        </delete>
        ```
        ##### sql
        这个元素可以被用来定义可重用的 SQL 代码段，可以包含在其他语句中。它可以被静态地(在加载参数) 参数化. 不同的属性值通过包含的实例变化
        ```
        <sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
        <select id="selectUsers" resultType="map">
          select
            <include refid="userColumns"><property name="alias" value="t1"/></include>,
            <include refid="userColumns"><property name="alias" value="t2"/></include>
          from some_table t1
            cross join some_table t2
        </select>
        ```
        ##### resultType
        用resultType的时候，要保证结果集的列名与java对象的属性相同
        ```
        <select id="selectUsers" parameterType="int" resultType="com.hand.hcf.app.base.user.domain.User">
          select
            user_id             as "id",
            user_name           as "userName",
            hashed_password     as "hashedPassword"
          from some_table
          where id = #{id}
        </select>
        ```
        ##### resultMap
        resultMap 元素是 MyBatis 中最重要最强大的元素。resultMap 的设计思想是，简单的语句不需要明确的结果映射，而复杂一点的语句只需要描述它们的关系就行了。
        通过里面的id标签和result标签来建立映射关系，由property和column分别指定实体类属性和数据表的列名。
        ```
        <resultMap id="UserLoginBindResultMap"
               type="com.hand.hcf.app.base.user.domain.UserLoginBind">
            <!-- 用id属性来映射主键字段 -->
            <id column="id" property="id"/>
            <!-- 用result属性来映射非主键字段 -->
            <result column="user_oid" property="userOid"/>
            <result column="login" property="login"/>
            <result column="bind_type" property="bindType"/>
        </resultMap>
        <select id="findOneByUserOid" resultMap="UserLoginBindResultMap">
           SELECT
        	  u.id,
              u.user_oid,
              u.bind_type,
              u.created_by,
              u.created_date
              FROM
             	sys_user_login_bind u
             WHERE
                 u.user_oid = #{userOid}
              and u.is_active=1
              and u.enabled=1
              and  u.deleted=0
        </select>
        <!--关联查询Map,带父部门，子部门集合-->
        <resultMap id="DepartmentAssociationResultMap"
                   type="com.hand.hcf.app.base.department.domain.Department">
            <id column="id" property="id"/>
            <result column="department_oid" property="departmentOid"/>
            <result column="name" property="name"/>
            <result column="path" property="path"/>
            <result column="status" property="status"/>
            <result column="manager_id" property="managerId"/>
            <result column="department_code" property="departmentCode"/>
            <result column="tenant_id" property="tenantId"/>
            <result column="data_source" property="dataSource"/>
            <association property="parent" column="parent_id" javaType="com.hand.hcf.app.base.department.domain.Department"
                         select="selectParentResult"/>
            <collection property="children" column="id" ofType="com.hand.hcf.app.base.department.domain.Department"
                        select="selectChildrenResult"/>
    
        </resultMap>
        <!--//父部门对象-->
        <select id="selectParentResult" parameterType="long" resultMap="DepartmentResultMap">
            SELECT * from sys_department where id = #{id}
        </select>
        <!--//子部门集合-->
        <select id="selectChildrenResult" parameterType="long" resultMap="DepartmentResultMap">
            SELECT * from sys_department where parent_id = #{id}
        </select>
        ```
    - [ ] **2.2.动态 SQL**
    
        MyBatis 的强大特性之一便是它的动态 SQL。
        动态 SQL 元素和 JSTL 或基于类似 XML 的文本处理器相似。
        MyBatis 采用功能强大的基于 OGNL 的表达式来淘汰其它大部分元素。
        动态sql元素|
        ---|
         if |
        choose (when, otherwise) |
        trim (where, set)|
        foreach|
        ##### if
        动态 SQL 通常要做的事情是根据条件包含 where 子句的一部分.
        ```
        <select id="findActiveBlogWithTitleLike"
             resultType="Blog">
          SELECT * FROM BLOG 
          WHERE state = ‘ACTIVE’ 
          <if test="title != null">
            AND title like #{title}
          </if>
          <if test="author != null and author.name != null">
            AND author_name like #{author.name}
          </if>
        </select>
        ```
        ##### choose (when, otherwise)
        有时我们不想应用到所有的条件语句，而只想从中择其一项。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。 
        ```
        <select id="findActiveBlogLike"
             resultType="Blog">
          SELECT * FROM BLOG WHERE state = ‘ACTIVE’
          <choose>
            <when test="title != null">
              AND title like #{title}
            </when>
            <when test="author != null and author.name != null">
              AND author_name like #{author.name}
            </when>
            <otherwise>
              AND featured = 1
            </otherwise>
          </choose>
        </select>
        ```
        ##### where
        ```
        <select id="findActiveBlogLike"
             resultType="Blog">
          SELECT * FROM BLOG 
          <where> 
            <if test="state != null">
                 state = #{state}
            </if> 
            <if test="title != null">
                AND title like #{title}
            </if>
            <if test="author != null and author.name != null">
                AND author_name like #{author.name}
            </if>
          </where>
        </select>
        ```
        ##### foreach
        动态 SQL 的另外一个常用的操作需求是对一个集合进行遍历，通常是在构建 IN 条件语句的时候。
        ```
        <select id="selectPostIn" resultType="domain.blog.Post">
          SELECT *
          FROM POST P
          WHERE ID in
          <foreach item="item" index="index" collection="list"
              open="(" separator="," close=")">
                #{item}
          </foreach>
        </select>
        <select id="selectPostIn" resultType="domain.blog.Post">
          SELECT *
          FROM POST P
          <where>
              <if test="list != null and list.size() > 0">
               ID in
                  <foreach item="item" index="index" collection="list"
                      open="(" separator="," close=")">
                        #{item}
                  </foreach>
              </if>
          </where>
        </select>
        ```    
> ## 二、mybatis plus2.0
- ### 0.简介
    MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。
    愿景是成为 MyBatis 最好的搭档，就像 魂斗罗 中的 1P、2P，基友搭配，效率翻倍。
- ### 1.通用 CRUD
    - [ ] **1.1.注解说明**
        ##### 表名注解 @TableName
        值 | 描述
        ---|---
        value | 表名（ 默认空 ）
        resultMap | xml 字段映射 resultMap ID
        ```
        @TableName("sys_user")
        @TableName(value = "sys_user")
        ```
        ##### 主键注解 @TableId
        值 | 描述
        ---|---
        value | 字段值（驼峰命名方式，该值可无）
        type  |	主键 ID 策略类型（ 默认 INPUT ，全局开启的是 ID_WORKER ）
        ```
        @TableId(value = "id")
        private Long id;
        ```
        ##### 字段注解 @TableField
        值 | 描述
        ---|---
        value|	字段值（驼峰命名方式，该值可无）
        update|	预处理 set 字段自定义注入
        condition|	预处理 WHERE 实体条件自定义运算规则
        el|	详看注释说明
        exist|	是否为数据库表字段（ 默认 true 存在，false 不存在 ）
        strategy|	字段验证 （ 默认 非 null 判断，查看 com.baomidou.mybatisplus.enums.FieldStrategy ）
        fill|	字段填充标记 （ FieldFill, 配合自动填充使用 ） 实体条件自定义运算规则
        ```
        @TableField(value="created_date",fill= FieldFill.INSERT)
        protected ZonedDateTime createdDate;
        DEFAULT 默认不处理
        INSERT 插入填充字段
        UPDATE 更新填充字段
        INSERT_UPDATE 插入和更新填充字段
        
        @TableField(exist = false)
        private Level level;
        
        @TableField(value="last_updated_date", fill = FieldFill.INSERT_UPDATE, update="NOW()")
        protected ZonedDateTime lastUpdatedDate;
        例如：@TableField(.. , update="now()") 使用数据库时间
        输出 SQL 为：update 表 set 字段=now() where ...
        
        @Version
        @TableField(value="version_number", fill = FieldFill.INSERT, update="%s+1")
        protected Integer versionNumber;
        例如：@TableField(.. , update="%s+1") 其中 %s 会填充为字段
        输出 SQL 为：update 表 set 字段=字段+1 where ...
        
        @TableField(condition = SqlCondition.LIKE)
        private String name;
        输出 SQL 为：select 表 where name LIKE CONCAT('%',值,'%')
        ```
        ##### 序列主键策略 注解 @KeySequence
        值 | 描述
        ---|---
        value|	序列名
        clazz|	id的类型
        ##### 乐观锁标记注解 @Version
        乐观锁是想要在数据库层面加锁控制并发
        ```
        @Version
        protected Integer versionNumber;
        ```
- ### 2.条件构造器
    实体包装器，用于处理 sql 拼接，排序，实体参数查询等！
    实体包装器 EntityWrapper 继承 Wrapper
    - [ ] **2.1.简单示例**
        ##### 翻页查询
        ```
        public Page<T> selectPage(Page<T> page, EntityWrapper<T> entityWrapper) {
          if (null != entityWrapper) {
              entityWrapper.orderBy(page.getOrderByField(), page.isAsc());
          }
          page.setRecords(baseMapper.selectPage(page, entityWrapper));
          return page;
        }
        ```
        ##### 拼接 sql 方式 一
        ```
        @Test
        public void testTSQL11() {
            /*
             * 实体带查询使用方法  输出看结果
             */
            EntityWrapper<User> ew = new EntityWrapper<User>();
            ew.setEntity(new User(1));
            ew.where("user_name={0}", "'zhangsan'").and("id=1")
                    .orNew("user_status={0}", "0").or("status=1")
                    .notLike("user_nickname", "notvalue")
                    .andNew("new=xx").like("hhh", "ddd")
                    .andNew("pwd=11").isNotNull("n1,n2").isNull("n3")
                    .groupBy("x1").groupBy("x2,x3")
                    .having("x1=11").having("x3=433")
                    .orderBy("dd").orderBy("d1,d2");
            System.out.println(ew.getSqlSegment());
        }
        ```
        ##### 拼接 sql 方式 二
        ```
        int buyCount = selectCount(Condition.create()
                .setSqlSelect("sum(quantity)")
                .isNull("order_id")
                .eq("user_id", 1)
                .eq("type", 1)
                .in("status", new Integer[]{0, 1})
                .eq("product_id", 1)
                .between("created_time", startDate, currentDate)
                .eq("weal", 1));
        ```
        ##### 自定义 SQL 方法如何使用 Wrapper
         ###### mapper java 接口方法
         ```
         List<User> selectMyPage(RowBounds rowBounds, @Param("ew") Wrapper<T> wrapper);
         ```
         ###### mapper xml 定义
         ```
         <select id="selectMyPage" resultType="User">
          SELECT * FROM user 
          <where>
          ${ew.sqlSegment}
          </where>
        </select>
        ```
    - [ ] **2.2.条件参数说明**
        查询方式|	说明
         ---|---
        setSqlSelect|	设置 SELECT 查询字段
        where|	WHERE 语句，拼接 + WHERE 条件
        and|	AND 语句，拼接 + AND 字段=值
        andNew|	AND 语句，拼接 + AND (字段=值)
        or|	OR 语句，拼接 + OR 字段=值
        orNew|	OR 语句，拼接 + OR (字段=值)
        eq|	等于=
        allEq|	基于 map 内容等于=
        ne|	不等于<>
        gt|	大于>
        ge|	大于等于>=
        lt|	小于<
        le|	小于等于<=
        like|	模糊查询 LIKE
        notLike|	模糊查询 NOT LIKE
        in|	IN 查询
        notIn|	NOT IN 查询
        isNull|	NULL 值查询
        isNotNull|	IS NOT NULL
        groupBy|	分组 GROUP BY
        having|	HAVING 关键词
        orderBy|	排序 ORDER BY
        orderAsc|	ASC 排序 ORDER BY
        orderDesc|	DESC 排序 ORDER BY
        exists|	EXISTS 条件语句
        notExists|	NOT EXISTS 条件语句
        between|	BETWEEN 条件语句
        notBetween|	NOT BETWEEN 条件语句
        addFilter|	自由拼接 SQL
        last|	拼接在最后，例如：last("LIMIT 1")
        ==注意！ xxNew 都是另起 ( ... ) 括号包裹。==
    - [ ] **2.3.分页**
        ##### UserMapper.java 方法内容
        ```
        public interface UserMapper{//可以继承或者不继承BaseMapper
            /**
             * <p>
             * 查询 : 根据state状态查询用户列表，分页显示
             * </p>
             *
             * @param page
             *            翻页对象，可以作为 xml 参数直接使用，传递参数 Page 即自动分页
             * @param state
             *            状态
             * @return
             */
            List<User> selectUserList(Pagination page, Integer state);
        }
        ```
        ##### UserServiceImpl.java 调用翻页方法，需要 page.setRecords 回传给页面
        ```
        public Page<User> selectUserPage(Page<User> page, Integer state) {
            // 不进行 count sql 优化，解决 MP 无法自动优化 SQL 问题
            // page.setOptimizeCountSql(false);
            // 不查询总记录数
            // page.setSearchCount(false);
            // 注意！！ 分页 total 是经过插件自动 回写 到传入 page 对象
            return page.setRecords(userMapper.selectUserList(page, state));
        }
        ```
        ##### UserMapper.xml 等同于编写一个普通 list 查询，mybatis-plus 自动替你分页
        ```
        <select id="selectUserList" resultType="User">
            SELECT * FROM user WHERE state=#{state}
        </select>
        ```

> ## 三、mybatis plus3.0
- ### 1.CRUD 接口
    - [ ] **1.1. Mapper CRUD 接口**
        ```
        说明:
            通用CRUD 封装BaseMapper接口，为 Mybatis-Plus 启动时自动解析实体表关系映射转换为 Mybatis 内部对象注入容器
            泛型 T 为任意实体对象
            参数 Serializable 为任意类型主键 Mybatis-Plus 不推荐使用复合主键约定每一张表都有自己的唯一 id 主键
            对象 Wrapper 为 条件构造器
        ```
        ##### insert
        ```
        /**
         * <p>
         * 插入一条记录
         * </p>
         *
         * @param entity 实体对象
         * @return 插入成功记录数
         */
        int insert(T entity);
        ```
        ##### deleteById
        ```
        /**
         * <p>
         * 根据 ID 删除
         * </p>
         *
         * @param id 主键ID
         * @return 删除成功记录数
         */
        int deleteById(Serializable id);
        ```
        ##### deleteByMap
        ```
        /**
         * <p>
         * 根据 columnMap 条件，删除记录
         * </p>
         *
         * @param columnMap 表字段 map 对象
         * @return 删除成功记录数
         */
        int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
        ```
        ##### delete
        ```
        /**
         * <p>
         * 根据 entity 条件，删除记录
         * </p>
         *
         * @param wrapper 实体对象封装操作类（可以为 null）
         * @return 删除成功记录数
         */
        int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
        ```
        ##### deleteBatchIds
        ```
        /**
         * <p>
         * 删除（根据ID 批量删除）
         * </p>
         *
         * @param idList 主键ID列表(不能为 null 以及 empty)
         * @return 删除成功记录数
         */
        int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
        ```
        ##### updateById
        updateById方法,更新不了空字符串
        
        IGNORED(0, "忽略判断"),
        NOT_NULL(1, "非 NULL 判断"),
        NOT_EMPTY(2, "非空判断");
        
        字段验证策略默认是 NOT_NULL
        
        @TableField(value = "price_unit", strategy = FieldStrategy.IGNORED)
        private String priceUnit;
        
        ```
        /**
         * <p>
         * 根据 ID 修改
         * </p>
         *
         * @param entity 实体对象
         * @return 修改成功记录数
         */
        int updateById(@Param(Constants.ENTITY) T entity);
        ```
        ##### update
        ```
        /**
         * <p>
         * 根据 whereEntity 条件，更新记录
         * </p>
         *
         * @param entity        实体对象 (set 条件值,可为 null)
         * @param updateWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
         * @return 修改成功记录数
         */
        int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
        ```
        ##### selectById
        ```
        /**
         * <p>
         * 根据 ID 查询
         * </p>
         *
         * @param id 主键ID
         * @return 实体
         */
        T selectById(Serializable id);
        ```
        ##### selectBatchIds
        ```
        /**
         * <p>
         * 查询（根据ID 批量查询）
         * </p>
         *
         * @param idList 主键ID列表(不能为 null 以及 empty)
         * @return 实体集合
         */
        List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
        ```
        ##### selectByMap
        ```
        /**
         * <p>
         * 查询（根据 columnMap 条件）
         * </p>
         *
         * @param columnMap 表字段 map 对象
         * @return 实体集合
         */
        List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
        ```
        ##### selectOne
        ```
        /**
         * <p>
         * 根据 entity 条件，查询一条记录
         * </p>
         *
         * @param queryWrapper 实体对象
         * @return 实体
         */
        T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
        ```
        ##### selectCount
        ```
        /**
         * <p>
         * 根据 Wrapper 条件，查询总记录数
         * </p>
         *
         * @param queryWrapper 实体对象
         * @return 满足条件记录数
         */
        Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
        ```
        ##### selectList
        ```
        /**
         * <p>
         * 根据 entity 条件，查询全部记录
         * </p>
         *
         * @param queryWrapper 实体对象封装操作类（可以为 null）
         * @return 实体集合
         */
        List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
        ```
        ##### selectMaps
        ```
        /**
         * <p>
         * 根据 Wrapper 条件，查询全部记录
         * </p>
         *
         * @param queryWrapper 实体对象封装操作类（可以为 null）
         * @return 字段映射对象 Map 集合
         */
        List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
        ```
        ##### selectObjs
        ```
        /**
         * <p>
         * 根据 Wrapper 条件，查询全部记录
         * 注意： 只返回第一个字段的值
         * </p>
         *
         * @param queryWrapper 实体对象封装操作类（可以为 null）
         * @return 字段映射对象集合
         */
        List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
        ```
        ##### selectPage
        ```
        /**
         * <p>
         * 根据 entity 条件，查询全部记录（并翻页）
         * </p>
         *
         * @param page         分页查询条件（可以为 RowBounds.DEFAULT）
         * @param queryWrapper 实体对象封装操作类（可以为 null）
         * @return 实体分页对象
         */
        IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
        ```
        ##### selectMapsPage
        ```
        /**
         * <p>
         * 根据 Wrapper 条件，查询全部记录（并翻页）
         * </p>
         *
         * @param page         分页查询条件
         * @param queryWrapper 实体对象封装操作类
         * @return 字段映射对象 Map 分页对象
         */
        IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
        ```
    - [ ] **1.2. Service CRUD 接口**
        ```
        说明:
            |通用 Service CRUD 封装IService接口，进一步封装 CRUD 采用 get 查询单行 remove 删除 list 查询集合 page 分页 前缀命名方式区分 Mapper 层避免混淆，
            |泛型 T 为任意实体对象
            |建议如果存在自定义通用 Service 方法的可能，请创建自己的 IBaseService 继承 Mybatis-Plus 提供的基类
            |对象 Wrapper 为 条件构造器
        ```
        ##### save
        ```
        /**
         * <p>
         * 插入一条记录（选择字段，策略插入）
         * </p>
         *
         * @param entity 实体对象
         */
        boolean save(T entity);
        ```
        ##### saveBatch
        ```
        /**
         * <p>
         * 批量插入记录
         * </p>
         *
         * @param entityList 实体对象集合
         */
        boolean saveBatch(Collection<T> entityList);
        ```
        ##### saveOrUpdateBatch
        ```
        /**
         * <p>
         * 批量修改插入
         * </p>
         *
         * @param entityList 实体对象集合
         */
        boolean saveOrUpdateBatch(Collection<T> entityList);
        ```
        ##### saveOrUpdateBatch
        ```
        /**
         * <p>
         * 批量修改插入
         * </p>
         *
         * @param entityList 实体对象集合
         * @param batchSize  每次的数量
         */
        boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
        ```
        ##### removeById
        ```
        /**
         * <p>
         * 根据 ID 删除
         * </p>
         *
         * @param id 主键ID
         */
        boolean removeById(Serializable id);
        ```
        ##### removeByMap
        ```
        /**
         * <p>
         * 根据 columnMap 条件，删除记录
         * </p>
         *
         * @param columnMap 表字段 map 对象
         */
        boolean removeByMap(Map<String, Object> columnMap);
        ```
        ##### remove
        /**
         * <p>
         * 根据 entity 条件，删除记录
         * </p>
         *
         * @param queryWrapper 实体包装类 {@link com.baomidou.mybatisplus.core.conditions.query.QueryWrapper}
         */
        boolean remove(Wrapper<T> queryWrapper);
        ##### removeByIds
        ```
        /**
         * <p>
         * 删除（根据ID 批量删除）
         * </p>
         *
         * @param idList 主键ID列表
         */
        boolean removeByIds(Collection<? extends Serializable> idList);
        ```
        ##### updateById
        ```
        /**
         * <p>
         * 根据 ID 选择修改
         * </p>
         *
         * @param entity 实体对象
         */
        boolean updateById(T entity);
        ```
        ##### update
        ```
        /**
         * <p>
         * 根据 whereEntity 条件，更新记录
         * </p>
         *
         * @param entity        实体对象
         * @param updateWrapper 实体对象封装操作类 {@link com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper}
         */
        boolean update(T entity, Wrapper<T> updateWrapper);
        ```
        ##### updateBatchById
        ```
        /**
         * <p>
         * 根据ID 批量更新
         * </p>
         *
         * @param entityList 实体对象集合
         * @param batchSize  更新批次数量
         */
        boolean updateBatchById(Collection<T> entityList, int batchSize);
        ```
        ##### saveOrUpdate
        ```
        /**
         * <p>
         * TableId 注解存在更新记录，否插入一条记录
         * </p>
         *
         * @param entity 实体对象
         */
        boolean saveOrUpdate(T entity);
        ```
        ##### getById
        ```
        /**
         * <p>
         * 根据 ID 查询
         * </p>
         *
         * @param id 主键ID
         */
        T getById(Serializable id);
        ```
        ##### listByIds
        ```
        /**
         * <p>
         * 查询（根据ID 批量查询）
         * </p>
         *
         * @param idList 主键ID列表
         */
        Collection<T> listByIds(Collection<? extends Serializable> idList);
        ``` 
- ### 2.条件构造器
    ```
    说明:
        以下出现的第一个入参boolean condition表示该条件是否加入最后生成的sql中
        以下代码块内的多个方法均为从上往下补全个别boolean类型的入参,默认为true
        以下出现的泛型This均为具体使用的Wrapper的实例
        以下方法在入参中出现的R为泛型,在普通wrapper中是String,在LambdaWrapper中是函数(例:Entity::getId,Entity为实体类,getId为字段id的getMethod)
        以下方法入参中的R column均表示数据库字段,当R为String时则为数据库字段名(字段名是数据库关键字的自己用转义符包裹!)!而不是实体类数据字段名!!!
        以下举例均为使用普通wrapper,入参为Map和List的均以json形式表现!
        使用中如果入参的Map或者List为空,则不会加入最后生成的sql中!!!
        有任何疑问就点开源码看,看不懂函数的点击我学习新知识
    ```
    - [ ] **2.1. AbstractWrapper**
        ```
        说明:
            QueryWrapper(LambdaQueryWrapper) 和 UpdateWrapper(LambdaUpdateWrapper) 的父类
            用于生成 sql 的 where 条件, entity 属性也用于生成 sql 的 where 条件
            注意: entity 生成的 where 条件与 使用各个 api 生成的 where 条件没有任何关联行为
        ```
        ##### eq(等于 =)
        ```
        eq
        eq(R column, Object val)
        eq(boolean condition, R column, Object val)
        例: eq("name", "老王")--->name = '老王'
        ```
        ##### ne(不等于 <>)
        ```
        ne(R column, Object val)
        ne(boolean condition, R column, Object val)
        例: ne("name", "老王")--->name <> '老王'
        ```
        ##### gt(大于 >)
        ```
        gt(R column, Object val)
        gt(boolean condition, R column, Object val)
        例: gt("age", 18)--->age > 18
        ```
        ##### ge(大于等于 >=)
        ```
        ge(R column, Object val)
        ge(boolean condition, R column, Object val)
        例: ge("age", 18)--->age >= 18
        ```
        ##### lt(小于 <)
        ```
        lt(R column, Object val)
        lt(boolean condition, R column, Object val)
        例: lt("age", 18)--->age < 18
        ```
        ##### le(小于等于 <=)
        ```
        le(R column, Object val)
        le(boolean condition, R column, Object val)
        例: le("age", 18)--->age <= 18
        ```
        ##### between(BETWEEN 值1 AND 值2)
        ```
        between(R column, Object val1, Object val2)
        between(boolean condition, R column, Object val1, Object val2)
        例: between("age", 18, 30)--->age between 18 and 30
        ```
        ##### notBetween(NOT BETWEEN 值1 AND 值2)
        ```
        notBetween(R column, Object val1, Object val2)
        notBetween(boolean condition, R column, Object val1, Object val2)
        例: notBetween("age", 18, 30)--->age not between 18 and 30
        ```
        ##### like(LIKE '%值%')
        ```
        like(R column, Object val)
        like(boolean condition, R column, Object val)
        like("name", "王")--->name like '%王%'
        ```
        ##### notLike(NOT LIKE '%值%')
        ```
        notLike(R column, Object val)
        notLike(boolean condition, R column, Object val)
        例: notLike("name", "王")--->name not like '%王%'
        ```
        ##### likeLeft(LIKE '%值')
        ```
        likeLeft(R column, Object val)
        likeLeft(boolean condition, R column, Object val)
        例: likeLeft("name", "王")--->name like '%王'
        ```
        ##### likeRight(LIKE '值%')
        ```
        likeRight(R column, Object val)
        likeRight(boolean condition, R column, Object val)
        例: likeRight("name", "王")--->name like '王%'
        ```
        ##### isNull(字段 IS NULL)
        ```
        isNull(R column)
        isNull(boolean condition, R column)
        例: isNull("name")--->name is null
        ```
        ##### isNotNull(字段 IS NULL)
        ```
        isNotNull(R column)
        isNotNull(boolean condition, R column)
        例: isNotNull("name")--->name is not null
        ```
        ##### in(字段 IN (value.get(0), value.get(1), ...))
        ```
        in(R column, Collection<?> value)
        in(boolean condition, R column, Collection<?> value)
        例: in("age",{1,2,3})--->age in (1,2,3)
        in(R column, Object... values)
        in(boolean condition, R column, Object... values)
        例: in("age", 1, 2, 3)--->age in (1,2,3)
        ```
        ##### notIn(字段 NOT IN (value.get(0), value.get(1), ...))
        ```
        notIn(R column, Object... values)
        notIn(boolean condition, R column, Object... values)
        例: notIn("age",{1,2,3})--->age not in (1,2,3)
        notIn(R column, Object... values)
        notIn(boolean condition, R column, Object... values)
        例: notIn("age", 1, 2, 3)--->age not in (1,2,3)
        ```
        ##### inSql(字段 IN ( sql语句 ))
        ```
        inSql(R column, String inValue)
        inSql(boolean condition, R column, String inValue)
        例: inSql("age", "1,2,3,4,5,6")--->age in (1,2,3,4,5,6)
        例: inSql("id", "select id from table where id < 3")--->id in (select id from table where id < 3)
        ```
        ##### notInSql(字段 NOT IN ( sql语句 ))
        ```
        notInSql(R column, String inValue)
        notInSql(boolean condition, R column, String inValue)
        例: notInSql("age", "1,2,3,4,5,6")--->age not in (1,2,3,4,5,6)
        例: notInSql("id", "select id from table where id < 3")--->age not in (select id from table where id < 3)
        ```
        ##### groupBy(分组：GROUP BY 字段, ...)
        ```
        groupBy(R... columns)
        groupBy(boolean condition, R... columns)
        例: groupBy("id", "name")--->group by id,name
        ```
        ##### orderByAsc(排序：ORDER BY 字段, ... ASC)
        ```
        orderByAsc(R... columns)
        orderByAsc(boolean condition, R... columns)
        例: orderByAsc("id", "name")--->order by id ASC,name ASC
        ```
        ##### orderByDesc(排序：ORDER BY 字段, ... DESC)
        ```
        orderByDesc(R... columns)
        orderByDesc(boolean condition, R... columns)
        例: orderByDesc("id", "name")--->order by id DESC,name DESC
        ```
        ##### orderBy(排序：ORDER BY 字段, ...)
        ```
        orderBy(boolean condition, boolean isAsc, R... columns)
        ```
        ##### having(HAVING ( sql语句 ))
        ```
        having(String sqlHaving, Object... params)
        having(boolean condition, String sqlHaving, Object... params)
        例: having("sum(age) > 10")--->having sum(age) > 10
        例: having("sum(age) > {0}", 11)--->having sum(age) > 11
        ```
        ##### or(拼接 OR)
        ```
        or()
        or(boolean condition)
        例: eq("id",1).or().eq("name","老王")--->id = 1 or name = '老王'
        or(Function<This, This> func)
        or(boolean condition, Function<This, This> func)
        例: or(i -> i.eq("name", "李白").ne("status", "活着"))--->or (name = '李白' and status <> '活着')
        ```
        ##### and(AND 嵌套)
        ```
        and(Function<This, This> func)
        and(boolean condition, Function<This, This> func)
        例: and(i -> i.eq("name", "李白").ne("status", "活着"))--->and (name = '李白' and status <> '活着')
        ```
    - [ ] **2.2. 使用 Wrapper 自定义SQL**
        ##### Service.java
        ```
        mysqlMapper.getAll(Wrappers.<MysqlData>lambdaQuery().eq(MysqlData::getGroup, 1));
        ```
        ##### 方案一 注解方式 Mapper.java
        ```
        @Select("select * from mysql_data ${ew.customSqlSegment}")
        List<MysqlData> getAll(@Param(Constants.WRAPPER) Wrapper wrapper);
        ```
        ##### 方案二 XML形式 Mapper.xml
        ```
        <select id="getAll" resultType="MysqlData">
        	SELECT * FROM mysql_data ${ew.customSqlSegment}
        </select>
        ```
> ## 四、mybatis插件
- [ ]         MyBatisX
- [ ]         Mybatis Log Plugin