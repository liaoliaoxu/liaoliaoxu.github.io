---
layout:     post
title:      Validated
subtitle:   参数校验
date:       2019-09-23
author:     Link
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - Java
    - Spring
---
# 参数校验

## 需求

参数校验代码繁复

   ```java
    P.throwMessageExceptionIf(null == planId, 200, "planId 不能为空");
    P.throwMessageExceptionIf(null == contentType, 200, "内容类型不能为空");
    P.throwMessageExceptionIf(null == jumpType, 200, "跳转类型不能空");
    P.throwMessageExceptionIf(StringUtils.isEmpty(fireworkName), 200, "弹屏命名不能为空");
    P.throwMessageExceptionIf(null == h5Type && StringUtil.isEmpty(jumpUrl), 200, "跳转链接不能为空");
   ```

## 已有轮子

### javax.validation

   ```xml
   <dependency>
         <groupId>javax.validation</groupId>
         <artifactId>validation-api</artifactId>
         <version>2.0.1.Final</version>
   </dependency>
   ```

### org.springframework.validation

> spring自带

### spring boot 中使用

   ```xml
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
   </dependency>

   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

如果有了spring-boot-starter-web就不需要spring-boot-starter-validation

## 基本使用

### 1. 字段注解

> 注解修饰字段，表示一种校验方式

#### 部分注解功能

```java
//修饰的字段在验证时必须不为null，否则验证无法通过。
@NotNull(message = "can't be null")
private String name;

////修饰的字段在验证时必须不为空，否则验证无法通过。
@NotEmpty(message = "can't be empty")
private String name;

//如下代码表示，修饰的字段长度不能超过5或者低于。
@Size(min = 1, max = 5)
private List<String> names;

//如下代码表示，该字段的最大值为19，否则无法通过验证。
@Max(value = 19)
private Integer age;

//同理，被该注解修饰的字段的最小值，不能低于某个值。
@Min(value = 1)
private Integer age;

//验证数字的最大值。
@DecimalMax(value = "12.35")
private double money;

//验证数字的最小值。
@DecimalMin(value = "0.3")
private double money;

//用于验证字段是否与给定的正则相匹配。
@Pattern(regexp = "[abc]")
private String name;
```

#### 注解通用属性

1. message, string, 校验失败后的报错信息
2. group, interface.class, 分组功能（默认为Default.class)
   即可以在校验时指定分组，针对不同的校验需求，采取不同策略。如下面代码，指定TestGroup.class进行校验时, 只会对appId进行校验。
3. 一个简单的校验实体类

   ```java
   @Data
   public class AppPublish {

      @NotNull(message = "appId is null", groups = TestGroup.class)
      private Integer appId;

      @NotNull(message = "version is null")
      @Pattern(regexp = "\\d+(\\.\\d+)*", message = "version is error, please check!")
      private String version;

      @Max(value = 100, message = "num is to big!")
      @Min(value = 1, message = "num is to small!")
      @NotNull(message = "percent can't be null!")
      private Integer percent;

      @Valid
      private InnerPublish publish;

      @Size(min = 1, max = 4, message = "list.size must be between 1 and 4")
      @NotNull(message = "list can't be null!")
      private List<String> list;
   }
   ```

#### 自定义校验方式

> 除了以上的注解，也可以自己实现注解

1. 自定义注解

   ```java
   @Documented
   @Constraint(validatedBy = { PasswordEqualsValidator.class })
   @Target({ ElementType.METHOD, ElementType.FIELD, ElementType.TYPE })
   @Retention(RetentionPolicy.RUNTIME)
   public @interface Password {

      String message() default "Password is not the same";

      Class<?>[] groups() default {};

      Class<? extends Payload>[] payload() default {};
   }
   ```

2. 校验器

   ```java
   public class PasswordEqualsValidator implements ConstraintValidator<Password, RegisterForm> {

      @Override
      public void initialize(Password anno) {
      }

      @Override
      public boolean isValid(RegisterForm form, ConstraintValidatorContext context) {
         String confirm = form.getConfirm();
         String password = form.getPassword();

         boolean match = password != null && password.equals(confirm);
         if (match) {
               return true;
         }

         String messageTemplate = context.getDefaultConstraintMessageTemplate();

         // disable default violation rule
         context.disableDefaultConstraintViolation();

         // assign error on password Confirm field
         context.buildConstraintViolationWithTemplate(messageTemplate).addPropertyNode("confirm")
                  .addConstraintViolation();
         return false;
      }
   }
   ```

3. 实体类

   ```java
   @Data
   @Password(message = "password is not same!")
   public class RegisterForm {

      @NotNull(message = "name can't be null")
      private String name;

      @NotNull(message = "password can't be null")
      private String password;

      @NotNull(message = "confirm can't be null")
      private String confirm;
   }
   ```

### 2. 注解加异常处理器使用

#### 注解

> @Valid, @Validated

1. controller层直接使用，可以使用@Validated和@Valid两个注解，后者无法使用分组功能，抛出BindException(表单数据), MethodArgumentNotValidException(json数据)

   ```java
   @PostMapping("/controller")
   public BaseResult controller(@Validated(TestGroup.class) @RequestBody AppPublish appPublish{
      return new DataResult("success");
   }

   //or
   @PostMapping("/controller")
   public BaseResult controller(@Valid @RequestBody AppPublish appPublish{
      return new DataResult("success");
   }
   ```

2. 任意地方使用，抛出ConstraintViolationException

   ```java
   @Service
   @Validated
   public class SignatureServiceImpl implements ISignatureService {

      @Override
      //如果默认分组，可以省略以下注解
      @Validated(TestGroup.class)
      public void testCheck(@Valid AppPublish appPublish, @NotNull(message = "desc can't be null") String desc){

      }
   }
   ```

#### 异常处理

> 实现异常处理器即可对校验失败的结果进行报警
> 这种方法有一个问题，不同地方校验所产生的异常不同

1. 一个简单的异常处理器

   ```java
   @Slf4j
   public class BindExceptionResolver extends AbstractHandlerExceptionResolver {

      private Charset charset = Charset.forName(Constant.UTF8);

      private static String JSON_CONTENT_TYPE = "text/plain;charset=UTF-8";

      @Override
      protected ModelAndView doResolveException(HttpServletRequest request, HttpServletResponse response,
                                                Object handler, Exception ex) {
         if(ex instanceof BindException){
            //controller params
         }
         if(ex instanceof ConstraintViolationException){
            //exception in others
         }
         if(ex instanceof MethodArgumentNotValidException){
            //controller json
         }
         return null;
      }

      @Override
      public int getOrder() {
         return Ordered.HIGHEST_PRECEDENCE;
      }
   ```

2. 注意getOrder()方法，这里将该异常处理器的优先级设置为最高，以免被其他默认的异常处理器（如spring自带的异常处理器）处理报出其他不想要的结果。

### util主动调用

> 不依赖spring
> 需要主动调用

1. 实现方式

   ```java
   ValidateUtils.validate(appPublish, Default.class);

   //utils
   public class ValidateUtils {

      //默认校验器
      private static Validator validator = Validation.buildDefaultValidatorFactory().getValidator();

      public static <T> void validate(T obj, Class group){
         //校验过程
         Set<ConstraintViolation<T>> validates = validator.validate(obj, group);
         if(CollectionUtils.isEmpty(validates)){
            return;
         }
         List<String> messages = validates.stream().map(ConstraintViolation::getMessage).collect(Collectors.toList());
         String message = splice(messages);
         throw new MessageException(message, 400);
      }

      private static String splice(List<String> str){
         StringBuilder sb = new StringBuilder();
         str.forEach(s -> sb.append(s).append(", "));
         String result = sb.toString();
         return result.substring(0, result.length() - 3);
      }
   }
   ```

2. 当然一般也是要实现异常处理的，只不过自定义可以控制抛出什么异常，比如mobile的MessageException
3. 无法实现该情况下对单一参数的校验，如String desc

   ```java
   @Service
   @Validated
   public class SignatureServiceImpl implements ISignatureService {

      @Override
      //如果默认分组，可以省略以下注解
      @Validated(TestGroup.class)
      public void testCheck(@Valid AppPublish appPublish, @NotNull(message = "desc can't be null") String desc){

      }
   }
   ```

## 总结

### 使用框架校验参数的优势

1. 代码简洁
2. 容易实现错误信息的整合
   1. 原来的校验方式，如果contentType，jumpType两个都出错，只能返回第一个错误（当然也可以整合，只是比较麻烦，而且代码更乱了）

      ```java
      P.throwMessageExceptionIf(null == contentType, 200, "内容类型不能为空");
      P.throwMessageExceptionIf(null == jumpType, 200, "跳转类型不能空");
      ```

   2. 采用校验框架是可以在一个地方实现信息的整合，因为一个异常包含了这次校验所有错误的信息

      ```java
      Set<ConstraintViolation<T>> validates = validator.validate(obj, group);
      if(CollectionUtils.isEmpty(validates)){
         return;
      }
      List<String> messages = validates.stream().map(ConstraintViolation::getMessage).collect(Collectors.toList());
      String message = splice(messages);
      ```

3. 框架校验方式对比

   | 方式       | @Validated                     | Util                     |
   | ----------| ------------------------------ | ------------------------ |
   | 代码整洁   | 注解更为整洁                   | 需要主动调用             |
   | 单参数校验 | 可以实现                       | 无法实现                 |
   | 实现      | 需实现spring指定的多歌异常处理 | 对结果进行统一的异常处理 |
