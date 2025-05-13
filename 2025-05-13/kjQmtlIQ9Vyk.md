diff --git a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
index 784b5e4..d88ecae 100644
--- a/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
+++ b/openai-code-review-sdk/src/main/java/cn/flybird/sdk/OpenAiCodeReview.java
@@ -61,7 +61,8 @@ public class OpenAiCodeReview {
             System.out.println("代码评审结果：" + log);
 
             // 3. 写入评审日志
-            String logUrl = writeLog(token, log);
+
+            String logUrl = writeLog(token, diffCode + log);
             System.out.println("writeLog: " + logUrl);
 
         } catch(Exception e) {
根据提供的`git diff`记录，以下是针对代码变更的评审：

### 代码变更分析
1. **行号61变更**：
   - 原代码：`String logUrl = writeLog(token, log);`
   - 新代码：`String logUrl = writeLog(token, diffCode + log);`
   
   **变更分析**：
   - 变更将`log`变量直接传递给了`writeLog`方法。新的代码中，`log`变量被附加了一个`diffCode`字符串。这表明可能是在记录代码评审日志时，希望包含代码差异信息。

### 评审意见
1. **目的和逻辑**：
   - 确认`diffCode`变量的含义。如果`diffCode`是包含代码差异的字符串，那么将其与评审日志`log`合并是合理的，这样可以提供更完整的评审信息。
   
2. **代码可读性和维护性**：
   - 代码的可读性略有降低，因为`diffCode`的具体含义没有在代码中明确说明。建议在代码中添加注释来解释`diffCode`变量的用途。
   - 如果`diffCode`是代码差异，那么在日志中包含差异信息可能过于冗长。建议考虑是否需要将差异信息单独记录，或者以更紧凑的方式呈现。

3. **异常处理**：
   - 异常处理代码块看起来是处理可能的异常。确保`writeLog`方法能够正确处理所有可能的异常情况，并且异常信息能够被正确记录。

4. **日志记录**：
   - 在打印`writeLog`方法的返回值时，确保该返回值是预期的。如果`writeLog`方法返回一个URL，那么打印该URL是合理的。如果返回值有其他含义，应相应地解释。

### 代码建议
```java
// 在OpenAiCodeReview类中添加注释
/**
 * 将代码评审日志与代码差异信息合并后写入日志。
 * @param token 访问API的令牌
 * @param diffCode 包含代码差异的字符串
 * @param log 评审日志
 * @return 日志URL或错误信息
 */
String logUrl = writeLog(token, diffCode + log);
System.out.println("writeLog: " + logUrl);
```

### 总结
代码变更看起来是为了记录更详细的评审信息，但建议增加注释以提高代码的可读性，并确保`diffCode`的使用是合理的。同时，需要检查`writeLog`方法的实现，确保它能够正确处理所有可能的异常情况。