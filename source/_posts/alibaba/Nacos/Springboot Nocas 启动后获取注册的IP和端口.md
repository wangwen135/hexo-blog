---
title: Springboot Nocas 启动后获取注册的IP和端口
date: 2021-11-18 17:10
tags: 
  - Springboot
  - Nacos
categories:
  - [alibaba,Nacos]
---

```
@Component
public class TestApplicationRunner implements ApplicationRunner {

    @Autowired
    private NacosDiscoveryProperties nacosDiscoveryProperties;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("启动后运行");
        System.out.println(nacosDiscoveryProperties.getIp());
        System.out.println(nacosDiscoveryProperties.getPort());
    }

}
```
