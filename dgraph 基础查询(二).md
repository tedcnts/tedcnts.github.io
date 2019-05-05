# 基本请求方式及数据准备
## http方式
Alpha server 端口 
1. /alter 用于修改表结构 包括删除数据库
1. /query 用户查询
1. /mutate 添加或修改数据
1. /commit 提交事务

alter 例子：

添加 name （string 类型字段） 。 用于后面查询。如果没有索引，不利于查询。
``` bash
curl -X POST http://127.0.0.1:8080/alter -d 'name: string @index(term) .'
# 返回 {"code":"Success","message":"Done"}
```

``` bash
# 删除全库字段
curl -X POST http://127.0.0.1:8080/alter -d 'name: string @index(term) .'
# 删除全库 学习时候 先执行一下
curl -X POST http://127.0.0.1:8080/alter -d '{"drop_all": true}'
```
## 测试数据准备
``` bash
# 添加数据库结构 用于后面搜索 
curl -X POST http://127.0.0.1:8080/alter -d $'
name: string @index(term) @lang .
age: int @index(int) .
friend: uid @count .
'
# -H "X-Dgraph-CommitNow: true" 代表事务自动提交
# 此节重点介绍查询，具体事务后面介绍。大体上 就是 mutate 后 用commit 提交
curl -X POST http://127.0.0.1:8080/mutate -H "X-Dgraph-CommitNow: true" -d $'
{
  set {
    _:michael <name> "Michael" .
    _:michael <age> "39" .
    _:michael <friend> _:amit .
    _:michael <friend> _:sarah .
    _:michael <friend> _:sang .
    _:michael <friend> _:catalina .
    _:michael <friend> _:artyom .
    _:michael <owns_pet> _:rammy .

    _:amit <name> "अमित"@hi .
    _:amit <name> "অমিত"@bn .
    _:amit <name> "Amit"@en .
    _:amit <age> "35" .
    _:amit <friend> _:michael .
    _:amit <friend> _:sang .
    _:amit <friend> _:artyom .

    _:luke <name> "Luke"@en .
    _:luke <name> "Łukasz"@pl .
    _:luke <age> "77" .

    _:artyom <name> "Артём"@ru .
    _:artyom <name> "Artyom"@en .
    _:artyom <age> "35" .

    _:sarah <name> "Sarah" .
    _:sarah <age> "55" .

    _:sang <name> "상현"@ko .
    _:sang <name> "Sang Hyun"@en .
    _:sang <age> "24" .
    _:sang <friend> _:amit .
    _:sang <friend> _:catalina .
    _:sang <friend> _:hyung .
    _:sang <owns_pet> _:goldie .

    _:hyung <name> "형신"@ko .
    _:hyung <name> "Hyung Sin"@en .
    _:hyung <friend> _:sang .

    _:catalina <name> "Catalina" .
    _:catalina <age> "19" .
    _:catalina <friend> _:sang .
    _:catalina <owns_pet> _:perro .

    _:rammy <name> "Rammy the sheep" .

    _:goldie <name> "Goldie" .

    _:perro <name> "Perro" .
  }
}
'
```
格式基本是
uid <属性名字> 值 .
uid <关系> 被关系的uid .

相当于 json
``` json
{
	"set": [{
			"uid": "_:michael",
			"name": "Michael",
			"age": 39,
			"owns_pet": {
				"uid": "_:rammy",
				"name": "Rammy the sheep"
			},
			"friend": [{
					"uid": "_:amit"
				},
				{
					"uid": "_:sarah"
				},
				{
					"uid": "_:sang"
				},
				{
					"uid": "_:catalina"
				},
				{
					"uid": "_:artyom"
				}
			]
		},
		{
			"uid": "_:amit",
			"name@hi": "अमित",
			"name@bn": "অমিত",
			"name@en": "Amit",
			"age": 35,
			"friend": [{
					"uid": "_:michael"
				},
				{
					"uid": "_:sang"
				},
				{
					"uid": "_:artyom"
				}
			]
		},
		{
			"uid": "_:luke",
			"name@pl": "Łukasz",
			"name@en": "Luke",
			"age": 77
		},
		{
			"uid": "_:artyom",
			"name@ru": "Артём",
			"name@en": "Artyom",
			"age": 35
		},
		{
			"uid": "_:sarah",
			"name": "Sarah",
			"age": 55
		},
		{
			"uid": "_:sang",
			"name@ko": "상현",
			"name@en": "Sang Hyun",
			"age": 24,
			"owns_pet": {
				"uid": "_:goldie",
				"name": "Goldie"
			},
			"friend": [{
					"uid": "_:amit"
				},
				{
					"uid": "_:catalina"
				},
				{
					"uid": "_:hyung"
				}
			]
		},
		{
			"uid": "_:hyung",
			"name@en": "Hyung Sin",
			"name@ko": "형신",
			"friend": {
				"uid": "_:sang"
			}
		},
		{
			"uid": "_:catalina",
			"name": "Catalina",
			"age": 19,
			"owns_pet": {
				"uid": "_:perro",
				"name": "Perro"
			},
			"friend": {
				"uid": "_:sang"
			}
		}
	]
}
```

## 基础查询

查询
```
{
    # #开头表示注释
    # everyone1 everyone2 代表查询 语句id 可以多条
    # anyofterms 代表查询类型  此例子为 对name 属性 为 Michael Amit
  everyone1(func: anyofterms(name, "Michael Amit")) {
    # 返回体 指定返回数据
    name
    friend {
        # 返回 friend的name属性 @ru @ko @en 如果ru存在 则返回ru 如果ko 存在 则返回ko 都没有 返回en 
      name@ru:ko:en
      # 对朋友不过滤 返回所有属性
      friend { expand(_all_) { expand(_all_) } }
    }
  }
  michaels_friends(func: eq(name, "Michael")) {
    name
    age
    friend {
      name@.
    }
  }
}

```
基本结构为
```
{
    # 查询id (func 查询方法){
    #    返回字段
    #    返回关系 { 返回条件 }
    #}
    michaels_friends(func: eq(name, "Michael")) {
    name
    age
    friend {
      name@.
    }
    # name 为 Michael  返回 name，age，friend. name 属性
  }
}
```

## 列属性查询
数据类型


| Dgraph Type |Description |
|  -----------|------------------|
| int	| signed 64 bit integer |
| float |	double precision floating point number |
| string |	string |
| bool |	boolean |
| id |	ID’s stored as strings |
| dateTime	| RFC3339 time format with optional timezone eg: 2006-01-02T15:04:05.999999999+10:00 or 2006-01-02T15:04:05.999999999 |
| geo |	geometries stored using go-geom |
```
# 对 name, age, friend, owns_pet 查询属性
schema(pred: [name, age, friend, owns_pet]) {
  type # 数据类型
  index # true 是索引 能被搜索
}
```

## 列返回 及别名
### 列可以被@ 修饰 
1. 如果什么都没有，返回值，如果没有值则不返回
2. 如果@标签1:标签2:标签3 。总返回最左边有值一列。
3. @标签1:标签2:. 如果标签1，标签2 都没有值，返回其他标签，如果其他标签也没有 返回“some”

### 别名 

别名 ： 属性

别名可以跟 属性名相同

```
{
  language_support(func: allofterms(name, "Michael")) {
    name@bn:hi:en
    name1 : name@bn:hi:en
    name2 : name
    name3 : name@bn:hi:en:.
    age : age
    friend {
      name@ko:ru
      age
    }
  }
}
```
## 支持嵌套

查询Michael的朋友的宠物：
```
{
  michael_friends_pets(func: eq(name, "Michael")) {
    name
    age
    friend {
      name@.
      owns_pet {
        name
      }
    }
  }
}
```
查询Michael的朋友的朋友：

注意 返回不是树形结构 具体查看返回
```
{
  michael_friends_friends(func: allofterms(name, "Michael")) {
    name
    age
    friend {
      name
      friend {
        name
      }
    }
  }
}
``` 
## 搜索方法
func| 含义
----| ----
allOfTerms(edge_name, "term1 ... termN")|匹配带有传出string边的节点，edge_name其中字符串包含所有列出的术语。
anyOfTerms(edge_name, "term1 ... termN")|与之相同allOfTerms，但至少匹配一个术语。
has(edge_name) | 是否存在
eq(edge_name, value)| 等于
ge(edge_name, value)| 大于或等于
le(edge_name, value)| 小于或等于
gt(edge_name, value)| 大于
lt(edge_name, value)| 小于
AND| 和
OR | 或
NOT | 非
orderasc | 生序
orderdesc | 降序
first: N | 返回N结果
offset: N | 跳过N结果
after: uid | 返回uid 之后的
count(edge_name) | 计数
```
{
  michaels_friends_filter(func: allofterms(name, "Michael")) {
    name
    age
    friend @filter(ge(age, 27)) {
      name@.
      age
    }
  }
  michael_friends_and(func: allofterms(name, "Michael")) {
    name
    age
    friend @filter(ge(age, 27) AND le(age, 48)) {
      name@.
      age
    }
  }
  michael_friends_sorted(func: allofterms(name, "Michael")) {
    name
    age
    friend @filter(ge(age, 1) AND le(age, 300))(orderasc: age) {
      name@.
      age
    }
  }
  michael_friends_first(func: allofterms(name, "Michael")) {
    name
    age
    friend (orderasc: name@., offset: 1, first: 2) {
      name@.
    }
  }
  michael_number_friends(func: allofterms(name, "Michael")) {
    name
    age
    count(friend)
  }
  # 聚合
  lots_of_friends(func: ge(count(friend), 2)) {
    name@.
    age
    friend {
      name@.
    }
  }
  # 跟节点 条件组合
  lots_of_friends(func: ge(count(friend), 2)) @filter(ge(age, 20) AND lt(age, 30)) {
    name@.
    age
    friend {
        name@.
    }
  }
  # 存在
  have_friends(func: has(friend)) {
    name@.
    age
    number_of_friends : count(friend)
  }

}
```