---
title: Hibernate使用Json存储List或Map对象
date: 2018-12-06 11:39:07
tags: JPA Hibernate hibernate-types
---
  
需求描述：使用Hibernate作为ORM框架，实体对象中包含List 或 Map 的对象，但是不希望使用OneToMany这种方式进行存储（可能数据比较简单，没必要用这种结构），而是直接用Json进行持久化。
具体的实现方法有两种，分别进行介绍。


### 一、 推荐方法
此方法比较简单，代码少，但是会增加依赖。
依赖： Lombok、hibernate-types、Jackson（必选）

#### 1. 引入依赖  

```xml
<dependency>  
  <groupId>org.projectlombok</groupId>  
  <artifactId>lombok</artifactId>  
  <version>1.18.2</version>  
  <scope>provided</scope>  
</dependency>
<dependency>  
  <groupId>com.vladmihalcea</groupId>  
  <artifactId>hibernate-types-5</artifactId>  
  <version>2.3.5</version>  
</dependency>
```
其他依赖自行添加，注意其中hibernate-types，请访问作者的仓库 [GitHub](https://github.com/vladmihalcea/hibernate-types)，根据指引导入对应的版本即可。

#### 2. 实体对象  
  

```
@Data
@Entity
@Table(name = "table_name")
public class VoteItems {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(columnDefinition = "LONGTEXT")
    @Type(type = "json")
    private Map<Long, Integer> scoreItemMap;
    
    private String name;

}
```
代码详解：
@Data  Lombok，自动生成Get、Set、toString啥的
@Type  重点，放在需要用Json存储的字段上，其中参数 ‘json’需要与后面的配置对应


#### 3. Spring SessionFactory配置  
配置如下：
```xml
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">  
 <property name="configLocation" value="classpath:conf/hibernate.cfg.xml"/>  
 <property name="mappingLocations" value="classpath:conf/hibernate.types.xml"/>  
 <property name="dataSource" ref="dataSourceProxy"/>  
</bean>
```
其中hibernate.cfg.xml 没什么特别的，大体如下：
```xml
<hibernate-configuration>  
 <session-factory>  
 <property name="dialect">org.hibernate.spatial.dialect.mysql.MySQLSpatialDialect</property>  
 <property name="show_sql">false</property>  
 <property name="format_sql">true</property>  
  <property name="hbm2ddl.auto">update</property>  
  
 <mapping class="com.*****.VoteItems"/>  
  
 </session-factory>  
</hibernate-configuration>
```
重点在hibernate.types.xml，配置如下：
```
<?xml version="1.0"?>  
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"  
 "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">  

<hibernate-mapping>  
 <typedef class="com.vladmihalcea.hibernate.type.json.JsonStringType" name="json"/>  
</hibernate-mapping>
```
其中定义了一个typedef ，class使用的hibernate-types提供的工具类，name则为与实体类中的对应名称


#### 4. 测试  
测试代码：
```
@RunWith(SpringJUnit4ClassRunner.class)  
@ContextConfiguration(locations = "classpath:spring-config.xml")  
@WebAppConfiguration  
public class TestVoteItemsLombok {  
  
  @Resource  
  private BaseService baseService;  
  
  @Test  
  public void test() throws ObjectPersistenceException {  
    val map = new HashMap<Long, Integer>();  
    map.put(1L, 1);  
    map.put(2L, 2);  
  
    val vi = new VoteItems();  
    vi.setName("测试");  
    vi.setScoreItemMap(map);  
  
    baseService.saveOrUpdate(vi);  
  
    val a = baseService.findById(VoteItems.class, 1L);  
    System.out.println(a);  
  }  
}
```
存储到数据库之后的效果如下所示：
{% asset_img effect-table.png 效果图 %}



### 二、 减少依赖的方法

前面提到的依赖： Lombok、hibernate-types
下面依次介绍在省去各个依赖后的做法：
#### 1. Lombok
跟前面的写法没有多大区别，自己写Get、Set就可以了
#### 2. hibernate-types
这个部分比较复杂了，省去有后两类方法可以解决，第一种当然是自己写一个JsonStringType，这个方法不推荐，此处不介绍；另一种是使用自定义个Get\Set来实现，具体如下：

注：其中需要有一个需要注意的地方，在Hibernate中@Id 、@Column这些可以写在实体类的字段上或是对应字段的Get/Set方法上，Hibernate是根据@Id所在的位置进行判断，如果@Id在字段上，则会用class.getFields 这种方式来获取，如果@Id在方法上，则会找成对的Get/Set，作为数据库的字段。
我们这种方法需要自定义Get/Set，所以@Id必须要放到Get/Set方法上才行。
最终的实体类写法如下：
```
@Entity  
@Table(name = "evadata_voteitems")  
public class VoteItem {  

    private Long id;  
    private String name;  
    private Map<Long, Integer> scoreItem;  
    
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    public Long getId() {  
        return id;  
    }  
    
    public void setId(Long id) {  
        this.id = id;  
    }  
    
    public String getName() {  
        return name;  
    }  
    
    public void setName(String name) {  
        this.name = name;  
    }  
    
    @Column(columnDefinition = "LONGTEXT")  
    public String getScoreItem() {  
        return JacksonUtils.obj2Json(scoreItem);  
    }  
    
    public void setScoreItem(String scoreItem) {  
        this.scoreItem = JacksonUtils.jsonToType(scoreItem, new TypeReference<Map<Long, Integer>>() {});  
    }  
    
    @Transient  
    public Map<Long, Integer> getScoreItemMap() {  
        return scoreItem;  
    }  
    
    public void setScoreItemMap(Map<Long, Integer> scoreItemMap) {  
        this.scoreItem = scoreItemMap;  
    }  
}
```
其中需要注意的地方：
    1、在getScoreItem()上我们增加了字段的描述，并且方法内容是从Map转Json的方式，在对应的setScoreItem中，我们实现了Json转Map。当然这里可以用Json，也可以用你喜欢的任一序列号框架
    2、在Map<Long, Integer> getScoreItemMap() 上，我们增加了@Transient注解，意思是告诉Hibernate忽略该字段。
这种写法可以达到和前面方式同样的效果。