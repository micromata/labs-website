---
layout: page
width: small
hero:
    title: Kapitel 3 - JPA und Spring Security
    subtitle: Dieser Beitrag beschreibt den Einsatz von JPA und einer H2-Datenbank im Rahmen des Frameworks Spring Security.
permalink: /publikationen/tutorial-spring-security/jpa-und-spring-security/
author: jfast
scope: [Tutorial Spring Security]
---

Den Source Code zu diesem Tutorial findet ihr auf dem [Micromata Github Bereich](https://github.com/micromata/labs-spring-security/tree/master/chapter3/jpa-and-spring-security).

## Vorbereitung

Als Basis dient das Projekt aus dem Beitrag [Thymeleaf und Spring Security](/publikationen/tutorial-spring-security/thymeleaf-und-spring-security/). Hier werden nun die Dependencies für JPA und H2 hinzugefügt.

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
 
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

Die H2-Datenbank wird von Spring automatisch konfiguriert. Es reicht hier also die Dependency in der Pom anzugeben.

## Die Datenbank Objekte

Zunächst wird eine JPA Entity des Users erstellt.

```java
package de.micromata.spring.security.example.data;
 
import org.hibernate.validator.constraints.NotEmpty;
 
import javax.persistence.*;
 
@Entity
public class User {
 
    @Id
    @GeneratedValue()
    private long id;
 
    @NotEmpty(message = "username is required")
    @Column(unique = true)
    private String username;
 
    @NotEmpty(message = "password is required")
    private String password;
 
    protected User() {}
 
    public User(String userName, String password) {
        this.username = userName;
        this.password = password;
    }
 
    public String getUsername() {
        return username;
    }
 
    public void setUsername(String username) {
        this.username = username;
    }
 
    public String getPassword() {
        return password;
    }
 
    public void setPassword(String password) {
        this.password = password;
    }
}
```

Die Entity wird mit der Annotation `@Entity` als solche gekennzeichnet und besitzt lediglich die Felder id, username und password. Die Id wird durch die Annotation `@GeneratedValue()` generiert. Außerdem wird mit `@Column(unique = true)` festgelegt, dass ein Benutzername nur einmalig vergeben werden darf. Die Annotation `@NotEmpty(message = "password is required")` sorgt dafür, dass beim Speichern des User-Objektes der Benutzername und das Passwort gesetzt werden müssen, andernfalls wird die entsprechend  konfigurierte Fehlermeldung ausgegeben.

Um User in der Datenbank finden zu können, wird ein Repository angelegt:

```java
package de.micromata.spring.security.example.data;
 
import org.springframework.data.repository.CrudRepository;
 
public interface UserRepository extends CrudRepository<User, Long> {
    User findByUsername(String username);
```

Das Repository erbt von der Klasse CrudRepository, welche als  Datentypen die Entity User und den Datentyp der Entity-Id erhält. Für  die Implementation des Repositorys und insbesondere der Methode `findByUsername(String username)` sorgt Spring selbst.

## Anpassung der Security Konfiguration

Die `InMemoryAuthentication` wird nun auf den `UserDetailsService` umgestellt.

```markup
package de.micromata.spring.security.example.security.conf;
 
import de.micromata.spring.security.example.security.user.AuthenticatedUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
 
@Configuration
@EnableWebSecurity
@ComponentScan(basePackageClasses = AuthenticatedUserService.class)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
 
    @Autowired
    private UserDetailsService userDetailsService;
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/noSecurity").permitAll()
            .anyRequest().authenticated()
            .and().formLogin().loginPage("/login").permitAll()
            .and().logout().permitAll();
    }
 
    @Autowired
    public void globalSecurityConfiguration(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService);
    }
 
}
```

Anstelle von `auth.inMemoryAuthentication()` wird `auth.userDetailsService()` in der Methode globalSecurityConfiguration() verwendet. Das übergebene  Interface UserDetailsService wird per @Autowired eingebunden. Damit  Spring die Implementation findet, wird an die Klasse die Annotation `@ComponentScan(basePackageClasses = AuthenticatedUserService.class)` eingefügt. `AuthenticatedUserService` ist dabei unsere Implementierung des Interfaces.

## Der UserDetailsService

Der `UserDetailsService` wird in diesem Beitrag mit `AuthenticatedUserService` implementiert.

```java
package de.micromata.spring.security.example.security.user;
 
import de.micromata.spring.security.example.data.User;
import de.micromata.spring.security.example.data.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
 
@Service
public class AuthenticatedUserService implements UserDetailsService {
 
    @Autowired
    private UserRepository userRepository;
 
    @Override
    public UserDetails loadUserByUsername(String username) {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("The user " + username + " does not exist");
        }
        return new AuthenticatedUser(user);
    }
```

Da es sich bei dieser Klasse um einen Service handelt, wird sie als solche mit `@Service` annotiert. Das zuvor erstellte Repository `UserRepository` wird per `@Autowired` eingebunden. In der Implementation der Methode `loadUserByUsername()` wird der Benutzer aus der Datenbank geladen und in der Klasse `AuthenticatedUser` gekapselt, welche das Interface `UserDetails` implementiert.

```java
package de.micromata.spring.security.example.security.user;
 
import de.micromata.spring.security.example.data.User;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.UserDetails;
 
import java.util.Collection;
 
public class AuthenticatedUser extends User implements UserDetails {
    protected AuthenticatedUser(User user) {
        super(user.getUsername(), user.getPassword());
    }
 
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return AuthorityUtils.createAuthorityList("ROLE_USER");
    }
 
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
 
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
 
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
 
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

Die Klasse erbt von der Klasse `User`, welche durch `getUsername()` und `getPassword()` bereits zwei Methoden von `UserDetails` implementiert. Als Benutzerrolle wird in diesem Beispiel immer `ROLE_USER` gesetzt.

## Initiale Daten

Um initiale Daten in die Datenbank zu spielen, besteht die Möglichkeit zum Beispiel unter `/resources` die Datei `data.sql` anzulegen. Diese wird dann von Spring automatisch angezogen. Näheres dazu kann [hier](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html) gefunden werden.

```java
insert into user(id,username,password) values (0,'user','password');
insert into user(id,username,password) values (1,'admin','password');
```

Durch dieses Script werden die zwei User user und admin angelegt.
