
Enlarge is a tool for operating mobile phone data on the PC web page. It aims to create an open source and customizable data communication system between PC and mobile phone, so that you can easily have the powerful functions of AirDroid!

# Enlarge-Android
Enlarge Android Server On the client side, nanohttpd is used as the http and websocket service, and the annotation processor is used to automatically generate code. The directory structure is:

### AnnotationProcessor Annotation Processor Project

### Annotation Annotations and public interface definitions

### app Main Project
libs - nanohttp, fastjson, rxjavalight library reference directory
 
src/main/assets/dist - Enlarge-Web project deployment directory

src/main/com/google/zxing - QR code scanning code

src/main/org/cmdmac/enlarge/server - http and websocket service code

##### Http service entry : AppNanolets

##### Add Controller Example
```java
@Controller(name = "filemanager")
//返回给桌面的入口配置
@DesktopApp(name = "FileManager", icon = "images/ic_launcher_document.png")
public class FileManagerHandler {

    @RequestMapping(path= "list")
    public Response list(@Param(name = "dir", value = "/sdcard") String path) {
        if (TextUtils.isEmpty(path)) {
            path = Environment.getExternalStorageDirectory().getAbsolutePath();
        }
        Dir d = new Dir(new java.io.File(path));
        String json = JSON.toJSONString(d);
        Response response = Response.newFixedLengthResponse(Status.OK, "application/json", json);
        response.addHeader("Access-Control-Allow-Origin", "*");
        return response;
    }
    @RequestMapping(path= "getThumb")
    public Response getThumb(@Param(name = "path") String path) {
        try {
            Bitmap bm = FileUtils.getImageThumbnail(path, 100, 100);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            bm.compress(Bitmap.CompressFormat.PNG, 100, baos);
            Response response = Response.newFixedLengthResponse(Status.OK, "image/*", baos.toByteArray());
            response.addHeader("Access-Control-Allow-Origin", "*");
            bm.recycle();
            return response;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return Response.newFixedLengthResponse("not found");
    }

    @RequestMapping(path= "mkDir")
    public Response mkDir(@Param(name = "dir") String path, @Param(name = "name") String dirName) {
        java.io.File f = new File(path, dirName);
        JSONObject jsonObject = new JSONObject();
        if (f.mkdir()) {
            jsonObject.put("code", 200);
        } else {
            jsonObject.put("code", 500);
        }
        Response response = Response.newFixedLengthResponse(Status.OK, "application/json", jsonObject.toJSONString());
        response.addHeader("Access-Control-Allow-Origin", "*");
        return response;
    }

    @RequestMapping(path= "test")
    public Response test(@Param(name = "dir") String path, @Param(name = "testName") String dirName, int test) {
        java.io.File f = new File(path, dirName);
        JSONObject jsonObject = new JSONObject();
        if (f.mkdir()) {
            jsonObject.put("code", 200);
        } else {
            jsonObject.put("code", 500);
        }
        Response response = Response.newFixedLengthResponse(Status.OK, "application/json", jsonObject.toJSONString());
        response.addHeader("Access-Control-Allow-Origin", "*");
        return response;
    }

}
```
Controller One-click injection entrance:
```java
 public static class UriRouter extends BaseRouter {

        RouterMatcher mStaticMatcher = new RouterMatcher("", StaticPageHandler.class);
        public UriRouter() {
            super();
            // 一键注入,否则controller不生效
            ControllerInject.inject(this);
//            addRoute(StaticPageHandler.class);
            mappings.add(mStaticMatcher);

        }

        @Override
        public void addRoute(Class<?> handler) {
            mappings.add(new ControllerMatcher("/", handler));
        }

        public void addRoute(String url, Class<?> handler) {
            mappings.add(new ControllerMatcher(url, handler));
        }

        @Override
        public Response process(IHTTPSession session) {
            Response response = super.process(session);
            if (response == null) {
                return mStaticMatcher.process(null, session);
            } else {
                return response;
            }
        }
    }
```
    
##### web socket accomplish : EnlargeWebSocket

            
