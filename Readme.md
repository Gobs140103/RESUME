Creating a login page using React for the frontend and Spring Boot for the backend involves several steps. Here’s a detailed guide to help you through the process:

### Prerequisites
- Basic knowledge of Java, Spring Boot, React, and JavaScript.
- Install Node.js and npm.
- Install Java and Maven.

### Step 1: Set Up the Spring Boot Backend

1. **Create a Spring Boot Project**
   - Use Spring Initializr (https://start.spring.io/) to generate a Spring Boot project with dependencies such as Spring Web, Spring Security, and Spring Data JPA.
   - Download the project and import it into your IDE (e.g., IntelliJ IDEA, Eclipse).

2. **Configure Spring Security**
   - Add Spring Security configuration to handle authentication.

   ```java
   // SecurityConfig.java
   package com.example.demo.security;

   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.security.config.annotation.web.builders.HttpSecurity;
   import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
   import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
   import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
   import org.springframework.security.crypto.password.PasswordEncoder;

   @Configuration
   @EnableWebSecurity
   public class SecurityConfig extends WebSecurityConfigurerAdapter {

       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http
               .csrf().disable()
               .authorizeRequests()
               .antMatchers("/login").permitAll()
               .anyRequest().authenticated()
               .and()
               .formLogin()
               .loginProcessingUrl("/login")
               .usernameParameter("username")
               .passwordParameter("password")
               .defaultSuccessUrl("/home", true)
               .failureUrl("/login?error=true");
       }

       @Bean
       public PasswordEncoder passwordEncoder() {
           return new BCryptPasswordEncoder();
       }
   }
   ```

3. **Create User Entity and Repository**

   ```java
   // User.java
   package com.example.demo.model;

   import javax.persistence.*;

   @Entity
   @Table(name = "users")
   public class User {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;

       private String username;
       private String password;

       // Getters and setters
   }
   ```

   ```java
   // UserRepository.java
   package com.example.demo.repository;

   import com.example.demo.model.User;
   import org.springframework.data.jpa.repository.JpaRepository;

   public interface UserRepository extends JpaRepository<User, Long> {
       User findByUsername(String username);
   }
   ```

4. **Create a REST Controller for Login**

   ```java
   // AuthController.java
   package com.example.demo.controller;

   import org.springframework.web.bind.annotation.*;

   @RestController
   @RequestMapping("/auth")
   public class AuthController {

       @PostMapping("/login")
       public String login(@RequestBody LoginRequest loginRequest) {
           // Implement login logic
           return "Logged in";
       }

       public static class LoginRequest {
           public String username;
           public String password;
       }
   }
   ```

### Step 2: Set Up the React Frontend

1. **Create a React Application**
   - Use Create React App to set up a new React project.

   ```bash
   npx create-react-app react-login
   cd react-login
   ```

2. **Install Axios for HTTP Requests**
   
   ```bash
   npm install axios
   ```

3. **Create a Login Component**

   ```jsx
   // Login.js
   import React, { useState } from 'react';
   import axios from 'axios';

   const Login = () => {
       const [username, setUsername] = useState('');
       const [password, setPassword] = useState('');

       const handleSubmit = async (e) => {
           e.preventDefault();
           try {
               const response = await axios.post('http://localhost:8080/auth/login', {
                   username,
                   password
               });
               console.log(response.data);
           } catch (error) {
               console.error(error);
           }
       };

       return (
           <form onSubmit={handleSubmit}>
               <div>
                   <label>Username:</label>
                   <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} />
               </div>
               <div>
                   <label>Password:</label>
                   <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
               </div>
               <button type="submit">Login</button>
           </form>
       );
   };

   export default Login;
   ```

4. **Update App Component**

   ```jsx
   // App.js
   import React from 'react';
   import Login from './Login';

   const App = () => {
       return (
           <div>
               <h1>Login</h1>
               <Login />
           </div>
       );
   };

   export default App;
   ```

### Step 3: Run the Applications

1. **Run Spring Boot Application**
   - Run the Spring Boot application from your IDE or using Maven:

   ```bash
   mvn spring-boot:run
   ```

2. **Run React Application**
   - Start the React application:

   ```bash
   npm start
   ```

### Summary
You now have a basic login page with React for the frontend and Spring Boot for the backend. This setup handles user login with a form in React, sends the credentials to the Spring Boot backend, and processes the login request. You can further enhance this by adding JWT authentication, error handling, and user feedback.
