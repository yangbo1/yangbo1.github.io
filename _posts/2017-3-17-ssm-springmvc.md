---
layout: post
title:  "Springmvc笔记"
categories: springmvc
tags:  springmvc ssm
author: yangbo
---

* content
{:toc}
springmvc学习笔记




# springmvc

Spring web mvc和Struts2都属于表现层的框架,它是Spring框架的一部分,我们可以从Spring的整体结构中看得出来：

![](http://a1.qpic.cn/psb?/V11TGf4q1ABjBk/sLrE183MCrJzbgIq7UDyX.DIlFSLXEipohc5Uuv*zkk!/m/dPgAAAAAAAAAnull&bo=UAJjAVACYwEFCSo!&rf=photolist&t=5)

# springmvc执行流程

![](http://a1.qpic.cn/psb?/V11TGf4q1ABjBk/fPizefZNKBOT1kxp*b*63hMKgB4fLDj7IGAvoc1Vetw!/m/dLAAAAAAAAAAnull&bo=PAPAATwDwAEFCSo!&rf=photolist&t=5)

# 入门程序

## Controller，实现Controller接口

```java
package com.itheima.springmvc.controller;

import com.itheima.springmvc.po.Items;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by 杨波 on 2017/3/7.
 */
public class ItemsController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        //模拟商品列表
        List<Items> itemsList = new ArrayList<Items>();
        //商品列表
        Items items_1 = new Items();
        items_1.setName("联想笔记本");
        items_1.setPrice(6000f);
        items_1.setDetail("ThinkPad T430 联想笔记本电脑！");

        Items items_2 = new Items();
        items_2.setName("苹果手机");
        items_2.setPrice(5000f);
        items_2.setDetail("iphone6苹果手机！");

        itemsList.add(items_1);
        itemsList.add(items_2);
        //使用request传递结果
        //request.setAttribute("itemsList", itemsList);
        //创建一个ModelAndView对象
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("itemsList", itemsList);
        //添加视图
        modelAndView.setViewName("itemsList");

        return modelAndView;
    }
}

```

## Springmvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-3.2.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-3.2.xsd ">
    <!-- 处理器的映射器,如果在映射器中添加扩展时必须在配置文件中配置-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
    <!-- 注解形式的处理器映射器 -->
    <!-- <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/> -->
    <!-- 处理器适配器 -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
    <bean class="org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter"/>
    <!-- 注解形式的处理器适配器 -->
    <!-- <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/> -->

    <!-- 注解驱动，配置了这个节点就相当于配置了注解形式的处理器映射器和适配器 -->
    <mvc:annotation-driven/>
    <!-- 配置扫描包 -->
    <context:component-scan base-package="com.itheima.springmvc.controller"/>

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 配置前缀和后缀 -->
        <property name="prefix" value="/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- 使用bean的name属性配置url和handler的映射关系 -->
    <bean name="/itemsList.action" id="itemsController" class="com.itheima.springmvc.controller.ItemsController"/>
    <!-- 实现HttpRequestHandler接口的controller -->
    <!--<bean name="/itemsList2.action" class="com.itheima.springmvc.controller.ItemsController2"/>-->
</beans>
```

## Controller实现HttpRequestHandler接口

```java
package com.itheima.springmvc.controller;

import com.itheima.springmvc.po.Items;
import org.springframework.web.HttpRequestHandler;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by 杨波 on 2017/3/7.
 */
public class ItemsController2 implements HttpRequestHandler {
    public void handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws ServletException, IOException {
        //模拟商品列表
        List<Items> itemsList = new ArrayList<Items>();
        //商品列表
        Items items_1 = new Items();
        items_1.setName("联想笔记本");
        items_1.setPrice(6000f);
        items_1.setDetail("ThinkPad T430 联想笔记本电脑！");

        Items items_2 = new Items();
        items_2.setName("苹果手机");
        items_2.setPrice(5000f);
        items_2.setDetail("iphone6苹果手机！");

        itemsList.add(items_1);
        itemsList.add(items_2);
        //使用request域向页面传递数据
        request.setAttribute("itemsList", itemsList);
        //跳转到页面
        request.getRequestDispatcher("/WEB-INF/jsp/itemsList.jsp").forward(request, response);
    }
}

```

Springmvc.xml

```xml
<bean name="/itemsList2.action" class="com.itheima.springmvc.controller.ItemsController2"/>
```

## 使用注解方式开发controller

```java
package com.itheima.springmvc.controller;

import com.itheima.springmvc.po.Items;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by 杨波 on 2017/3/7.
 */
@Controller
public class ItemsController3 {
    //@RequestMapping配置url和方法之间的映射关系。
    @RequestMapping("/itemsList3.action")
    public ModelAndView itemsList() throws Exception {
        //模拟商品列表
        List<Items> itemsList = new ArrayList<Items>();
        //商品列表
        Items items_1 = new Items();
        items_1.setName("联想笔记本");
        items_1.setPrice(6000f);
        items_1.setDetail("ThinkPad T430 联想笔记本电脑！");

        Items items_2 = new Items();
        items_2.setName("苹果手机");
        items_2.setPrice(5000f);
        items_2.setDetail("iphone6苹果手机！");

        itemsList.add(items_1);
        itemsList.add(items_2);

        //创建ModelAndView对象
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("itemsList", itemsList);
        //添加视图
        modelAndView.setViewName("itemsList");
        return modelAndView;
    }
}

```

springmvc.xml

```xml
<!-- 配置扫描包 -->
    <context:component-scan base-package="com.itheima.springmvc.controller"/>
```

# 总结

非注解形式

* 实现Controller接口


* 实现HttpRequestHandler接口


* 配置url和handler直接的映射关系

在springmvc.xml中配置bean的name属性。

注解形式

* 在Controller类上添加@Controller注解
* 映射关系使用@RequestMapping在方法上配置。
# springmvc的框架结构

![](http://a4.qpic.cn/psb?/V11TGf4q1ABjBk/nk*VyGQ6WXnDznTeibisxiJcQ645WsRMO69Ta6Hss*U!/m/dIMBAAAAAAAAnull&bo=FwNIAhcDSAIFCSo!&rf=photolist&t=5)

流程

* 前端控制器：接收用户请求，响应用户。负责整个处理流程的调度。可以降低各个组件之间耦合。
* 处理器映射器：根据url查找handler处理器。记录了url和handler之间的映射关系。可以使用bean的name属性建立也可以使用注解方法建立。返回一个执行链包含拦截器和handler。
* 处理器适配器：根据不同的handler选择不同的适配器。可以有多种适配器。
* **处理器**：handler，相当于struts2的action，需要程序员开发。可以有多种方式，实现Controller接口，实现HttprequestHandler接口，使用注解。
* 视图解析器：把逻辑视图转换成物理视图。最常见的视图就是jsp。其他的视图，freemaker、pdf、excel。
* **视图**：包含很多种，最常见的就是jsp。渲染视图把model中的数据填充到jsp中。默认支持jstl。Jsp也需要开发的

# 参数绑定

## 修改商品信息

controller

```java
package com.itheima.ssm.controller;

import com.itheima.ssm.po.Items;
import com.itheima.ssm.service.ItemsService;
import com.sun.javafx.sg.prism.NGShape;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

/**
 * Created by 杨波 on 2017/3/8.
 */
@Controller
public class ItemsController {
    @Autowired
    private ItemsService itemsService;
    @RequestMapping("/itemsList")
    //参数Model，当springmvc执行时，会自动创建一个model对象传递过来。
    public String itemsList(Model model) throws Exception {
        //查询商品列表
        List<Items> itemsList = itemsService.queryItemsList();
        model.addAttribute("itemsList", itemsList);
        //返回逻辑视图
        return "itemsList";
    }

    //根据页面参数id查询items回显
    @RequestMapping("/editItems")
    public String editItems(Integer id,Model model) throws  Exception{
        Items items = itemsService.findItemsById(id);
        model.addAttribute("items",items);
        return "editItems";
    }
    //修改数据提交
    @RequestMapping("/editItemsSubmit")
    public String editItemsSubmit(Items items) throws Exception{
        itemsService.updateItems(items);
        return "success";
    }
}

```

service

```java
package com.itheima.ssm.service.impl;

import com.itheima.ssm.mapper.ItemsMapper;
import com.itheima.ssm.po.Items;
import com.itheima.ssm.po.ItemsExample;
import com.itheima.ssm.service.ItemsService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * Created by 杨波 on 2017/3/8.
 */
@Service
public class ItemsServiceImpl implements ItemsService {
    @Autowired
    private ItemsMapper itemsMapper;
    @Override
    public List<Items> queryItemsList() throws Exception {
        List<Items> list = itemsMapper.selectByExampleWithBLOBs(new ItemsExample());
        return list;
    }

    @Override
    public Items findItemsById(int id) {
        Items items = itemsMapper.selectByPrimaryKey(id);
        return items;
    }

    @Override
    public void updateItems(Items items) {
        itemsMapper.updateByPrimaryKeyWithBLOBs(items);
    }
}

```

# springmvc默认支持的参数类型

* HttpServletRequest

* HttpServletResponse

* HttpSession

* Model/ModelMap

# @RequestParam

* 可以指定要绑定参数的名称

  `@RequestParam(value="iid")Integer id`


* 可以指定默认值

  `@RequestParam(value="iid", defaultValue="1")Integer id`

* 可以指定参数是不是可以为null，如果不加此注解默认可以为null，加上了默认是不可为null。

#  实现自定义converter

创建一个converter类需要实现Converter接口

```java
//<S, T>
//s：源类型
//t:目标数据类型
public class CustomerConverter implements Converter<String, Date> {

	@Override
	public Date convert(String source) {
		//创建日期格式
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		try {
			Date date = dateFormat.parse(source);
			return date;
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		return null;
	}
```

在springmvc.xml中配置

```xml
<!-- 配置Converter -->
		<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="converters">
			<list>
				<bean class="com.itheima.ssm.controller.converter.CustomerConverter"/>
			</list>
		</property>
		</bean>
```

```xml
<!-- 注解驱动，配置了这个节点就相当于配置了注解形式的处理器映射器和适配器 -->
	<mvc:annotation-driven conversion-service="conversionService"/>
```

# ssm整合的案例
[github地址](https://github.com/yangbo1/ssm_01)
