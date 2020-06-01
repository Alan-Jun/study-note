# 创建的jsp文件没有内容

![1542259134122](assets\1542259134122.png)

模板内容：

```jsp
<%--
  Created by IntelliJ IDEA.
  User: name
  Date: ${DATE}
  Time: ${TIME}
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<html>
  <head>
    
    <base href="<%=basePath%>">
    <title>#[[$Title$]]#</title>
  </head>
  <body>
  #[[$END$]]#
  </body>
</html>
```

