# Spring Boot-Konfiguration für den SEW-Unterricht in MEDT an der HTL3R
Maven-Repository für die Grundkonfiguration von Spring Boot-Unterrichtsprojekten
mit den folgenden Artefakten:

## spring-boot
Globales Parent-POM für alle Projekte; enthält keinen Code, sondern definiert die Spring Boot-Version
und die verfügbaren Spring Boot Starter. Dies sind:
+ Web
+ JPA
+ REST Repositories
+ Jersey (JAX-RS)
+ Security
+ OAuth2-Client für Github, Google, Facebook und Okta
+ Test
+ Configuration Processor

Als weitere Abhängigkeiten sind enthalten:
+ Spring Session
+ OAuth2-Client für Microsoft Azure AD
+ Jirutka Collection Validators
+ H2-Datenbank
+ Hibernate
+ Ehcache

Plugins:
+ Spring Boot Maven Plugin
+ Keytool Maven Plugin

## autoconfig
Wird dieses Artefakt im POM eines Projekts als Abhängigkeit angegeben, so werden 
folgende Spring Boot-Einstellungen wirksam:

### HTTP-Server
+ Der Server läuft auf <code>127.0.0.1:8080</code>.
+ Auf <code>http://127.0.0.1:8080/index.html</code> wird ein Hinweistext ausgeliefert.

### Authentifizierung, Autorisierung

#### Anmeldung mit Benutzernamen und Passwort
Dafür muss
1. die Benutzer-<code>@Entity</code> das Interface <code>UserInfo</code> implementieren,
1. das zugehörige REST-Repository muss von <code>UserInfoRepository</code> abgeleitet
werden und
1. dort muss die Methode <code>toUser()</code> implementiert werden.

Dies bewirkt:
+ Die Anmeldung erfolgt durch <code>POST</code> von <code>username</code> und
<code>password</code> auf <code>/login</code>
(Property <code>sew.login</code> in <code>application.properties</code>).
Die JSON-Response enthält Details zur angemeldeten Benutzerin.
+ Bei jedem Anmeldeversuch wird im <code>UserDetailsService</code> der Benutzer zum Benutzernamen ermittelt.
Das gespeicherte Passwort muss als BCrypt-Hash vorliegen.
+ Die Benutzerdetails stehen später auch an <code>/api/me</code> zur Verfügung.
+ Zum Abmelden dient ein <code>POST</code> auf <code>/logout</code> (Property <code>sew.logout</code>).
+ Nach 600 Sekunden Inaktivität (Property <code>sew.session-timeout</code>) wird man automatisch abgemeldet.
+ In der Response wird das Zugriffstoken sowohl im <code>SESSION</code>-Cookie gesetzt als auch im
<code>x-auth-token</code>-Header geliefert. In Requests wird es aber nur im <code>x-auth-token</code>-Header
akzeptiert.
+ Alle Pfade sind ohne Authentifizierung zugänglich, d.h. nur <i>Methoden</i> 
können mit <code>@PreAuthorize</code> abgesichert werden.
+ Auditing: stehen Benutzer in einer <code>@OneToMany</code>-Beziehung 
zu einer anderen <code>@Entity</code>, so können sie dort mit <code>@CreatedBy</code> 
oder <code>@LastModifiedBy</code> annotiert werden. 
Dazu muss die andere Entity zusätzlich mit 
<code>@EntityListeners(AuditingEntityListener.class)</code> annotiert werden.

#### Anmeldung über OAuth2
Die OAuth2-Provider <code>github</code>, <code>google</code>, <code>facebook</code>
<code>okta</code> und <code>microsoft</code> (Azure Active Directory) werden unterstützt.

Die Konfiguration erfolgt über die in <code>src/main/resources/application-oauth2.properties</code>
angeführten Properties. Dies bewirkt folgende Unterschiede gegenüber der Anmeldung
mit Benutzernamen und Passwort:
+ Den Authentifizierungsvorgang startet man durch ein <code>GET</code> auf 
<code>/oauth2/authorization/{provider}</code>. Daraufhin wird der Browser zur Anmeldeseite
des OAuth2-Providers umgeleitet.
+ Nach erfolgreicher Authentifizierung wird der ursprüngliche <code>GET</code>-Request schließlich
auf den im Property <code>sew.oauth2-login-success</code> angegebenen Pfad umgeleitet.

### REST-API und CORS
Um CORS zu ermöglichen, gibt man die <code>Allowed-Origins</code> als Liste im Property 
<code>sew.allowed-origins</code> an. Andernfalls wird SOP erzwungen. 

### Profile
+ <code>logging</code> erzeugt ausführliche Debugging-Protokolle.
+ <code>tuning</code> optimiert Datenbankzugriffe; jede cachebare <code>@Entity</code> 
muss zusätzlich mit <code>@javax.persistence.Cacheable</code> annotiert werden.

