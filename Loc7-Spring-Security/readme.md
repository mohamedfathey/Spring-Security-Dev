# Spring Security Demo Application  
##   Overview  
This is a simple Spring Security demo showcasing **Basic Authentication** and **role-based access control** using both configuration-level and method-level security. 

التطبيق ده بيوضح ازاي نأمن الـ Endpoints باستخدام تسجيل دخول أساسي (Username و Password) وتحكم في الصلاحيات بطريقتين مختلفتين.

---

##  Application Components  

### 1. SecurityConfig - تهيئة الأمان  
- **File**: `SecurityConfig.java`  
- **الغرض - Purpose**: الكلاس ده بيعرف إزاي التطبيق هيأمن الطلبات ومين المستخدمين المسموح لهم يدخلوا.  

#### الكود وتفسيره  
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true) // بيفعل الأمان على مستوى الدوال
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .httpBasic() // بيفعل Basic Authentication
                .and()
                .authorizeRequests()
                .anyRequest().authenticated() // كل الطلبات لازم تكون متصادقة
                .and()
                .build();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        // بيعمل مستخدمين في الذاكرة
        var u1 = User.withUsername("mhmd")
                .password(encoder().encode("123"))
                .authorities("read")
                .build();
        var u2 = User.withUsername("saif")
                .password(encoder().encode("456"))
                .authorities("play")
                .build();
        var u3 = User.withUsername("khalid")
                .password(encoder().encode("789"))
                .authorities("manager")
                .build();
        return new InMemoryUserDetailsManager(u1, u2, u3);
    }

    @Bean
    public BCryptPasswordEncoder encoder() {
        return new BCryptPasswordEncoder(); // بيشفر كلمات السر
    }
}
```  
<!-- - **ازاي بيشتغل؟ - How it works?**:  
  - `.httpBasic()`:   بيفعل تسجيل الدخول بـ Username و Password عبر الـ HTTP Headers. 
  - `.anyRequest().authenticated()`: بيطلب إن كل طلب لازم يكون متصادق، يعني لازم تسجل دخول عشان تدخل أي Endpoint.  
  - `userDetailsService()`: بيعمل 3 مستخدمين في الذاكرة (mhmd, saif, khalid) مع صلاحياتهم (`read`, `play`, `manager`).  
  - `BCryptPasswordEncoder`: بيشفر كلمات السر عشان تبقى آمنة (مثلاً `123` بتتحول لـ `$2a$10$...`).  
  - `@EnableGlobalMethodSecurity(prePostEnabled = true)`: بيسمح باستخدام `@PreAuthorize` في الـ Controller للتحكم في الصلاحيات على مستوى الدوال.   -->

---

### 2. DemoController - الكنترولر  
- **File**: `DemoController.java`  
- **الغرض - Purpose**: بيعرف الـ Endpoints اللي هتكون متاحة ومين يقدر يدخلها باستخدام `@PreAuthorize`.  

#### الكود وتفسيره  
```java
@RestController
@RequestMapping("/demo")
public class DemoController {

    @GetMapping("/demo1")
    @PreAuthorize("hasAuthority('read')") // بس اللي عنده "read" يدخل
    public String test1() {
        return "demo1";
    }

    @GetMapping("/demo2")
    @PreAuthorize("hasAuthority('play')") // بس اللي عنده "play" يدخل
    public String test2() {
        return "demo2";
    }

    @GetMapping("/demo3")
    @PreAuthorize("hasAuthority('manager')") // بس اللي عنده "manager" يدخل
    public String test3() {
        return "demo3";
    }
}
```  
- **ازاي بيشتغل؟ - How it works?**:  
  - كل دالة (Endpoint) ليها شرط معين باستخدام `@PreAuthorize`:  
    - `/demo/demo1`: `"read"`لازم يكون عندك صلاحية  (يعني `mhmd` بس يقدر يدخل).  
    - `/demo/demo2`: `"play"`لازم صلاحية  (يعني `saif` بس).  
    - `/demo/demo3`: `"manager"`لازم صلاحية  (يعني `khalid` بس).  
  - لو حاولت تدخل من غير الصلاحية المطلوبة، هيطلعلك 403 (Forbidden).  

---
# كيف يعمل التطبيق؟ - How the Application Works?

## الخطوات الأساسية  
### 1. إرسال طلب لنقطة نهاية (Endpoint)  
-   لما تبعت طلب (زي `/demo/demo1`)، الـ `SecurityFilterChain` بيطلب منك تسجل دخول باستخدام اسم المستخدم وكلمة السر (Basic Auth). لو البيانات صحيحة (مثل `mhmd:123`)، بيسمح لك تدخل، لكن لازم تكون عندك الصلاحية المطلوبة للنقطة دي.  
  -  When you send a request (like `/demo/demo1`), the `SecurityFilterChain` asks you to log in using a username and password (Basic Auth). If your credentials are correct (e.g., `mhmd:123`), it lets you in, but only if you have the required authority for that endpoint.

### 2. التحقق من الصلاحية باستخدام `@PreAuthorize`  
-  الـ `@PreAuthorize` بيتأكد إن الصلاحية اللي عندك (زي `"read"`) متطابقة مع اللي مكتوب في الدالة. لو الصلاحية مش موجودة، بيمنعك من الدخول.  
   The `@PreAuthorize` checks if your authority (like `"read"`) matches what’s required in the method. If you don’t have it, it blocks your access.

#### مثال بسيط  
-  لو جربت تدخل `/demo/demo1` بـ `mhmd:123`، هيدخل عادي لأن عنده صلاحية `"read"`. لكن لو جربت بـ `saif:456`، هيطلعلك خطأ 403 لأن معندوش `"read"`.  
  -  If you try `/demo/demo1` with `mhmd:123`, it works because he has the `"read"` authority. But if you try with `saif:456`, you’ll get a 403 error because he doesn’t have `"read"`.

---
## الفرق بين `@PreAuthorize("hasAuthority('manager')")` وبين إعدادها في الـ Config  

### 1. استخدام `@PreAuthorize("hasAuthority('manager')")` في الـ Controller  
-   الـ `@PreAuthorize` ده أمر بتحطه فوق الدالة في الـ Controller، وبيقول "الدالة دي (`demo3`) محدش يدخلها غير اللي عنده صلاحية `manager`".  
  -   The `@PreAuthorize` is a command you place above a method in the Controller, saying, "Only users with the `manager` authority can access this method (`demo3`)".  

### **ازاي بيشتغل؟**:  
بيشتغل بسبب إعداد `@EnableGlobalMethodSecurity` في الـ Config، وبيفحص صلاحيتك قبل ما الدالة تشتغل. لو معندكش `"manager"`، بيرفض الطلب فورًا. 

 It works because of the `@EnableGlobalMethodSecurity` setting in the Config. It checks your authority before the method runs, and if you don’t have `"manager"`, it rejects the request immediately.  

### **المميزات**:  
 مرن جدًا، يعني تقدر تحدد قاعدة مختلفة لكل دالة. الكود بيبقى واضح لأن القاعدة مكتوبة جنب الدالة.  

 Very flexible you can set a different rule for each method. The code is clear because the rule is right next to the method.  

### **العيوب**:  
  لو عندك دوال كتير، هتكتب `@PreAuthorize` كتير، وممكن ده يخلّي الكود متكرر ومش منظم.  
 
  If you have many methods, you’ll repeat `@PreAuthorize` a lot, which can make the code repetitive and messy.  

#### مثال عملي  
  لو كتبت الكود ده، بس `khalid` يقدر يدخل `/demo/demo3` لأن عنده `"manager"`. 
   
  If you write this code, only `khalid` can access `/demo/demo3` because he has `"manager"`.  
```java
@GetMapping("/demo3")
@PreAuthorize("hasAuthority('manager')")
public String test3() {
    return "demo3";
}
```

---
 
## 2. إعداد نفس القاعدة في الـ `SecurityConfig`  
   بدل ما تكتب القاعدة في الـ Controller، بتحطها في الـ `SecurityFilterChain` في الـ Config، وب تقول "أي طلب لـ `/demo/demo3` لازم يكون عنده صلاحية `manager`".  

  Instead of writing the rule in the Controller, you put it in the `SecurityFilterChain` in the Config, saying, "Any request to `/demo/demo3` must have the `manager` authority".  

### **ازاي بيشتغل؟**:  
 بتستخدم `.antMatchers()` مع `.hasAuthority()` في الـ Config عشان تحدد القاعدة على مستوى الـ URL. التحقق بيحصل في الفلاتر قبل ما الطلب يوصل للـ Controller.

  You use `.antMatchers()` with `.hasAuthority()` in the Config to set the rule at the URL level. The check happens in the filter chain before the request reaches the Controller.  

### **المميزات**:  
  مركزي، يعني كل قواعد الأمان في مكان واحد (الـ Config). أسهل لو عندك Endpoints كتير بنفس القاعدة.  

  Centralized—all security rules are in one place (the Config). Easier if you have many endpoints with the same rule.  

### **العيوب**:  
  أقل مرونة لو عايز قواعد مختلفة لدوال في نفس الـ URL. الكود بيبقى بعيد عن الـ Endpoints، فمش واضح على طول.  
 
  Less flexible if you want different rules for methods under the same URL. The code is far from the endpoints, so it’s not immediately clear.  

#### مثال عملي  
  لو كتبت الكود ده في الـ Config، برضو بس `khalid` يقدر يدخل `/demo/demo3` لأن عنده `"manager"`.  
 
  If you write this code in the Config, still only `khalid` can access `/demo/demo3` because he has `"manager"`.  
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .httpBasic()
            .and()
            .authorizeRequests()
            .antMatchers("/demo/demo3").hasAuthority("manager") // نفس القاعدة في الـ Config
            .anyRequest().authenticated()
            .and()
            .build();
}
```

---

## الفرق الرئيسي - Key Difference  
| النقطة             | `@PreAuthorize` في الـ Controller          | في الـ `SecurityConfig`                  |  
|--------------------|--------------------------------------------|------------------------------------------|  
| **المكان**        | على الدالة نفسها في الـ Controller         | في الـ Config على مستوى الـ URL          |  
| **المرونة**       | أعلى، بتقدر تعمل قواعد لكل دالة مختلفة    | أقل، بتعمل قواعد على مستوى الـ Endpoint |  
| **الوضوح**        | أوضح لأن القاعدة جنب الدالة                | أقل وضوح لأن القواعد بعيدة عن الكود     |  
| **التنظيم**       | ممكن يكون متشتت لو دوال كتير               | مركزي ومنظم لو قواعد كتير               |  

## **امتى تستخدم ايه؟**:  
  استخدم `@PreAuthorize` لو عايز تحكم دقيق في كل دالة (Method-Level Security). استخدم الـ Config لو عايز قواعد عامة على مستوى الـ URLs (Global Security).  
 
  Use `@PreAuthorize` for precise control over each method (Method-Level Security). Use the Config for general rules at the URL level (Global Security).  

---

## ازاي تستخدم التطبيق؟ - How to Use It?  
.1  افتح `/demo/demo1` وحط `mhmd:123` (هينجح لأن عنده `"read"`).  
   
Open `/demo/demo1` and use `mhmd:123` (it’ll work because he has `"read"`). 

.2   جرب `/demo/demo2` بـ `saif:456` (هينجح لأن عنده `"play"`).  
  
  Try `/demo/demo2` with `saif:456` (it’ll work because he has `"play"`).  

.3  جرب `/demo/demo3` بـ `khalid:789` (هينجح لأن عنده `"manager"`).  

 Try `/demo/demo3` with `khalid:789` (it’ll work because he has `"manager"`).  
 
. 4  لو جربت بيانات غلط أو صلاحية ناقصة، هيطلعلك 401 (غير متصادق) أو 403 (ممنوع).  
 
  If you use wrong credentials or lack the authority, you’ll get 401 (Unauthorized) or 403 (Forbidden).  

---

**Happy Coding!** 🚀