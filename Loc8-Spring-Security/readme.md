# Spring Security Demo Application  
## Overview  
This is a small app that shows how to use **Spring Security** to protect your web pages. It uses **Basic Authentication** (just a username and password) and something called `@PreAuthorize` to decide who can see what. Think of it like a locked door – only people with the right key (username, password, and permission) can get in! 😊  

---

## Application Components  

### 1. SecurityConfig  
**File**: `SecurityConfig.java`  
**What it does**:  
This part sets up the security rules and makes a list of users who can use the app. It’s like writing a guest list for a party and giving each person a special pass.  

#### How it works (Simple Version):  
- **`.httpBasic()`**: When someone tries to visit the app, it asks for a username and password. You send these in the request (like a secret code in the background).  
- **`.anyRequest().authenticated()`**: This means EVERY page in the app needs you to log in first. No login = no entry!  
- **`userDetailsService()`**: This creates 3 users in the app’s memory (not a real database, just for this demo):  
  - `mhmd` with password `123` can only do “read” stuff.  
  - `saif` with password `456` can only do “play” stuff.  
  - `khalid` with password `789` can do “manager” stuff (he’s the boss!).  
- **`BCryptPasswordEncoder`**: This scrambles the passwords so they’re safe. For example, `123` turns into a long, weird code like `$2a$10$...`. Nobody can guess it!  
- **`@EnableGlobalMethodSecurity(prePostEnabled = true)`**: This is a switch that lets us use `@PreAuthorize` later to control who gets into specific pages.  

---

### 2. DemoController  
**File**: `DemoController.java`  
**What it does**:  
This part creates the pages (called endpoints) you can visit in the app and decides who’s allowed to see them using `@PreAuthorize`. It’s like putting a guard at each door who checks your pass.  

#### How it works (Simple Version):  
- **`/demo/demo1`**: Only people with “read” permission can visit (that’s just `mhmd`). If you’re not `mhmd`, the guard says “No way!”  
- **`/demo/demo2`**: Only people with “play” permission can visit (that’s just `saif`). Others get blocked.  
- **`/demo/demo3`**: Only people with “manager” permission can visit (that’s just `khalid`). He’s the VIP!  
- **`/demo/demo4/{smth}`**: This one’s a bit tricky:  
  - You can get in if your username matches the word you put in the URL (like `/demo/demo4/mhmd` works for `mhmd`).  
  - OR if you’re `khalid` with “manager” permission, you can get in no matter what word you use (like `/demo/demo4/anything`).  
  - **Examples**:  
    - `mhmd` tries `/demo/demo4/mhmd` → Works because his name matches! ✅  
    - `saif` tries `/demo/demo4/saif` → Works because his name matches! ✅  
    - `khalid` tries `/demo/demo4/anything` → Works because he’s the manager! ✅  
    - `mhmd` tries `/demo/demo4/saif` → Fails because his name isn’t `saif` and he’s not a manager. ❌  

---

### 3. PasswordValidator  
**File**: `PasswordValidator.java`  
**What it does**:  
This is a helper that checks if a username and password are correct, but it’s not being used in this app right now. It’s like a spare key that’s just sitting there.  

#### How it works (Simple Version):  
- It says “Yes” (true) only if the username is `user` and the password is `pass`. Any other combo gets a “No” (false).  
- **Note**: This isn’t connected to the app yet. It’s just sitting there waiting. If you want to use it, you’d need to add more code (like a custom checker).  
- **Example Use**: You could use it later if you want to make your own way of checking passwords instead of the built-in user list.  

---

## How the Application Works?  
### 1. Sending a Request  
When you try to visit a page like `/demo/demo1`, the app says, “Hey, log in first!” You give it a username and password (like `mhmd:123`), and if they’re right, it lets you keep going.  

### 2. Checking Permission  
The `@PreAuthorize` is like a guard who looks at your permission slip. If it matches what the page needs (like “read” for `/demo/demo1`), you’re in! If not, you get a “403 Forbidden” error, which means “Sorry, you can’t come in.”  

#### Examples:  
- Try `/demo/demo1` with `mhmd:123` → Works because `mhmd` has “read” permission! ✅  
- Try `/demo/demo1` with `saif:456` → Fails because `saif` doesn’t have “read” (he gets 403). ❌  
- Try `/demo/demo4/khalid` with `khalid:789` → Works because `khalid` is a manager! ✅  
- Try `/demo/demo4/mhmd` with `mhmd:123` → Works because the name matches! ✅  

---

## How to Use It?  
- Visit `/demo/demo1` and use `mhmd:123` → Works because he has “read”! ✅  
- Visit `/demo/demo2` and use `saif:456` → Works because he has “play”! ✅  
- Visit `/demo/demo3` and use `khalid:789` → Works because he has “manager”! ✅  
- Visit `/demo/demo4/mhmd` with `mhmd:123` → Works because the name matches! ✅  
- Visit `/demo/demo4/anything` with `khalid:789` → Works because he’s the manager! ✅  

---

## Extra Notes  
- The `@PreAuthorize` in `/demo/demo4` uses a special trick (called SpEL) to check two things: your name or if you’re a manager. It’s like a smart guard! 🤓  
- The `PasswordValidator` is there but not doing anything yet. If you want it to work, you’d need to connect it with extra code. 🔧  

**Happy Coding!** 🎉