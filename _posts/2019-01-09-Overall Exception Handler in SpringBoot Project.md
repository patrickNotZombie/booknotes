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
	public class GlobalExceptionHandler {
 
	    Logger log = LoggerFactory.getLogger(OverallExceptionResolver.class);
	 
	    @ExceptionHandler(value = IllegalArgumentException.class)
	    @ResponseBody
	    public ResponseEntity<String> parameterErrorHandler(HttpServletRequest req, IllegalArgumentException e) {
	        return new ResponseEntity(e.getMessage(), HttpStatus.BAD_REQUEST);
	    }
	
			
	    @ExceptionHandler(value = Exception.class)
		@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
		public @ResponseBody
		String serviceCommonExceptionHandler(HttpServletRequest req, Exception exception) {
			String url = req.getRequestURI();
			log.error("request error at " + url, exception);
			JSONObject json = new JSONObject();
			json.put("detail", exception.getMessage());
			json.put("code",500);
			return json.toString();
		}
	}
