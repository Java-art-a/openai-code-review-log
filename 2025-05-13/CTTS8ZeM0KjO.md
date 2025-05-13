diff --git a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
index d67ca87..def40c2 100644
--- a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
+++ b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
@@ -181,6 +181,7 @@ public class OpenAiCodeReview {
         String accessToken = WxAccessTokenUtils.getAccessToken();
 
         Message message = new Message();
+        message.setUrl(logUrl);
         message.put("project", "big-market");
         message.put("review", logUrl);
 
diff --git a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/domain/model/Message.java b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/domain/model/Message.java
index 8d07964..b9ea631 100644
--- a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/domain/model/Message.java
+++ b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/domain/model/Message.java
@@ -7,7 +7,7 @@ public class Message {
 
     private String touser = "oHUCK6gQcrxx13jEJvA4Bmk42NnM";
     private String template_id = "H4KIXvvHsWtbEEnRSE_ZUzRqZbVi9jZrN4cIVfFfdX8";
-    private String url = "https://github.com/Java-art-a/openai-code-review-log/blob/main/2025-05-13/kjQmtlIQ9Vyk.md";
+    private String url = "";
     private String topcolor = "#FF0000";
     private Map<String, Map<String, String>> data = new HashMap<>();
 
以下是对提供的Git diff记录的代码评审：

**文件：a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java**

变更：
- 在`OpenAiCodeReview`类的某个方法中，给`Message`对象设置了`url`属性，并将其值设置为`logUrl`。

**评审：**
- 这个变更看起来是为了将某个日志的URL传递给`Message`对象，以便在消息中包含该URL。这是一个合理的功能，因为可能需要将日志链接发送给用户。
- 确保`logUrl`变量已经被正确定义，并且包含有效的URL。
- 考虑到`logUrl`可能包含敏感信息或指向不可公开的内容，可能需要验证该URL是否安全，以及它是否应该被公开。

**文件：a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/domain/model/Message.java**

变更：
- 在`Message`类中，`url`字段的默认值从`https://github.com/Java-art-a/openai-code-review-log/blob/main/2025-05-13/kjQmtlIQ9Vyk.md`更改为空字符串。

**评审：**
- 将`url`字段的默认值设置为空字符串可能意味着该字段需要被外部设置，或者不再需要包含一个默认的URL。
- 如果`url`字段不再需要默认值，这个变更可能是合理的。
- 如果`url`字段仍然需要默认值，那么应该设置一个合理的默认URL，或者删除该字段，并确保调用者负责提供该值。

**总体建议：**
- 确保所有的变更都有充分的文档记录，特别是当它们涉及外部资源（如URL）时。
- 在代码审查过程中，检查是否有任何潜在的硬编码或敏感信息泄露。
- 如果`logUrl`是一个动态生成的URL，确保它不会暴露任何敏感数据。
- 如果这个变更是为了修复一个已知的问题或者添加一个新的功能，那么应该更新相关的测试用例以确保功能按预期工作。