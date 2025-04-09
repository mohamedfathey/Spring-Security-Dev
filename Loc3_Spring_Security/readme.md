# Custom Authentication Implementation in Spring Security  
## Overview 

This document explains a custom authentication setup in **Spring Security** .

#### بدل ما نستخدم الطريقة التقليدية زي Username و Password، هنستخدم مفتاح مخصص (Custom Key) ييجي في الـ Header بتاع الطلب. The codebase is broken into 5 main parts:

1️⃣ **CustomAuthentication**:⏩  الكائن اللي بيمثل بيانات المصادقة.  
2️⃣ **CustomAuthenticationFilter**:⏩ الفلتر اللي بيستخرج المفتاح من الطلب.  
3️⃣ **CustomAuthenticationManager**:⏩ المدير اللي بينسق عملية التحقق.  
4️⃣ **CustomAuthenticationProvider**:⏩ المزود اللي بيتأكد من المفتاح.  
5️⃣ **SecurityConfig**:⏩ التهيئة اللي بتربط كل حاجة مع Spring Security.  

---

---
---
<!-- 🖌 1️⃣ 2️⃣ 3️⃣ 4️⃣ 5️⃣ ➡ ⬅⬇↗☑🔴🟠🔵🟣🟣🟪🟦🟩 
💫💥🚀 
6️⃣7️⃣8️⃣9️⃣
❌💯❎✅⏩➖
-->
## 1️⃣ CustomAuthentication - كائن المصادقة المخصص  
- **File**: `CustomAuthentication.java`  
- **Purpose**: ده الكلاس اللي بيمثل بيانات المصادقة ومتوافق مع 
`Authentication` interface بتاع Spring Security.  
- **Key Parts**:  
  - `boolean authentication`: بيحدد إذا كنت متصادق ولا لأ.  
  - `String kay`: المفتاح المخصص اللي بييجي من الطلب.  

### الكود وتفسيره  
```java
public class CustomAuthentication implements Authentication {
    private boolean authentication; // الحالة (متصادق أو لا)
    private String kay; // المفتاح المخصص

    public CustomAuthentication(boolean authentication, String kay) {
        this.authentication = authentication; // بتحدد الحالة
        this.kay = kay; // بتخزن المفتاح
    }

    public boolean isAuthenticated() { 
        return authentication; // بترجع الحالة لو متصادق أو لا
    }
    // Placeholder methods زي getAuthorities و getCredentials للتوافق مع Spring
}
```  
- **ازاي بيشتغل؟**: زي بطاقة هوية بتحمل المفتاح وبتقول إذا تم التحقق منه أو لسه.  

---

## 2️⃣ CustomAuthenticationFilter - فلتر المصادقة المخصص  
- **File**: `CustomAuthenticationFilter.java`  
- **Purpose**: الفلتر ده بيوقف كل طلب (Request) ويطلع المفتاح من الـ Header.  
- **Key Method**: `doFilterInternal` - هنا بيحصل الشغل الأساسي.  

### الكود وتفسيره  
```java
@Component
public class CustomAuthenticationFilter extends OncePerRequestFilter {
    private final CustomAuthenticationManager customAuthenticationManager;

    public CustomAuthenticationFilter(CustomAuthenticationManager customAuthenticationManager) {
        this.customAuthenticationManager = customAuthenticationManager; // بيحقن الـ Manager
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String kay = request.getHeader("kay"); // بيطلع المفتاح من الـ Header
        CustomAuthentication ca = new CustomAuthentication(false, kay); // بيعمل كائن غير متصادق
        var a = customAuthenticationManager.authenticate(ca); // بيبعت الكائن للتحقق

        if (a.isAuthenticated()) { 
            SecurityContextHolder.getContext().setAuthentication(a); // لو صح، بيحط الكائن في الـ Context
            filterChain.doFilter(request, response); // بيسمح للطلب يكمل
        }
    }
}
```  
**ازاي بيشتغل؟**:  
1- بياخد المفتاح من الـ Header.  
2- بيعمل كائن جديد غير متصادق.  
3-  بيبعت الكائن للـ Manager.  
4- لو نجح، بيسمح للطلب يكمل.  

---

## 3️⃣ CustomAuthenticationManager - مدير المصادقة المخصص  
- **File**: `CustomAuthenticationManager.java`  
- **Purpose**: المدير اللي بينسق عملية التحقق وبيمرر الكائن للمزود.  
- **Key Method**: `authenticate`.  

### الكود وتفسيره  
```java
@Component
public class CustomAuthenticationManager implements AuthenticationManager {
    private final CustomAuthenticationProvider authenticationProvider;

    public CustomAuthenticationManager(CustomAuthenticationProvider authenticationProvider) {
        this.authenticationProvider = authenticationProvider; // بيحقن الـ Provider
    }

    @Override
    public Authentication authenticate(Authentication authentication) {
        if (authenticationProvider.supports(authentication.getClass())) {
            return authenticationProvider.authenticate(authentication); // بيمرر الكائن للـ Provider
        }
        throw new BadCredentialsException("Invalid authentication type"); // لو النوع غلط، بيرمي Exception
    }
}
```  
 **ازاي بيشتغل؟**:  
1-  بيتأكد إن الـ Provider بيدعم الكائن.  
2-  لو آه، بيمرره للتحقق.  
3- لو لأ، بيرفض العملية.  

---

## 4️⃣ CustomAuthenticationProvider - مزود المصادقة المخصص  
- **File**: `CustomAuthenticationProvider.java`  
- **Purpose**: هنا بيحصل التحقق الفعلي بمقارنة المفتاح مع السري.  
- **Key Methods**: `authenticate` و `supports`.  

### الكود وتفسيره  
```java
@Component
public class CustomAuthenticationProvider implements AuthenticationProvider {
    @Value("${our.very.very.very.secret.key}")
    private String kay; // المفتاح السري من الـ Config

    @Override
    public Authentication authenticate(Authentication authentication) {
        CustomAuthentication ca = (CustomAuthentication) authentication;
        if (kay.equals(ca.getKay())) { // بيقارن المفتاح السري بالمفتاح اللي جه
            return new CustomAuthentication(true, null); // لو متطابق، بيرجع كائن متصادق
        }
        throw new BadCredentialsException("Invalid key"); // لو مش متطابق، بيرمي Exception
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return CustomAuthentication.class.equals(authentication); // بيدعم بس CustomAuthentication
    }
}
```  
 **ازاي بيشتغل؟**:  
1- بيقارن المفتاح اللي جه مع السري.  
2- لو متطابق، بيرجع كائن متصادق.  
3- لو لأ، بيرفض.  

---

## 5️⃣ SecurityConfig - تهيئة الأمان  
- **File**: `SecurityConfig.java`  
- **Purpose**: بتربط الفلتر مع سلسلة الفلاتر بتاعة Spring Security.  
- **Key Method**: `securityFilterChain`.  

### الكود وتفسيره  
```java
@Configuration
public class SecurityConfig {
    private final CustomAuthenticationFilter customAuthenticationFilter;

    public SecurityConfig(CustomAuthenticationFilter customAuthenticationFilter) {
        this.customAuthenticationFilter = customAuthenticationFilter; // بيحقن الفلتر
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        return httpSecurity
                .addFilterAt(customAuthenticationFilter, UsernamePasswordAuthenticationFilter.class) // بيضيف الفلتر
                .authorizeRequests().anyRequest().authenticated() // كل الطلبات لازم تكون متصادقة
                .and().build(); // بيبني السلسلة
    }
}
```  
 **ازاي بيشتغل؟**:  
1- بيضيف الفلتر المخصص في السلسلة.  
2- بيطلب إن كل الـ Requests تكون متصادقة.  

---

## ازاي الدنيا بتشتغل؟ - How It Works

#### 1. **الطلب بييجي**: الـ `CustomAuthenticationFilter` بيمسك الـ Request.  
#### 2. **استخراج المفتاح**: بيطلع المفتاح من الـ Header (مثلاً "kay").  
#### 3. **محاولة التحقق**: بيعمل كائن `CustomAuthentication` ويبعت للـ Manager.  
#### 4. **التحقق**: الـ Provider بيقارن المفتاح بالسري.  
#### 5. **النتيجة**:  
 **لو صح**: بيتخزن في `SecurityContextHolder` والطلب بيكمل.  
 **لو غلط**: بيترفض مع Exception.  

---

## الاعتماديات والتهيئة - Dependencies & Configuration  
- **Dependencies**: ضيف `spring-boot-starter-security` في المشروع.  
- **Configuration**: في `application.properties`:  
```properties
our.very.very.very.secret.key=secret
```

---

## التخصيص - Customization  
- غير المفتاح السري (`our.very.very.very.secret.key`) زي ما تحب.  
- لو عايز تضيف Roles أو بيانات زيادة، عدل في `CustomAuthentication`.  

---

## ملاحظات أخيرة - Final Thoughts  
1- ده مثال بسيط لـ Custom Authentication، لكن بتقدر توسع فيه.  
2- Spring Security مرن جدًا، فممكن تستخدم الطريقة دي مع أي بيانات مش بس مفتاح.  

**Happy Coding!** 🚀