## 简单使用

`Java API`规范 (`JSR303`) 定义了`Bean`校验的标准`validation-api`，但没有提供实现。`hibernate validation`是对这个规范的实现，并增加了校验注解如`@Email`、`@Length`等。`Spring Validation`是对`hibernate validation`的二次封装，用于支持`spring mvc`参数自动校验。接下来，我们以`spring-boot`项目为例，介绍`Spring Validation`的使用。

### 引入依赖

如果`spring-boot`版本小于`2.3.x`，`spring-boot-starter-web`会自动传入`hibernate-validator`依赖。如果`spring-boot`版本大于`2.3.x`，则需要手动引入依赖：

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.1.Final</version>
</dependency>
```

对于`web`服务来说，为防止非法参数对业务造成影响，在`Controller`层一定要做参数校验的！大部分情况下，请求参数分为如下两种形式：

1. `POST`、`PUT`请求，使用`requestBody`传递参数；
2. `GET`请求，使用`requestParam/PathVariable`传递参数。

下面我们简单介绍下`requestBody`和`requestParam/PathVariable`的参数校验实战！

### `requestBody`参数校验

`POST`、`PUT`请求一般会使用`requestBody`传递参数，这种情况下，后端使用** DTO 对象**进行接收。只要给 DTO 对象加上`@Validated`注解就能实现自动参数校验**。比如，有一个保存`User`的接口，要求`userName`长度是`2-10`，`account`和`password`字段长度是`6-20`。如果校验失败，会抛出`MethodArgumentNotValidException`异常，`Spring`默认会将其转为`400（Bad Request）`请求。

**DTO 表示数据传输对象（Data Transfer Object），用于服务器和客户端之间交互传输使用的**。在 spring-web 项目中可以表示用于接收请求参数的`Bean`对象。

- **在`DTO`字段上声明约束注解**

```java
@Data
public class UserDTO {

    private Long userId;

    @NotNull
    @Length(min = 2, max = 10)
    private String userName;

    @NotNull
    @Length(min = 6, max = 20)
    private String account;

    @NotNull
    @Length(min = 6, max = 20)
    private String password;
}
```

- **在方法参数上声明校验注解**

```java
@PostMapping("/save")
public Result saveUser(@RequestBody @Validated UserDTO userDTO) {

    return Result.ok();
}
```

这种情况下，**使用`@Valid`和`@Validated`都可以**。

### `requestParam/PathVariable`参数校验

`GET`请求一般会使用`requestParam/PathVariable`传参。如果参数比较多 (比如超过 6 个)，还是推荐使用`DTO`对象接收。否则，推荐将一个个参数平铺到方法入参中。在这种情况下，**必须在`Controller`类上标注`@Validated`注解，并在入参上声明约束注解 (如`@Min`等)**。如果校验失败，会抛出`ConstraintViolationException`异常。代码示例如下：

```java
@RequestMapping("/api/user")
@RestController
@Validated
public class UserController {

    @GetMapping("{userId}")
    public Result detail(@PathVariable("userId") @Min(10000000000000000L) Long userId) {

        UserDTO userDTO = new UserDTO();
        userDTO.setUserId(userId);
        userDTO.setAccount("11111111111111111");
        userDTO.setUserName("xixi");
        userDTO.setAccount("11111111111111111");
        return Result.ok(userDTO);
    }

    @GetMapping("getByAccount")
    public Result getByAccount(@Length(min = 6, max = 20) @NotNull String  account) {

        UserDTO userDTO = new UserDTO();
        userDTO.setUserId(10000000000000003L);
        userDTO.setAccount(account);
        userDTO.setUserName("xixi");
        userDTO.setAccount("11111111111111111");
        return Result.ok(userDTO);
    }
}
```

### 统一异常处理

前面说过，如果校验失败，会抛出`MethodArgumentNotValidException`或者`ConstraintViolationException`异常。在实际项目开发中，通常会用**统一异常处理**来返回一个更友好的提示。比如我们系统要求无论发送什么异常，`http`的状态码必须返回`200`，由业务码去区分系统的异常情况。

```java
@RestControllerAdvice
public class CommonExceptionHandler {

    @ExceptionHandler({MethodArgumentNotValidException.class})
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Result handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {
        BindingResult bindingResult = ex.getBindingResult();
        StringBuilder sb = new StringBuilder("校验失败:");
        for (FieldError fieldError : bindingResult.getFieldErrors()) {
            sb.append(fieldError.getField()).append("：").append(fieldError.getDefaultMessage()).append(", ");
        }
        String msg = sb.toString();
       return Result.fail(BusinessCode.参数校验失败, msg);
    }

    @ExceptionHandler({ConstraintViolationException.class})
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Result handleConstraintViolationException(ConstraintViolationException ex) {
        return Result.fail(BusinessCode.参数校验失败, ex.getMessage());
    }
}
```

## 进阶使用

### 分组校验

在实际项目中，可能多个方法需要使用同一个`DTO`类来接收参数，而不同方法的校验规则很可能是不一样的。这个时候，简单地在`DTO`类的字段上加约束注解无法解决这个问题。因此，`spring-validation`支持了**分组校验**的功能，专门用来解决这类问题。还是上面的例子，比如保存`User`的时候，`UserId`是可空的，但是更新`User`的时候，`UserId`的值必须`>=10000000000000000L`；其它字段的校验规则在两种情况下一样。这个时候使用**分组校验**的代码示例如下：

- **约束注解上声明适用的分组信息`groups`**

```java
@Data
public class UserDTO {

    @Min(value = 10000000000000000L, groups = Update.class)
    private Long userId;

    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 2, max = 10, groups = {Save.class, Update.class})
    private String userName;

    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 6, max = 20, groups = {Save.class, Update.class})
    private String account;

    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 6, max = 20, groups = {Save.class, Update.class})
    private String password;

    public interface Save {
    }

    public interface Update {
    }
}
```

- **`@Validated`注解上指定校验分组**

```java
@PostMapping("/save")
public Result saveUser(@RequestBody @Validated(UserDTO.Save.class) UserDTO userDTO) {

    return Result.ok();
}

@PostMapping("/update")
public Result updateUser(@RequestBody @Validated(UserDTO.Update.class) UserDTO userDTO) {

    return Result.ok();
}
```

### 嵌套校验

前面的示例中，`DTO`类里面的字段都是`基本数据类型`和`String`类型。但是实际场景中，有可能某个字段也是一个对象，这种情况先，可以使用`嵌套校验`。

比如，上面保存`User`信息的时候同时还带有`Job`信息。需要注意的是，**此时`DTO`类的对应字段必须标记`@Valid`注解**。

```java
@Data
public class UserDTO {

    @Min(value = 10000000000000000L, groups = Update.class)
    private Long userId;

    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 2, max = 10, groups = {Save.class, Update.class})
    private String userName;

    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 6, max = 20, groups = {Save.class, Update.class})
    private String account;

    @NotNull(groups = {Save.class, Update.class})
    @Length(min = 6, max = 20, groups = {Save.class, Update.class})
    private String password;

    @NotNull(groups = {Save.class, Update.class})
    @Valid
    private Job job;

    @Data
    public static class Job {

        @Min(value = 1, groups = Update.class)
        private Long jobId;

        @NotNull(groups = {Save.class, Update.class})
        @Length(min = 2, max = 10, groups = {Save.class, Update.class})
        private String jobName;

        @NotNull(groups = {Save.class, Update.class})
        @Length(min = 2, max = 10, groups = {Save.class, Update.class})
        private String position;
    }

    public interface Save {
    }

    public interface Update {
    }
}
```

嵌套校验可以结合分组校验一起使用。还有就是`嵌套集合校验`会对集合里面的每一项都进行校验，例如`List<Job>`字段会对这个`list`里面的每一个`Job`对象都进行校验。

### 集合校验

如果请求体直接传递了`json`数组给后台，并希望对数组中的每一项都进行参数校验。此时，如果我们直接使用`java.util.Collection`下的`list`或者`set`来接收数据，参数校验并不会生效！我们可以使用自定义`list`集合来接收参数：

- **包装`List`类型，并声明`@Valid`注解**

```java
public class ValidationList<E> implements List<E> {

    @Delegate
    @Valid
    public List<E> list = new ArrayList<>();

    @Override
    public String toString() {
        return list.toString();
    }
}
```

`@Delegate`注解受`lombok`版本限制，`1.18.6`以上版本可支持。如果校验不通过，会抛出`NotReadablePropertyException`，同样可以使用统一异常进行处理。

比如，我们需要一次性保存多个`User`对象，`Controller`层的方法可以这么写：

```java
@PostMapping("/saveList")
public Result saveList(@RequestBody @Validated(UserDTO.Save.class) ValidationList<UserDTO> userList) {

    return Result.ok();
}
```

### 自定义校验

业务需求总是比框架提供的这些简单校验要复杂的多，我们可以自定义校验来满足我们的需求。自定义`spring validation`非常简单，假设我们自定义`加密id`（由数字或者`a-f`的字母组成，`32-256`长度）校验，主要分为两步：

- **自定义约束注解**

```java
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = {EncryptIdValidator.class})
public @interface EncryptId {

    String message() default "加密id格式错误";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

- 实现`ConstraintValidator`接口编写约束校验器

```java
public class EncryptIdValidator implements ConstraintValidator<EncryptId, String> {

    private static final Pattern PATTERN = Pattern.compile("^[a-f\\d]{32,256}$");

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {

        if (value != null) {
            Matcher matcher = PATTERN.matcher(value);
            return matcher.find();
        }
        return true;
    }
}
```

这样我们就可以使用`@EncryptId`进行参数校验了！

### 编程式校验

上面的示例都是基于`注解`来实现自动校验的，在某些情况下，我们可能希望以`编程方式`调用验证。这个时候可以注入`javax.validation.Validator`对象，然后再调用其`api`。

```java
@Autowired
private javax.validation.Validator globalValidator;

@PostMapping("/saveWithCodingValidate")
public Result saveWithCodingValidate(@RequestBody UserDTO userDTO) {
    Set<ConstraintViolation<UserDTO>> validate = globalValidator.validate(userDTO, UserDTO.Save.class);

    if (validate.isEmpty()) {

    } else {
        for (ConstraintViolation<UserDTO> userDTOConstraintViolation : validate) {

            System.out.println(userDTOConstraintViolation);
        }
    }
    return Result.ok();
}
```

### 快速失败 (Fail Fast)

`Spring Validation`默认会校验完所有字段，然后才抛出异常。可以通过一些简单的配置，开启`Fali Fast`模式，一旦校验失败就立即返回。

```java
@Bean
public Validator validator() {
    ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
            .configure()

            .failFast(true)
            .buildValidatorFactory();
    return validatorFactory.getValidator();
}
```

### `@Valid`和`@Validated`区别

| 区别         | @Valid                                          | @Validated              |
| :----------- | :---------------------------------------------- | :---------------------- |
| 提供者       | JSR-303 规范                                    | Spring                  |
| 是否支持分组 | 不支持                                          | 支持                    |
| 标注位置     | METHOD, FIELD, CONSTRUCTOR, PARAMETER, TYPE_USE | TYPE, METHOD, PARAMETER |
| 嵌套校验     | 支持                                            | 不支持                  |

## 实现原理

### `requestBody`参数校验实现原理

在`spring-mvc`中，`RequestResponseBodyMethodProcessor`是用于解析`@RequestBody`标注的参数以及处理`@ResponseBody`标注方法的返回值的。显然，执行参数校验的逻辑肯定就在解析参数的方法`resolveArgument()`中：

```java
public class RequestResponseBodyMethodProcessor extends AbstractMessageConverterMethodProcessor {
    @Override
    public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

        parameter = parameter.nestedIfOptional();

        Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());
        String name = Conventions.getVariableNameForParameter(parameter);

        if (binderFactory != null) {
            WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
            if (arg != null) {

                validateIfApplicable(binder, parameter);
                if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
                    throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
                }
            }
            if (mavContainer != null) {
                mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());
            }
        }
        return adaptArgumentIfNecessary(arg, parameter);
    }
}
```

可以看到，`resolveArgument()`调用了`validateIfApplicable()`进行参数校验。

```java
protected void validateIfApplicable(WebDataBinder binder, MethodParameter parameter) {

    Annotation[] annotations = parameter.getParameterAnnotations();
    for (Annotation ann : annotations) {

        Validated validatedAnn = AnnotationUtils.getAnnotation(ann, Validated.class);

        if (validatedAnn != null || ann.annotationType().getSimpleName().startsWith("Valid")) {
            Object hints = (validatedAnn != null ? validatedAnn.value() : AnnotationUtils.getValue(ann));
            Object[] validationHints = (hints instanceof Object[] ? (Object[]) hints : new Object[] {hints});

            binder.validate(validationHints);
            break;
        }
    }
}
```

看到这里，大家应该能明白为什么这种场景下`@Validated`、`@Valid`两个注解可以混用。我们接下来继续看`WebDataBinder.validate()`实现。

```java
@Override
public void validate(Object target, Errors errors, Object... validationHints) {
    if (this.targetValidator != null) {
        processConstraintViolations(

            this.targetValidator.validate(target, asValidationGroups(validationHints)), errors);
    }
}
```

最终发现底层最终还是调用了`Hibernate Validator`进行真正的校验处理。

### 方法级别的参数校验实现原理

上面提到的将参数一个个平铺到方法参数中，然后在每个参数前面声明`约束注解`的校验方式，就是方法级别的参数校验。实际上，这种方式可用于任何`Spring Bean`的方法上，比如`Controller`/`Service`等。**其底层实现原理就是`AOP`，具体来说是通过`MethodValidationPostProcessor`动态注册`AOP`切面，然后使用`MethodValidationInterceptor`对切点方法织入增强**。

```java
public class MethodValidationPostProcessor extends AbstractBeanFactoryAwareAdvisingPostProcessorimplements InitializingBean {
    @Override
    public void afterPropertiesSet() {

        Pointcut pointcut = new AnnotationMatchingPointcut(this.validatedAnnotationType, true);

        this.advisor = new DefaultPointcutAdvisor(pointcut, createMethodValidationAdvice(this.validator));
    }

    protected Advice createMethodValidationAdvice(@Nullable Validator validator) {
        return (validator != null ? new MethodValidationInterceptor(validator) : new MethodValidationInterceptor());
    }
}
```

接着看一下`MethodValidationInterceptor`：

```java
public class MethodValidationInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {

        if (isFactoryBeanMetadataMethod(invocation.getMethod())) {
            return invocation.proceed();
        }

        Class<?>[] groups = determineValidationGroups(invocation);
        ExecutableValidator execVal = this.validator.forExecutables();
        Method methodToValidate = invocation.getMethod();
        Set<ConstraintViolation<Object>> result;
        try {

            result = execVal.validateParameters(
                invocation.getThis(), methodToValidate, invocation.getArguments(), groups);
        }
        catch (IllegalArgumentException ex) {
            ...
        }

        if (!result.isEmpty()) {
            throw new ConstraintViolationException(result);
        }

        Object returnValue = invocation.proceed();

        result = execVal.validateReturnValue(invocation.getThis(), methodToValidate, returnValue, groups);

        if (!result.isEmpty()) {
            throw new ConstraintViolationException(result);
        }
        return returnValue;
    }
}
```

**实际上，不管是`requestBody参数校验`还是`方法级别的校验`，最终都是调用`Hibernate Validator`执行校验，`Spring Validation`只是做了一层封装**。