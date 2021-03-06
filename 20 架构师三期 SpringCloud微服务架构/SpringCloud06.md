# SpringCloud 06

## Feign

### 请求

#### API

```
@FeignClient(name = "user-provider")
public interface ConsumerApi extends UserApi {

	@GetMapping("/getMap")
	Map<Integer, String> getMap(@RequestParam("id") Integer id);
	@GetMapping("/getMap2")
	Map<Integer, String> getMap2(@RequestParam("id") Integer id,@RequestParam("name") String name);
	
	@GetMapping("/getMap3")
	Map<Integer, String> getMap3(@RequestParam Map<String, Object> map);
	
	@PostMapping("/postMap")
	Map<Integer, String> postMap(Map<String, Object> map);

}
```



#### Controller

```
package com.mashibing.UserConsumer;

import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.mashibing.UserAPI.UserApi;

@RestController
public class MainController {

	@Autowired
	ConsumerApi api;
	
	@Autowired
	MashibingApi mapi;

	@GetMapping("/alive")
	public String alive() {
		/**
		 * URL 不能变 
		 * 
		 * jar
		 * 文档
		 */
		return api.alive();
	}
	
	
	@GetMapping("/vip")
	public String vip() {
		
		return mapi.getVip();
	}
	
	@GetMapping("/map")
	public Map<Integer, String> map(Integer id) {
		System.out.println(id);
		return api.getMap(id);
	}
	
	@GetMapping("/map2")
	public Map<Integer, String> map2(Integer id,String name) {
		System.out.println(id);
		return api.getMap2(id,name);
	}
	
	
	@GetMapping("/map3")
	public Map<Integer, String> map3(@RequestParam Map<String, Object> map) {
//		System.out.println(id);
//		HashMap<String, Object> map = new HashMap<>(2);
//		
//		map.put("id", id);
//		map.put("name", name);
//		syso
		System.out.println(map);
		return api.getMap3(map);
	}
	
	
	@GetMapping("/map4")
	public Map<Integer, String> map4(@RequestParam Map<String, Object> map) {
//		System.out.println(id);
//		HashMap<String, Object> map = new HashMap<>(2);
//		
//		map.put("id", id);
//		map.put("name", name);
//		syso
		System.out.println(map);
		return api.postMap(map);
	}
}

```

#### Provider

```
package com.mashibing.UserProvider;

import java.util.Collections;
import java.util.Map;
import java.util.concurrent.atomic.AtomicInteger;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.mashibing.UserAPI.UserApi;

@RestController
public class UserController implements UserApi {

	@Value("${server.port}")
	String port;
	
	
	private AtomicInteger count = new AtomicInteger();
	
	@Override
	public String alive() {

		try {
			System.out.println("准备睡");
			
			Thread.sleep(500);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		int i = count.getAndIncrement();
		System.out.println("====好的第：" + i + "次调用");
		return "port:" + port;
	}

	@Override
	public String getById(Integer id) {
		// TODO Auto-generated method stub
		return null;
	}
	
	@GetMapping("/getMap")
	public Map<Integer, String> getMap(@RequestParam("id") Integer id) {
		// TODO Auto-generated method stub
		System.out.println(id);
		return Collections.singletonMap(id, "mmeme");
	}
	@GetMapping("/getMap2")
	public Map<Integer, String> getMap2(Integer id,String name) {
		// TODO Auto-generated method stub
		System.out.println(id);
		return Collections.singletonMap(id, name);
	}
	
	@GetMapping("/getMap3")
	public Map<Integer, String> getMap3(@RequestParam Map<String, Object> map) {
		// TODO Auto-generated method stub
		System.out.println(map);
		return Collections.singletonMap(Integer.parseInt(map.get("id").toString()), map.get("name").toString());
	}
	
	
	@PostMapping("/postMap")
	public Map<Integer, String> postMap(@RequestBody Map<String, Object> map) {
		// TODO Auto-generated method stub
		System.out.println(map);
		return Collections.singletonMap(Integer.parseInt(map.get("id").toString()), map.get("name").toString());
	}

	

}

```



### 开启日志

#### 配置文件

```
logging.level.com.mashibing.UserConsumer:debug
```

#### 重写日志等级

```java
package com.mashibing.UserConsumer;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import feign.Logger;

@Configuration
public class FeiginConfig {

	@Bean
	Logger.Level logLevel(){
		
		return Logger.Level.BASIC;
	}
}

```

### 超时

Feign默认支持Ribbon；Ribbon的重试机制和Feign的重试机制有冲突，所以源码中默认关闭Feign的重试机制,使用Ribbon的重试机制

```
#连接超时时间(ms)
ribbon.ConnectTimeout=1000
#业务逻辑超时时间(ms)
ribbon.ReadTimeout=6000
```

### 重试

```
#同一台实例最大重试次数,不包括首次调用
ribbon.MaxAutoRetries=1
#重试负载均衡其他的实例最大重试次数,不包括首次调用
ribbon.MaxAutoRetriesNextServer=1
#是否所有操作都重试
ribbon.OkToRetryOnAllOperations=false
```

使用ribbon重试机制，请求失败后，每个6秒会重新尝试

## Hystrix

spring cloud 用的是 hystrix，是一个容错组件。

Hystrix实现了 超时机制和断路器模式。

Hystrix是Netflix开源的一个类库，用于隔离远程系统、服务或者第三方库，防止级联失败，从而提升系统的可用性与容错性。主要有以下几点功能：

1. 为系统提供保护机制。在依赖的服务出现高延迟或失败时，为系统提供保护和控制。
2. 防止雪崩。
3. 包裹请求：使用HystrixCommand（或HystrixObservableCommand）包裹对依赖的调用逻辑，每个命令在独立线程中运行。
4. 跳闸机制：当某服务失败率达到一定的阈值时，Hystrix可以自动跳闸，停止请求该服务一段时间。
5. 资源隔离：Hystrix为每个请求都的依赖都维护了一个小型线程池，如果该线程池已满，发往该依赖的请求就被立即拒绝，而不是排队等候，从而加速失败判定。防止级联失败。
6. 快速失败：Fail Fast。同时能快速恢复。侧重点是：（不去真正的请求服务，发生异常再返回），而是直接失败。
7. 监控：Hystrix可以实时监控运行指标和配置的变化，提供近实时的监控、报警、运维控制。
8. 回退机制：fallback，当请求失败、超时、被拒绝，或当断路器被打开时，执行回退逻辑。回退逻辑我们自定义，提供优雅的服务降级。
9. 自我修复：断路器打开一段时间后，会自动进入“半开”状态，可以进行打开，关闭，半开状态的转换。前面有介绍。

### hystrix独立使用脱离spring cloud

```java
package com.mashibing.UserConsumer;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;

public class HystrixTest extends HystrixCommand {

	protected HystrixTest(HystrixCommandGroupKey group) {
		super(group);
		// TODO Auto-generated constructor stub
	}

	public static void main(String[] args) {

		
	//	HystrixTest hystrixTest = new HystrixTest(HystrixCommandGroupKey.Factory.asKey("ext"));
		/**
		 * execute()：以同步阻塞方式执行run()。以demo为例，调用execute()后，
		 * hystrix先创建一个新线程运行run()，
		 * 	接着调用程序要在execute()调用处一直阻塞着，直到run()运行完成 
		 */
	//	System.out.println("result:" + hystrixTest.execute());
		
		/**
		 * queue()：以异步非阻塞方式执行run()。以demo为例，
		 * 	一调用queue()就直接返回一个Future对象，
		 * 	同时hystrix创建一个新线程运行run()，
		 * 	调用程序通过Future.get()拿到run()的返回结果，
		 * 	而Future.get()是阻塞执行的
		 */
		Future<String> futureResult = new HystrixTest(HystrixCommandGroupKey.Factory.asKey("ext")).queue();
		String result = "";
		try {
			result = futureResult.get();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ExecutionException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		System.out.println("程序结果："+result);
	}

	@Override
	protected Object run() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("执行逻辑");
		int i = 1/0;
		return "ok";
	}

	@Override
	protected Object getFallback() {
		// TODO Auto-generated method stub
		return "getFallbackgetFallback";
	}
	
	
	
}

```

### 整合Resttemplate

#### Service

```
	@HystrixCommand(fallbackMethod = "back")
	public String alive() {
		// 自动处理URL
		
		RestTemplate restTemplate = new RestTemplate();
		
		String url ="http://user-provider/User/alive";
		String object = restTemplate.getForObject(url, String.class);
		
		return object;
		
	}
	
	
	public String back() {
		
		return "请求失败~bbb...";
	}
```

#### 启动类

```
@EnableCircuitBreaker
```

### 整合Feign

#### 接口

```
@FeignClient(name = "user-provider",fallback = AliveBack.class)
public interface ConsumerApi {

	@RequestMapping(value = "/User/alive",method = RequestMethod.GET)
	public String alive();
	
	@RequestMapping(value = "/User/getById",method = RequestMethod.GET)
	public String getById(Integer id);
}
```

#### 实现

```
package com.mashibing.UserConsumer;

import java.util.Map;

import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMapping;
@Component
public class AliveBack implements ConsumerApi{

	@Override
	public String alive() {
		// TODO Auto-generated method stub
		return "aaa";
	}

	@Override
	public String getById(Integer id) {
		// TODO Auto-generated method stub
		return null;
	}

}

```

## 作业：



Feign 远程调用 必须自己敲

自己整合 RestTemplate 、整合Feign