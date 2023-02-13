# JavaSe

## 字符串时间比较

- `int i = str1.compareTo(str2) `
  - i < 0 str1 < str2

## 比较两个集合对象的异同

```java
C.stream().filter(c -> {
    return I.stream().filter(i -> i.getname() == c.getname()).count() > 0;
}).collect(Collectors.toList()).forEach(hjUserContactMapper::insert);
```

- 大于0则相同，小于等于0不同

## 除法

- Math.round()
- `DecimalFormat decimalFormat = new DecimalFormat(".00");`

### 小数点后取两位

- `String.format("%.2f", num)`

## 各项求比例后和为100%

#### 最大余额法

```java
/**
* 最大余额法 求百分比
*/
public static List<Integer> calculatePercent(List<Long> list) {
    Long sum = list.stream().reduce(Long::sum).get();
    List<Integer> valueInts = new ArrayList<>();
    List<Double> ds = new ArrayList<>();
    for (Long percent : list) {
        double value = percent * 100 / (double) sum;
        //设置对应的百分比
        int valueInt = (int) value;
        valueInts.add(valueInt);
        //获取小数点后的值
        double d = value - valueInt;
        ds.add(d);
    }
    //求和：当前各项百分比合计。由于我们舍弃了小数位，所以该合计只会小于等于100
    int curSum = valueInts.stream().mapToInt(e -> e).sum();
    while (curSum < 100) {
        //找出小数余额最大的组，对其进行加1
        Integer max = valueInts.stream().max(Comparator.comparingDouble(e -> ds.get(valueInts.indexOf(e))
                                                                       )).get();
        valueInts.set(valueInts.indexOf(max), max + 1);
        //当前这个数已经加1了，不应该参与下一轮的竞选
        ds.set(valueInts.indexOf(max + 1), 0.0);
        curSum++;
    }
    return valueInts;
}
```

## 创建Map时添加值

```java
new HashMap<K, V>(){
    {
        put(key, value)
    }
}
```

## Stream流

### 取对象列表中日期最大值最小值

```java
m.getValue().stream().min(Comparator.comparing(HjUserWorkHourDTO::getWorkDate)).get().getWorkDate();
m.getValue().stream().max(Comparator.comparing(HjUserWorkHourDTO::getWorkDate)).get().getWorkDate();
```



# 数据库

## 两个表有关联时

- 用一张表保留两个表的id做对应关系

## 时间截取

#### pg

- `to_char(now()::timestamp, 'yyyy-MM-dd')`

## Xml中使用Where条件查询

```xml
<Where>
    <!--数字类型-->
	<if test="query.id != null and query.id != '0'.toString()">
    	and sys_user.user_id = '${query.userId}'
    </if>
    <!--文本类型-->
    <if test="param.name != null">
        and sys_user.username LIKE '%${query.username}%'
    </if>
</Where>
```

## 时间日期比较

- datediff()
  - 返回两个时间的差值

## 大于小于转义

```xml
&lt;			<
&gt;			>
&amp;			&
&apos;			'
&quot;			"
```

## 截取身份证年月日

`DATE_FORMAT(cast(substring('44050919990409401X', 7, 8) as date), '%Y-%m-%d')`

## xml返回HashMap<Long, List<HjUserInfoDTO>>

```java
@MapKey("deptId")
HashMap<Long, List<HjUserInfoDTO>> getDeptAndUser();
```

```xml
<resultMap id="deptAndUser" type="java.util.Map">
    <result property="deptId" column="deptId"/>
    <collection property="users" javaType="java.util.ArrayList" ofType="com.bda.huijun.hr.personnel.entity.vo.HjUserInfoVO">
        <result column="ID" property="id" />
        <result column="UID_TYPE" property="uidType" />
        <result column="AID" property="aid" />
        <result column="ANAME" property="aname" />
        <result column="UNAME" property="uname" />
        <result column="UAGE" property="uage" />
        <result column="USEX" property="usex" />
        <result column="UPICTURE" property="upicture" />
        <result column="ULOCAL" property="ulocal" />
        <result column="UNATION" property="unation" />
        <result column="UP_HEIGHT" property="upHeight" />
        <result column="UP_WEIGHT" property="upWeight" />
        <result column="UP_BLOOD" property="upBlood" />
        <result column="UP_HEALTH" property="upHealth" />
        <result column="UP_HEALTH_DES" property="upHealthDes" />
        <result column="UP_MEDICAL_HIS" property="upMedicalHis" />
        <result column="UP_MEDICAL_HIS_DES" property="upMedicalHisDes" />
        <result column="UID_CODE" property="uidCode" />
        <result column="UID_ADDERSS" property="uidAdderss" />
        <result column="UADDRESS" property="uaddress" />
        <result column="UPHONE" property="uphone" />
        <result column="UMARITAL" property="umarital" />
        <result column="UPOLOTICAL" property="upolotical" />
        <result column="UREPRODUCTIVE" property="ureproductive" />
        <result column="UHOUSEHOLD" property="uhousehold" />
        <result column="UCOUNTRY" property="ucountry" />
        <result column="UNATIVE_PLACE" property="unativePlace" />
        <result column="SORT_NUM" property="sortNum" />
    </collection>
</resultMap>

<select id="getDeptAndUser" resultMap="deptAndUser">
    SELECT su.dept_id as deptId, hui.*
    FROM hj_user_info hui
    LEFT JOIN sys_user su ON hui.aid = su.user_id
    LEFT JOIN sys_dept sd ON su.dept_id = sd.dept_id
    WHERE hui.del_flag = '0'
</select>
```

## 类型转换

type可以是以下类型之一：BINARY，CHAR，DATE，DATETIME，TIME，DECIMAL，SIGNED，UNSIGNED。

### **CAST()**

- `CAST(字段 AS TYPE)`

### **CONVERT()**

- `CONVERT(字段, TYPE)`
