## Retrofit
1. 请求方式  
  `@GET("/users/list")` or `@GET("/users/list?sort=desc")`
2. URL 处理  
  请求的URL可以根据函数的参数动态更新。一个可替换的区块用`{}`包围字段，然后用`@Path`注解的方式标明，注意：注解的参数为同样的字符串
  ```
  @GET("/group/{id}/users") // 注意 字符串id
  List<User> groupList(@Path("id") int groupId); // 注意 Path注释的参数要和前面的字符串一样
  ```
3. 支持查询参数
  ```
  @GET("/group/{id}/users")
  List<User> groupList(@Pah(id) int groupId, @Query("sort") String sort);
  ```
4. 请求体  
  通过`@Body`注解声明一个对象作为请求体发送到服务器  
  ```
  @POST("/users/new")
  void createUser(@Body User user, Callback<User> cb);
  ```
  对象将被`RestAdapter` 使用对应的转换器转换为字符串或者字节流提交到服务器.

5. `FOTM ENCODED AND mULTIPART` 表单和`Multipart`
  函数也可以注解为发送表单和multipart数据  
  使用@FormUrlEncoded 注解来发送表单数据;使用@Field注解和参数来指定每个表单项的key,valuse为参数的值。
  ```
  @FormUrlEncoded
  @POST("user/edit")
  User update(@Field(first_name) sring fisrt, @Field("last_name") String last);
  ```
  使用@Multipart 注解来发送multipart数据。使用@Part注解 要发送的每文件
  ```
  @Multipart
  @PUT("/user/photo")
  User updateUser(@Part("photo") TypedFile photo, @Part(descript) TypedString description);
  ```
  Multipart 中的Part使用RestAdapter 的转换器来转换也可以实现TypeOutput 来自己处理序列化。
6. 异步和同步
  每个函数都可以定义为异步或者同步。  
  具有返回值的函数为同步执行。  
  ```
  @GET("/user/{id}/photo")
  Photo listUsers(@Path("id") int id);
  ```
  异步执行函数没有返回值并且要求函数最后一个参数为Callback对象
  ```
  @GET("/user/{id}/photo")
  void listUsers(@Path("id") int id, Callback<Photo> cb);
  ```
  Android 上，callback会在主线程中调用，而普通java应用中callback在请求执行的线程中调用

7. 服务器结果转换为Java对象
  使用`RestAdapter`的转换器把Http请求的结果默认为JSON，转换为Java对象，对象通过函数返回值或者Callback接口定义  
  ```
  @GET("/users/list")
  List<User> userList();

  @GET("/users/list")
  void userList(Callback<List<User>> cb);
  ```
  直接获取HTTP返回的对象，则使用`Response`对象
  ```
  @GET("/users/list")
  Response userList();

  @GET("/users/list")
  void userList(Callback<Response> cb);
  ```
