---
layout:     post
title:      "SpringBoot下返回Json的方法"
subtitle:   ""
date:       2018-12-2 16：19：00
author:     "Patrick"
header-img: ""
catalog:    true
tags:
    - Java
    - SpringBoot
---


**1.直接返回对象**

代码

	 
    @PostMapping("/test")
    public User test(@RequestParam String account, @RequestParam String password) {
        User user = userService.findUserByAccount(account);
        return user;    
	}

测试结果

	{
		"lastModified":null,
		"id":12061,
		"account":"guochao",
		"name":"guochao",
		"company":"",
		"department":"",
		"position":null,
		"email":"",
		"phone":"",
		"mobile":null,
		"im":"",
		"userCreateType":"add",
		"userbaseStatus":"normal",
		"publicKey":null,
		"privateKey":null,
		"serialNumber":null,
		"userbaseIsAvailable":null,
		"appName":null,
		"photo":null,
		"registerFrom":null,
		"creator":null,
		"lastLoginTime":null,
		"lastLoginIp":null
	}

**2.返回List**

代码

	@PostMapping("/test")
    public List<User> test(@RequestParam String account, @RequestParam String password) {
        List<User> list = new ArrayList<>();
        User user = userService.findUserByAccount(account);
        list.add(user);
        list.add(user);
        return list;
    }

测试结果

	[
		{
			"lastModified":null,
			"id":12061,
			"account":"guochao",
			"name":"guochao",
			"company":"",
			"department":"",
			"position":null,
			"email":"",
			"phone":"",
			"mobile":null,
			"im":"",
			"userCreateType":"add",
			"userbaseStatus":"normal",
			"publicKey":null,
			"privateKey":null,
			"serialNumber":null,
			"userbaseIsAvailable":null,
			"appName":null,
			"photo":null,
			"registerFrom":null,
			"creator":null,
			"lastLoginTime":null,
			"lastLoginIp":null
		},
		{
			"lastModified":null,
			"id":12061,
			"account":"guochao",
			"name":"guochao",
			"company":"",
			"department":"",
			"position":null,
			"email":"",
			"phone":"",
			"mobile":null,
			"im":"",
			"userCreateType":"add",
			"userbaseStatus":"normal",
			"publicKey":null,
			"privateKey":null,
			"serialNumber":null,
			"userbaseIsAvailable":null,
			"appName":null,
			"photo":null,
			"registerFrom":null,
			"creator":null,
			"lastLoginTime":null,
			"lastLoginIp":null
		}
	]

**3.返回Map**

代码

    @PostMapping("/test")
    public Map<String,User> test(@RequestParam String account, @RequestParam String password) {
        Map<String,User> map = new HashMap<>();
        User user = userService.findUserByAccount(account);
        map.put("user1", user);
        map.put("user2", user);
        return map;
    }

测试结果
	
	{
	"user1":{
		"lastModified":null,
		"id":12061,
		"account":"guochao",
		"name":"guochao",
		"company":"",
		"department":"",
		"position":null,
		"email":"",
		"phone":"",
		"mobile":null,
		"im":"",
		"userCreateType":"add",
		"userbaseStatus":"normal",
		"publicKey":null,
		"privateKey":null,
		"serialNumber":null,
		"userbaseIsAvailable":null,
		"appName":null,
		"photo":null,
		"registerFrom":null,
		"creator":null,
		"lastLoginTime":null,
		"lastLoginIp":null
	},
	"user2":{
		"lastModified":null,
		"id":12061,
		"account":"guochao",
		"name":"guochao",
		"company":"",
		"department":"",
		"position":null,
		"email":"",
		"phone":"",
		"mobile":null,
		"im":"",
		"userCreateType":"add",
		"userbaseStatus":"normal",
		"publicKey":null,
		"privateKey":null,
		"serialNumber":null,
		"userbaseIsAvailable":null,
		"appName":null,
		"photo":null,
		"registerFrom":null,
		"creator":null,
		"lastLoginTime":null,
		"lastLoginIp":null
	}
	}

**4.返回JsonObject**

代码

    @PostMapping("/test")
    public String test(@RequestParam String account, @RequestParam String password) {
        User user = userService.findUserByAccount(account);
        JSONObject json = new JSONObject();
        json.element("resultFlag", "success");
        json.element("user", user);
        return json.toString();
    }

测试结果

	{
		"resultFlag":"success",
		"user":{
			"userbaseIsAvailable":"",
			"registerFrom":"",
			"publicKey":"",
			"lastLoginIp":"",
			"userbaseStatus":"normal",
			"userCreateType":"add",
			"company":"",
			"id":12061,
			"department":"",
			"email":"",
			"creator":"",
			"serialNumber":"",
			"im":"",
			"appName":"",
			"mobile":"",
			"photo":"",
			"lastLoginTime":null,
			"privateKey":"",
			"phone":"",
			"name":"guochao",
			"lastModified":null,
			"position":"",
			"account":"guochao"
		}
	}

**5.直接拼接String**

代码

	    public String verifyPassword(@RequestParam String account, @RequestParam String password) {
	        User user = userService.findUserByAccount(account);
	        Boolean flag = false;
	        if (UserServiceImpl.validatePassword(password,user.getPassword())) {
	            flag = true;
	        }
	        return "{ \"verifyResult\":"+flag+",\"success\":true}";    
		}


测试结果


	{
		"verifyResult":true,
		"success":true
	}