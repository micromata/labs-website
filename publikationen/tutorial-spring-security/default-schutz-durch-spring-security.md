---
layout: page
width: small
hero:
    title: Kapitel 4 - Default Schutz durch Spring Security
    subtitle: 
    image:
permalink: /publikationen/tutorial-spring-security/default-schutz-durch-spring-security/
author: jfast
scope: [Tutorial Spring Security]
---

Als Basis hierfür dient das Projekt aus dem Beitrag [JPA und Spring Security](/publikationen/tutorial-spring-security/jpa-und-spring-security/). Den Source Code zu diesem Tutorial findet ihr auf dem [Micromata Github Bereich](https://github.com/micromata/labs-spring-security/tree/master/chapter4/default-protection-in-spring-security).

## CSRF

Schauen wir uns beispielsweise das HTML unserer Login Form an:

```java
<form name="f" method="post" action="/login">
    <fieldset>
        <legend>Please Login</legend>
            <div class="form-group">
                <label for="username">Username</label>
                <input type="text" id="username" name="username" />
            </div>
            <div class="form-group">
                <label for="password">Password</label>
                <input type="password" id="password" name="password" />
            </div>
            <button type="submit" class="btn btn-primary">Log in</button>
    </fieldset>
    <input type="hidden" name="_csrf" value="b1f1cfe2-56b4-407b-8b53-5764701c2199" />
</form>
```

Hier sieht man bereits eines dieser per Default aktiven Sicherheitsmechanismen. Bei dem Hidden Field mit dem Namen `_csrf` handelt es sich um einen CSRF-Token zum Schutz vor Cross-Site-Request-Forgery-Angriffen ([CSRF](https://de.wikipedia.org/wiki/Cross-Site-Request-Forgery)).

## Security Headers

Neben dem CSRF-Token bietet Spring Security diverse Security Header, die in die Response eingefügt werden und in der `WebSecurityConfig` durch den Aufruf von `http.headers()`.`defaultsDisabled()` deaktiviert werden können. Nachfolgend ist diese Methode zu sehen:

```java
public HeadersConfigurer<H> defaultsDisabled() {
    contentTypeOptions.disable();
    xssProtection.disable();
    cacheControl.disable();
    hsts.disable();
    frameOptions.disable();
    return this;
}
```

Die einzelnen Felder haben dabei folgenden Inhalt:

| Feld               | Header                    | Wert                                           |
| ------------------ | ------------------------- | ---------------------------------------------- |
| contentTypeOptions | X-Content-Type-Options    | nosniff                                        |
| xssProtection      | X-XSS-Protection          | 1; mode=block                                  |
| cacheControl       | Cache-Control             | no-cache, no-store, max-age=0, must-revalidate |
| cacheControl       | Pragma                    | no-cache                                       |
| cacheControl       | Expires                   | 0                                              |
| hsts               | Strict-Transport-Security | max-age=31536000 ; includeSubDomains           |
| frameOptions       | X-Frame-Options           | DENY                                           |

Der HSTS-Eintrag ist nur dann im Header zu finden, wenn die  Applikation unter HTTPS aufgerufen wird. Wie dies mit einer Spring Boot  Applikation gemacht wird, wird im noch ausstehenden Beitrag HTTPS mit  Spring Boot erklärt.

## Aktivierung der H2 Console

Um die in Spring Boot integrierte H2 Console aufrufen zu können,  genügt es diese in der Konfigurationsdatei der Applikation einzutragen.  Die Konfigurationsdatei ist dabei unter `/resources/application.properties` oder `/resources/application.yml` zu finden. Welche Dateiendung gewählt wird, ist Geschmacksache und beeinflusst die spätere Notation der Konfiguration.

*application.properties*

```java
spring.datasource.url=jdbc:h2:mem:mydb
spring.h2.console.enabled=true
spring.h2.console.path=/console
```

*application.yml*

```java
spring:
    datasource.url: jdbc:h2:mem:mydb
    h2.console:
        enabled: true
        path: /console
```

Nach einem Aufruf von `/console` sollte nun die H2 Console erscheinen:![Spring Security H2-Console](https://labs.micromata.de/wp-content/uploads/2017/02/console.png)

## CSRF Ausnahmen

Bei einem Klick auf `Connect` erscheint jedoch die folgende Fehlerseite:![Spring Security csrfError](https://labs.micromata.de/wp-content/uploads/2017/02/csrfError.png)

Damit wir uns in die H2 Console einloggen können, müssen wir den CSRF Sicherheitsmechanismus für `/console` deaktivieren.

Dazu wird die `WebSecurityConfig` angepasst:

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers("/noSecurity").permitAll()
        .anyRequest().authenticated()
        .and().formLogin().loginPage("/login").permitAll()
        .and().logout().permitAll();
 
    http.csrf().ignoringAntMatchers("/console/**");
}
```

Durch `http.csrf().ignoringAntMatchers("/console/**")` erreichen wir genau dieses Verhalten. Alles unter `/console/*` wird nun nicht mehr auf ein CSRF-Token überprüft.

## Frame Options Anpassung

Klicken wir nun auf Connect in der H2 Console, dann sehen wir nichts. Die Console des Chrome zeigt jedoch folgendes:

![Spring Security Chrome ](https://labs.micromata.de/wp-content/uploads/2017/08/consoleChrome.png)
 Diesen Fehler erhalten wir, da die H2 Console in einem Frame läuft und das durch die Security Headers per Default verboten wird.
 Aus diesem Grund müssen die Headers angepasst und die `X-Frame-Options` von `Deny` auf `SAMEORIGIN` gesetzt werden.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers("/noSecurity").permitAll()
        .anyRequest().authenticated()
        .and().formLogin().loginPage("/login").permitAll()
        .and().logout().permitAll();

    http.csrf().ignoringAntMatchers("/console/**")
        .and().headers().frameOptions().sameOrigin();
}
```

Nun können wir uns erfolgreich in die H2 Console einloggen.
