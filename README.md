# ShopKasei


## 通用请求响应封装
### 响应结果枚举类
ResponseCode是枚举类，存储了多种响应情况的code和对应desc  
###### 枚举类一般内部先定义属性，使用逗号分割每个属性，名字都全大写。 
```java
public enum ResponseCode {
    SUCCESS(0,"SUCCESS"),
    ERROR(1,"ERROR");

    private static final int code;
    private static final String desc;

    ResponseCode(int code,String desc){
        this.code = code;
        this.desc = desc;
    }
}
```
上面代码貌似类似于这样
```jaba
public class ResponseCode{
    private final int code;
    private final String desc;

    ResponseCode(int code,String desc){
        this.code = code;
        this.desc = desc;
    }

    public ResponseCode SUCCESS = new ResponseCode(0,"SUCCESS");
    public ResponseCode ERROR = new ResponseCode(1,"ERROR");

}
```
###### final关键字：修饰后继承类不能修改该属性。

### 服务响应类
封装了多种方法如createBySuccess, createByError等，调用后可返回各个情况下成功或失败的json格式数据
```java
@JsonSerialize(include =  JsonSerialize.Inclusion.NON_NULL)
public class ServerResponse<T> implements Serializable {
    private int status;
    private String msg;
    private T data;

    private ServerResponse(int status) {
        this.status = status;
    }

    private ServerResponse(int status, String msg) {
        this.status = status;
        this.msg = msg;
    }

    @JsonIgnore
    public boolean isSuccess() {
        return this.status == ResponseCode.SUCCESS.getCode();
    }

    public static <T> ServerResponse<T> createBySuccess() {
        return new ServerResponse<T>(ResponseCode.SUCCESS.getCode());
    }

    public static <T> ServerResponse<T> createByError() {
        return new ServerResponse<T>(ResponseCode.ERROR.getCode(), ResponseCode.ERROR.getDesc());
    }
}
```
@JsonSerialize(include =  JsonSerialize.Inclusion.NON_NULL)注解表示类自动转json，参数可以让null的数据不出现在返回的json中    
@JsonIgnore作用使之不在json序列化结果当中  
implements Serializable因为Java规定继承该类才能序列化（转json）  
ServerResponse<T> 泛型<T>表示该类实例化时，必须传入任意类型的一个参数。  
public static <T> ServerResponse<T>这里static后面的\<T>是语法规定，后面的ServerResponse<T>这个<>中出现的，必须要在前面static<>中有出现过。

## 用户模块
###### 横向越权：攻击者尝试访问与他拥有相同权限的用户的资源。例如重置密码api如果没有token的情况
###### 纵向越权：低级别攻击者尝试访问高级别用户的资源  
