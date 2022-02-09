---
title: Logback und Elasticsearch
author:
categories: [Best Practices]
tags: [elastisearch]
shortdesc:
---

Einer der gebräuchlichen Ansätze, um Daten in einen ElastiSearch-Cluster zu importieren, ist Logstash. In verschiedenen Situationen, z. B. wenn keine explizite technische  Infrastruktur aufgebaut werden soll, um Nachrichten aus einer einzelnen  Quelle zu verarbeiten, ist diese Kombination nicht immer sinnvoll.

Als Alternative wird im Folgenden ein einfacher Appender für das u. a. von [Spring-boot](http://projects.spring.io/spring-boot/) standardmäßig verwendete Logging-Framework [logback](http://logback.qos.ch/) vorgestellt, der ohne den Umweg über einen Logstash-Import und  Elasticsearch-Export direkt auf die Elasticsearch-REST-Schnittstelle  zugreift und die Logdaten übermittelt.

### Benötigte Abhängigkeiten

Die Maven-Artifact-IDs der benötigten Abhängigkeiten sind:

– **spring-web** für die Klasse ***[RestTemplate](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)*** zum Verschicken von HTTP-POST-Nachrichten

– **logstash-logback-encoder** zur Umwandlung von ***[ILoggingEvent](http://logback.qos.ch/apidocs/ch/qos/logback/classic/spi/ILoggingEvent.html)*** aus Logback in von Elasticsearch nutzbares JSON

Statt **spring-web** kann natürlich auch eine andere Bibliothek zum Senden von HTTP-POST-Nachrichten, z. B. [Apache HttpClient](http://hc.apache.org/httpcomponents-client-ga/), verwendet werden.

### Konfiguration von Logback

Die Einbindung des Appenders erfolgt klassisch über einen entsprechenden Eintrag in die Datei logback.xml durch:

```xml
<appender name="ES_APPENDER" class="ElasticSearchAppender">
  <url>http://elasticnode:9200/logs/backend</url>
</appender>
  ...
<root level="DEBUG">
  <appender-ref ref="ES_APPENDER"/>
  <appender-ref ref="CONSOLE"/>
</root>
```

### Implementierung des Appenders

Der Appender wird von ***AppenderBase*** abgeleitet und ermöglicht damit das Behandeln von ***ILoggingEvent***s. Jeder zu loggende Event wird über den Logstash-Encoder in ein  elasticsearch-kompatibles JSON-Format überführt und anschließend an die im Appender konfigurierte URL per REST gePOSTed.

```xml
public class ElasticSearchAppender extends AppenderBase<ILoggingEvent> {
  private RestTemplate template = new RestTemplate();
  private String url;
  private ThreadLocal<Encoder<ILoggingEvent>> encoder;
  public ElasticSearchAppender() {
    encoder = new ThreadLocal<Encoder<ILoggingEvent>>() {
      @Override
      protected Encoder<ILoggingEvent> initialValue() {
        LogstashEncoder encoder = new LogstashEncoder();
        encoder.start();
        return encoder;
      }
    };
  }
  @Override
  protected void append(ILoggingEvent eventObject) {
    String postData = null;
    try {
      ByteArrayOutputStream os = new ByteArrayOutputStream();
      encoder.get().init(os);
      encoder.get().doEncode(eventObject);
      postData = os.toString();
    } catch (IOException e) {
      // Wird bewusst ignoriert.
      return;
    }
    // Sende Event an ES.
    postToElasticSearch(url, postData);
  }
  private void postToElasticSearch(String url, String postData) {
    try {
      template.postForLocation(new URI(url), postData);
    } catch (RestClientException | URISyntaxException e) {
    // Wird bewusst ignoriert.
    }
  }
  public String getUrl() {
    return url;
  }
  public void setUrl(String url) {
    this.url = url;
  }
}
```
