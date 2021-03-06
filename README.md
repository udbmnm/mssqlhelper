# nodejs Microsoft SQL Server Helper 
 nodejs的一个用于连接mssql数据库的工具类

# Features 介绍

 * 采用微软tds协议，不需要任何C/C++扩展，跨平台使用
 * 执行sql语句，获得结果行
 * 执行存储过程，获得输出参数以及结果行

# TODO 待实现内容

 * 将支持获得多个结果集（Table）
 * 将支持连接池
 * 更多性能加强


# Use 使用
```
    $ git clone git@github.com:play175/mssqlhelper.git
    $ cd mssqlhelper
    $ node test.js
```
# Example 测试代码
```javascript
var db = require('./index');

db.config({
    host: '192.168.1.100'
	,port: 1433
	,userName: 'sa'
	,password: '123'
	,database:'testdb'
});

//test query sql 执行sql

db.query(
	'select @Param1 Param1,@Param2 Param2'
	,{
		 Param1: { type : 'NVarChar', size: 7,value : 'myvalue' }
		 ,Param2: { type : 'Int',value : 321 }
	}
	,function(res){
		if(res.err)throw new Error('database error:'+res.err.msg);
		var rows = res.tables[0].rows;
		for (var i = 0; i < rows.length; i++) {
			console.log(rows[i].getValue(0),rows[i].getValue('Param2'));
		}
	}
);

//test excute sp 执行存储过程

db.exec(
	'test_sp'
	,{
		 Param1: {direction:'out', type : 'NVarChar', size: 50,value : 'my Param1 value' }
		 ,Param2: { type : 'Int',value : 123 }
		 ,Param3: {direction:'out', type : 'VarChar', size: 50,value : '789' }
	}
	,function(res){
		if(res.err)throw new Error('database error:'+res.err.msg);

		//get output paramater value
		console.log('output @Param1='+res.params.Param1.value);

		//get rows
		var rows = res.tables[0].rows;
		for (var i = 0; i < rows.length; i++) {
			var rp = '';
			for(var j=0,len = rows[i].metadata.columns.length;j<len;j++){
				var col = rows[i].metadata.columns[j];
				rp += ' ' +(rows[i].getValue(j));
			}
			console.log(rp);
		}
	}
);
```


///////////////////////////////问题和解决方案//////////////////////////////////////

问题/Issues：

1：不能输出匿名列，比如select *

2：如果不能排序，比如 select a from table oerder by b desc，目前的解决方法：
```sql
	;with result as(
		SELECT Actor,ActorName FROM [GameActor] order by time desc
	)
	select * from result
```	
3：输出中文乱码，引发这个问题有几个方面，解决方法：

	（1）把所有js文件用utf-8编码保存
	
	（2）数据库中含有中文的字段，必须是unicode类型，比如varchar应该改为nvarchar

4:在多进程cluster的的使用：
```javascript
	var db = require('./index');	
	var cluster = require('cluster');	
	if (cluster.isMaster) {
	    var numCPUs = require('os').cpus().length;
	    for (var i = 0; i < numCPUs; i++) {
	        var worker = cluster.fork();
	    }
	} else {
	    db.config(..);
	    db.query(..);
	}
```

