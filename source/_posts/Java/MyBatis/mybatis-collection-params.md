---
title: MyBatis 传入参数为 List、Array、Map
author: Joden_He
tags: 
  - Java
  - MyBatis
categories: 
  - Java
  - MyBatis
description: MyBatis 传入参数为 List、Array、Map 
---



### foreach 标签简单介绍

主要用于在SQL语句中构建循环体

标签的主要属性有 `item`，`index`，`collection`，`open`，`separator`，`close`。

| 属性       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| item       | 集合中每一个元素进行迭代时的别名。该参数为必选。             |
| index      | 指定一个名字，用于表示在迭代过程中，没错迭代到的位置         |
| open       | 表示该语句以什么开始                                         |
| close      | 表示该语句以什么结束                                         |
| separator  | 表示在每次进行迭代之间以什么符号作为分隔符                   |
| collection | 属性是在使用foreach的时候最关键的也是最容易出错的，该属性是必须指定的，但是在不同情况下，该属性的值是不一样的，主要有一下3种情况： <br />(1) 如果传入的是单参数且参数类型是一个List的时候，collection属性值为list .<br />(2) 如果传入的是单参数且参数类型是一个array数组的时候，collection的属性值为array .<br />(3) 如果传入的参数是多个的时候，我们就需要把它们封装成一个Map了，当然单参数也可以封装成map，实际上如果你在传入参数的时候，在MyBatis里面也是会把它封装成一个Map的，map的key就是参数名，所以这个时候collection属性值就是传入的List或array对象在自己封装的map里面的key.<br />(4) 如果参数使用了@Param注解，那必须是参数名 |

### 示例

#### 建表语句

```sql
DROP TABLE IF EXISTS `XXGL_CODE_COMBINATIONS`;
CREATE TABLE `XXGL_CODE_COMBINATIONS`(
    `COMBINATION_ID` bigint AUTO_INCREMENT NOT NULL COMMENT '表ID，主键，供其他表做外键' ,
    `CODE_COMBINATION_ID`  bigint NOT NULL ,
    `ENABLED_FLAG`  varchar(1) NOT NULL COMMENT 'Y/N' ,
    `ACCOUNT_TYPE`  varchar(10) NOT NULL COMMENT '账户类型' ,
    `SEGMENT1`  varchar(25) NULL COMMENT '段值1',
    `SEGMENT2`  varchar(25) NULL COMMENT '段值2',
    `SEGMENT3`  varchar(25) NULL COMMENT '段值3',
    `SEGMENT4`  varchar(25) NULL COMMENT '段值4',
    `SEGMENT5`  varchar(25) NULL COMMENT '段值5',
    `SEGMENT6`  varchar(25) NULL COMMENT '段值6',
    `SEGMENT7`  varchar(25) NULL COMMENT '段值7',
    `SEGMENT8`  varchar(25) NULL COMMENT '段值8',
    `SEGMENT9`  varchar(25) NULL COMMENT '段值9',
    `SEGMENT10`  varchar(25) NULL COMMENT '段值10',
    `SEGMENT11`  varchar(25) NULL COMMENT '段值11',
    `SEGMENT12`  varchar(25) NULL COMMENT '段值12',
    `SEGMENT13`  varchar(25) NULL COMMENT '段值13',
    `SEGMENT14`  varchar(25) NULL COMMENT '段值14',
    `SEGMENT15`  varchar(25) NULL COMMENT '段值15',
    `SEGMENT16`  varchar(25) NULL COMMENT '段值16',
    `SEGMENT17`  varchar(25) NULL COMMENT '段值17',
    `SEGMENT18`  varchar(25) NULL COMMENT '段值18',
    
    PRIMARY KEY (`COMBINATION_ID`),
  	UNIQUE KEY `XXGL_CODE_COMBINATIONS_U1` (`COMBINATION_ID`),
  	KEY `XXGL_CODE_COMBINATIONS_N1` (`CODE_COMBINATION_ID`)
) ENGINE=InnoDB AUTO_INCREMENT=1000 DEFAULT CHARSET=utf8;
```



#### 测试数据

```sql
INSERT INTO `XXGL_CODE_COMBINATIONS` (
	`CODE_COMBINATION_ID`,
	`ENABLED_FLAG`,
	`ACCOUNT_TYPE`,
	`SEGMENT1`,
	`SEGMENT2`,
	`SEGMENT3`,
	`SEGMENT4`,
	`SEGMENT5`,
	`SEGMENT6`,
	`SEGMENT7`,
	`SEGMENT8`,
	`SEGMENT9`,
	`SEGMENT10`,
	`SEGMENT11`,
	`SEGMENT12`,
	`SEGMENT13`,
	`SEGMENT14`,
	`SEGMENT15`,
	`SEGMENT16`,
	`SEGMENT17`,
	`SEGMENT18`
)
VALUES
	(
		300000003238305,
		'Y',
		'E',
		'050',
		'0',
		'71205',
		'S101005',
		'50010',
		'1101',
		'I101',
		'0',
		'0',
		'0',
		'0',
		'0',
		NULL,
		NULL,
		NULL,
		NULL,
		NULL,
		NULL
	);
```



#### 实体类（CodeCombinations.java）

```java
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import org.hibernate.validator.constraints.Length;
import javax.persistence.Table;
import javax.validation.constraints.NotNull;

/**
 * Created by xiaofeng.he on 2018/03/08
 * 帐号组合DTO
 */
@Table(name = "XXGL_CODE_COMBINATIONS")
public class CodeCombinations {

    @Id
    @GeneratedValue
    private Long combinationId; // 表ID，主键，供其他表做外键

    @NotNull
    private Long codeCombinationId; // ERP中CODE_COMBINATION_ID

    private String accountType;

    private String enabledFlag;

    @Length(max = 25)
    private String segment1;

    @Length(max = 25)
    private String segment2;

    @Length(max = 25)
    private String segment3;

    @Length(max = 25)
    private String segment4;

    @Length(max = 25)
    private String segment5;

    @Length(max = 25)
    private String segment6;

    @Length(max = 25)
    private String segment7;

    @Length(max = 25)
    private String segment8;

    @Length(max = 25)
    private String segment9;

    @Length(max = 25)
    private String segment10;

    @Length(max = 25)
    private String segment11;

    @Length(max = 25)
    private String segment12;

    @Length(max = 25)
    private String segment13;

    @Length(max = 25)
    private String segment14;

    @Length(max = 25)
    private String segment15;

    @Length(max = 25)
    private String segment16;

    @Length(max = 25)
    private String segment17;

    @Length(max = 25)
    private String segment18;

    // getter setter...
}
```



#### java mapper

```java
import org.apache.ibatis.annotations.Param;

/**
 * Created by xiaofeng.he on 2018/03/08
 * 账户组合mapper
 */
public interface CodeCombinationsMapper {

    /**
     * 根据多个 segment value 查询唯一
     * @param segmentValues
     * @return
     */
    CodeCombinations selectOneBySegmentArray(@Param("segmentValues") String[] segmentValues);
    
    /**
     * 根据多个 segment value 查询唯一
     * @param segmentValues
     * @return
     */
    CodeCombinations selectOneBySegmentList(List<String> segmentValues);
    
    /**
     * 根据多个 segment value 查询唯一
     * @param params
     * @return
     */
    CodeCombinations selectOneBySegmentMap(Map<String, Object> params);

}
```



#### xml mapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="xxx.mapper.CodeCombinationsMapper">
    <resultMap id="BaseResultMap" type="xxx.dto.CodeCombinations">
        <result column="COMBINATION_ID" property="combinationId" jdbcType="DECIMAL" />
        <result column="CODE_COMBINATION_ID" property="codeCombinationId" jdbcType="DECIMAL" />
        <result column="ACCOUNT_TYPE" property="accountType" jdbcType="VARCHAR" />
        <result column="ENABLED_FLAG" property="enabledFlag" jdbcType="VARCHAR" />
        <result column="SEGMENT1" property="segment1" jdbcType="VARCHAR" />
        <result column="SEGMENT2" property="segment2" jdbcType="VARCHAR" />
        <result column="SEGMENT3" property="segment3" jdbcType="VARCHAR" />
        <result column="SEGMENT4" property="segment4" jdbcType="VARCHAR" />
        <result column="SEGMENT5" property="segment5" jdbcType="VARCHAR" />
        <result column="SEGMENT6" property="segment6" jdbcType="VARCHAR" />
        <result column="SEGMENT7" property="segment7" jdbcType="VARCHAR" />
        <result column="SEGMENT8" property="segment8" jdbcType="VARCHAR" />
        <result column="SEGMENT9" property="segment9" jdbcType="VARCHAR" />
        <result column="SEGMENT10" property="segment10" jdbcType="VARCHAR" />
        <result column="SEGMENT11" property="segment11" jdbcType="VARCHAR" />
        <result column="SEGMENT12" property="segment12" jdbcType="VARCHAR" />
        <result column="SEGMENT13" property="segment13" jdbcType="VARCHAR" />
        <result column="SEGMENT14" property="segment14" jdbcType="VARCHAR" />
        <result column="SEGMENT15" property="segment15" jdbcType="VARCHAR" />
        <result column="SEGMENT16" property="segment16" jdbcType="VARCHAR" />
        <result column="SEGMENT17" property="segment17" jdbcType="VARCHAR" />
        <result column="SEGMENT18" property="segment18" jdbcType="VARCHAR" />
    </resultMap>
    
	<!--Array:forech 中的 collection 属性类型是 Array, collection 的值必须是:array, item 的值可以随意, mapper 接口中参数名字随意 -->
    <select id="selectOneBySegmentArray" resultMap="BaseResultMap">
        select c.* from XXGL_CODE_COMBINATIONS c
        <where>
            and c.ENABLED_FLAG = 'Y'
            <choose>
                <when test="segmentValues">
                    <!-- 使用了@Param 后collection不能填array，要填写别名 -->
                    <foreach collection="segmentValues" item="segmentValue" index="index">
                        and c.SEGMENT${index+1} = #{segmentValue}
                    </foreach>
                </when>
                <otherwise>
                    and 1=2
                </otherwise>
            </choose>
        </where>
    </select>
    
    <!--List:forech 中的 collection 属性类型是 List, collection 的值必须是:list, item 的值可以随意, mapper 接口中参数名字随意 -->
    <select id="selectOneBySegmentList" resultMap="BaseResultMap">
        select c.* from XXGL_CODE_COMBINATIONS c
        <where>
            and c.ENABLED_FLAG = 'Y'
            <choose>
                <when test="list">
                    <foreach collection="list" item="segmentValue" index="index">
                        and c.SEGMENT${index+1} = #{segmentValue}
                    </foreach>
                </when>
                <otherwise>
                    and 1=2
                </otherwise>
            </choose>
        </where>
    </select>
    
    <!--Map:不单单 forech 中的 collection 属性是 map.key,其它所有属性都是 map.key, 比如下面的 enabledFlag -->
    <select id="selectOneBySegmentMap" resultMap="BaseResultMap">
        select c.* from XXGL_CODE_COMBINATIONS c
        <where>
            and c.ENABLED_FLAG = #{enabledFlag}
            <choose>
                <when test="segmentArray">
                    <foreach collection="segmentArray" item="segmentValue" index="index">
                        and c.SEGMENT${index+1} = #{segmentValue}
                    </foreach>
                </when>
                <otherwise>
                    and 1=2
                </otherwise>
            </choose>
            <choose>
                <when test="segmentList">
                    <foreach collection="segmentList" item="segmentValue" index="index">
                        /* 2 */
                        and c.SEGMENT${index+1} = #{segmentValue}
                    </foreach>
                </when>
                <otherwise>
                    and 1=2
                </otherwise>
            </choose>
        </where>
    </select>
</mapper>
```

#### 测试 controller

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import xxx.dto.CodeCombinations;
import xxx.mapper.CodeCombinationsMapper;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
public class TestController {
    @Autowired
    private CodeCombinationsMapper codeCombinationsMapper;

    @GetMapping("/test/code/combination/selectBySegments")
    @ResponseBody
    public List<CodeCombinations> test() {
        String[] a = {"050", "0", "71205", "S101005", "50010", "1101", "I101"};

        CodeCombinations codeCombinations1 = codeCombinationsMapper.selectOneBySegments(a);
        CodeCombinations codeCombinations2 = codeCombinationsMapper.selectOneBySegmentList(Arrays.asList(a));
        Map<String, Object> params = new HashMap<>();
        params.put("enabledFlag", "Y");
        params.put("segmentArray", a);
        params.put("segmentList", Arrays.asList(a));
        CodeCombinations codeCombinations3 = codeCombinationsMapper.selectOneBySegmentMap(params);

        return Arrays.asList(codeCombinations1, codeCombinations2, codeCombinations3);
    }
}
```

#### 测试结果

**log**

```verilog
2019-02-08 22:40:40.523 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegments - ==>  Preparing: select c.* from XXGL_CODE_COMBINATIONS c WHERE c.ENABLED_FLAG = 'Y' and c.SEGMENT1 = ? and c.SEGMENT2 = ? and c.SEGMENT3 = ? and c.SEGMENT4 = ? and c.SEGMENT5 = ? and c.SEGMENT6 = ? and c.SEGMENT7 = ? 
2019-02-08 22:40:40.524 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegments - ==> Parameters: 050(String), 0(String), 71205(String), S101005(String), 50010(String), 1101(String), I101(String)
2019-02-08 22:40:40.532 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegments - <==      Total: 1

2019-02-08 22:40:43.259 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegmentList - ==>  Preparing: select c.* from XXGL_CODE_COMBINATIONS c WHERE c.ENABLED_FLAG = 'Y' and c.SEGMENT1 = ? and c.SEGMENT2 = ? and c.SEGMENT3 = ? and c.SEGMENT4 = ? and c.SEGMENT5 = ? and c.SEGMENT6 = ? and c.SEGMENT7 = ? 
2019-02-08 22:40:43.259 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegmentList - ==> Parameters: 050(String), 0(String), 71205(String), S101005(String), 50010(String), 1101(String), I101(String)
2019-02-08 22:40:43.263 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegmentList - <==      Total: 1

2019-02-08 22:40:44.284 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegmentMap - ==>  Preparing: select c.* from XXGL_CODE_COMBINATIONS c WHERE c.ENABLED_FLAG = ? and c.SEGMENT1 = ? and c.SEGMENT2 = ? and c.SEGMENT3 = ? and c.SEGMENT4 = ? and c.SEGMENT5 = ? and c.SEGMENT6 = ? and c.SEGMENT7 = ? /* 2 */ and c.SEGMENT1 = ? /* 2 */ and c.SEGMENT2 = ? /* 2 */ and c.SEGMENT3 = ? /* 2 */ and c.SEGMENT4 = ? /* 2 */ and c.SEGMENT5 = ? /* 2 */ and c.SEGMENT6 = ? /* 2 */ and c.SEGMENT7 = ? 
2019-02-08 22:40:44.284 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegmentMap - ==> Parameters: Y(String), 050(String), 0(String), 71205(String), S101005(String), 50010(String), 1101(String), I101(String), 050(String), 0(String), 71205(String), S101005(String), 50010(String), 1101(String), I101(String)
2019-02-08 22:40:44.289 DEBUG [10001] [e5c47fa7c79b461a806c2d6135d485c2] xxx.mapper.CodeCombinationsMapper.selectOneBySegmentMap - <==      Total: 1
```

**json**

```json
[
{
"combinationId": 1000,
"codeCombinationId": 300000003238305,
"accountType": "E",
"enabledFlag": "Y",
"segment1": "050",
"segment2": "0",
"segment3": "71205",
"segment4": "S101005",
"segment5": "50010",
"segment6": "1101",
"segment7": "I101",
"segment8": "0",
"segment9": "0",
"segment10": "0",
"segment11": "0",
"segment12": "0",
"segment13": null,
"segment14": null,
"segment15": null,
"segment16": null,
"segment17": null,
"segment18": null
},
{
"combinationId": 1000,
"codeCombinationId": 300000003238305,
"accountType": "E",
"enabledFlag": "Y",
"segment1": "050",
"segment2": "0",
"segment3": "71205",
"segment4": "S101005",
"segment5": "50010",
"segment6": "1101",
"segment7": "I101",
"segment8": "0",
"segment9": "0",
"segment10": "0",
"segment11": "0",
"segment12": "0",
"segment13": null,
"segment14": null,
"segment15": null,
"segment16": null,
"segment17": null,
"segment18": null
},
{
"combinationId": 1000,
"codeCombinationId": 300000003238305,
"accountType": "E",
"enabledFlag": "Y",
"segment1": "050",
"segment2": "0",
"segment3": "71205",
"segment4": "S101005",
"segment5": "50010",
"segment6": "1101",
"segment7": "I101",
"segment8": "0",
"segment9": "0",
"segment10": "0",
"segment11": "0",
"segment12": "0",
"segment13": null,
"segment14": null,
"segment15": null,
"segment16": null,
"segment17": null,
"segment18": null
}
]
```

### 感谢

https://blog.csdn.net/s592652578/article/details/52871884