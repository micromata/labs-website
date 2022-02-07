---
layout: page
width: small
hero:
    title: Kapitel 7– Tests mit Spring Security
    subtitle: Das siebte Kapitel der Tutorial-Beitragsreihe zu Spring Security führt die Entwicklung von Tests im Hinblick auf Spring Security vor.
    image:
permalink: /publikationen/tutorial-spring-security/tests-mit-spring-security/
author: jfast
scope: [Tutorial Spring Security]
---

Dies ist das siebte Kapitel der **Tutorial-Beitragsreihe** zu **Spring Security**. Dieser Beitrag führt die Entwicklung von Tests im Hinblick auf Spring  Security vor. Die dabei gezeigten und weitere Testfunktionalitäten von  Spring Security können [hier](http://docs.spring.io/spring-security/site/docs/current/reference/html/test-method.html) und [hier](http://docs.spring.io/spring-security/site/docs/current/reference/html/test-mockmvc.html) nachgeschlagen werden. Den Source Code zu diesem Tutorial findet ihr auf dem [Micromata Github Bereich](https://github.com/micromata/labs-spring-security/tree/master/chapter7/testing-spring-security).

## Vorbereitung

Als Basis dient das Projekt aus dem Beitrag [**Rollen und Zugriffsrechte mit Spring Security**](https://labs.micromata.de/best-practices/tutorial-spring-security-rollen-und-zugriffsrechte-mit-spring-security/). Hier werden nun zwei weitere Dependencies hinzugefügt:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
 
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>4.1.3.RELEASE</version>
</dependency>
```

## Grundgerüst der Backend-Tests

Das Grundgerüst der Backend-Tests sieht wie folgt aus:

```java
package de.micromata.spring.security.example.tests;
 
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
 
@RunWith(SpringRunner.class)
@SpringBootTest()
public class ExampleTest {
}
```

Die Annotation `@RunWith(SpringRunner.class)` ist ein Alias zu der Annotation `@RunWith(SpringJUnit4ClassRunner.class)`. Sie erweitert die Unit-Klassen um das **Spring-TestContext**-Framework. Die Annotation `@SpringBootTest()` fügt diesem Framework noch weitere Funktionen hinzu und ermöglicht uns beispielsweise den Einsatz von `@Autowired` in unseren Tests.
 <h2

```java
@Test
@WithMockUser
public void exampleTest() {
}
```

`@WithMockUser` ist eine Annotation, um einen  eingeloggten User zu simulieren. Dabei kann die Annotation auch über der Testklasse direkt platziert werden, wodurch alle Tests mit einem  simulierten User ausgeführt werden. Um dies wiederum zu umgehen und  innerhalb solch einer Testklasse einen Test ohne simulierten User zu  erstellen, gibt es die Annotation `@WithAnonymousUser`.

`@WithMockUser` funktioniert jedoch nur, solange UserDetails nicht angepasst wurde, was wir jedoch in dem Kapitel [JPA und Spring Security](https://labs.micromata.de/best-practices/jpa-und-spring-security/) gemacht haben, weshalb bei Nutzung der Annotation folgende Exception auftritt:

```java
org.springframework.security.core.userdetails.User 
cannot be cast to de.micromata.spring.security.example.data.User
```

Für diesen Fall ist die Annotation `@WithUserDetails` vorgesehen. Der Einsatz dieser Annotation ist im folgenden Codebeispiel zu sehen.

```java
@RunWith(SpringRunner.class)
@SpringBootTest()
public class MessageTest {
 
    @Autowired
    MessageRepository messageRepository;
 
    @Autowired
    UserRepository userRepository;
 
    @Test(expected = AuthenticationCredentialsNotFoundException.class)
    public void getMessageWithOutUser() {
        messageRepository.findById(110l);
    }
 
    @Test
    @WithUserDetails(value = "tom")
    public void getMessage() {
        messageRepository.findById(110l);
        messageRepository.findOne(110l);
    }
 
    @Test
    @WithUserDetails(value = "tom")
    public void getMessageFromAnOtherUser() {
        try {
            messageRepository.findById(100l);
            assertTrue("Tom can see the message of max by findById", false);
        } catch (AccessDeniedException e) {
        }
        try {
            messageRepository.findOne(100l);
            assertTrue("Tom can see the message of max by findOne", false);
        } catch (AccessDeniedException e) {
        }
    }
 
    @Test
    @WithUserDetails(value = "admin")
    public void getMessageFromAnOtherUserAsAdmin() {
        try {
            messageRepository.findById(110l);
        } catch (AccessDeniedException e) {
            assertTrue("The admin should see the message of tom by findById", false);
        }
        try {
            messageRepository.findOne(110l);
            assertTrue("The admin can see the message of max by findOne", false);
        } catch (AccessDeniedException e) {
        }
    }
}
```

Der erste Test erwartet eine Exception beim Suchen von  Nachrichten, da kein User simuliert wird, während die anderen Tests die  im vorherigen Kapitel [Rollen und Zugriffsrechte mit Spring Security](https://labs.micromata.de/best-practices/tutorial-spring-security-rollen-und-zugriffsrechte-mit-spring-security/) eingeführten Berechtigungen testen. Im Gegensatz zur Annoation `WithMockUser` muss der mit `WithUserDetails` angegebene User auch existieren. Wird kein Wert für `value` angegeben, so wird per Default `user` benutzt. Da solch ein User in unserer Applikation nicht existiert, setzen wir den Wert auf `tom` bzw. `admin`.

## Benutzerspezifische Annotation zum Simulieren von Usern

Damit der Kontext der Applikation geändert werden kann, ohne dabei beispielsweise das Script `data.sql` anpassen zu müssen, kann eine eigene Annotation erstellt werden, die  genau dieses Verhalten bewirkt. Die Annotation selbst ist dabei sehr  überschaubar.

```java
package de.micromata.spring.security.example.utils;
 
import org.springframework.security.test.context.support.WithSecurityContext;
 
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
 
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithCustomMockUserSecurityContextFactory.class)
public @interface WithCustomMockUser {
 
    String username() default "newUser";
 
}
```

Der wichtigste Punkt ist die Annotaion `WithSecurityContext`. Darüber geben wir unsere eigene Implementierung von `WithSecurityContextFactory` an.

```java
package de.micromata.spring.security.example.utils;
 
import de.micromata.spring.security.example.data.Message;
import de.micromata.spring.security.example.data.MessageRepository;
import de.micromata.spring.security.example.data.User;
import de.micromata.spring.security.example.data.UserRepository;
import de.micromata.spring.security.example.security.user.AuthenticatedUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.test.context.support.WithSecurityContextFactory;
 
public class WithCustomMockUserSecurityContextFactory
    implements WithSecurityContextFactory<WithCustomMockUser> {
 
    @Autowired
    private AuthenticatedUserService authenticatedUserService;
 
    @Autowired
    private UserRepository userRepository;
 
    @Autowired
    private MessageRepository messageRepository;
 
    @Override
    public SecurityContext createSecurityContext(WithCustomMockUser customUser) {
        String username = customUser.username();
        createCustomValues(username);
        return authenticate(username);
    }
 
    public void createCustomValues(String username) {
        User user = userRepository.save(new User(4, username, "password", "ROLE_USER"));
        Message message = new Message();
        message.setTitle("custom");
        message.setText("custom");
        message.setUser(user);
        messageRepository.save(message);
    }
 
    public SecurityContext authenticate(String username) {
        UserDetails principal = authenticatedUserService.loadUserByUsername(username);
        Authentication authentication = new UsernamePasswordAuthenticationToken(principal, principal.getPassword(),
            principal.getAuthorities());
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authentication);
        return context;
    }
}
```

Als generischer Typ wir die zuvor gezeigte Annotation übergeben. In der Funktion `createSecurityContext` kann der Applikationskontext beliebig angepasst werden. In unserem Fall legen wir den über das Attribut `username` aus der Annotation` WithCustomMockUser` angegebenen User an. Danach wird für diesen User eine Nachricht  angelegt und abschließend eine Authentifizierung durchgeführt. Nun kann  die Annotation in den Tests verwendet werden:

```java
@Test
@WithCustomMockUser()
public void getMessageWithCustomMockUser() {
    User user = userRepository.findByUsername("newUser");
    assertTrue("user is null", user != null);
 
    Iterable<Message> messages = messageRepository.findByUserId(user.getId());
    assertTrue("newUser has no messages", messages != null && messages.iterator().hasNext());
}
```

In diesesem Test wird lediglich geprüft, dass der User `newUser` existiert und auch mindestens eine Nachricht besitzt.

## Grundgerüst der Controller Tests

Um die Endpunkte der Applikation zu testen, wird zunächst folgendes Grundgerüst angelegt:

```java
package de.micromata.spring.security.example.tests;
 
import org.junit.Before;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;
 
import static org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers.springSecurity;
 
@RunWith(SpringRunner.class)
@SpringBootTest()
public class ControllerTest {
 
    @Autowired
    private WebApplicationContext context;
 
    private MockMvc mvc;
 
    @Before
    public void setup() {
        mvc = MockMvcBuilders
            .webAppContextSetup(context)
            .apply(springSecurity())
            .build();
    }
}
```

Die Annotationen `@RunWith(SpringRunner.class)` und `@SpringBootTest()` haben wir bereits kennengelernt. Neu hinzugekommen sind an dieser Stelle der über `@Autowired` eingebundene `WebApplicationContext` sowie Springs `MockMvc`, welches vor jedem Test neu initialisiert wird.

## Controller Testbeispiele

Um den Einsatz von MockMvc vorzustellen, folgen nun einige Codebeispiele:

```java
@Test
public void getIndexWithoutLogin() throws Exception{
    ResultActions action = mvc.perform(get("/"));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 302 ; current status code = " + status, status == 302);
    String redirectURL = action.andReturn().getResponse().getHeader("Location");
    assertTrue("no redirect to the login page", "http://localhost/login".equals(redirectURL));
}
 
@Test
public void getIndexWithLogin() throws Exception{
    ResultActions action = mvc.perform(get("/").with(user("user")));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 200 ; current status code = " + status, status == 200);
}
```

Der erste Test führt einen GET Request gegen den Index-Endpunkt  aus. Das erwartete verhalten ist, dass ein Redirect zur Login-Seite  vollzogen wird, da kein Login stattgefunden hat. Im zweiten Test wird  dies mit einem eingeloggten User probiert, weshalb hier auch der  erfolgreiche Statuscode 200 erwartet wird. Der User braucht dabei nicht  zu existieren.

```java
@Test
public void getConsoleWithLoginUser() throws Exception{
    ResultActions action = mvc.perform(get("/console").with(user("user")));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 403 ; current status code = " + status, status == 403);
}
 
@Test
public void getConsoleWithLoginAdmin() throws Exception{
    ResultActions action = mvc.perform(post("/console").with(user("admin").roles("ADMIN")));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 404 ; current status code = " + status, status == 404);
}
 
@Test
public void getNoSecurity() throws Exception{
    ResultActions action = mvc.perform(get("/noSecurity"));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 200 ; current status code = " + status, status == 200);
}
```

In diesen Tests werden weitere Endpunkte getestet und der Einsatz von `roles` vorgeführt.

```java
@Test
public void checkCsrfWithToken() throws Exception{
    ResultActions action = mvc.perform(post("/noSecurity").with(csrf()).with(user("user")));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 200 ; current status code = " + status, status == 200);
}
 
@Test
public void checkCsrfWithOutToken() throws Exception{
    ResultActions action = mvc.perform(post("/noSecurity").with(user("user")));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 403 ; current status code = " + status, status == 403);
}
```

Hier wird der CSRF-Schutz unserer Applikation getestet.

```java
@Test
public void checkLogin() throws Exception{
    ResultActions action = mvc.perform(formLogin("/login").user("admin").password("password"));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 302 ; current status code = " + status, status == 302);
    String redirectURL = action.andReturn().getResponse().getHeader("Location");
    assertTrue("login with valid user and password not possible", "/".equals(redirectURL));
}
 
@Test
public void checkLoginWithWrongPassword() throws Exception{
    ResultActions action = mvc.perform(formLogin("/login").user("admin").password("pass"));
    int status = action.andReturn().getResponse().getStatus();
    assertTrue("expected status code = 302 ; current status code = " + status, status == 302);
    String redirectURL = action.andReturn().getResponse().getHeader("Location");
    assertTrue("possible login with wrong password", "/login?error".equals(redirectURL));
}
```

Abschließend sind an dieser Stelle noch zwei Tests zum Login zu sehen.
