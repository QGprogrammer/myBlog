#  MyBatis(一) 3.5.6新特性

>工作中一直直接或间接（tk_mapper、mybatis_plus等框架）地使用MyBatis。现在框架的大版本也已升到了3.5.6，期间也增加了不少新功能，就此总结整理一番。有时间的话也对其核心源码进行阅读解析。



#### 鉴别器

`<collection/>和<association/>`标签大家可能都用过，至少听说过。两个标签都是用在`mapper.xml`文件里。

`<collection/>`是一对多的结果映射。

`<association/>`是一对一的结果映射。

有了以上两种标签，那么在联合查询后对复杂结果的映射有了便利的解决方案，解放了不少的生产力。但是这个标签，估计很少用到，但是在实际的开发需求中也是很常见的。

**在resultMap中使用**

```
<resultMap id="vehicleResult" type="Vehicle">
  <id property="id" column="id" />
  <result property="vin" column="vin"/>
  <result property="year" column="year"/>
  <result property="make" column="make"/>
  <result property="model" column="model"/>
  <result property="color" column="color"/>
  <discriminator javaType="int" column="vehicle_type">
    <case value="1" resultType="carResult">
      <result property="doorCount" column="door_count" />
    </case>
    <case value="2" resultType="truckResult">
      <result property="boxSize" column="box_size" />
      <result property="extendedCab" column="extended_cab" />
    </case>
    <case value="3" resultType="vanResult">
      <result property="powerSlidingDoor" column="power_sliding_door" />
    </case>
    <case value="4" resultType="suvResult">
      <result property="allWheelDrive" column="all_wheel_drive" />
    </case>
  </discriminator>
</resultMap>
```

**在`<collection/>`中使用**

```
<resultMap id="userRole" type="com.gitee.peterven.domain.UserDO">
        <id property="id" column="id"/>
        <result property="age" column="age"/>
        <result property="name" column="name"/>
        <result property="age" column="age"/>
        <collection property="roleList" ofType="com.gitee.peterven.domain.RoleDO">
        <discriminator javaType="int" column="rid">
            <case value="1">
                <id column="rid" property="id"/>
                <result property="roleName" column="roleName"/>
            </case>
        </discriminator>
        </collection>
        <collection property="roleStrList" ofType="com.gitee.peterven.domain.UserRoleDO">
            <discriminator javaType="int" column="rid">
                <case value="2">
                    <result column="rid" property="roleId"/>
                </case>
            </discriminator>
        </collection>
    </resultMap>
```

其实就是`if-else`判断，对结果进行判断分类。

比方说，在客户的证件信息表中，有证件类型、证件号等字段。如果想按类型将所有证件分类查询返回。

不用`<discriminator/>`的话就只能全部查出来后在内存中进行分类，或者执行多条sql查询，相较之下，使用鉴别器还是省了不少功夫。