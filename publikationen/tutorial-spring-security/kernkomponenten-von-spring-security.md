---
layout: page
width: small
hero:
    title: Kapitel 8 – Kernkomponenten von Spring Security
    subtitle: Dies ist das achte Kapitel der Tutorial-Reihe zu Spring Security. Dieser Beitrag soll einen Überblick über die Kernkomponenten von Spring Security schaffen.
    image:
permalink: /publikationen/tutorial-spring-security/kernkomponenten-von-spring-security/
author: jfast
scope: [Tutorial Spring Security]
---

Zu den zentralen Klassen gehören dabei:

- SecurityContextHolder
- SecurityContext
- Authentication
- GrantedAuthority
- UserDetails
- UserDetailsService

Die Codebeispiele stammen aus Spring Boot in der Version 1.5.2.RELEASE.

## SecurityContextHolder

```java
package org.springframework.security.core.context;

import org.springframework.util.ReflectionUtils;
import org.springframework.util.StringUtils;

import java.lang.reflect.Constructor;

public class SecurityContextHolder {

    public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";
    public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
    public static final String MODE_GLOBAL = "MODE_GLOBAL";
    public static final String SYSTEM_PROPERTY = "spring.security.strategy";
    private static String strategyName = System.getProperty(SYSTEM_PROPERTY);
    private static SecurityContextHolderStrategy strategy;
    private static int initializeCount = 0;

    static {
        initialize();
    }

    public static void clearContext() {
        strategy.clearContext();
    }

    public static SecurityContext getContext() {
        return strategy.getContext();
    }

    public static int getInitializeCount() {
        return initializeCount;
    }

    private static void initialize() {
        if (!StringUtils.hasText(strategyName)) {
            // Set default
            strategyName = MODE_THREADLOCAL;
        }

        if (strategyName.equals(MODE_THREADLOCAL)) {
            strategy = new ThreadLocalSecurityContextHolderStrategy();
        }
        else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
            strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
        }
        else if (strategyName.equals(MODE_GLOBAL)) {
            strategy = new GlobalSecurityContextHolderStrategy();
        }
        else {
            // Try to load a custom strategy
            try {
                Class<?> clazz = Class.forName(strategyName);
                Constructor<?> customStrategy = clazz.getConstructor();
                strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
            }
            catch (Exception ex) {
                ReflectionUtils.handleReflectionException(ex);
            }
        }

        initializeCount++;
    }

    public static void setContext(SecurityContext context) {
        strategy.setContext(context);
    }

    public static void setStrategyName(String strategyName) {
        SecurityContextHolder.strategyName = strategyName;
        initialize();
    }

    public static SecurityContextHolderStrategy getContextHolderStrategy() {
        return strategy;
    }

    public static SecurityContext createEmptyContext() {
        return strategy.createEmptyContext();
    }

    public String toString() {
        return "SecurityContextHolder[strategy='" + strategyName + "'; initializeCount="
                + initializeCount + "]";
    }
}
```

`SecurityContextHolder` ist die Basisklasse von Spring Security, welche den aktuellen `SecurityContext` der Applikation und somit auch Informationen zum aktuellen Nutzer enthält. Standardmäßig wird der `SecurityContext` im Threadlocal (mehr dazu hier) gehalten. Desweiteren bietet Spring noch INHERITABLETHREADLOCAL und GLOBAL als `SecurityContextHolderStrategy` an. Mit GLOABAL haben alle Threads einer JVM und mit  INHERITABLETHREADLOCAL alle Threads, die durch den Security Thread  gestartet wurden, den gleichen `SecurityContext`.

## SecurityContext

```java
package org.springframework.security.core.context;

import org.springframework.security.core.Authentication;
import java.io.Serializable;

public interface SecurityContext extends Serializable {

    Authentication getAuthentication();

    void setAuthentication(Authentication authentication);
}
```

Das Interface `SecurityContext` hält das `Authentication` Objekt. Die Standardimplementation ist `SecurityContextImpl`.

## Authentication

```java
package org.springframework.security.core;

import java.io.Serializable;
import java.security.Principal;
import java.util.Collection;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.core.context.SecurityContextHolder;


public interface Authentication extends Principal, Serializable {

    Collection<? extends GrantedAuthority> getAuthorities();

    Object getCredentials();

    Object getDetails();

    Object getPrincipal();

    boolean isAuthenticated();

    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

Das Interface `Authentication` ermöglicht unter anderem mit `getPrincipal()` den Zugriff auf den aktuellen Nutzer. Die Klasse des Nutzer Objektes implementiert dabei für gewöhnlich das Interface `UserDetails`. Nach der `WebSecurityConfig` aus dem ersten Kapitel Einstieg in Spring Security ist dies die Klasse `org.springframework.security.core.userdetails.User`. Ab dem dritten Kapitel JPA und Spring Security ist es die Klasse `de.micromata.spring.security.example.security.user.AuthenticatedUser`. Die Standardimplementation des Interfaces `Authentication` ist die abstrakte Klasse `AbstractAuthenticationToken`, welche wiederum von `UsernamePasswordAuthenticationToken` geerbt wird. Über `getAuthorities()` können die Rollen des Nutzers abgefragt werden.

Ein Nutzer gilt als authentifiziert, wenn der `SecurityContextHolder` ein vollständiges `Authentication` Objekt enthält. Dabei spielt es keine Rolle wie dieses Objekt gesetzt wurde, solange dies geschieht bevor der `AbstractSecurityInterceptor` greift. So ist es auch möglich die Authentifizierung außerhalb von  Spring Security durchzuführen. Hier ist jedoch dann auch darauf zu  achten, dass das Erzeugen und Setzen einer HTTP Session selbst  implementiert werden muss, was normalerweise durch den  Authentifizerungsprozess von Spring Security geschieht.

## GrantedAuthority

```java
package org.springframework.security.core;

import java.io.Serializable;
import org.springframework.security.access.AccessDecisionManager;

public interface GrantedAuthority extends Serializable {

    String getAuthority();
}
```

Dieses Interface enthält lediglich einen „Getter“ für die Rolle, welche im Spring Kontext für gewöhnlich die Form `ROLE_USER` hat. Die Standardimplementation ist `SimpleGrantedAuthority`.

## UserDetails

```java
package org.springframework.security.core.userdetails;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import java.io.Serializable;
import java.util.Collection;

public interface UserDetails extends Serializable {

    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}
```

Dieses Interface repräsentiert das User Objekt. Die bisher  verwendeten Implementationen wurden oben unter Authentication bereits  erwähnt. Es enthält die notwendigen Daten, um ein `Authentication` Objekt zu erzeugen.

## UserDetailsService

```java
package org.springframework.security.core.userdetails;

public interface UserDetailsService {

    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

Dieses Interface wird benutzt, um anhand des `username` ein `UserDetails` Objekt zu erzeugen. Im ersten Kapitel Einstieg in Spring Security wurde dazu die Implementation `org.springframework.security.provisioning.InMemoryUserDetailsManager` und ab dem dritten Kapitel JPA und Spring Security `de.micromata.spring.security.example.security.user.AuthenticatedUserService ` verwendet.
