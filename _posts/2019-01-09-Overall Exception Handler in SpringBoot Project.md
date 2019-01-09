---
layout:     post
title:      "SpringBoot中异常统一处理"
subtitle:   ""
date:       2019-01-09 16：19：00
author:     "Patrick"
header-img: ""
catalog:    true
tags:
    - SpringBoot
    - Java
    - Exception
---
异常处理的代码如下，可以根据不同的异常设置不同的Status_code,在这个类中进行异常的日志记录和返回处理。

	/**
	 * 异常处理拦截器
	 * @author patrick
	 *
	 */
	@ControllerAdvice
	public class OverallExceptionResolver {
		@Autowired
		private LogService logService;
		 //记录数据库最大字符长度  
	    private static final int WIRTE_DB_MAX_LENGTH = 50; 
	    ActionMessage actionMessage = null;
		
		private static Log log = Log.getLog(OverallExceptionResolver.class);
		
	    @ExceptionHandler(value = Exception.class)
		@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
	    public @ResponseBody String serviceCommonExceptionHandler(Exception exception) {
	        //对捕获的异常进行处理并打印日志等，之后返回json数据，方式与Controller相同
			Logger logger = LoggerFactory.getLogger("debug");
	
	    	
			if (exception instanceof NotAuthorizedException) {
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_NOT_LOGIN, exception.getMessage());
			} else if(exception instanceof UnauthenticatedException) {
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_NOT_LOGIN, "请登录后再进行操作！");
			} else if (exception instanceof PermissionsException){
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_NO_PERMISSION, exception.getMessage());
			} else if (exception instanceof GroupsException){
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_BAD_INPUT, exception.getMessage());
			}else {
				exception.printStackTrace();
				actionMessage = new ActionMessage(ActionMessage.Type.WARN, 
						ActionMessage.CODE_FORBIDDEN, exception.getMessage());
				actionMessage.getDetail();
				
			}
			
			String exceptionMessage = actionMessage.getDetail();  
	        if(exceptionMessage != null){  
	            if(exceptionMessage.length() > WIRTE_DB_MAX_LENGTH){  
	                exceptionMessage = exceptionMessage.substring(0,WIRTE_DB_MAX_LENGTH);  
	            }  
	        } 
	        
			if (!(exception instanceof NotAuthorizedException || exception instanceof UnauthenticatedException)) {
				logService.addErrorLog(actionMessage.getType().name(), exceptionMessage);
			}
	        return "{\"type\":\"error\",\"success\":\"false\",\"code\":\"300\",\"detail\":\""+exception.getMessage()+"\"}";
	    }
	    @ExceptionHandler(value = NotAuthorizedException.class)
		@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
	    public @ResponseBody String serviceAuthExceptionHandler(Exception exception) {
	        //对捕获的异常进行处理并打印日志等，之后返回json数据，方式与Controller相同
			Logger logger = LoggerFactory.getLogger("debug");
	
	    	
			if (exception instanceof NotAuthorizedException) {
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_NOT_LOGIN, exception.getMessage());
			} else if(exception instanceof UnauthenticatedException) {
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_NOT_LOGIN, "请登录后再进行操作！");
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_SYSTEM_STOPED, exception.getMessage());
			} else if (exception instanceof PermissionsException){
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_NO_PERMISSION, exception.getMessage());
			} else if (exception instanceof GroupsException){
				actionMessage = new ActionMessage(ActionMessage.Type.ERROR,
						ActionMessage.CODE_BAD_INPUT, exception.getMessage());
			}else {
				exception.printStackTrace();
				actionMessage = new ActionMessage(ActionMessage.Type.WARN, 
						ActionMessage.CODE_FORBIDDEN, exception.getMessage());
				actionMessage.getDetail();
				
			}
			
			String exceptionMessage = actionMessage.getDetail();  
	        if(exceptionMessage != null){  
	            if(exceptionMessage.length() > WIRTE_DB_MAX_LENGTH){  
	                exceptionMessage = exceptionMessage.substring(0,WIRTE_DB_MAX_LENGTH);  
	            }  
	        } 
	        
			if (!(exception instanceof NotAuthorizedException || exception instanceof UnauthenticatedException)) {
				logService.addErrorLog(actionMessage.getType().name(), exceptionMessage);
			}
	        return "{\"type\":\"error\",\"success\":\"false\",\"code\":\"300\",\"detail\":"+exception.getMessage()+"\"}";
	    }
	
	}

