
# Spring Security Demo Application  

## Overview  
This application demonstrates how to use **Spring Security** to secure endpoints with **Basic Authentication** and **role-based access control**.   

#### بدل ما نعتمد على حاجة معقدة، التطبيق ده بيستخدم طريقةبسيطة لتأمين الـ Endpoints باستخدام تسجيل دخول أساسي (Username و Password) وتحديد الصلاحيات.

---

## Controller  

### DemoController  

- **`/demo` Endpoint**: Accessible **only** for users with the `"read"` authority.  
  `"read"` نقطة النهاية دي متاحة بس للمستخدمين اللي عندهم صلاحية .

- **`/null` Endpoint**: Open to **any authenticated user**, regardless of authority.  
  متاحة لأي مستخدم متصادق من غير قيود صلاحيات.

#### How It Works – ازاي بيشتغل؟  
##### لو حاولت تدخل `/demo` من غير صلاحية `"read"`، هتظهرلك رسالة 403 Forbidden.  
##### أما `/null` فهي متاحة لأي حد متصادق، حتى لو معندوش صلاحيات معينة.

---

## Security Configuration – تهيئة الأمان  

### SecurityFilterChain  

- **Basic Authentication**: Enables HTTP Basic login for all requests.  
#####  بيفعل تسجيل الدخول الأساسي (Username و Password) لكل الطلبات.

- **Authorization Rules**:  
  - `/demo`: Requires `"read"` authority.  
  - All other endpoints: Require authentication only.  
  - `/null`: Just needs authentication.

#### How It Works – ازاي بيشتغل؟  
`SecurityFilterChain` acts like a gatekeeper. It decides who can enter based on the rules above.

---

### UserDetailsService  

- **InMemoryUserDetailsManager**: Stores users in-memory (not in a database) for simplicity.  
  بيخزن بيانات المستخدمين في الذاكرة كمثال بسيط.

- **Users**:  
  - `mhmd` / `123` → Authority: `"read"`  
  - `saif` / `456` → Authority: `"play"`  
  - `khalid` / `789` → Authority: `"manager"`

#### How It Works – ازاي بيشتغل؟  
لما تحاول تسجل دخول، النظام بيشوف إذا كنت موجود في القائمة وبتتطابق بياناتك مع الصلاحيات المطلوبة.

---

### Password Encoding – تشفير كلمات السر  

- Uses **BCryptPasswordEncoder** for secure password hashing.  
  بيشفر كلمات السر باستخدام `BCrypt`، فـ `123` مثلاً بتتخزن في شكل مشفر زي:  
  `$2a$10$...` لحماية أقوى.

---

## Key Notes – ملاحظات مهمة  

1. **401 Unauthorized**: لما تدخل بيانات غلط أو مش متصادق.  
2. **403 Forbidden**: لما تكون متصادق بس مش عندك الصلاحية المطلوبة.

---

## Usage – ازاي تستخدم التطبيق؟  

1. ادخل على `/demo` باستخدام بيانات المستخدم اللي عنده `read` authority (مثلاً: `mhmd:123`).  
2. باقي الـ Endpoints هيطلبوا بس بيانات تسجيل دخول صحيحة (زي `saif:456` أو `khalid:789`).

---

## Method Explanations – تفسير مفصل للدوال  

### `.anyRequest().hasAnyAuthority("play", "read")`  
- **Explanation**: Requires user to have either `"play"` or `"read"` authority.  
- **Use Case**: When multiple authorities are allowed.

```java
.anyRequest().hasAnyAuthority("admin", "user")
```

---

### `.anyRequest().access("isAuthenticated() and hasAuthority('play')")`  
- **Explanation**: Uses SpEL to combine multiple conditions:  
  - Must be authenticated.  
  - Must have `"play"` authority.

```java
.anyRequest().access("hasRole('ADMIN') or (hasAuthority('read') and isRememberMe())")
```

---

### `denyAll()`  
- **Explanation**: `Denies` access to **everyone**, even if authenticated.  
- **Use Case**: Useful for maintenance mode.

---

### `permitAll()`  
- **Explanation**: `Grants` access to **everyone**, even if not logged in.  
- **Use Case**: For public pages like login or registration.

---

### `authenticated()`  
- **Explanation**: Requires only authentication, no specific authority.  
- **Use Case**: Restrict access to logged-in users only.

---

## Complete Example – مثال كامل  

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .httpBasic()
            .and()
            .authorizeRequests()
            .antMatchers("/public").permitAll()
            .antMatchers("/admin").hasAuthority("admin")
            .antMatchers("/user").hasAnyAuthority("user", "editor")
            .antMatchers("/confidential").access("isAuthenticated() and hasAuthority('confidential')")
            .antMatchers("/maintenance").denyAll()
            .anyRequest().authenticated()
            .and()
            .build();
}
```

**Explanation – التفسير**:  
- `/public`: Open to everyone.  
- `/admin`: Requires `"admin"` authority.  
- `/user`: Requires `"user"` or `"editor"` authority.  
- `/confidential`: Must be authenticated **and** have `"confidential"` authority.  
- `/maintenance`: No one can access.  

---

## Final Thoughts – ملاحظات أخيرة  

- The project is a simple but clear demonstration of how to implement **Spring Security** effectively.  
- Try out the users (`mhmd`, `saif`, `khalid`) and observe how access changes based on their roles!  

**Thanks & Happy Coding!** 🚀
