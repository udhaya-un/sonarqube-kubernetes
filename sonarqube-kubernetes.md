Index: src/main/kotlin/com/zimidy/api/layers/storage/entities/node/nodes/Email.kt
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- src/main/kotlin/com/zimidy/api/layers/storage/entities/node/nodes/Email.kt	(revision ca1a32a0d6e9405ba7a02772e577cabdfcd5b848)
+++ src/main/kotlin/com/zimidy/api/layers/storage/entities/node/nodes/Email.kt	(date 1537459274000)
@@ -8,14 +8,14 @@
 
 class Email(
     @Index(unique = true)
-    var value: String? = null,
+    var value: String = "",
     var verified: Boolean = false,
     @Relationship(type = User.EMAIL, direction = Relationship.INCOMING)
     var user: User = User()
 ) : Node() {
     companion object {
         fun create(
-            value: String? = null,
+            value: String = "",
             verified: Boolean = false,
             user: User
         ) = Email(
@@ -27,14 +27,14 @@
 }
 
 data class CreateEmailData(
-    var value: String?,
+    var value: String,
     var verified: Boolean,
     var userId: NodeId
 )
 
 data class UpdateEmailData(
     var id: NodeId,
-    var value: String?,
+    var value: String,
     var verified: Boolean
 )
 
Index: src/main/kotlin/com/zimidy/api/layers/api/graphql/resolvers/nodes/user_resolvers.kt
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- src/main/kotlin/com/zimidy/api/layers/api/graphql/resolvers/nodes/user_resolvers.kt	(revision ca1a32a0d6e9405ba7a02772e577cabdfcd5b848)
+++ src/main/kotlin/com/zimidy/api/layers/api/graphql/resolvers/nodes/user_resolvers.kt	(date 1537768021000)
@@ -43,6 +43,7 @@
         var validateNickName = isValidNickName(input.data.nickname)
         if (validateNickName) {
             val user = repository.create(input.data)
+            println("createUser%%%%%%%%%%%%%%%%%%%%%%%%%%%%% $user")
             return UserPayload(clientMutationId = input.clientMutationId, node = UserNode(user))
         }
         return null
Index: src/main/kotlin/com/zimidy/api/layers/api/graphql/resolvers/nodes/email_resolvers.kt
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- src/main/kotlin/com/zimidy/api/layers/api/graphql/resolvers/nodes/email_resolvers.kt	(revision ca1a32a0d6e9405ba7a02772e577cabdfcd5b848)
+++ src/main/kotlin/com/zimidy/api/layers/api/graphql/resolvers/nodes/email_resolvers.kt	(date 1537463784000)
@@ -13,7 +13,11 @@
 import com.zimidy.api.layers.api.graphql.types.node.nodes.EmailPayload
 import com.zimidy.api.layers.api.graphql.types.node.nodes.DeleteEmailInput
 import com.zimidy.api.layers.api.graphql.types.node.nodes.EmailConnectionFactory
+import com.zimidy.api.layers.storage.entities.emailVerification.UserMailService
 import com.zimidy.api.layers.storage.repositories.node.nodes.EmailRepository
+import org.springframework.beans.factory.annotation.Autowired
+import org.springframework.mail.javamail.JavaMailSender
+import org.thymeleaf.spring4.SpringTemplateEngine
 
 @Component
 class EmailNodeQueryResolver(private val repository: EmailRepository) : QueryResolver() {
@@ -32,7 +36,11 @@
 }
 
 @Component
-class EmailNodeMutationResolver(private val repository: EmailRepository) : MutationResolver() {
+class EmailNodeMutationResolver(
+    private val repository: EmailRepository,
+    private val sender: JavaMailSender,
+    var templateEngine: SpringTemplateEngine
+) : MutationResolver() {
 
     fun isValidEmailAddress(email: String?): Boolean {
         val ePattern = "^[a-zA-Z0-9]+@((\\[[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\])|(([a-zA-Z\\-0-9]+\\.)+[a-zA-Z]{2,}))$"
@@ -46,6 +54,8 @@
         var testdata = isValidEmailAddress(input.data.value)
         if (testdata) {
             val email = repository.create(input.data)
+            val data = UserMailService(sender, templateEngine)
+            data.sendMail(email)
             return EmailPayload(clientMutationId = input.clientMutationId, node = EmailNode(email))
         }
         return null
Index: src/main/kotlin/com/zimidy/api/layers/storage/entities/emailVerification/UserMailService.kt
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- src/main/kotlin/com/zimidy/api/layers/storage/entities/emailVerification/UserMailService.kt	(date 1537463143000)
+++ src/main/kotlin/com/zimidy/api/layers/storage/entities/emailVerification/UserMailService.kt	(date 1537463143000)
@@ -0,0 +1,54 @@
+package com.zimidy.api.layers.storage.entities.emailVerification
+
+import com.zimidy.api.layers.storage.entities.node.nodes.Email
+import com.zimidy.api.layers.storage.entities.node.nodes.User
+import org.springframework.beans.factory.annotation.Autowired
+import javax.mail.MessagingException
+import org.springframework.mail.javamail.JavaMailSender
+import org.springframework.mail.javamail.MimeMessageHelper
+import org.springframework.stereotype.Component
+import org.springframework.stereotype.Controller
+import org.springframework.ui.Model
+import org.springframework.ui.ModelMap
+import org.springframework.web.bind.annotation.RequestMapping
+import org.thymeleaf.context.Context
+import org.thymeleaf.spring4.SpringTemplateEngine
+import java.nio.charset.StandardCharsets
+import java.util.UUID
+
+@Controller
+class UserMailService(
+    @Autowired
+    private val sender: JavaMailSender,
+    @Autowired
+    private val templateEngine: SpringTemplateEngine
+) {
+    @RequestMapping("/sendMail")
+    fun sendMail(email: Email): String {
+        val message = sender.createMimeMessage()
+        val helper = MimeMessageHelper(message,
+            MimeMessageHelper.MULTIPART_MODE_MIXED_RELATED,
+            StandardCharsets.UTF_8.name())
+        val token = UUID.randomUUID().toString()
+        val verifyUrl = "http://localhost:8080/sendMail?" + token
+
+        val model = ModelMap()
+        model.addAttribute("verifyUrl", verifyUrl)
+
+        val context = Context()
+        val html = templateEngine.process("email-template", context)
+
+        try {
+            helper.setTo(email.value)
+            helper.setText(html, true)
+            helper.setSubject("Mail From Zimidy")
+        } catch (e: MessagingException) {
+            e.printStackTrace()
+            return "Error while sending mail .."
+        }
+        sender.send(message)
+        return "email-template"
+    }
+}
+
+
Index: src/main/kotlin/com/zimidy/api/layers/storage/repositories/node/nodes/UserRepository.kt
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- src/main/kotlin/com/zimidy/api/layers/storage/repositories/node/nodes/UserRepository.kt	(revision ca1a32a0d6e9405ba7a02772e577cabdfcd5b848)
+++ src/main/kotlin/com/zimidy/api/layers/storage/repositories/node/nodes/UserRepository.kt	(date 1537768222000)
@@ -42,6 +42,7 @@
             roles = data.roles,
             nickname = data.nickname
         )
+        println("useruser@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ $user")
         return create(user)
     }
 
Index: src/main/kotlin/com/zimidy/api/layers/storage/entities/emailVerification/ThymeleafConfig.kt
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- src/main/kotlin/com/zimidy/api/layers/storage/entities/emailVerification/ThymeleafConfig.kt	(date 1537429689000)
+++ src/main/kotlin/com/zimidy/api/layers/storage/entities/emailVerification/ThymeleafConfig.kt	(date 1537429689000)
@@ -0,0 +1,28 @@
+package com.zimidy.api.layers.storage.entities.emailverifiacation
+
+import org.springframework.context.annotation.Bean
+import org.springframework.context.annotation.Configuration
+import org.thymeleaf.spring4.SpringTemplateEngine
+import org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver
+import java.nio.charset.StandardCharsets
+
+@Configuration
+class ThymeleafConfig {
+
+    @Bean
+    fun springTemplateEngine(): SpringTemplateEngine {
+        val templateEngine = SpringTemplateEngine()
+        templateEngine.addTemplateResolver(htmlTemplateResolver())
+        return templateEngine
+    }
+
+    @Bean
+    fun htmlTemplateResolver(): SpringResourceTemplateResolver {
+        val emailTemplateResolver = SpringResourceTemplateResolver()
+        emailTemplateResolver.prefix = "classpath:/templates/"
+        emailTemplateResolver.suffix = ".html"
+        emailTemplateResolver.setTemplateMode("HTML5")
+        emailTemplateResolver.characterEncoding = StandardCharsets.UTF_8.name()
+        return emailTemplateResolver
+    }
+}
Index: src/main/resources/application.yml
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- src/main/resources/application.yml	(revision ca1a32a0d6e9405ba7a02772e577cabdfcd5b848)
+++ src/main/resources/application.yml	(date 1537464842000)
@@ -13,17 +13,19 @@
     username: neo4j
     password: @{neo4j.password}
     auto-index: assert
+
 #  todo: mailing features are temporarily switched of, because authentication to mail account doesn't work
-#  mail:
-#    host: smtp.gmail.com
-#    port: 587
-#    username: accountcreate@zimidy.com
-#    password: Down4Anything!
-#    properties.mail.smtp: # available properties: https://goo.gl/anLq4j
-#      starttls:
-#        enable: true
-#        required: true
-#      auth: true
+  mail:
+    host: smtp.gmail.com
+    port: 587
+    username: rajkumar.sellakannu@zimidy.com
+    password: Rajkumar@123
+    properties.mail.smtp: # available properties: https://goo.gl/anLq4j
+      starttls:
+        enable: true
+        required: true
+      auth: true
+
   freemarker:
     enabled: false
     cache: true
Index: pom.xml
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
--- pom.xml	(revision ca1a32a0d6e9405ba7a02772e577cabdfcd5b848)
+++ pom.xml	(date 1537459139000)
@@ -1,7 +1,7 @@
 <?xml version="1.0" encoding="UTF-8"?>
-<project xmlns="http://maven.apache.org/POM/4.0.0"
+<project xmlns="http://maven.apache.org//4.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
-         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
+         xsi:schemaLocation="http://maven.apache.org//4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
     <modelVersion>4.0.0</modelVersion>
 
@@ -26,8 +26,9 @@
         <module>api</module>
 
         <java.version>1.8</java.version>
-        <kotlin.version>1.2.50</kotlin.version>
+        <kotlin.version>1.2.30</kotlin.version>
         <kotlin.compiler.incremental>true</kotlin.compiler.incremental>
+        <graphql-spring-boot.version>4.0.0.M1</graphql-spring-boot.version>
     </properties>
 
     <repositories>
@@ -67,9 +68,11 @@
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-data-neo4j</artifactId>
         </dependency>
+
         <dependency>
-            <groupId>org.springframework.boot</groupId>
-            <artifactId>spring-boot-starter-web</artifactId>
+            <groupId>org.neo4j</groupId>
+            <artifactId>neo4j-ogm-bolt-driver</artifactId>
+            <version>${neo4j-ogm.version}</version>
         </dependency>
         <dependency>
             <groupId>org.springframework.boot</groupId>
@@ -83,7 +86,10 @@
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-freemarker</artifactId>
         </dependency>
-
+        <dependency>
+            <groupId>org.springframework.boot</groupId>
+            <artifactId>spring-boot-starter-web</artifactId>
+        </dependency>
         <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-security</artifactId>
@@ -93,6 +99,50 @@
             <artifactId>jjwt</artifactId>
             <version>0.9.0</version>
         </dependency>
+
+        <dependency>
+            <groupId>org.thymeleaf</groupId>
+            <artifactId>thymeleaf-spring4</artifactId>
+            <version>3.0.9.RELEASE</version>
+        </dependency>
+        <dependency>
+            <groupId>nz.net.ultraq.thymeleaf</groupId>
+            <artifactId>thymeleaf-layout-dialect</artifactId>
+            <version>2.3.0</version>
+        </dependency>
+
+
+        <dependency>
+            <groupId>com.plivo</groupId>
+            <artifactId>plivo-java</artifactId>
+            <version>3.0.8</version>
+        </dependency>
+        <dependency>
+            <groupId>org.projectlombok</groupId>
+            <artifactId>lombok</artifactId>
+            <version>1.16.10</version>
+            <scope>provided</scope>
+        </dependency>
+        <dependency>
+            <groupId>com.google.apis</groupId>
+            <artifactId>google-api-services-storage</artifactId>
+            <version>v1-rev65-1.21.0</version>
+        </dependency>
+        <dependency>
+            <groupId>org.apache.poi</groupId>
+            <artifactId>poi-ooxml</artifactId>
+            <version>3.14</version>
+        </dependency>
+        <dependency>
+            <groupId>com.google.maps</groupId>
+            <artifactId>google-maps-services</artifactId>
+            <version>0.1.15</version>
+        </dependency>
+        <dependency>
+            <groupId>org.apache.commons</groupId>
+            <artifactId>commons-text</artifactId>
+            <version>1.1</version>
+        </dependency>
 
         <!--
         GraphQL Spring Boot (https://github.com/graphql-java/graphql-spring-boot) depends on:
@@ -102,13 +152,13 @@
         <dependency>
             <groupId>com.graphql-java</groupId>
             <artifactId>graphql-spring-boot-starter</artifactId>
-            <version>4.2.0</version>
+            <version>${graphql-spring-boot.version}</version>
         </dependency>
         <!-- GraphQL Java Tools (https://github.com/graphql-java/graphql-java-tools) -->
         <dependency>
             <groupId>com.graphql-java</groupId>
             <artifactId>graphql-java-tools</artifactId>
-            <version>5.1.0</version>
+            <version>4.3.0</version>
         </dependency>
 
         <!--
@@ -124,6 +174,47 @@
             <artifactId>spring-boot-starter-test</artifactId>
             <scope>test</scope>
         </dependency>
+        <dependency>
+            <groupId>org.springframework</groupId>
+            <artifactId>spring-core</artifactId>
+            <version>5.0.4.RELEASE</version>
+        </dependency>
+
+
+        <!-- Social Login -->
+
+        <dependency>
+            <groupId>org.springframework.security</groupId>
+            <artifactId>spring-security-oauth2-client</artifactId>
+        </dependency>
+        <dependency>
+            <groupId>org.springframework.security</groupId>
+            <artifactId>spring-security-oauth2-jose</artifactId>
+        </dependency>
+
+        <dependency>
+            <groupId>org.springframework.security</groupId>
+            <artifactId>spring-security-config</artifactId>
+            <version>5.0.2.RELEASE</version>
+        </dependency>
+
+        <dependency>
+            <groupId>org.springframework.boot</groupId>
+            <artifactId>spring-boot-starter-thymeleaf</artifactId>
+        </dependency>
+        <dependency>
+            <groupId>org.springframework.social</groupId>
+            <artifactId>spring-social-twitter</artifactId>
+            <version>1.1.2.RELEASE</version>
+        </dependency>
+        <dependency>
+            <groupId>org.twitter4j</groupId>
+            <artifactId>twitter4j-core</artifactId>
+            <version>[4.0,)</version>
+        </dependency>
+
+        <!-- Social Login end-->
+
     </dependencies>
 
     <build>
@@ -283,80 +374,80 @@
                     </execution>
                 </executions>
             </plugin>
-            <plugin>
-                <groupId>org.apache.maven.plugins</groupId>
-                <artifactId>maven-antrun-plugin</artifactId>
-                <version>1.8</version>
-                <executions>
-                    <execution>
-                        <id>ktlint</id>
-                        <phase>verify</phase>
-                        <configuration>
-                            <target name="ktlint">
-                                <java taskname="ktlint"
-                                      dir="${basedir}"
-                                      fork="true"
-                                      failonerror="true"
-                                      classpathref="maven.plugin.classpath"
-                                      classname="com.github.shyiko.ktlint.Main"
-                                >
-                                    <arg value="src/**/*.kt"/>
-                                    <arg value="--reporter=plain?group_by_file"/>
-                                    <arg value="--reporter=plain?group_by_file,output=${project.build.directory}/ktlint-report.txt"/>
-                                    <arg value="--verbose"/>
-                                </java>
-                            </target>
-                        </configuration>
-                        <goals><goal>run</goal></goals>
-                    </execution>
-                    <execution>
-                        <id>ktlint-format</id>
-                        <configuration>
-                            <target name="ktlint">
-                                <java taskname="ktlint"
-                                      dir="${basedir}"
-                                      fork="true"
-                                      failonerror="true"
-                                      classpathref="maven.plugin.classpath"
-                                      classname="com.github.shyiko.ktlint.Main"
-                                >
-                                    <arg value="-F"/>
-                                    <arg value="src/**/*.kt"/>
-                                    <arg value="--reporter=plain?group_by_file"/>
-                                    <arg value="--reporter=plain?group_by_file,output=${project.build.directory}/ktlint-report.txt"/>
-                                    <arg value="--verbose"/>
-                                </java>
-                            </target>
-                        </configuration>
-                        <goals><goal>run</goal></goals>
-                    </execution>
-                    <execution>
-                        <id>ktlint-apply-to-idea-project</id>
-                        <configuration>
-                            <target name="ktlint">
-                                <java taskname="ktlint"
-                                      dir="${basedir}"
-                                      fork="true"
-                                      failonerror="true"
-                                      classpathref="maven.plugin.classpath"
-                                      classname="com.github.shyiko.ktlint.Main"
-                                >
-                                    <arg value="--apply-to-idea-project"/>
-                                </java>
-                            </target>
-                        </configuration>
-                        <goals><goal>run</goal></goals>
-                    </execution>
-                </executions>
-                <dependencies>
-                    <dependency>
-                        <groupId>com.github.shyiko</groupId>
-                        <artifactId>ktlint</artifactId>
-                        <version>0.23.1</version>
-                    </dependency>
-                    <!-- additional 3rd party rule-set(s) can be specified here -->
-                </dependencies>
-            </plugin>
+            <!--   <plugin>
+                   <groupId>org.apache.maven.plugins</groupId>
+                   <artifactId>maven-antrun-plugin</artifactId>
+                   <version>1.8</version>
+                   <executions>
+                       <execution>
+                           <id>ktlint</id>
+                           <phase>verify</phase>
+                           <configuration>
+                               <target name="ktlint">
+                                   <java taskname="ktlint"
+                                         dir="${basedir}"
+                                         fork="true"
+                                         failonerror="true"
+                                         classpathref="maven.plugin.classpath"
+                                         classname="com.github.shyiko.ktlint.Main"
+                                   >
+                                       <arg value="src/**/*.kt"/>
+                                       <arg value="&#45;&#45;reporter=plain?group_by_file"/>
+                                       <arg value="&#45;&#45;reporter=plain?group_by_file,output=${project.build.directory}/ktlint-report.txt"/>
+                                       <arg value="&#45;&#45;verbose"/>
+                                   </java>
+                               </target>
+                           </configuration>
+                           <goals><goal>run</goal></goals>
+                       </execution>
+                       <execution>
+                           <id>ktlint-format</id>
+                           <configuration>
+                               <target name="ktlint">
+                                   <java taskname="ktlint"
+                                         dir="${basedir}"
+                                         fork="true"
+                                         failonerror="true"
+                                         classpathref="maven.plugin.classpath"
+                                         classname="com.github.shyiko.ktlint.Main"
+                                   >
+                                       <arg value="-F"/>
+                                       <arg value="src/**/*.kt"/>
+                                       <arg value="&#45;&#45;reporter=plain?group_by_file"/>
+                                       <arg value="&#45;&#45;reporter=plain?group_by_file,output=${project.build.directory}/ktlint-report.txt"/>
+                                       <arg value="&#45;&#45;verbose"/>
+                                   </java>
+                               </target>
+                           </configuration>
+                           <goals><goal>run</goal></goals>
+                       </execution>
+                       <execution>
+                           <id>ktlint-apply-to-idea-projectktlint-apply-to-idea-project</id>
+                           <configuration>
+                               <target name="ktlint">
+                                   <java taskname="ktlint"
+                                         dir="${basedir}"
+                                         fork="true"
+                                         failonerror="true"
+                                         classpathref="maven.plugin.classpath"
+                                         classname="com.github.shyiko.ktlint.Main"
+                                   >
+                                       <arg value="&#45;&#45;apply-to-idea-project"/>
+                                   </java>
+                               </target>
+                           </configuration>
+                           <goals><goal>run</goal></goals>
+                       </execution>
+                   </executions>
+                   <dependencies>
+                       <dependency>
+                           <groupId>com.github.shyiko</groupId>
+                           <artifactId>ktlint</artifactId>
+                           <version>0.23.1</version>
+                       </dependency>
+                       &lt;!&ndash; additional 3rd party rule-set(s) can be specified here &ndash;&gt;
+                   </dependencies>
+               </plugin>-->
         </plugins>
     </build>
 
