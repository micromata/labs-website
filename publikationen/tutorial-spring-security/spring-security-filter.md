---
layout: page
width: small
hero:
    title: Kapitel 9 – Spring Security Filter
    subtitle: Dies ist das neunte Kapitel der Tutorial-Beitragsreihe zu Spring Security. Dieser Beitrag beschreibt einige für den Authentifizierungsprozess von Spring Security innerhalb der Webapplikation verantwortlichen Filter.
    image:
permalink: /publikationen/tutorial-spring-security/spring-security-filter/
author: jfast
scope: [Tutorial Spring Security]
---

## Die Filter
Im Folgenden sind die Filter der in diesem Tutorial programmierten Webapplikation aufgelistet:

### Reihenfolge Name Filterklasse

Von Interesse ist dabei die springSecurityFilterChain. Dies ist eine  eigene Filterchain innerhalb eines Filters und fasst die Filter von  Spring Security innerhalb der Klasse FilterChainProxy.VirtualFilterChain zusammen:

| #    | Name                             | Filterklasse                                                 |
| ---- | -------------------------------- | ------------------------------------------------------------ |
| 1    | characterEncodingFilter          | org.springframework.boot.web.filter.OrderedCharacterEncodingFilter |
| 2    | hiddenHttpMethodFilter           | org.springframework.boot.web.filter.OrderedHiddenHttpMethodFilter |
| 3    | httpPutFormContentFilter         | org.springframework.boot.web.filter.OrderedHttpPutFormContentFilter |
| 4    | requestContextFilter             | org.springframework.boot.web.filter.OrderedRequestContextFilter |
| 5    | springSecurityFilterChain        | org.springframework.boot.web.servlet.DelegatingFilterProxyRegistrationBean |
| 6    | Tomcat WebSocket (JSR356) Filter | org.apache.tomcat.websocket.server.WsFilter                  |

Von Interesse ist dabei die `springSecurityFilterChain`. Dies ist eine eigene Filterchain innerhalb eines Filters und fasst die Filter von Spring Security innerhalb der Klasse `FilterChainProxy.VirtualFilterChain` zusammen:

```java
/**
 * Internal {@code FilterChain} implementation that is used to pass a request through
 * the additional internal list of filters which match the request.
 */
private static class VirtualFilterChain implements FilterChain {
    private final FilterChain originalChain;
    private final List<Filter> additionalFilters;
    ...
}
```

`originalChain` enthält die originale Filterchain aus der obigen Tabelle und `additionalFilters` die aufgelisteten Filter von Spring Security:

- WebAsyncManagerIntegrationFilter
- SecurityContextPersistenceFilter
- HeaderWriterFilter
- CsrfFilter
- LogoutFilter
- UsernamePasswordAuthenticationFilter
- RequestCacheAwareFilter
- SecurityContextHolderAwareRequestFilter
- AnonymousAuthenticationFilter
- SessionManagementFilter
- ExceptionTranslationFilter
- FilterSecurityInterceptor

 Für die folgenden Erläuterungen betrachten wir die Filter **UsernamePasswordAuthenticationFilter**, **AnonymousAuthenticationFilter**, **ExceptionTranslationFilter** und **FilterSecurityInterceptor**.

## Aufruf einer geschützten Ressource ohne Login

Da wir nicht eingeloggt sind, sorgt der Filter  AnonymousAuthenticationFilter dafür, dass ein  AnonymousAuthenticationToken als Authentication Objekt gesetzt wird.

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
        throws IOException, ServletException {
 
    if (SecurityContextHolder.getContext().getAuthentication() == null) {
        SecurityContextHolder.getContext().setAuthentication(
                createAuthentication((HttpServletRequest) req));
 
        if (logger.isDebugEnabled()) {
            logger.debug("Populated SecurityContextHolder with anonymous token: '"
                    + SecurityContextHolder.getContext().getAuthentication() + "'");
        }
    }
    else {
        if (logger.isDebugEnabled()) {
            logger.debug("SecurityContextHolder not populated with anonymous token, as it already contained: '"
                    + SecurityContextHolder.getContext().getAuthentication() + "'");
        }
    }
 
    chain.doFilter(req, res);
}
 
protected Authentication createAuthentication(HttpServletRequest request) {
    AnonymousAuthenticationToken auth = new AnonymousAuthenticationToken(key,
            principal, authorities);
    auth.setDetails(authenticationDetailsSource.buildDetails(request));
 
    return auth;
}
```

Anschließend wird im `ExceptionTranslationFilter` versucht den Filter `FilterSecurityInterceptor` auszuführen, welcher der letzte Filter von `additionalFilters` ist.

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
        throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest) req;
    HttpServletResponse response = (HttpServletResponse) res;
 
    try {
        chain.doFilter(request, response);
        ...
    }
    ...
    catch (Exception ex) {  
        ...
        handleSpringSecurityException(request, response, chain, ase);
        ...
    }
    ...
}
```

Dies führt in unserem Fall zu einer `AccessDeniedException`, die von der Methode `handleSpringSecurityException` behandelt wird. Da wir einen anonymen Benutzer besitzen, wird `sendStartAuthentication` aufgerufen, was in dieser Applikation zu einem Redirect auf die Login Seite führt.

```java
private void handleSpringSecurityException(HttpServletRequest request,
        HttpServletResponse response, FilterChain chain, RuntimeException exception)
        throws IOException, ServletException {
    ...
    else if (exception instanceof AccessDeniedException) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authenticationTrustResolver.isAnonymous(authentication) || authenticationTrustResolver.isRememberMe(authentication)) {
            logger.debug(
                    "Access is denied (user is " + (authenticationTrustResolver.isAnonymous(authentication) ? "anonymous" : "not fully authenticated") + "); redirecting to authentication entry point",
                    exception);
 
            sendStartAuthentication(
                    request,
                    response,
                    chain,
                    new InsufficientAuthenticationException(
                            "Full authentication is required to access this resource"));
        }
        else {
            logger.debug(
                    "Access is denied (user is not anonymous); delegating to AccessDeniedHandler",
                    exception);
 
            accessDeniedHandler.handle(request, response,
                    (AccessDeniedException) exception);
        }
    }
}
```

Die `AccessDeniedException` wird dabei in der Klasse `AffirmativeBased` nach der Überprüfung der Zugriffsberechtigung geworfen. Die nachfolgenden Codeausschnitte zeigen dabei die Aufrufreihenfolge.

**FilterSecurityInterceptor:**

```java
public class FilterSecurityInterceptor extends AbstractSecurityInterceptor implements
        Filter {
    ...
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException {
        FilterInvocation fi = new FilterInvocation(request, response, chain);
        invoke(fi);
    }
    ... 
    public void invoke(FilterInvocation fi) throws IOException, ServletException {
        ...
        InterceptorStatusToken token = super.beforeInvocation(fi);
        ...
    }
    ...
}
```

AbstractSecurityInterceptor:

```java
public abstract class AbstractSecurityInterceptor implements InitializingBean,
        ApplicationEventPublisherAware, MessageSourceAware {
    ...
    protected InterceptorStatusToken beforeInvocation(Object object) {
        ...
        // Attempt authorization
        try {
            this.accessDecisionManager.decide(authenticated, object, attributes);
        }
        catch (AccessDeniedException accessDeniedException) {
            publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated,
                    accessDeniedException));
 
            throw accessDeniedException;
        }
        ...
    }
    ...
}
```

AffirmativeBased:

```java
public class AffirmativeBased extends AbstractAccessDecisionManager {
    ...
    public void decide(Authentication authentication, Object object,
            Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
        int deny = 0;
 
        for (AccessDecisionVoter voter : getDecisionVoters()) {
            int result = voter.vote(authentication, object, configAttributes);
            ...
            switch (result) {
            case AccessDecisionVoter.ACCESS_GRANTED:
                return;
 
            case AccessDecisionVoter.ACCESS_DENIED:
                deny++;
 
                break;
 
            default:
                break;
            }
        }
 
        if (deny > 0) {
            throw new AccessDeniedException(messages.getMessage(
                    "AbstractAccessDecisionManager.accessDenied", "Access is denied"));
        }
 
        // To get this far, every AccessDecisionVoter abstained
        checkAllowIfAllAbstainDecisions();
    }
    ...
}
```

<h2

Der erste interessante Filter für den Login ist der `UsernamePasswordAuthenticationFilter` bzw. die Methode `doFilter` der Elternklasse `AbstractAuthenticationProcessingFilter`.

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
        throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest) req;
    HttpServletResponse response = (HttpServletResponse) res;
    if (!requiresAuthentication(request, response)) {
        chain.doFilter(request, response);
        return;
    }
    ...
    Authentication authResult;
    ...
        authResult = attemptAuthentication(request, response);
    ...
    successfulAuthentication(request, response, chain, authResult);
}
```

Solange es sich nicht um einen Login Aufruf handelt, wird der nächste Filter aufgerufen. Ansonsten wird die Methode `attemptAuthentication` aufgerufen, welche von `UsernamePasswordAuthenticationFilter` implementiert wird. Dadurch wird die Authentifizierung vollzogen.  Weitere Details zu diesem Login-Prozess werden im kommenden Kapitel  erläutert. Anschließend wird die Methode `successfulAuthentication` aufgerufen

```java
protected void successfulAuthentication(HttpServletRequest request,
        HttpServletResponse response, FilterChain chain, Authentication authResult)
        throws IOException, ServletException {
    ...
    SecurityContextHolder.getContext().setAuthentication(authResult);
    ...
    successHandler.onAuthenticationSuccess(request, response, authResult);
}
```

Hier wird das Authentication Objekt gesetzt, welches in unserem Fall das `UsernamePasswordAuthenticationToken` ist. Anschließend wird in der Methode `onAuthenticationSuccess` ein Redirect zu der vor dem Login aufgerufenen Seite oder, falls die  Login Seite direkt aufgerufen wurde, zur Index Seite ausgeführt.

## Aufruf einer geschützten Ressource nach erfolgreichem Login

Wird eine geschütze Ressource mit eingeloggtem und berechtigtem Nutzer aufgerufen, dann wird in der Klasse `AffirmativeBased` keine `AccessDeniedException` geworfen und der Filter `FilterSecurityInterceptor` kann die Methode doFilter abschließen.

```java
public class FilterSecurityInterceptor extends AbstractSecurityInterceptor implements
        Filter {
    ...
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException {
        FilterInvocation fi = new FilterInvocation(request, response, chain);
        invoke(fi);
    }
    ... 
    public void invoke(FilterInvocation fi) throws IOException, ServletException {
        ...
        InterceptorStatusToken token = super.beforeInvocation(fi);
        try {
            fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
        } finally {
            super.finallyInvocation(token);
        }
        super.afterInvocation(token, null);
    }
    ...
}
```

n unserem Fall wird im try-Block der Filter `WsFilter`, der letzte Filter der `originalChain`, aufgerufen. Es folgt der Aufruf von `finallyInvocation`:

```java
protected void finallyInvocation(InterceptorStatusToken token) {
    if (token != null && token.isContextHolderRefreshRequired()) {
        if (logger.isDebugEnabled()) {
            logger.debug("Reverting to original Authentication: "
                    + token.getSecurityContext().getAuthentication());
        }
     
        SecurityContextHolder.setContext(token.getSecurityContext());
    }
}
```

Dies führt zu keiner Änderung, da die enthaltene if-Abfrage nicht zutrifft, weil `runAs` in der Methode `beforeInvocation` null ist und somit `contextHolderRefreshRequired` auf `false` gesetzt wird:

```java
public abstract class AbstractSecurityInterceptor implements InitializingBean,
        ApplicationEventPublisherAware, MessageSourceAware {
    ...
    protected InterceptorStatusToken beforeInvocation(Object object) {
        ...
        // Attempt to run as a different user
        Authentication runAs = this.runAsManager.buildRunAs(authenticated, object,
                attributes);
        if (runAs == null) {
            ...
            return new InterceptorStatusToken(SecurityContextHolder.getContext(), false,
                    attributes, object);
        }
        else {
            ...
            return new InterceptorStatusToken(origCtx, true, attributes, object);
        }
    }
    ...
}
```

Abschließend wird `afterInvocation` aufgerufen:

```java
protected Object afterInvocation(InterceptorStatusToken token, Object returnedObject) {
    if (token == null) {
        // public object
        return returnedObject;
    }
     
    finallyInvocation(token); // continue to clean in this method for passivity
 
    if (afterInvocationManager != null) {
        // Attempt after invocation handling
        try {
            returnedObject = afterInvocationManager.decide(token.getSecurityContext()
                    .getAuthentication(), token.getSecureObject(), token
                    .getAttributes(), returnedObject);
        }
        catch (AccessDeniedException accessDeniedException) {
            AuthorizationFailureEvent event = new AuthorizationFailureEvent(
                    token.getSecureObject(), token.getAttributes(), token
                            .getSecurityContext().getAuthentication(),
                    accessDeniedException);
            publishEvent(event);
 
            throw accessDeniedException;
        }
    }
    return returnedObject;
}
```

Dies hat jedoch ebenfalls keine Auswirkungen, da lediglich ein weiteres Mal `finallyInvocation` aufgerufen wird und `afterInvocationManager` null ist.
