Yes, you'll need a database to store user login information. For simplicity, we'll use H2, an in-memory database, for development. In a production environment, you would use a more robust database like MySQL, PostgreSQL, etc.

### Step-by-Step with Database

#### 1. Setting Up Spring Boot Backend

**1.1. Create a Spring Boot Project:**

- Use Spring Initializr (https://start.spring.io/) to generate a Spring Boot project with dependencies for Spring Web, Spring Security, Spring Data JPA, H2 Database (for development), and Lombok.

**1.2. Configure Application Properties:**

In `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.h2.console.enabled=true
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

# H2 Console URL
spring.h2.console.path=/h2-console

# Security configuration
spring.security.user.name=admin
spring.security.user.password=admin
```

**1.3. Define User Entity:**

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private String role;

    // Getters and Setters
}
```

**1.4. Create User Repository:**

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}
```

**1.5. Implement UserDetailsService for Authentication:**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.stereotype.Service;

import java.util.Collections;

@Service
public class CustomUserDetailsService implements UserDetailsService {
    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("User not found");
        }
        return new org.springframework.security.core.userdetails.User(user.getUsername(), user.getPassword(), 
                Collections.singletonList(new SimpleGrantedAuthority(user.getRole())));
    }
}
```

**1.6. Configure Spring Security:**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private CustomUserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .antMatchers("/login", "/register", "/h2-console/**").permitAll()
            .anyRequest().authenticated()
            .and()
            .formLogin().loginPage("/login").permitAll()
            .and()
            .logout().permitAll();
        
        http.headers().frameOptions().disable(); // To enable H2 console
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**1.7. Create REST Controller for Authentication:**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.web.bind.annotation.*;

import java.security.Principal;

@RestController
@RequestMapping("/api")
public class AuthController {
    @Autowired
    private UserRepository userRepository;

    @PostMapping("/register")
    public ResponseEntity<?> registerUser(@RequestBody User user) {
        if(userRepository.findByUsername(user.getUsername()) != null) {
            return ResponseEntity.badRequest().body("Username is already taken");
        }
        user.setPassword(new BCryptPasswordEncoder().encode(user.getPassword()));
        userRepository.save(user);
        return ResponseEntity.ok("User registered successfully");
    }

    @GetMapping("/user")
    public ResponseEntity<?> getUserDetails(Principal principal) {
        if(principal != null) {
            return ResponseEntity.ok(userRepository.findByUsername(principal.getName()));
        }
        return ResponseEntity.status(401).body("Unauthorized");
    }
}
```

#### 2. Setting Up React Frontend

**2.1. Create a React Application:**

```sh
npx create-react-app banking-app
cd banking-app
npm install axios react-router-dom
```

**2.2. Implement User Login Page:**

Create `Login.js`:

```jsx
import React, { useState } from 'react';
import axios from 'axios';

const Login = () => {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (event) => {
        event.preventDefault();
        try {
            const response = await axios.post('/login', { username, password });
            localStorage.setItem('token', response.data.token);
            window.location.href = '/main';
        } catch (error) {
            console.error('Login failed', error);
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

**2.3. Implement Main Page:**

Create `Main.js`:

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const Main = () => {
    const [user, setUser] = useState(null);

    useEffect(() => {
        const fetchUser = async () => {
            try {
                const response = await axios.get('/api/user', {
                    headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
                });
                setUser(response.data);
            } catch (error) {
                console.error('Failed to fetch user', error);
            }
        };

        fetchUser();
    }, []);

    if (!user) {
        return <div>Loading...</div>;
    }

    return (
        <div>
            <h1>Welcome, {user.username}</h1>
        </div>
    );
};

export default Main;
```

**2.4. Set Up Routing:**

In `App.js`:

```jsx
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Login from './Login';
import Main from './Main';

const App = () => {
    return (
        <Router>
            <Switch>
                <Route path="/login" component={Login} />
                <Route path="/main" component={Main} />
                <Route path="/" component={Login} />
            </Switch>
        </Router>
    );
};

export default App;
```

**2.5. Connect React to Spring Boot:**

In `package.json`, add a proxy to the Spring Boot server:

```json
"proxy": "http://localhost:8080"
```

**2.6. Start the Applications:**

- Start the Spring Boot application:

```sh
mvn spring-boot:run
```

- Start the React application:

```sh
npm start
```

### Conclusion

This setup includes a database to store user login information. You can extend the functionality by adding more features and improving security measures as needed. In a production environment, you should use a more robust database system, secure passwords properly, and consider other security best practices.
