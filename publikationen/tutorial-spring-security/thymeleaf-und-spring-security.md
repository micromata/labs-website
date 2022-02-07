---
layout: page
width: small
hero:
    title: Kapitel 2 – Thymeleaf und Spring Security
    subtitle: Dies ist das zweite Kapitel der Tutorial-Beitragsreihe zu Spring Security. Dieser Beitrag beschreibt den Einsatz von Thymeleaf und Bootstrap sowie den Einbau der Logout-Funktionalität.
permalink: /publikationen/tutorial-spring-security/thymeleaf-und-spring-security/
author: jfast
scope: [Tutorial Spring Security]
---

Den Source Code zu diesem Tutorial findet ihr auf dem [Micromata Github Bereich](https://github.com/micromata/labs-spring-security/tree/master/chapter2/thymeleaf-and-spring-security).

## Vorbereitung

Als Basis dient das Projekt aus dem Beitrag [Einstieg in Spring Security](/publikationen/tutorial-spring-security/einstieg-in-spring-security/). Hier wird nun die Dependency für Thymeleaf hinzugefügt.

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## Anpassung des Controllers

Damit keine festen Strings, sondern mit Thymeleaf erstellte  Template-Dateien gerendert werden, muss der Controller angepasst werden.

```java
package de.micromata.spring.security.example;
 
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
 
@Controller
public class DefaultController {
     
    @RequestMapping("/")
    public String index() {
        return "index";
    }
 
    @RequestMapping("/login")
    public String login(Model model) {
        return "login";
    }
 
    @RequestMapping("/noSecurity")
    public String noSecurity() {
        return "noSecurity";
    }
 
}
```

Der Controller erhält nun `@Controller` als Annotation. Der Unterschied zu `@RestController` ist, dass `@RestController` eine Kombination der Annotationen `@Controller` und `@ResponseBody `darstellt. Durch die Annotation `@ResponseBody` wird der String, den ein Controller als Rückgabewert übergibt, direkt gerendert. Die Annotation `@Controller` sorgt dafür, dass der Rückgabewert als Name des Templates, welches gerendert  werden soll, interpretiert wird. Wird zum Beispiel `/noSecurity` aufgerufen, so wird die Datei `/resources/templates/noSecurity.html` gerendert. Außerdem wurde ein Mapping für `/login` eingefügt, um ein eigenes Template für die Login-Seite zu erstellen.

**headerAndNav.html:**

```java
<html xmlns:th="http://www.thymeleaf.org" xmlns:tiles="http://www.thymeleaf.org">
    <head th:fragment="header">
        <title tiles:fragment="title">Spring Ecurity Example</title>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7" crossorigin="anonymous"/>
    </head>
    <body>
        <nav class="navbar navbar-inverse" th:fragment="navbar">
            <div class="container-fluid" >
                <div class="navbar-inner" th:with="currentUser=${#httpServletRequest.userPrincipal?.name}">
                    <a class="navbar-brand" th:href="@{/}">Home</a>
                    <a class="navbar-brand" th:href="@{/noSecurity}">No Security</a>
                    <div th:if="${currentUser != null}">
                        <form class="navbar-form navbar-right" th:action="@{/logout}" method="post">
                            <input type="submit" class="btn btn-primary" value="Logout" />
                        </form>
                        <p class="navbar-text navbar-right" th:text="${currentUser}">
                            example_user
                        </p>
                    </div>
                </div>
            </div>`
        </nav>
    </body>
</html>
```

Das Fragment `headerAndNav.html` bindet unter anderem Bootstrap im Header ein. Der Header wird hierbei mit `th:fragment` als Thymeleaf-Fragment mit dem Namen `header` gekennzeichnet. Im Body wird eine Navigationsbar angelegt, die ebenfalls als  Thymeleaf-Fragment gekennzeichnet wird und den Namen `navbar` enthält. Die folgenden Templates binden den Header und die Navigationsbar über `th:replace` ein.

**index.html:**

```java
<html>
    <head th:replace="fragments/headerAndNav :: header"/>
    <body>
        <div th:replace="fragments/headerAndNav :: navbar"/>
        <div class="container">
            You can only see this, if you are logged in!
        </div>
    </body>
</html>
```

Dies ist das Template zum Mapping `/`.

**noSecurity.html:**

```java
<html xmlns:th="http://www.thymeleaf.org" xmlns:tiles="http://www.thymeleaf.org">
    <head th:replace="fragments/headerAndNav :: header"/>
    <body>
        <div th:replace="fragments/headerAndNav :: navbar"/>
        <div class="container">
            Everybody can see this!
        </div>
    </body>
</html>
```

Dies ist das Template zum Mapping `/noSecurity`.

**login.html:**

```java
<html xmlns:th="http://www.thymeleaf.org" xmlns:tiles="http://www.thymeleaf.org">
    <head th:replace="fragments/headerAndNav :: header"/>
    <body>
        <div th:replace="fragments/headerAndNav :: navbar"/>
        <div class="container">
            <form name="f" th:action="@{/login}" method="post">
                <fieldset>
                    <legend>Please Login</legend>
                    <div th:if="${param.error}" class="alert alert-error">
                        Invalid username and password.
                    </div>
                    <div th:if="${param.logout}" class="alert alert-success">
                        You have been logged out.
                    </div>
                    <div class="form-group">
                        <label for="username">Username</label>
                        <input type="text" id="username" name="username"/>
                    </div>
                    <div class="form-group">
                        <label for="password">Password</label>
                        <input type="password" id="password" name="password"/>
                    </div>
                    <button type="submit" class="btn btn-primary">Login</button>
 
                </fieldset>
            </form>
        </div>
    </body>
</html>
```

Das Template `login.html` ist das Template der  Login-Seite. Es wird sowohl beim Logout als auch bei einem fehlerhaften  Login-Versuch eine Nachricht ausgegeben. Dies kann mit `th:if="${param.error}"` und `th:if="${param.logout}"` abgefragt werden.

### Anpassung der Security Konfiguration

Die Methode `configure(HttpSecurity http)` der Elternklasse von `WebSecurityConfig` (siehe dazu auch den Beitrag [**Einstieg in Spring Security**](https://labs.micromata.de/einstieg-spring-security.html)) wird im Folgenden angepasst.

**Vorher**

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers("/noSecurity").permitAll()
        .anyRequest().authenticated()
        .and().formLogin().permitAll();
}
```

**Nachher**

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http.authorizeRequests().antMatchers("/noSecurity").permitAll()
    .anyRequest().authenticated()
    .and().formLogin().loginPage("/login").permitAll()
    .and().logout().permitAll();
}
```

Durch `loginPage("/login")` wird festgelegt, dass die Login-Seite unter `/login` aufrufbar ist. Ein nicht eingeloggter User wird auf diese Seite weitergeleitet. Mit `.and().logout().permitAll()` wird festgelegt, dass der Aufruf des Logouts von jeder Seite aus möglich ist.


[← Zurück zur Übersicht](/publikationen/tutorial-spring-security/)