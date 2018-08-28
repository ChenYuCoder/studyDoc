
http://www.cnblogs.com/huangxincheng/archive/2012/02/18/2356595.html
# show dbs
# show collections
# show users
# db.createCollection(“mycollection”)
# db.createCollection(“mycol”, { capped : true, autoIndexID : true, size : 6142800, max : 10000 } )
     字段	类型	描述
     capped	Boolean	（可选）如果为true，则启用封顶集合。封顶集合是固定大小的集合，会自动覆盖最早的条目，当它达到其最大大小。如果指定true，则需要也指定尺寸参数。
     autoIndexID	Boolean	（可选）如果为true，自动创建索引_id字段的默认值是false。
     size	number	（可选）指定最大大小字节封顶集合。如果封顶如果是 true，那么你还需要指定这个字段。
     max	number	（可选）指定封顶集合允许在文件的最大数量。
# db.CollectionName.drop()
  
# 条件：
     <	$lt
     <=	$lte
     >	$gt
     >=	$gte
    !=	$ne 代表不等于 db.Student.find({age:{$ne:33}}) 查询age不等于33
    in	$in
    !	$not
    or	$or
    and	$all
    $exists   null可以匹配自身,而且可以匹配"不存在的"
    $where function(){return this.name=="Jack"}
补充
# 插入数据：
insert({})
# 删除数据：
remove({})
# 修改数据：
update({})
## 1.$inc
    用法：{$inc:{field:value}}
    作用：对一个数字字段的某个field增加value
    示例：db.students.update({name:"student"},{$inc:{age:5}})  
## 2.$set
    用法：{$set:{field:value}}
    作用：把文档中某个字段field的值设为value
    示例：db.students.update({name:"student"},{$set:{age:23}})
## 3.$unset
    用法：{$unset:{field:1}}
    作用：删除某个字段field
    示例： db.students.update({name:"student"},{$unset:{age:1}})
## 4.$push
    用法：{$push:{field:value}}
    作用：把value追加到field里。注：field只能是数组类型，如果field不存在，会自动插入一个数组类型
    示例：db.students.update({name:"student"},{$push:{"title":"major"}}
## 5.$rename
    用法：{$rename:{old_field_name:new_field_name}}
    作用：对字段进行重命名(不是值,是字段)
    示例：db.students.update({name:"student"},{$rename:{"name":"newname"}}) 
# 查1询数据：
find()/findOne()
    第二个参数显示过滤，1显示，0不显示  ：db.MyFirstCollection.findOne({“title”:”new title”},{“description”:1,”_id”:0}); 
# 排序：
sort({})
    1:正序
    -1:倒叙
# 分页：
limit(10):读取前十个
skip(10):排除前十个

# 聚合：
## count:({'key':'value'})
## distinct:会以数组的形式返回指定键的不同值
## group
    key： 这个就是分组的key，我们这里是对年龄分组。
    keyf:有时候不能简单的用一个键进行分组，分组条件也许更复杂，$keyf函数必须返回一个对象。比如上面的例子中，想把type为1和5的商品归为一类商品来统计，那可以如下这样做：'$keyf':function(x){return {'type':x.type%4?x.type%4:4}}
    initial: 每个分组开始的函数，一般用于分组数据初始化。
    $reduce: 这个函数的第一个参数是当前的文档对象，第二个参数是上一次function操作的累计对象，第一次
    finalize:这是个函数，每一组文档执行完后，多会触发此方法，那么在每组集合里面加上count也就是它的活了。
    condition: 这个就是过滤条件。
####### FE:
        db.getCollection("SouthAir").group({"key":{"type":true},
                                                    "initial":{"analyties":[]},         
                                                    "$reduce":function(cur,prev){prev.analyties.push(cur.ts);},
                                                    "finalize":function(out){out.cout=out.analyties.length},
                                                    "condition":{"ts":{$gt:0}}
                                            })
## mapReduce
    map：这个称为映射函数，里面会调用emit(key,value)，集合会按照你指定的key进行映射分组。
    reduce：这个称为简化函数，会对map分组后的数据进行分组简化，注意：在reduce(key,value)中的key就是emit中的key，vlaue为emit分组后的emit(value)的集合，这里也就是很多{"count":1}的数组。
    mapReduce:这个就是最后执行的函数了，参数为map，reduce和一些可选参数。具体看图可知：
    finalize函数，同group的finalize完成器一样，可以对reduce的结果做一些处理；
    query文档，在map函数前对文档过滤；
    sort文档，在map函数前对文档排序，必须先对排序的字段建立索引；
    limit整数，在map函数前设定文档数量；
    scope文档，js函数中用到的变量，客户端可以通过scope传递一些值；
    jsMode布尔，指定了map和reduce函数间传递的对象使用BSON格式还是javascript对象，默认值false，表示采用BSON格式，优点是中间的BSON数据会被存在硬盘上，所以传递的数据量可以很大，但会影响性能；采用javascript对象，性能较高，但只能传递50万个不同的key值；
    verbos布尔，默认true，显示详细的时间统计信息。

# 游标
    var list=db.person.find();
    list其实并没有获取到person中的文档，而是申明一个“查询结构”，等我们需要的时候通过for或者next()一次性加载过来，然后让游标逐行读取，当我们枚举完了之后，游标销毁.



# 1、find()/findOne()条件过滤

只获取name等于jack的Student。findOne()则只获取第一条

# 2、find()/findOne()指定返回的fileds

说明：find()的第二个参数限制返回的filed的个数，0代表不返回，1代表返回。"_id"键总是会被返回。  
如果不带条件，只限制返回的filed个数的话，命令如下：db.Student.find({},{sex:0})。只需要第一个参数为{}空字典就可以。
 只获取name等于jack的Student，并且filed为name，age的数据。


# 3、查询条件

"$lt","$lte","$gt","$gte"分别对应<,<=,>,>= 
如下代码：
db.Student.find({age:{$gt:33}}) 查询age大于33的 
db.Student.find({age:{$gte:33}})
db.Student.find({age:{$lt:33}})
db.Student.find({age:{$lte:33}})
db.Student.find({age:{$gt:23,$lt:43}})

$ne 代表不等于 
db.Student.find({age:{$ne:33}}) 查询age不等于33
$in,$not和$or

# 4、特殊查询--null和exists

null可以匹配自身,而且可以匹配"不存在的"

--插入测试数据
db.Student.insert({name:null,sex:1,age:18})
db.Student.insert({sex:1,age:24})

db.Student.find({name:null})        --上面两条都能查到
db.Student.find({name:{$in:[null],$exists:true}})  ---只能查到第一条   
# 5、数组数据查询

db.Student.insert({name:"wjh",sex:1,age:18,color:["red","blue","black"]})
db.Student.insert({name:"lpj",sex:1,age:22,color:["white","blue","black"]})
db.Student.find()

--color数组中所有包含white的文档都会被检索出来
db.Student.find({color:"white"})
--color数组中所有包含red和blue的文档都会被检索出来，数组中必须同时包含red和blue，但是他们的顺序无关紧要。
db.Student.find({color:{$all:["red","blue"]}}) 
--精确匹配，即被检索出来的文档，color值中的数组数据必须和查询条件完全匹配，即不能多，也不能少，顺序也必须保持一致。
db.Student.find({color:["red","blue","black"]})
--匹配数组中指定下标元素的值。数组的起始下标是0。注意color要加引号。
db.Student.find({"color.0":"white"}) 
# 6、内嵌文档查询

----待完成-------   
# 7、排序

db.Student.find().sort({age:1})
db.Student.find().sort({age:1,sex:1})  

--1代表升序，-1代表降序
# 8、分页

db.Student.find().sort({age:1}).limit(3).skip(3) 

--limit代表取多少个document，skip代表跳过前多少个document。  
# 9、获取数量

db.Student.count({name:null})   
--或者
db.Student.find({name:null}).count()

# 删除数据

db.Student.remove({name:null})
