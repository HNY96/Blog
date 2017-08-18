## Node.js 之数据库

mysql 开放了 Node.js 的驱动程序，即可以用 npm 安装 `mysql`。

官方提供的接口比较底层，因此查询代码：

	connection.query('select * from database where id = ?', ['123'], function(err, rows) {
			if (err) {...}
			else {
				for (let row in rows) {
					processRow(row);
				}
			}
	});

### ORM

> ORM: Object-Relational Mapping

ORM 技术把数据库的表结构映射到了对象上，键与键，值与值之间相互对应。因此 `sequelize` 就是一个 ORM 框架，来做这个转换。对于一个 `sequelize` 对象（是 promise），有这两个函数：

- `then()` 异步响应成功
- `catch()` 异步响应失败

还有一种更简单的写法：

	(async () => {
			// Pet 是一个 sequelize 对象
			// await 必须在 async 中调用
			var pets = await Pet.findAll();
	})();

### 实战

	hello-sequelize/
	|
	+- .vscode/
	|  |
	|  +- launch.json <-- VSCode 配置文件
	|
	+- init.txt <-- 初始化SQL命令
	|
	+- config.js <-- MySQL配置文件
	|
	+- app.js <-- 使用koa的js
	|
	+- package.json <-- 项目描述文件
	|
	+- node_modules/ <-- npm安装的所有依赖包

1. 先在 mysql 中定义一个数据库和表
2. 安装依赖包 `sequelize`
3. 在 `config.js` 中写一个简单的配置文件，包括数据库名、用户名、密码、主机名、端口号；然后将其暴露出去
4. 在 `app.js` 中实例化 sequelize  `var sequelize = new Sequelize(database, username, password, {host, dialect, pool});`
5. 定义模型 Pet，_告诉 Sequelize 如何映射数据库表_ `var Pet = sequelize.define('pet', {name: Sequelize.STRING(100), ...}, {timestamps: false});`，这样就定义了一个 **Model**。

对于这个 Model 有一些方法：

`create()`:
	(async () => {
			var dog = await Pet.create({
				id: 'd-' + now,
				name: 'Odie',
				//...
			});
			console.log('created: ' + JSON.stringify(dog));
	})();

`findAll()`:
	(async () => {
			var pets = await Pet.findAll({
				where: {
					name: 'Gaffey',
				}
			});
			console.log('find ${pets.length} pets');
			for (let p of pets) {
			console.log(JSON.stringify(p));
		}
	})();

`save()`:
	(async () => {
			var p = await queryFromSomewhere();
			p.gender = true;
			await p.save();
	})();

`destroy()`:
	(async () => {
			var p = await queryFromSomewhere();
			await p.destroy();
	})();

