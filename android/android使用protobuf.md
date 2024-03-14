# Android使用protobuf
## 1. 安装 protobuf
mac 安装方式：
```
// 确保 mac 已安装 brew，执行如下命令
brew install protobuf
```

## 2. 编写 proto 文件
语法可参考：https://juejin.cn/post/7135365943282122765#heading-19
```
message Weather{
  int32 query = 1;

  //季节
  enum Season{
    //允许对枚举常量设置别名
    option allow_alias = true;
    //枚举里面的 = 操作是对常量进行赋值操作
    //春
    SPRING = 0;
    //夏
    SUMMER = 1;
    //秋 如果不设置别名，不允许存在两个相同的值
    FALL = 2;
    AUTUMN = 2;
    //冬
    WINTER = 3;
  }

  //对 season 进行编号
  Season season = 2;
}
```

## 3. 编译 protoc 生成 java 文件
```
protoc --java_out=${"你要生成的 Java 文件目录"} ${"Protobuf 文件位置"}

//因为android使用的是javalite版本，这里命令如下，举例：
protoc --java_out=lite:./app/src/main/java ./app/src/main/proto/student.proto

//如果使用的是java版本则：
protoc --java_out=:./app/src/main/java ./app/src/main/proto/student.proto
```

## 4. android 项目导入 protobuf 依赖库
因为生成的 Java 文件中会使用 protobuf 依赖库里面的功能，需添加该依赖到项目的 build.gradle
```
implementation 'com.google.protobuf:protobuf-javalite:3.19.2'
```