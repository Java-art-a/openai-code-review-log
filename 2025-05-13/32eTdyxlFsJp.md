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
根据提供的 `git diff` 记录，以下是对代码变更的评审：

### 1. 新增 `Message` 类
- **优点**:
  - 新增了 `Message` 类来封装微信消息的发送数据，有助于代码的封装和重用。
  - 使用 `HashMap` 来存储消息内容，增加了灵活性。

- **缺点**:
  - 类中使用了硬编码的 `touser` 和 `template_id`，这在实际应用中可能需要动态获取或配置。
  - 缺少异常处理，如果 `put` 方法中的键或值不合法，可能会导致运行时错误。

### 2. 新增 `WxAccessTokenUtils` 类
- **优点**:
  - 新增了 `WxAccessTokenUtils` 类来获取微信的访问令牌，增加了与微信API的集成。
  - 使用了 `JSON.parseObject` 来解析JSON响应，方便获取访问令牌。

- **缺点**:
  - 缺少异常处理，如果获取访问令牌失败，可能会导致程序中断。
  - 令牌的有效期未处理，可能需要实现缓存机制或定期刷新访问令牌。

### 3. 修改 `OpenAiCodeReview` 类
- **优点**:
  - 在 `main` 方法中添加了发送微信消息的逻辑，增加了与微信API的集成。
  - 使用了 `JSON.toJSONString` 来序列化消息对象，方便发送。

- **缺点**:
  - 在发送微信消息时，直接使用 `System.out.println` 打印日志，这在生产环境中可能不够健壮。
  - 在发送消息之前，没有检查微信访问令牌的有效性，可能导致发送失败。

### 4. 测试代码
- **优点**:
  - 新增了 `ApiTest` 类中的 `test_wx` 方法来测试微信消息发送功能。
  - 使用了 `sendPostRequest` 方法来发送POST请求，增加了代码的复用性。

- **缺点**:
  - 测试代码中的 `Message` 类与实际项目中使用的类不一致，可能会导致测试失败。
  - 测试代码没有模拟网络请求，可能无法准确测试微信API的响应。

### 总结
总的来说，这次代码变更增加了与微信API的集成，并提高了代码的封装性。然而，代码中存在一些潜在的问题，需要进一步改进和优化。建议增加异常处理、缓存机制和更健壮的日志记录。