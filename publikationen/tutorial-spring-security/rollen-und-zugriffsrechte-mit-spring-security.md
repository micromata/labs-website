---
layout: page
width: small
hero:
    title: Kapitel 6 - Rollen und Zugriffsrechte mit Spring Security
    subtitle: Im sechsten Kapitel der Tutorial-Beitragsreihe zu Spring Security geht es darum, den Zugriff auf die H2-Konsole auf Administratoren zu beschränken sowie Nachrichten für die Benutzer zu erstellen, deren Zugriff eingeschränkt wird.
permalink: /publikationen/tutorial-spring-security/rollen-und-zugriffsrechte-mit-spring-security/
author: jfast
scope: [Tutorial Spring Security]
---

Dies ist das sechste Kapitel der **Tutorial-Beitragsreihe** zu **Spring Security**. In diesem Beitrag geht es darum, den Zugrif auf die H2 Console auf  Administratoren zu beschränken sowie Nachrichten für die Benutzer zu  erstellen, deren Aufruf eingeschränkt wird. Dieser Beitrag baut auf dem  Projekt aus [HTTPS mit Spring Boot](/publikationen/tutorial-spring-security/https-mit-spring-boot/) auf. Den Source Code zu diesem Tutorial findet ihr auf dem [Micromata Github Bereich](https://github.com/micromata/labs-spring-security/tree/master/chapter6/roles-and-permissions-in-spring-security).

## URL’s auf Rollen beschränken

Um beispielsweise alle URL’s unter `/console/*` auf Administratoren zu beschränken, genügt es, die Zeile `.antMatchers("/console/**").hasRole("ADMIN")` in der `WebSecurityConfig` hinzuzufügen.

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers("/noSecurity").permitAll()
        .antMatchers("/console/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        .and().formLogin().loginPage("/login").permitAll()
        .and().logout().permitAll();
 
    http.csrf().ignoringAntMatchers("/console/**")
        .and().headers().frameOptions().sameOrigin();
}
```

Da unsere Benutzer jedoch noch über keine Rollen verfügen, müssen diese nun entsprechend in der Klasse `User` hinzugefügt werden.

```java
public User(long id, String userName, String password, String role) {
    this.id = id;
    this.username = userName;
    this.password = password;
    this.role = role;
}
 
@NotEmpty(message = "role is required")
private String role;
 
public String getRole() {
    return role;
}
 
public void setRole(String role) {
    this.role = role;
}
```

… und in der Klasse `AuthenticatedUser` gesetzt werden.

```java
protected AuthenticatedUser(User user) {
    super(user.getId(), user.getUsername(), user.getPassword(), user.getRole());
}
 
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    return AuthorityUtils.createAuthorityList(getRole());
}
```

## Nachrichten

Um die Funktionsweise der Zugriffsrechte vorzuführen, erstellen wir zunächst ein Nachrichtensystem. Dazu legen wir die Klasse `Message` an.

```java
package de.micromata.spring.security.example.data;
 
import org.hibernate.validator.constraints.NotEmpty;
 
import javax.persistence.*;
import javax.validation.constraints.NotNull;
 
@Entity
public class Message {
    @Id
    @GeneratedValue()
    private long id;
 
    @OneToOne
    @NotNull
    private User user;
 
    @NotEmpty(message = "text is required")
    private String text;
 
    @NotEmpty(message = "title is required")
    private String title;
 
    public User getUser() {
        return user;
    }
 
    public void setUser(User user) {
        this.user = user;
    }
 
    public String getText() {
        return text;
    }
 
    public void setText(String text) {
        this.text = text;
    }
 
    public String getTitle() {
        return title;
    }
 
    public void setTitle(String title) {
        this.title = title;
    }
 
    public long getId() {
        return id;
    }
 
    public void setId(long id) {
        this.id = id;
    }
}
```

… sowie ein entsprechendes Repository:

```java
package de.micromata.spring.security.example.data;
 
import org.springframework.data.repository.CrudRepository;
 
public interface MessageRepository extends CrudRepository<Message, Long> {
 
    Message findById(Long id);
 
    Iterable<Message> findByUserId(Long id);
}
```

Außerdem wird der Controller um die folgenden Methoden erweitert:

```java
@RequestMapping("/messages")
public String listMessages(@AuthenticationPrincipal User user, Model model) {
    Iterable<Message> messages = messageRepository.findByUserId(user.getId());
    model.addAttribute("messages", messages);
    return "listMessages";
}
 
@RequestMapping("/message/{id}")
public String viewMessage(@PathVariable Long id, Model model) {
    Message message = messageRepository.findById(id);
    model.addAttribute("message", message);
    return "viewMessage";
}
```

Das Template `listMessages`:

```java
<html xmlns:th="http://www.thymeleaf.org">
    <head th:replace="fragments/headerAndNav :: header"/>
    <body>
        <div th:replace="fragments/headerAndNav :: navbar"/>
        <div class="container">
            <ul th:each="message : ${messages}">
                <li><a href="viewMessage.html" th:href="@{'/message/' + ${message.id}}" th:text="${message.title}">The title</a></li>
            </ul>
        </div>
    </body>
</html>
```

Das Template `viewMessage`:

```java
<html xmlns:th="http://www.thymeleaf.org">
    <head th:replace="fragments/headerAndNav :: header"/>
    <body>
        <div th:replace="fragments/headerAndNav :: navbar"/>
        <div class="container">
            <h1><span th:text="${message.title}">The message title</span></h1>
            <span th:text="${message.text}">The message text</span>
        </div>
    </body>
</html>
```

Das Startscript `data.sql` wird um Nachrichten und weitere User erweitert:

```java
insert into user(id,username,password,role) values (0,'max','password', 'ROLE_USER');
insert into user(id,username,password,role) values (1,'tom','password', 'ROLE_USER');
insert into user(id,username,password,role) values (2,'admin','password', 'ROLE_ADMIN');
 
insert into message(id,user_id,title,text) values (100,0,'Message for Max','This message is for Max. Under /message/100 only Max or an admin should see this message. Under /privateMessage/100 only Max should see this message.');
insert into message(id,user_id,title,text) values (110,1,'Message for Tom','This message is for Tom. Under /message/110 only Tom or an admin should see this message. Under /privateMessage/110 only Tom should see this message.');
```

Unter https://localhost:8443/messages kann ein eingeloggter Nutzer nun eine Liste der eigenen Nachrichten sehen.  Ist man z.B. als Max eingeloggt, so wird man bei einem Klick auf die  angezeigte Nachricht nach https://localhost:8443/message/100 weitergeleitet. Durch anpassen der URL auf https://localhost:8443/message/110 kann Max jedoch die Nachricht von Tom sehen.

## Zugriff auf Nachrichten beschränken

Damit Max keinen Zugang mehr zu den Nachrichten von Tom hat, legen wir zunächst die Klasse `SimplePermissionEvaluator` an:

```java
package de.micromata.spring.security.example.security.permission;
 
import de.micromata.spring.security.example.data.Message;
import de.micromata.spring.security.example.data.User;
import org.springframework.security.access.PermissionEvaluator;
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Component;
 
import java.io.Serializable;
 
@Component
public class SimplePermissionEvaluator implements PermissionEvaluator {
 
    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        if (authentication == null) {
            return false;
        }
        Message message = (Message) targetDomainObject;
        if (message == null) {
            return true;
        }
        User user = (User) authentication.getPrincipal();
 
        if (user.getId() == message.getUser().getId()) {
            return true;
        }
 
        if ("privateMessage".equals(permission)) {
            return false;
        } else if ("message".equals(permission) && "ROLE_ADMIN".equals(user.getRole())) {
            return true;
        } else {
            return false;
        }
    }
 
    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        return false;
    }
}
```

Die Klasse implementiert das Interface `PermissionEvaluator`. Da nachfolgend nur die erste Methode benutzt wird, sparen wir uns an  dieser Stelle eine vernünftige Implementierung der zweiten Funktion und  liefern lediglich `false` zurück. Die obere `hasPermission-`Methode stellt sicher, dass eine Nachricht zum aktuellen Nutzer gehört, falls der Wert von `permisssion` „privateMessage“ beträgt. Wenn der Wert jedoch auf „message“ gesezt  ist, dann darf die Nachricht auch von einem anderen Benutzer eingesehen  werden, falls dieser die Rolle `ADMIN` hat.

Damit wir unsere Implementierung von `PermissionEvaluator` über eine Annotation verwenden dürfen, müssen wir zunächst die Annotation @`EnableGlobalMethodSecurity(prePostEnabled = true)` zu unserer Klasse `WebSecurityConfig` hinzfügen. Neben `prePostEnabled` existieren noch `securedEnabled `und `jsr250Enabled`. Die durch `securedEnabled` aktivierten Annotationen können lediglich mit einfachen Listen von z. B. Rollen umgehen, während die durch `prePostEnabled` aktivierten Annotationen mit komplexen [Spring-EL](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html)-Ausdrücken arbeiten können. `jsr250Enabled` aktiviert spezielle `jsr250-Annotationen`. Wir werden nachfolgend die Annotation `@PostAuthorize` verwenden, da diese benutzt wird, um angeforderte Daten einzuschränken. `@PreAuthorize` wird verwendet, um beispielsweise Vorgänge wie das Anlegen oder Löschen von Nutzern einzuschränken.

Die Einschränkung findet in der Klasse `MessageRepository` statt:

```java
package de.micromata.spring.security.example.data;
 
import org.springframework.data.repository.CrudRepository;
import org.springframework.security.access.prepost.PostAuthorize;
 
public interface MessageRepository extends CrudRepository<Message, Long> {
 
    @PostAuthorize("hasPermission(returnObject, 'message')")
    Message findById(Long id);
 
    Iterable<Message> findByUserId(Long id);
 
    @PostAuthorize("hasPermission(returnObject, 'privateMessage')")
    Message findOne(Long id);
 
}
```

Neben der Annotation `@PostAuthorize`, welche über `hasPermission` unsere Implementierung von `PermissionEvaluator` aufruft, wurde noch die Methode `findOne` hinzugefügt, welche in diesem Fall von Spring identisch zu `findById` implementiert wird. Dies dient dazu, möglichst einfach zwei  verschiedene Rechte vorzuführen. Nachrichten, die über findById gefunden werden, können also sowohl vom aktuellen Nutzer als auch von einem  Administrator eingesehen werden, während Nachrichten, die über `findOne` gefunden werden, nur vom aktuellen Nutzer eingesehen werden können.

Damit dies getestet werden kann, wird der Controller um eine weitere Funktion erweitert:

```java
@RequestMapping("/privateMessage/{id}")
public String viewPrivateMessage(@PathVariable Long id, Model model) {
    Message message = messageRepository.findOne(id);
    model.addAttribute("message", message);
    return "viewMessage";
}
```

Nun kann ein Nutzer unter /message/{id} und /privateMessage/{id}  nur die eigenen Nachrichten einsehen, während ein Administrator unter  /message/{id} die Nachrichten aller Nutzer sehen kann, jedoch über  /privateMessage/{id} nur die eigenen.
