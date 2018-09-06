log打点总结
===
原则：
* 目的：可以通过message确定所在代码位置，根据data可以重现问题，如果message过于普通，可写成className-methodName：message，data：{}。
* log等级为：
  *  info: 在某些重要正常流程
  	```
	logger.info("begin refresh weather.." + System.currentTimeMillis());
	正确：
	logger.info("begin refresh weather,time:{}",System.currentTimeMillis());
	//time 指数据类型
	```
  *  warn: 数据异常、数据不合法等导致数据被抛弃
	```
	if (imagePairs == null) {
		logger.info("news id:" + jc.getId());
		continue;
	}
	正确：
	if (imagePairs == null) {
		logger.warn("the news's imagePairs is null,news:{}",JSON.toJSONString(jc));
		continue;
	}
	```
  *  error: 异常报错
  	```
	try{
		...
	}catch(Exception e){
		logger.error(e.getMessage(), e);
	}
	正确：
	try{
		...
	}catch(Exception e){
		logger.error("className-methodName：do something has error，data：{}",data, e);
		//do something 指try内做的具体业务
	}
	```
  *  LogMarker.MAIL标记级别: 严重异常报错，导致重要业务被阻塞
  	```
	try{
		...
	}catch(Exception e){
		logger.error(e.getMessage(), e);
	}
	正确：
	try{
		...
	}catch(Exception e){
		logger.error(LogMarker.MAIL,"className-methodName：do something has error，data：{}",data, e);
		//do something 指try内做的具体业务
	}
	```






