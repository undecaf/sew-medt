# Spring Boot-Konfiguration für den SEW-Unterricht in MEDT an der HTL3R
Maven-Repository für die Grundkonfiguration von Spring Boot-Unterrichtsprojekten
mit den folgenden Artefakten:

## spring-boot
Globales Parent-POM für alle Projekte; enthält keinen Code, sondern definiert die Spring Boot-Version
(2.1.13) und die verfügbaren Spring Boot Starter. Dies sind:
+ Web
+ JPA
+ REST Repositories
+ Jersey (JAX-RS)
+ Security
+ AOP
+ OAuth2-Client für Github, Google, Facebook und Okta
+ Test
+ Configuration Processor
+ Developer Tools

Als weitere Abhängigkeiten sind enthalten:
+ [Spring Session](https://spring.io/projects/spring-session)
+ [AspectJ](https://www.eclipse.org/aspectj/)
  für [„Load-time weaving“](http://www.eclipse.org/aspectj/doc/released/devguide/ltw.html)
+ [OAuth2-Client für Microsoft Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)
+ [Jirutka Collection Validators](https://github.com/jirutka/validator-collection)
+ [Project Lombok](https://projectlombok.org/)
+ JDBC-Treiber für [H2](https://www.h2database.com/), 
  [PostgreSQL](https://www.postgresql.org/),
  [MS SQL Server](https://www.microsoft.com/en-us/sql-server/default.aspx), 
  [MariaDB](https://mariadb.org/) und
  [MySQL](https://www.mysql.com/) 
+ Embedded [H2-Datenbank](https://www.h2database.com/)
+ [Hibernate ORM](https://hibernate.org/)
+ [Ehcache](https://www.ehcache.org/)
+ [JUnit5](https://junit.org/junit5/)
+ [Mockito](https://site.mockito.org/)

Plugins:
+ Spring Boot Maven Plugin
+ Keytool Maven Plugin

## autoconfig
Wird dieses Artefakt im POM eines Projekts als Abhängigkeit angegeben, so werden 
folgende Spring Boot-Einstellungen wirksam:

### Load-time weaving (LTW) mit AspectJ
+ LTW wird auf Klassen inner- oder unterhalb des Pakets `server`
  angewendet, ohne dass ein externer Instrumentation-Agent benötigt wird.
+ Somit kann man auch Klassen mit `@Configurable` annotieren, wenn sie keine Spring Beans
  sondern z.B. Entities sind. Innerhalb dieser Klassen können somit `@Autowired`, `@PreAuthorize` etc. verwendet werden.
  
### HTTP-Server
+ Der Server läuft auf `localhost:8080`.
+ Auf `http://localhost:8080/index.html` wird ein Hinweistext ausgeliefert.

### Authentifizierung

#### Anmeldung mit Benutzernamen und Passwort
Voraussetzungen:
1. Man muss mit der Benutzer-`@Entity` das Interface `UserInfo` implementieren.
1. Das zugehörige REST-Repository muss man von `UserInfoRepository<T>` ableiten statt von
z.B. `PagingAndSortingRepository<T,ID>`.

Dies bewirkt:
+ Die Anmeldung erfolgt durch `POST` von `username` und
`password` auf `/login`
(Property `sew.login` in `application.properties`).
Die JSON-Response enthält Details zur angemeldeten Benutzerin.
+ Bei jedem Anmeldeversuch ermittelt das `UserInfoRepository` den Benutzer 
zum Anmelde-Benutzernamen. Das Passwort muss als BCrypt-Hash gespeichert sein.
+ Die Benutzerdetails stehen später auch an `/api/me` zur Verfügung.
+ Zum Abmelden dient ein `POST` auf `/logout` (Property `sew.logout`).
+ Nach 600 Sekunden Inaktivität (Property `sew.session-timeout`) wird man automatisch abgemeldet.
+ In der Response wird das Zugriffstoken sowohl im `SESSION`-Cookie gesetzt als auch im
`x-auth-token`-Header geliefert. In Requests wird es aber nur im `x-auth-token`-Header
akzeptiert.

#### Anmeldung über OAuth2
Die OAuth2-Provider `github`, `google`, `facebook`,
`okta`, `microsoft` und `azure` (Azure Active Directory) werden unterstützt. Die Voraussetzungen sind
dieselben wie für die 
[Anmeldung mit Benutzernamen und Passwort](#anmeldung-mit-benutzernamen-und-passwort).

Die Konfiguration erfolgt über die in `src/main/resources/application-oauth2.properties`
beispielhaft angeführten Properties. Dies bewirkt folgende Unterschiede gegenüber der 
Anmeldung mit Benutzernamen und Passwort:
+ Den Authentifizierungsvorgang startet man durch ein `GET` auf 
`/oauth2/authorization/{provider}`. Daraufhin wird der Browser zur Anmeldeseite
des OAuth2-Providers umgeleitet.
+ Nach der Authentifizierung wird temporär auf `/login/oauth2/code/{provider}` umgeleitet.
+ Abschließend wird der ursprüngliche `GET`-Request auf den im Property `sew.oauth2-login-success` angegebenen Pfad umgeleitet (redirect URI).
+ Nach der _ersten_ erfolgreichen Authentifizierung wird eine Benutzer-Entity im 
`UserInfoRepository` gespeichert. Diese Entity steht als `principal` in `@PreAuthorize`
zur Verfügung.

### Autorisierung
+ Alle _Pfade_ sind ohne Authentifizierung zugänglich, d.h. nur _Methoden_ 
können mit `@PreAuthorize` abgesichert werden.
+ Um `@PreAuthorize` in einer Klasse (z.B. Entity) zu verwenden, die kein Spring-Bean ist,
muss diese mit `@Configurable` annotiert werden, und sie muss sich inner- oder unterhalb 
des Pakets `server` befinden.
+ Auditing: stehen Benutzer in einer `@OneToMany`-Beziehung 
zu einer anderen `@Entity`, so können sie dort mit `@CreatedBy` 
oder `@LastModifiedBy` annotiert werden. 
Dazu muss die andere Entity zusätzlich mit 
`@EntityListeners(AuditingEntityListener.class)` annotiert werden.

### Datenbankanbindung
In `src/main/resources/application-db.properties` sind die Properties aufgezählt, die
für die Anbindung diverser Datenbanken benötigt werden. 

### REST-API und CORS
Das REST-API ist unter dem Pfad `/api`erreichbar (Property `spring.data.rest.base-path`).

Um CORS zu ermöglichen, gibt man alle `Allowed-Origins` als Liste in 
`sew.allowed-origins` an. Andernfalls wird die SOP erzwungen. 

### Profile
+ `logging` erzeugt ausführliche Debugging-Protokolle.
+ `tuning` optimiert Datenbankzugriffe; jede cachebare `@Entity` 
muss zusätzlich mit `@javax.persistence.Cacheable` annotiert werden.

