# 源码分析

解析注解的部分先不做具体分析，解析注解部分是学习使用 Okhttp 的绝佳案例 

```java
  public <T> T create(final Class<T> service) {
    // 检查传入的 service 是否是接口，如果不是接口抛出异常
    validateServiceInterface(service);
    return (T)
        // 使用动态代理生成代理类
        Proxy.newProxyInstance(
            service.getClassLoader(),
            new Class<?>[] {service},
            new InvocationHandler() {
              private final Platform platform = Platform.get();
              private final Object[] emptyArgs = new Object[0];

              @Override
              public @Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args)
                  throws Throwable {
                // 如果是Object的方法，直接调用原始实现
                if (method.getDeclaringClass() == Object.class) {
                  return method.invoke(this, args);
                }

                args = args != null ? args : emptyArgs;
                // isDefaultMethod 是Java8 添加的默认方法，如果是默认方法则直接调用原始实现，如果不是则调用 retrofit 实现
                return platform.isDefaultMethod(method)
                    ? platform.invokeDefaultMethod(method, service, proxy, args)
                    : loadServiceMethod(method).invoke(args);
              }
            });
  }
```

```java
  ServiceMethod<?> loadServiceMethod(Method method) {
    // 先从缓存中获取方法实现
    ServiceMethod<?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = ServiceMethod.parseAnnotations(this, method);
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```

```java
  static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
    // 根据 method 上面的注解信息生成 RequestFactory 对象，RequestFactory 用于生成 okhttp3.Request
    RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);
    // 获取到返回类型
    Type returnType = method.getGenericReturnType();
    // 不支持 WildcardType 和  TypeVariable
    if (Utils.hasUnresolvableType(returnType)) {
      throw methodError(
          method,
          "Method return type must not include a type variable or wildcard: %s",
          returnType);
    }
    // 如果返回类型不可以是 void.class
    if (returnType == void.class) {
      throw methodError(method, "Service methods cannot return void.");
    }
    // 根据注解生成 ServiceMethod 方法，可以调用 invoke 发起网络请求
    return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
  }
```
```java
  static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
      Retrofit retrofit, Method method, RequestFactory requestFactory) {
    boolean isKotlinSuspendFunction = requestFactory.isKotlinSuspendFunction;
    boolean continuationWantsResponse = false;
    boolean continuationBodyNullable = false;

    Annotation[] annotations = method.getAnnotations();
    Type adapterType;
    // 如果是 kotlin 的挂起函数
    if (isKotlinSuspendFunction) {
      Type[] parameterTypes = method.getGenericParameterTypes();
      Type responseType =
          Utils.getParameterLowerBound(
              0, (ParameterizedType) parameterTypes[parameterTypes.length - 1]);
      if (getRawType(responseType) == Response.class && responseType instanceof ParameterizedType) {
        // Unwrap the actual body type from Response<T>.
        responseType = Utils.getParameterUpperBound(0, (ParameterizedType) responseType);
        continuationWantsResponse = true;
      } else {
      }
      // 挂起函数，如果返回值做一层 Call 封装，用于 “返回值适配器” 做类型匹配  
      adapterType = new Utils.ParameterizedTypeImpl(null, Call.class, responseType);
      annotations = SkipCallbackExecutorImpl.ensurePresent(annotations);
    } else {
      adapterType = method.getGenericReturnType();
    }
    // 匹配返回值适配器， Call 适配器
    CallAdapter<ResponseT, ReturnT> callAdapter =
        createCallAdapter(retrofit, method, adapterType, annotations);
    Type responseType = callAdapter.responseType();
    if (responseType == okhttp3.Response.class) {
      throw methodError(
          method,
          "'"
              + getRawType(responseType).getName()
              + "' is not a valid response body type. Did you mean ResponseBody?");
    }
    if (responseType == Response.class) {
      throw methodError(method, "Response must include generic type (e.g., Response<String>)");
    }
    // TODO support Unit for Kotlin?
    // HEAD 请求必须使用 void 返回类型
    if (requestFactory.httpMethod.equals("HEAD") && !Void.class.equals(responseType)) {
      throw methodError(method, "HEAD method must use Void as response type.");
    }
    // 查找 响应数据适配器 例如 Gson 适配器，String 适配器   
    Converter<ResponseBody, ResponseT> responseConverter =
        createResponseConverter(retrofit, method, responseType);

    okhttp3.Call.Factory callFactory = retrofit.callFactory;
    if (!isKotlinSuspendFunction) {
      // 包装成一个最终的适配器，包含 请求参数生成器，okhttp.call 生成器，响应数据转换器，返回类型适配器
      return new CallAdapted<>(requestFactory, callFactory, responseConverter, callAdapter);
    } else if (continuationWantsResponse) {
      //noinspection unchecked Kotlin compiler guarantees ReturnT to be Object.
      return (HttpServiceMethod<ResponseT, ReturnT>)
          new SuspendForResponse<>(
              requestFactory,
              callFactory,
              responseConverter,
              (CallAdapter<ResponseT, Call<ResponseT>>) callAdapter);
    } else {
      //noinspection unchecked Kotlin compiler guarantees ReturnT to be Object.
      return (HttpServiceMethod<ResponseT, ReturnT>)
          new SuspendForBody<>(
              requestFactory,
              callFactory,
              responseConverter,
              (CallAdapter<ResponseT, Call<ResponseT>>) callAdapter,
              continuationBodyNullable);
    }
  }
```
