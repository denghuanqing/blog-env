---
title: jfinal源码05：jdbc
date: 2018-05-23 14:42:22
categories: JFinal
---
jfinal以插件的形式封装了jdbc
<!-- more -->
##### 5.1 Model一个实例对应一张表
ActiveRecordPlugin类 通过List集合维护了项目中所有的表
````
//配置文件
arp.addMapping("tb_item", User.class);
// 存放了所有的Table，对应数据库的表
private List<Table> tableList = new ArrayList<Table>();
// 226   维护TableMapping类的modelToTableMap 存放<class,table>映射
new TableBuilder().build(tableList, config);
````
Table类
````
        private String name;//表名
	private String[] primaryKey = null;
        // 
	private Map<String, Class<?>> columnTypeMap;	// config.containerFactory.getAttrsMap();
	
	private Class<? extends Model<?>> modelClass;
````
![image.png](http://upload-images.jianshu.io/upload_images/7027953-58820eb2e892251d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ModelBuilder类的build方法，**完成了数据库表列名，到Model对象参数名的映射**
````
public <T> List<T> build(ResultSet rs, Class<? extends Model> modelClass) throws SQLException, InstantiationException, IllegalAccessException {
		List<T> result = new ArrayList<T>();
		ResultSetMetaData rsmd = rs.getMetaData();
		int columnCount = rsmd.getColumnCount();
		String[] labelNames = new String[columnCount + 1];
		int[] types = new int[columnCount + 1];
      // 从结果集中获取每一列的  列名称 和 列类型 集合
		buildLabelNamesAndTypes(rsmd, labelNames, types);
		while (rs.next()) {
			Model<?> ar = modelClass.newInstance();
			Map<String, Object> attrs = ar._getAttrs();
			for (int i=1; i<=columnCount; i++) {
				Object value;
				if (types[i] < Types.BLOB)
					value = rs.getObject(i);
				else if (types[i] == Types.CLOB)
					value = handleClob(rs.getClob(i));
				else if (types[i] == Types.NCLOB)
					value = handleClob(rs.getNClob(i));
				else if (types[i] == Types.BLOB)
					value = handleBlob(rs.getBlob(i));
				else
					value = rs.getObject(i);
				// 填充Model的字段信息 <参数名，参数值>
				attrs.put(labelNames[i], value);
			}
			result.add((T)ar);
		}
      // 返回Model对象的结果集
		return result;
	}
````
Model类主要就关注attrs属性 用一个map集合替代了传统javaBean的属性（从resultSet中取出来）

tips：
1.getModel 其实调原理就是从request中获取Map参数集合，遍历Map在set到model
2.可以同过TableMapping获取表相关信息，Table user = TableMapping.me().getTable(User.class);
##### 5.2 Db
model有很多方法都是间接的在调用Db -> DbPro
model 和Db存在数据源的问题，大体调用方法一致  核心代码就是ModelBuild类build方法
