# Http 接口命名规范

## 请求方式
* get：获取数据，抓取数据
* post：增加，修改，删除

## 命名规则
> 动词-名词-目的-条件，根据功能进行具体修改modifyTemplateToOn或者publishTemplate都可以，描述的都是将模版上架
* 功能命名
  * 增加：create
    * createOrder：创建订单
  * 查询：get
    * getOrderDetail：获取个体详情（通过ID查询）
    * getOrdersCount：获取集合数量
    * getOrdersList：获取集合数据
    * getOrderStatus：获取个体某字段
    * getOrderDetailByCondition：通过特殊条件查询个体（Condition代指条件）
  * 修改：modify
    * modifyOrder：全量修改（通过ID，如果不想修改某些字段需要将原值传回，取消只传部分字段的逻辑）
    * modifyOrderStatus：修改个别字段（但是如果某个修改存在逻辑限制，则一定要按业务进行拆分）
  * 删除：remove
    * removeOrder：删除数据（通过ID）
    * removeOrderByCondition：通过条件删除数据

* 业务命名
  * modifyTemplateStatus 修改模版状态（发布，编辑，下架其实都是对该记录进行状态修改，但是每个业务实现都不一样，所以此时需要按着业务进行拆分）
    * modifyTemplateToOn：修改模版为上架
    * modifyTemplateToEdit：修改模版为编辑中
    * modifyTemplateToOff：修改模版为下架