---
title: 使用UEditor(JSP 1.4.3版本)遇到的问题
date: 2017-10-12 17:02:01
tags: [富文本编辑器,UEditor]
categories : 问题记录
---

![](/problems-of-using-ueditor/header-img.jpg)

## ueditor简介:

>UEditor是由百度web前端研发部开发所见即所得富文本web编辑器，具有轻量，可定制，注重用户体验等特点，开源基于MIT协议，允许自由使用和修改代码...

## 遇到问题及解决方案

### setContent报错

#### 描述
当使用setContent给初始化容器设置内容时出现如下错误：
![](/problems-of-using-ueditor/2-1.png)

#### 原因
异步，初始化容器并未创建完成，需等待编辑器创建完成后才能使用。

#### 解决

##### 方法一
使用setTimeout延迟一段时间后调用（不建议：由于setTimeout设置的时间与实际渲染需要的时间不一致）；

##### 方法二
使用addListener('ready',callback)，待编辑器创建完成后执行callback回调。
```javascript
ue.addListener('ready', function () {
   ue.setContent(content);
});
```
### 上传文件——“未找到上传数据”

#### 描述
无论上传什么文件均出现如下错误：
![](/problems-of-using-ueditor/2-2.png)

#### 原因

1. struts2：
struts2 框架默认使用 apache 的 commons-fileUpload 组件和内建的 FileUploadInterceptor 拦截器实现上传，会将 request 文件域封装到 action 中一个 File 类型的属性中，并删除 request 中的文件域，因此会上传文件失败。
2. springMVC：
UEditor 默认使用 commons 组件，而 springMVC 对 commons 组件进行了封装，使得上传后获取不到文件。

#### 解决

1. struts2：
自定义一个 struts 过滤器，指定不对 ueditor/jsp/ 目录下的 jsp 页面进行过滤。
```java
public class MyStrutsFilter extends StrutsPrepareAndExecuteFilter{
    @Override
    public void doFilter(ServletRequest req, ServletResponse res,
            FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        String url = request.getRequestURI();         
        System.out.println(url);         
        if (url.contains(request.getContextPath()+"/js/utf8-jsp/jsp/controller.jsp")) {             
            System.out.println("使用自定义过滤器");             
            chain.doFilter(req, res);         
        }else{             
            System.out.println("使用默认过滤器");             
            super.doFilter(req, res, chain);         
        } 
    }
}
```
2. springMVC：
修改源码 upload 包下的 BinaryUploader.java 文件。
下载并使用源码：
![](/problems-of-using-ueditor/2-3.png)
重写 BinaryUploader.java ：
```java
public class BinaryUploader {
    public static final State save(HttpServletRequest request,Map<String, Object> conf) {
        InputStream fileStream = null;
        if (!ServletFileUpload.isMultipartContent(request)) {
            return new BaseState(false, AppInfo.NOT_MULTIPART_CONTENT);
        }
        try {
            //修改了百度使用原生的commons上传方式
            DefaultMultipartHttpServletRequest multipartRequest=(DefaultMultipartHttpServletRequest)request;
            Iterator<String> fileNames=multipartRequest.getFileNames();
            MultipartFile file=null;
            while (fileNames.hasNext()){
                file=multipartRequest.getFiles(fileNames.next()).get(0);
                fileStream=file.getInputStream();
            }
            if (fileStream == null) {
                return new BaseState(false, AppInfo.NOTFOUND_UPLOAD_DATA);
            }
            String savePath = (String) conf.get("savePath");
            String originFileName = file.getOriginalFilename();
            String suffix = FileType.getSuffixByFilename(originFileName);
            originFileName = originFileName.substring(0,originFileName.length() - suffix.length());
            savePath = savePath + suffix;
            long maxSize = ((Long) conf.get("maxSize")).longValue();
            if (!BinaryUploader.validType(suffix, (String[]) conf.get("allowFiles"))) {
                return new BaseState(false, AppInfo.NOT_ALLOW_FILE_TYPE);
            }
            savePath = PathFormat.parse(savePath, originFileName);
            String physicalPath = (String) conf.get("rootPath") + savePath;
            State storageState = StorageManager.saveFileByInputStream(fileStream,physicalPath, maxSize);
            fileStream.close();
            if (storageState.isSuccess()) {
                storageState.putInfo("url", PathFormat.format(savePath));
                storageState.putInfo("type", suffix);
                storageState.putInfo("original", originFileName + suffix);
            }
            return storageState;
        } catch (ClassCastException e) {
            return new BaseState(false, AppInfo.PARSE_REQUEST_ERROR);
        }catch (IOException e){
            return new BaseState(false, AppInfo.IO_ERROR);
        }
    }
    
    private static boolean validType(String type, String[] allowTypes) {
        List<String> list = Arrays.asList(allowTypes);
        return list.contains(type);
    }
}
```

### 上传文件消失
    
#### 描述
使用 tomcat ，已上传文件在重新部署项目后丢失。

#### 原因
UEditor 上传配置文件 config.json 默认将上传文件上传到项目中，重新部署会清空项目文件。

#### 解决

##### 方法一：修改上传路径到webapps目录下
第一步：
将配置文件中上传保存路径 
`/ueditor/jsp/upload/image/{yyyy}{mm}{dd}/{time}{rand:6}`
修改为 
`/../ueditor/jsp/upload/image/{yyyy}{mm}{dd}/{time}{rand:6}`

第二步：
修改返回文件请求地址，将重写后 BinaryUploader.java 第33行的 
`PathFormat.format(savePath)`
修改为 
`PathFormat.format(savePath).replace("/..", "")`

##### 方法二：配置 tomcat 虚拟目录
第一步：
打开 %TOMCAT_HOME%/conf/server.xml（即tomcat的安装目录下面相关的文件），在 Host 中加入如下代码：
```xml
<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Context path="/ueditor" docBase="D:\ueditor" reloadable="true"></Context>       
</Host>
```

第二步：
修改上传路径，将重写后 BinaryUploader.java 第29行的 
`(String) conf.get("rootPath") + savePath`
修改为 
`"D://" + savePath`

**上传路径与虚拟目录 docBase 保持一致**