diff --git a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
index d88ecae..d67ca87 100644
--- a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
+++ b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
@@ -2,8 +2,10 @@ package cn.flybird.sdk;
 
 import cn.flybird.sdk.domain.model.ChatCompletionRequest;
 import cn.flybird.sdk.domain.model.ChatCompletionSyncResponse;
+import cn.flybird.sdk.domain.model.Message;
 import cn.flybird.sdk.domain.model.Model;
 import cn.flybird.sdk.types.utils.BearerTokenUtils;
+import cn.flybird.sdk.types.utils.WxAccessTokenUtils;
 import com.alibaba.fastjson2.JSON;
 import org.eclipse.jgit.api.Git;
 import org.eclipse.jgit.api.errors.GitAPIException;
@@ -18,6 +20,7 @@ import java.text.SimpleDateFormat;
 import java.util.ArrayList;
 import java.util.Date;
 import java.util.Random;
+import java.util.Scanner;
 
 public class OpenAiCodeReview {
     public static void main(String[] args) {
@@ -65,6 +68,10 @@ public class OpenAiCodeReview {
             String logUrl = writeLog(token, diffCode + log);
             System.out.println("writeLog: " + logUrl);
 
+            //4. 发送通知
+            pushMessage(logUrl);
+            System.out.println("pushMessage: " + logUrl);
+
         } catch(Exception e) {
             e.printStackTrace();
             Thread.currentThread().interrupt();
@@ -170,4 +177,42 @@ public class OpenAiCodeReview {
 
     }
 
+    private static void pushMessage(String logUrl) {
+        String accessToken = WxAccessTokenUtils.getAccessToken();
+
+        Message message = new Message();
+        message.put("project", "big-market");
+        message.put("review", logUrl);
+
+        String url = String.format("https://api.weixin.qq.com/cgi-bin/message/template/send?access_token=%s", accessToken);
+        sendPostRequest(url, JSON.toJSONString(message));
+    }
+
+    private static void sendPostRequest(String urlString, String jsonString) {
+        try {
+
+            URL url = new URL(urlString);
+            HttpURLConnection connection = (HttpURLConnection)url.openConnection();
+            connection.setRequestMethod("POST");
+            connection.setRequestProperty("Content-Type", "application/json; utf-8");
+            connection.setRequestProperty("Accept", "application/json");
+            connection.setDoOutput(true);
+
+            try (OutputStream os = connection.getOutputStream()) {
+                byte[] input = jsonString.getBytes(StandardCharsets.UTF_8);
+                os.write(input, 0, input.length);
+            }
+
+            try (Scanner scanner = new Scanner(connection.getInputStream(), StandardCharsets.UTF_8.name())) {
+                String response = scanner.useDelimiter("\\A").next();
+                System.out.println(response);
+            }
+
+        } catch (Exception e) {
+            e.printStackTrace();
+        }
+
+
+    }
+
 }
diff --git a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/domain/model/Message.java b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/domain/model/Message.java
new file mode 100644
index 0000000..8d07964
--- /dev/null
+++ b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/domain/model/Message.java
@@ -0,0 +1,62 @@
+package cn.flybird.sdk.domain.model;
+
+import java.util.HashMap;
+import java.util.Map;
+
+public class Message {
+
+    private String touser = "oHUCK6gQcrxx13jEJvA4Bmk42NnM";
+    private String template_id = "H4KIXvvHsWtbEEnRSE_ZUzRqZbVi9jZrN4cIVfFfdX8";
+    private String url = "https://github.com/Java-art-a/openai-code-review-log/blob/main/2025-05-13/kjQmtlIQ9Vyk.md";
+    private String topcolor = "#FF0000";
+    private Map<String, Map<String, String>> data = new HashMap<>();
+
+    public void put(String key, String value) {
+        data.put(key, new HashMap<String, String>() {
+            {
+                put("value", value);
+            }
+        });
+    }
+
+    public String getTouser() {
+        return touser;
+    }
+
+    public void setTouser(String touser) {
+        this.touser = touser;
+    }
+
+    public Map<String, Map<String, String>> getData() {
+        return data;
+    }
+
+    public void setData(Map<String, Map<String, String>> data) {
+        this.data = data;
+    }
+
+    public String getTopcolor() {
+        return topcolor;
+    }
+
+    public void setTopcolor(String topcolor) {
+        this.topcolor = topcolor;
+    }
+
+    public String getUrl() {
+        return url;
+    }
+
+    public void setUrl(String url) {
+        this.url = url;
+    }
+
+    public String getTemplate_id() {
+        return template_id;
+    }
+
+    public void setTemplate_id(String template_id) {
+        this.template_id = template_id;
+    }
+}
+
diff --git a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/types/utils/WxAccessTokenUtils.java b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/types/utils/WxAccessTokenUtils.java
new file mode 100644
index 0000000..83a0a14
--- /dev/null
+++ b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/types/utils/WxAccessTokenUtils.java
@@ -0,0 +1,74 @@
+package cn.flybird.sdk.types.utils;
+
+import com.alibaba.fastjson2.JSON;
+
+import java.io.BufferedReader;
+import java.io.InputStreamReader;
+import java.net.HttpURLConnection;
+import java.net.URL;
+
+public class WxAccessTokenUtils {
+
+    private static final String APPID = "wx64852572c187cdd8";
+    private static final String APPSECRET = "2ae89a546d3275e33b4a59b6e710d0c9";
+    private static final String GRANT_TYPE = "client_credential";
+    private static final String URL_TEMPLATE = "https://api.weixin.qq.com/cgi-bin/token?grant_type=%s&appid=%s&secret=%s";
+
+    public static String getAccessToken() {
+        try {
+            String urlString = String.format(URL_TEMPLATE, GRANT_TYPE, APPID, APPSECRET);
+            URL url = new URL(urlString);
+            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
+            connection.setRequestMethod("GET");
+
+            int responseCode = connection.getResponseCode();
+            System.out.println("Response code: " + responseCode);
+
+            if (responseCode == HttpURLConnection.HTTP_OK) {
+                BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream()));
+                String inputLine;
+                StringBuilder response = new StringBuilder();
+
+                while((inputLine = in.readLine()) != null) {
+                    response.append(inputLine);
+                }
+                in.close();
+
+                System.out.println("Response: " + response.toString());
+
+                Token token = JSON.parseObject(response.toString(), Token.class);
+
+                return token.getAccess_token();
+            } else {
+                System.out.println("GET request failed");
+                return null;
+            }
+
+        } catch (Exception e) {
+            e.printStackTrace();
+            return null;
+        }
+    }
+
+    public static class Token {
+        private String access_token;
+        private Integer expires_in;
+
+        public String getAccess_token() {
+            return access_token;
+        }
+
+        public void setAccess_token(String access_token) {
+            this.access_token = access_token;
+        }
+
+        public Integer getExpires_in() {
+            return expires_in;
+        }
+
+        public void setExpires_in(Integer expires_in) {
+            this.expires_in = expires_in;
+        }
+    }
+
+}
diff --git a/openai-code-review-sdk/src/test/java/cn/flybird/sdk/test/ApiTest.java b/openai-code-review-sdk/src/test/java/cn/flybird/sdk/test/ApiTest.java
index 3d7310a..b48970a 100644
--- a/openai-code-review-sdk/src/test/java/cn/flybird/sdk/test/ApiTest.java
+++ b/openai-code-review-sdk/src/test/java/cn/flybird/sdk/test/ApiTest.java
@@ -2,6 +2,7 @@ package cn.flybird.sdk.test;
 
 import cn.flybird.sdk.domain.model.ChatCompletionSyncResponse;
 import cn.flybird.sdk.types.utils.BearerTokenUtils;
+import cn.flybird.sdk.types.utils.WxAccessTokenUtils;
 import com.alibaba.fastjson2.JSON;
 import lombok.extern.slf4j.Slf4j;
 import org.junit.Test;
@@ -17,6 +18,9 @@ import java.net.HttpURLConnection;
 import java.net.URL;
 import java.nio.charset.StandardCharsets;
 import java.sql.SQLOutput;
+import java.util.HashMap;
+import java.util.Map;
+import java.util.Scanner;
 
 public class ApiTest {
 
@@ -51,8 +55,8 @@ public class ApiTest {
                 + "    }"
                 + "]"
                 + "}";
-        
-        try(OutputStream os = connection.getOutputStream()) {
+
+        try (OutputStream os = connection.getOutputStream()) {
             byte[] input = jsonInputString.getBytes(StandardCharsets.UTF_8);
             os.write(input);
         }
@@ -64,7 +68,7 @@ public class ApiTest {
         String inputLine;
 
         StringBuilder content = new StringBuilder();
-        while((inputLine = in.readLine()) != null) {
+        while ((inputLine = in.readLine()) != null) {
             content.append(inputLine).append(System.lineSeparator());
         }
 
@@ -77,4 +81,101 @@ public class ApiTest {
 
     }
 
+    @Test
+    public void test_wx() {
+        String accessToken = WxAccessTokenUtils.getAccessToken();
+
+        Message message = new Message();
+        message.put("project", "big-market");
+        message.put("review", "feat: 新加功能");
+
+        String url = String.format("https://api.weixin.qq.com/cgi-bin/message/template/send?access_token=%s", accessToken);
+        sendPostRequest(url, JSON.toJSONString(message));
+
+    }
+
+    private static void sendPostRequest(String urlString, String jsonString) {
+
+        try {
+
+            URL url = new URL(urlString);
+            HttpURLConnection connection = (HttpURLConnection)url.openConnection();
+            connection.setRequestMethod("POST");
+            connection.setRequestProperty("Content-Type", "application/json; utf-8");
+            connection.setRequestProperty("Accept", "application/json");
+            connection.setDoOutput(true);
+
+            try (OutputStream os = connection.getOutputStream()) {
+                byte[] input = jsonString.getBytes(StandardCharsets.UTF_8);
+                os.write(input, 0, input.length);
+            }
+
+            try (Scanner scanner = new Scanner(connection.getInputStream(), StandardCharsets.UTF_8.name())) {
+                String response = scanner.useDelimiter("\\A").next();
+                System.out.println(response);
+            }
+
+        } catch (Exception e) {
+            e.printStackTrace();
+        }
+
+
+    }
+
+    public static class Message {
+        private String touser = "oHUCK6gQcrxx13jEJvA4Bmk42NnM";
+        private String template_id = "H4KIXvvHsWtbEEnRSE_ZUzRqZbVi9jZrN4cIVfFfdX8";
+        private String url = "https://github.com/Java-art-a/openai-code-review-log/blob/main/2025-05-13/kjQmtlIQ9Vyk.md";
+        private String topcolor = "#FF0000";
+        private Map<String, Map<String, String>> data = new HashMap<>();
+
+        public void put(String key, String value) {
+            data.put(key, new HashMap<String, String>() {
+                {
+                    put("value", value);
+                }
+            });
+        }
+
+        public String getTouser() {
+            return touser;
+        }
+
+        public void setTouser(String touser) {
+            this.touser = touser;
+        }
+
+        public Map<String, Map<String, String>> getData() {
+            return data;
+        }
+
+        public void setData(Map<String, Map<String, String>> data) {
+            this.data = data;
+        }
+
+        public String getTopcolor() {
+            return topcolor;
+        }
+
+        public void setTopcolor(String topcolor) {
+            this.topcolor = topcolor;
+        }
+
+        public String getUrl() {
+            return url;
+        }
+
+        public void setUrl(String url) {
+            this.url = url;
+        }
+
+        public String getTemplate_id() {
+            return template_id;
+        }
+
+        public void setTemplate_id(String template_id) {
+            this.template_id = template_id;
+        }
+    }
+
 }
根据提供的Git diff记录，以下是对代码变更的评审：

### OpenAiCodeReview.java
1. **新增依赖**: 引入了`Message`和`WxAccessTokenUtils`类。这表明代码的功能有所扩展，加入了微信消息推送功能。
2. **main方法修改**: 在`main`方法中，新增了发送微信消息的代码。这可能是为了在代码审查完成后通知相关人员。
3. **异常处理**: 在`main`方法中，异常处理被修改为调用`Thread.currentThread().interrupt()`。这是正确的做法，因为它会重新设置线程的中断状态，允许调用者有机会检查中断状态并相应地处理。

### Message.java
1. **新增类**: `Message`类被添加到代码库中。这个类看起来是用来构造微信模板消息的。
2. **Map结构**: 类中使用了一个嵌套的`HashMap`来存储消息的键值对。这是一种常见的方式，但需要注意保持代码的可读性和可维护性。

### WxAccessTokenUtils.java
1. **新增类**: `WxAccessTokenUtils`类被添加到代码库中。这个类负责获取微信的访问令牌。
2. **API调用**: 类中包含了获取访问令牌的HTTP GET请求，这是一个常见的做法，但需要注意异常处理和错误处理。

### ApiTest.java
1. **新增测试**: 新增了一个测试类`ApiTest`，其中包含了对微信消息发送功能的测试。
2. **测试方法**: 测试方法中模拟了发送微信消息的过程，并打印了响应。这是一个很好的实践，因为它可以帮助确保功能按预期工作。

### 评审总结
- **功能扩展**: 新增的微信消息推送功能是一个有用的特性，但需要确保其稳定性和安全性。
- **代码质量**: 新增的类和方法应该遵循良好的编码规范，包括适当的命名、注释和异常处理。
- **测试**: 新增的测试是很好的，但应该进一步扩展以确保所有功能都得到测试。

### 建议
- **代码审查**: 在合并这些更改之前，应该进行代码审查，以确保代码质量和一致性。
- **测试**: 确保所有新功能都经过充分的测试，包括单元测试和集成测试。
- **文档**: 如果这些功能是公共API的一部分，那么应该更新文档以反映这些变化。