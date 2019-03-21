---
title: id自增下的单元测试
id: unit-test-with-auto-increase-id
date: 2018-12-07 14:47:54
updated: 2018-12-07 14:47:54
categories:
  - 那些古怪又令人忧心的问题
tags:
  - java
  - 单元测试
---
最新为新接手的项目写程序，除了了解需求，查看一下项目整体架构外，第一件做的事就是改集成测试。因为有了可靠的测试，才敢动手改别人的代码。

项目数据库主要是MySQL，框架用的mybatis-plus，到集成测试中则是使用H2模拟MySQL。项目原本使用的方法是在resources下有一个schema.sql和data.sql。

schema.sql里放创建表结构的sql语句，data.sql里放delete和insert数据的sql语句。运行集成测试时会执行这两个sql文件中的代码，完成数据初始化。所有测试用例都是基于这一套数据上进行代码运行的。

这样有一个问题，就是所有测试用例操作同一套数据，这个测试用例的运行结果会影响下一个测试的数据环境。

如果有人想要加入一个新的测试，很可能会导致其他测试失败，如果有人想要修改数据，也会导致其他测试失败。只能不断加新数据，然后告诉其他人不要动，非常难受。

<!-- more -->

个人的常用做法是每个测试方法执行前灌入一套数据，方法执行后再清空数据。

```
[
	{
		"id":1,
		"name":"zhangsan",
		"password":"123456"
	},
	{
		"id":2,
		"name":"lisi",
		"password":"654321"
	}
]
```

```
public class UserControllerTest extends AbstractControllerTest {

	@Autowired
	protected JdbcTemplate jdbcTemplate;
  	@Autowired
  	private UserMapper userMapper;

  	/**
   	 * 初始化数据.
   	 */
  	@Before
  	public void init() {
    	List<User> users = listFromJson(
        	"init-data/users.json", new TypeReference<List<User>>() {}
    	);
    	users.forEach(userMapper::insert);
  	}

  	/**
   	 * 清空数据.
   	 */
  	@After
  	public void destory() {
    	jdbcTemplate.execute("delete from user");
  	}

  	@Test
  	public void test1() {
  		// 测试1
  	}

  	@Test
  	public void test2() {
  		// 测试2
  	}
}
```
但是这个项目有个问题，就是所有的entity类都继承自一个BaseEntity，而BaseEntity使用的mybatis-plus的id自增策略
```
public class BaseEntity {

  	@TableId(type = IdType.AUTO)
  	protected Long id;
 	protected Long createUser;
  	protected LocalDateTime createTime;
  	protected Long updateUser;
  	protected LocalDateTime updateTime;
}
```
虽然json里写的id是1和2，但是test1运行完后test2的init()方法执行之后，插入的数据id其实是3和4。

那有没有办法解决这个问题呢？

第一个是自增数字回拨，也就是在test2的init()执行之前把自增数刷新回1。这个.....我不会！

第二个是绕过自增，不用mybatis，直接用jdbc执行insert sql，这个就比较简单了。

但是通常来讲，我们都是喜欢写json的，写insert sql既不直观，也不方便。

所以写了一段代码，将json转换成list，再将list转换成sql

```
public String listToInsertSql(List<?> list) throws IllegalAccessException {

	StringBuilder sql = new StringBuilder();

	Class clazz = list.get(0).getClass();
	List<Field> fields = getFields(clazz);
	String tableName = getLowerName(clazz.getSimpleName());
	List<String> columnNames = getColumnNames(fields);

	StringBuilder columns = new StringBuilder(64);
	StringBuilder values = new StringBuilder(64);
	Field field;
	Object valueObj;
	String valueStr;

	for (Object entity : list) {
		if (entity == null) {
	    	continue;
	  	}
	  	// 清空
	  	columns.delete(0, sql.length());
	  	values.delete(0, sql.length());

	  	for (int i = 0; i < fields.size(); i++) {
	    	field = fields.get(i);
	    	valueObj = field.get(entity);
	    	if (valueObj == null) {
	      		continue;
	    	}
	    	if (field.getGenericType().toString().endsWith("LocalDateTime")) {
	      		valueStr = ((LocalDateTime) valueObj).format(DateTimeFormatter.ISO_DATE_TIME);
	    	} else {
	      		valueStr = String.valueOf(valueObj);
	    	}
	  		columns.append(columnNames.get(i)).append(", ");
	  		values.append("'").append(valueStr).append("', ");
	  	}
	  	columns.delete(columns.length() - 2, columns.length());
	  	values.delete(values.length() - 2, values.length());
	  	sql.append("insert into ").append(tableName).append(" ( ").append(columns)
	      .append(" ) values (").append(values).append(" ); ");
		}
	return sql.toString();
}

public List<Field> getFields(Class clazz) {
	List<Field> fields = new ArrayList();
	for (; clazz != Object.class; clazz = clazz.getSuperclass()) {
	 	fields.addAll(Arrays.asList(clazz.getDeclaredFields()));
	}
	return fields;
}

public List<String> getColumnNames(List<Field> fields) {
	if (CollectionUtils.isEmpty(fields)) {
	  	return Collections.EMPTY_LIST;
	}
	List<String> names = new ArrayList<>(fields.size());
	for (Field field : fields) {
	  	field.setAccessible(true);
	  	names.add(getLowerName(field.getName()));
	}
	return names;
}

public String getLowerName(String name) {
	StringBuilder sb = new StringBuilder(name.length() * 2);
	for (char c : name.toCharArray()) {
  		if (c >= 'A' && c <= 'Z') {
    		sb.append("_").append(Character.toLowerCase(c));
  		} else {
    		sb.append(c);
		}
	}
	String newName = sb.toString();
	if (newName.startsWith("_")) {
  		newName = newName.substring(1, newName.length());
	}
	return newName;
}
```

主要有2个注意点：
- 不要忘记读取父类的属性
- 要处理特殊类型的数据格式，比如日期
