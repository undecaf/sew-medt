# Spring Boot-Konfiguration für den SEW-Unterricht in MEDT an der HTL3R
Maven-Repository für die Grundkonfiguration von Spring Boot-Unterrichtsprojekten
mit den folgenden Artefakten:

## spring-boot
Globales Parent-POM für alle Projekte; enthält keinen Code, sondern definiert die Spring Boot-Version
(2.5.3) und die verfügbaren Spring Boot Starter. Dies sind:
+ Web
+ JPA
+ REST Repositories
+ Jersey (JAX-RS)
+ Security
+ AOP
+ OAuth2-Client für Github, Google, Facebook und Okta
+ Test
+ Configuration Processor

Als weitere Abhängigkeiten sind enthalten:
+ [Spring Session](https://spring.io/projects/spring-session)
+ [AspectJ](https://www.eclipse.org/aspectj/)
+ [OAuth2-Client für Microsoft Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)
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
+ [WireMock](http://wiremock.org/)

Plugins:
+ Spring Boot Maven Plugin
+ Jib Maven Plugin (Goals `jib:dockerBuild` und `jib:build`)
+ Keytool Maven Plugin (Goal `keytool:generateKeyPair`)

## autoconfig
Wird dieses Artefakt im POM eines Projekts als Abhängigkeit angegeben, so werden 
folgende Spring Boot-Einstellungen wirksam:

### Dependency Injection in Beans, die nicht von Spring verwaltet werden
+ `@Autowired` kann man auch in Klassen verwenden, die keine Spring-Beans sind. 
  Dazu annotiert man diese Klassen mit `@Configurable`. Dies funktioniert für
  alle Klassen inner- oder unterhalb der Pakete `server`, `repositories`
  und `at.rennweg.htl.sew`.
+ Für dieses Verhalten wird [Load-time weaving (LTW)]((http://www.eclipse.org/aspectj/doc/released/devguide/ltw.html))
  verwendet, ohne dass ein externer Instrumentation-Agent benötigt wird.
  
### HTTP-Server
+ Der Server läuft auf `localhost:8080` (Properties `server.address`, `server.port`).
+ Auf `http://localhost:8080/index.html` werden Hinweise ausgeliefert, wie man eigene
  Inhalte auf dem Server platziert.

### Authentifizierung

#### Anmeldung mit Benutzernamen und Passwort
Voraussetzungen:
1. Man muss mit der Benutzer-`@Entity` das Interface `UserInfo<ID>` implementieren.
2. Das zugehörige REST-Repository muss man _zusätzlich_ auch von `UserInfoRepository<T, ID>` ableiten, z.B.
   ```java
   @RepositoryRestResource
   public interface MyUserRepository
       extends PagingAndSortingRepository<MyUser, Long>, UserInfoRepository<MyUser, Long> {
   }
   ```

Dies bewirkt:
+ Die Anmeldung erfolgt durch ein `POST` von `username` und `password` im
`application/x-www-form-urlencoded`-Format auf `/login`
(Property `sew.login` in `application.properties`).
Die `application/json`-Response enthält Details zur angemeldeten Benutzerin.
+ Bei jedem Anmeldeversuch ermittelt das `UserInfoRepository` den Benutzer 
zum Anmelde-Benutzernamen. Das Passwort muss dort als BCrypt-Hash gespeichert sein.
+ In der Login-Response wird das Zugriffstoken sowohl im `SESSION`-Cookie gesetzt als auch im
  `x-auth-token`-Header übergeben. In den weiteren Requests wird es zum Schutz 
  gegen XSS-Attacken aber nur im `x-auth-token`-Header akzeptiert.
+ Die Benutzerdetails stehen später auch an `/api/me` zur Verfügung.
+ Zum Abmelden dient ein `POST` auf `/logout` (Property `sew.logout`), der nur den
  `x-auth-token`-Header enthält.
+ Nach 600 Sekunden Inaktivität (Property `sew.session-timeout`) wird man automatisch abgemeldet,
  und das `x-auth-token` wird ungültig.

#### Anmeldung über OAuth2
Die OAuth2-Provider `github`, `google`, `facebook`,
`okta`, `microsoft` und `azure` (Azure Active Directory) werden unterstützt. Die Voraussetzungen sind
dieselben wie für die 
[Anmeldung mit Benutzernamen und Passwort](#anmeldung-mit-benutzernamen-und-passwort).

Die Konfiguration erfolgt über die in `src/main/resources/application-oauth2.properties.template`
beispielhaft angeführten Properties. Dies bewirkt folgende Verhaltensänderungen gegenüber der 
Anmeldung mit Benutzernamen und Passwort:
+ Den Authentifizierungsvorgang startet man durch ein `GET` auf 
  `/oauth2/authorization/{provider}`. Daraufhin wird der Browser zur Anmeldeseite
  des OAuth2-Authentifizierungsservers umgeleitet.
+ Nach der Authentifizierung wird temporär auf `/login/oauth2/code/{provider}` umgeleitet.
+ Abschließend wird der ursprüngliche `GET`-Request auf den im Property `sew.oauth2-login-success`
  angegebenen Pfad umgeleitet (redirect URI). Das mitgelieferte `SESSION`-Cookie muss man bei den
  weiteren Requests zur Authentifizierung als `x-auth-token`-Header mitsenden.
+ Nach der _ersten_ erfolgreichen Authentifizierung wird eine Benutzer-Entity im 
`UserInfoRepository` gespeichert. Diese Entity steht als `principal` in `@PreAuthorize`
zur Verfügung.

### Autorisierung
+ Alle _Pfade_ sind ohne Authentifizierung zugänglich.
+ Nur _Methoden_ können mit `@PreAuthorize` abgesichert werden.
+ Auditing: stehen Benutzer in einer `@OneToMany`-Beziehung 
zu einer anderen `@Entity`, so können sie dort mit `@CreatedBy` 
oder `@LastModifiedBy` annotiert werden. 
Dazu muss die andere Entity zusätzlich mit 
`@EntityListeners(AuditingEntityListener.class)` annotiert werden.

### Datenbankanbindung
In `src/main/resources/application-db.properties.template` sind die Properties aufgezählt, die
für die Anbindung diverser Datenbanken benötigt werden. 

### REST-API und CORS
Das REST-API ist unter dem Pfad `/api`erreichbar (Property `spring.data.rest.base-path`).

Um CORS zu ermöglichen, gibt man alle `Allowed-Origins` als Liste in 
`sew.allowed-origins` an. Andernfalls werden `XMLHttpRequest`s in Browsern wegen SOP
im Allgemeinen scheitern. 

### Zusätzliche Spring-Profile
+ `logging` erzeugt ausführliche Debugging-Protokolle.
+ `optimize` optimiert Datenbankzugriffe; jede cachebare `@Entity` 
muss zusätzlich mit `@javax.persistence.Cacheable` annotiert werden.

