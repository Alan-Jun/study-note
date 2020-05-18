## maven快速自动更新子模块项目的版本号

```
mvn versions:set -DnewVersion=[版本号]
```

## 重新拉取jar

```
mvn clean install -U
```

### 查看依赖关系**注意：** 

**注意输出结果中：**

* “+-”符号表示该包后面还有其它依赖包

* “\-”表示该包后面不再依赖其它jar包

#### 查看部分依赖 

```
mvn dependency:tree
```

#### 查看完整依赖

```
mvn dependency:tree  -Dverbose
```

#### 查看感兴趣的依赖部分

需要增加以下参数

  **-Dincludes=\*guava\* **

***includes 支持通配符的形式***

```
mvn dependency:tree -Dverbose -Dincludes=*guava*
```

## 本地打包发送到仓库

```
mvn deploy -Dmaven.test.skip=true
```

## 修改子pom中parent的version

```
mvn -N versions:update-child-modules
```

则会自动把子POM的<parent>标签中的version更新为和父POM一致。