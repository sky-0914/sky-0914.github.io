---
title: SpringBoot 全局异常处理
tags:
  - SpringBoot
  - 全局异常
categories:
  - SpringBoot
---
# SpringBoot 全局异常处理

```java
@Slf4j
@RestControllerAdvice
public class ExceptionController {

    /**
     * 校验错误拦截处理
     *
     * @param ex 异常
     * @return 返回值
     * StringBuilder sb = new StringBuilder();
     * for (FieldError error : ex.getBindingResult().getFieldErrors()) {
     * sb.append(error.getDefaultMessage()).append(";");
     * }
     */
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public ApiMessage<Object> methodArgumentNotValidHandler(MethodArgumentNotValidException ex) {
        //按需重新封装需要返回的错误信息
        List<ArgumentInvalid> invalidArguments = new ArrayList<>();
        //解析原错误信息，封装后返回，此处返回非法的字段名称，原始值，错误信息
        ex.getBindingResult().getFieldErrors().forEach(fieldError -> invalidArguments.add(new ArgumentInvalid(fieldError.getField(), fieldError.getRejectedValue(), fieldError.getDefaultMessage())));
        return new ApiMessage<>(ExceptionCode.PARAMETER_ERROR, invalidArguments);
    }
  
    @ExceptionHandler(value = BindException.class)
    public ApiMessage<Object> bindExceptionHandler(BindException ex) {
        //按需重新封装需要返回的错误信息
        List<ArgumentInvalid> invalidArguments = new ArrayList<>();
        //解析原错误信息，封装后返回，此处返回非法的字段名称，原始值，错误信息
        ex.getBindingResult().getFieldErrors().forEach(fieldError -> invalidArguments.add(new ArgumentInvalid(fieldError.getField(), fieldError.getRejectedValue(), fieldError.getDefaultMessage())));
        return new ApiMessage<>(ExceptionCode.PARAMETER_ERROR, invalidArguments);
    }

    /**
     * HTTP请求方式不正确
     *
     * @param ex 异常
     * @return 返回值
     */
    @ExceptionHandler(value = HttpRequestMethodNotSupportedException.class)
    public ApiMessage<Object> httpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException ex) {
        log.error("HTTP请求方式不正确：【{}】", ex.getMessage());
        return new ApiMessage<>(ex);
    }

    /**
     * 请求参数不全
     *
     * @param ex 异常
     * @return 返回值
     */
    @ExceptionHandler(value = MissingServletRequestParameterException.class)
    public ApiMessage<Object> missingServletRequestParameterException(MissingServletRequestParameterException ex) {
        log.error("请求参数不全：【{}】", ex.getMessage());
        return new ApiMessage<>(ex);
    }

    /**
     * 请求参数类型不正确
     *
     * @param ex 异常
     * @return 返回值
     */
    @ExceptionHandler(value = TypeMismatchException.class)
    public ApiMessage<Object> typeMismatchException(TypeMismatchException ex) {
        log.error("请求参数类型不正确：【{}】", ex.getMessage());
        return new ApiMessage<>(ex);
    }

    /**
     * 数据格式不正确
     *
     * @param ex 异常
     * @return 返回值
     */
    @ExceptionHandler(value = DataFormatException.class)
    public ApiMessage<Object> dataFormatException(DataFormatException ex) {
        log.error("数据格式不正确：【{}】", ex.getMessage());
        return new ApiMessage<>(ex);
    }

    /**
     * 非法输入或断言错误
     *
     * @param ex 异常
     * @return 返回值
     */
    @ExceptionHandler(value = IllegalArgumentException.class)
    public ApiMessage<Object> illegalArgumentException(IllegalArgumentException ex) {
        log.error("非法输入或断言错误：【{}】", ex.getMessage());
        return new ApiMessage<>(ex);
    }

    /**
     * 请求参数错误
     *
     * @param ex 异常
     * @return 返回值
     */
    @ExceptionHandler(value = ConstraintViolationException.class)
    public ApiMessage<Object> constraintViolationException(ConstraintViolationException ex) {
        log.error("请求参数错误：【{}】", ex.getMessage());
        return new ApiMessage<>(ex);
    }

    /**
     * 操作数据库出现异常
     *
     * @param ex 异常
     * @return 返回值
     */
    @ExceptionHandler(value = DataAccessException.class)
    public ApiMessage<Object> dataDoException(DataAccessException ex) {
        log.error("操作数据库出现异常：【{}】", ex.getMessage());
        return new ApiMessage<>(ex);
    }

    /**
     * 系统异常
     *
     * @param ex 异常
     * @return 返回值
     */
    @ExceptionHandler(Exception.class)
    public ApiMessage<Object> apiExceptionHandler(Exception ex) {
        //只打印15行的错误堆栈
        int count = 1;
        StringBuilder sb = new StringBuilder();
        for (StackTraceElement stackTraceElement : ex.getStackTrace()) {
            sb.append(stackTraceElement.toString());
//            if (count++ >= 30) {
//                break;
//            }
            sb.append("\n");
        }
        log.error("系统异常：【{}】", sb.toString());
        return new ApiMessage<>(ex);
    }

    /**
     * 自定义异常
     *
     * @param apiException 自定义异常
     * @return 返回值
     */
    @ExceptionHandler(ApiException.class)
    public ApiMessage<Object> apiException(ApiException apiException) {
        return new ApiMessage<>(apiException);
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @EqualsAndHashCode(callSuper = false)
    static class ArgumentInvalid {
        /**
         * 字段
         */
        private String field;
        /**
         * 字段值
         */
        private Object rejectedValue;
        /**
         * 默认值
         */
        private String defaultMessage;
    }
}
```

