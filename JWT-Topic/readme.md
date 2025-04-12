# 🚀 JWT TOPIC - JWT Authentication with Spring Boot

Welcome to **JWT TOPIC**! 🎉 This project demonstrates a secure authentication and authorization system using **JWT (JSON Web Token)** in a Spring Boot application. It allows users to log in, receive a token, and access protected resources based on their roles (e.g., ADMIN or USER). Let’s dive into the details! 🧠

---

## 🌟 Overview
This project builds a secure system where users can authenticate using **JWT tokens** 🔒 and access resources based on their **roles**. It uses **Spring Security** to manage security rules, ensuring only *`authorized`* users can access protected endpoints. JWT tokens are **`stateless`**, meaning the server doesn’t need to store session data—everything is in the token! 💡

---

## 🔧 Key Components (Detailed Breakdown)

### 1️⃣ **JWTService** 🛠️
- 🟠 **What It Does**: The `heart` of JWT handling! It **`generates`** and **`validates`** JWT tokens for secure authentication.
- 🟠 **Key Functions**:
  - 🔵 **Generate Token (`generateJWTForUser`)**: Creates a JWT token for a user after login. It includes:
    - The user’s `username` (e.g., "john").
    - The user’s `role` (e.g., ADMIN or USER).
    - An expiration time (e.g., 1 hour ⏰).
    - Signed with a secret key using the HMAC256 algorithm 🔐.
  - 🔵 **Extract Username (`getUsername`)**: Decodes the token to get the `username`.
  - 🔵 **Extract Role (`getRole`)**: Decodes the token to get the `role` for authorization.
- 🟠 **How It Works**:
  - Uses the `jjwt` library to create and decode tokens.
  - The token is signed with a secret key (loaded from `application.properties`) to prevent tampering.
  - Example token: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` (don’t worry, it’s just a secure string! 😄)
- 🟠 **Why It Matters**: The JWT token acts like a digital ID card 🪪—it proves who the user is and what they’re allowed to do.

### 2️⃣ **JWTAuthenticationFilter** 🕵️‍♂️
- 🟠 **What It Does**: A gatekeeper for every HTTP request! It checks if the request has a valid JWT token.
- 🟠**Key Functions**:
  - 🔵 **Extract Token**: Looks for the "Authorization" header (e.g., `Authorization: Bearer <token>`).
  - 🔵 **Validate Token**: Uses `JWTService` to decode the token and extract `username` and `role`. If the token is `invalid` (e.g., expired ⏳ or tampered), it returns a 401 **`(Unauthorized)`** response.
  - 🔵 **Authenticate User**: Creates a `UsernamePasswordAuthenticationToken` with the user’s details and sets it in Spring Security’s context.
  - 🔵 **Add Request Details**: Adds metadata like the user’s IP address to the authentication.
- 🟠 **How It Works**:
  - Extends `OncePerRequestFilter`, so it runs once per request.
  - If no token is found, it skips authentication (Spring Security will block the request if needed).
  - After authentication, it passes the request to the next filter or controller.
- 🟠 **Why It Matters**: Ensures every request is secure by validating the JWT token, keeping unauthorized users out! 🚫



### 3️⃣ **WebSecurityConfig** ⚙️
- 🟠 **What It Does**: Configures Spring Security to define the app’s security rules.
- 🟠 **Key Configurations**:
  - 🔵 **Disable CSRF**: `http.csrf(c -> c.disable())`—CSRF isn’t needed since we’re using JWT (stateless authentication).
  - 🔵 **Disable CORS**: `http.cors(c -> c.disable())`—Disabled for simplicity (in production, you’d configure allowed origins 🌐).
  - 🔵 **Endpoint Rules**:
    - `/auth/**` and `/public/**` are public (e.g., `/auth/login` can be accessed by anyone ✅).
    - All other endpoints require authentication (e.g., `/api/users` needs a valid token 🔑).
  - 🔵 **Add JWT Filter**: Adds `JWTAuthenticationFilter` to the security chain to process tokens.
- 🟠 **How It Works**:
  - Defines rules for which endpoints are public or protected.
  - Integrates the JWT filter to validate tokens before Spring Security processes the request.
- 🟠 **Why It Matters**: Sets the boundaries of what’s public and what’s protected, ensuring security across the app! 🛡️

### 4️⃣ **User Entity** 👤
- **What It Does**: Represents a user in the database with all their details.
- **Key Fields**:
  - `id`: Unique ID (auto-generated).
  - `username`: User’s login name (unique).
  - `password`: Hashed password (for security 🔒).
  - `email`: User’s email (unique).
  - `firstName`, `lastName`: Personal details.
  - `role`: Enum (e.g., ADMIN, USER) for permissions.
  - `verified`: Boolean to track email verification.
  - `otp`, `otpExpiration`: For email verification.
  - `resetOtp`, `resetOtpExpiration`: For password reset.
- 🟠 **How It Works**:
  - Maps to a `user` table in the database using JPA annotations.
  - Sensitive fields like `password` are hidden from API responses with `@JsonIgnore`.
- 🟠 **Why It Matters**: Stores user data and roles, which are used for authentication and authorization.

### 5️⃣ **CustomUserDetailsService** 📋
- 🟠 **What It Does**: Loads user data from the database for Spring Security to authenticate users.
- 🟠 **Key Functions**:
  - 🔵 **Load User (`loadUserByUsername`)**: Fetches the user by `username` from the database.
  - 🔵 **Build UserDetails**: Creates a `UserDetails` object with the user’s `username`, `password`, and `role`.
- 🟠 **How It Works**:
  - 🔵 Uses `UserRepo` to query the database.
  - 🔵 Throws `UsernameNotFoundException` if the user isn’t found.
- 🟠 **Why It Matters**: Connects Spring Security to the database, allowing user authentication during login.

---

<!-- 🖌 1️⃣ 2️⃣ 3️⃣ 4️⃣ 5️⃣ ➡ ⬅⬇↗☑🔴🟠🔵🟣🟣🟪🟦🟩 
💫💥🚀 
6️⃣7️⃣8️⃣9️⃣
❌💯❎✅⏩➖
-->


## 🛠️ How It Works (Step-by-Step Flow)

- 1️⃣ **User Logs In**:
   - 🟣 User sends `POST /auth/login` with `username` and `password` (e.g., `{"username": "john", "password": "pass123"}`).
   - 🟣 Server verifies credentials using `CustomUserDetailsService` and a password encoder (e.g., BCrypt).

- 2️⃣ **Token Generation**:
   - 🟣 If credentials are valid, `JWTService` generates a JWT token with the user’s `username` and `role`.
   - 🟣 Token example: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` (a secure, encoded string).

- 3️⃣ **Token Sent to Client**:
   - 🟣 Server responds with the token, and the client stores it (e.g., in local storage or a cookie).

- 4️⃣ **Access Protected Endpoint**:
   - 🟣 Client sends a request to a protected endpoint (e.g., `GET /api/users`) with the token in the header: `Authorization: Bearer <token>`.

- 5️⃣ **Token Validation**:
   - 🟣 `JWTAuthenticationFilter` extracts and validates the token using `JWTService`.
   - 🟣 If valid, it sets the user in the `SecurityContextHolder` with their `role`.

- 6️⃣ **Authorization Check**:
   - 🟣 Spring Security checks if the user’s role allows access to the endpoint (e.g., "ROLE_ADMIN" for `/api/admin`).
   - 🟣 If allowed, the request proceeds; if not, a 403 (Forbidden) response is returned.

- 7️⃣ **Token Expiration**:
   - 🟣 If the token expires (e.g., after 1 hour ⏰), the user must log in again to get a new token.

---

## ⚡ Setup Instructions

1. **Add Dependencies** to `pom.xml` 📦:
   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-security</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-data-jpa</artifactId>
       </dependency>
       <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-api -->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-api</artifactId>
			<version>0.12.3</version>
		</dependency>
		<dependency>
			<groupId>com.auth0</groupId>
			<artifactId>java-jwt</artifactId>
			<version>4.2.1</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-impl -->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-impl</artifactId>
			<version>0.12.3</version>
			<scope>runtime</scope>
		</dependency>

		<!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-jackson -->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-jackson</artifactId>
			<version>0.12.3</version>
			<scope>runtime</scope>
		</dependency>


       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>8.0.33</version>
       </dependency>
   </dependencies>
   ```

2. **Configure Application Properties** (`application.properties`) ⚙️:
   - JWT settings:
     ```
     jwt.algorithm.key=your-very-secure-secret-key
     jwt.issuer=ProjectGraduation
     jwt.expiryInSeconds=3600
     ```
   - Database settings (e.g., MySQL):
     ```
     spring.datasource.url=jdbc:mysql://localhost:3306/yourdb
     spring.datasource.username=root
     spring.datasource.password=yourpassword
     spring.jpa.hibernate.ddl-auto=update
     ```

3. **Database Setup** 🗄️:
   - Create a database (e.g., `yourdb` in MySQL).
   - JPA will auto-create the `user` table based on the `User` entity.

4. **Run the Application** 🚀:
   - Start the Spring Boot app.
   - Test `/auth/login` to get a token.
   - Use the token to access protected endpoints like `/api/users`.

---

## 🌐 Example Flow
1. **Login**:
   - Request: `POST /auth/login` with `{"username": "john", "password": "pass123"}`.
   - Response: `{"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}`.

2. **Access Resource**:
   - Request: `GET /api/users` with header `Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`.
   - Server validates the token and responds with the resource (or a 401/403 if invalid).

---

## 🎯 Why Use JWT?
- **Stateless**: No server-side session storage—everything is in the token! 🗿
- **Scalable**: Perfect for distributed systems like microservices 🌍.
- **Secure**: Tokens are signed to prevent tampering, and expiration ensures they don’t last forever 🔐.

---
 
**Happy coding! 💻**