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
+ Security inkl. OAuth2-Client
+ Test
+ Configuration Processor

Als weitere Abhängigkeiten sind enthalten:
+ Spring Session
+ OAuth2-Support für Microsoft Azure AD
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
Sofern keine andere Authentifizierungsmethode definiert ist, kann man sich nur als <code>user</code>
mit einem Passwort anmelden, das bei jedem Start zufällig bestimmt und an der Konsole protokolliert wird.

##### Allgemeines
+ Die Anmeldung erfolgt durch <code>POST</code> von <code>username</code> und
<code>password</code> auf <code>/login</code>
(Property <code>sew.login</code> in <code>application.properties</code>).
Die JSON-Response enthält Details zur angemeldeten Benutzerin.
+ Die Benutzerdetails stehen später auch an <code>/api/me</code> zur Verfügung.
+ Nach erfolgreicher Anmeldung wird ein <code>AuthenticationSuccessEvent</code> ausgelöst, das
mit einem entsprechenden <code>@EventListener</code> verarbeitet werden kann.
+ Zum Abmelden dient ein <code>POST</code> auf <code>/logout</code> (Property <code>sew.logout</code>).
+ Nach 600 Sekunden Inaktivität (Property <code>sew.session-timeout</code>) wird man automatisch abgemeldet.
+ In der Response wird das Zugriffstoken sowohl im <code>SESSION</code>-Cookie gesetzt als auch im
<code>x-auth-token</code>-Header geliefert. In Requests wird es aber nur im <code>x-auth-token</code>-Header
akzeptiert.
+ Alle Pfade sind ohne Authentifizierung zugänglich, d.h. nur <i>Methoden</i> 
können mit <code>@PreAuthorize</code> abgesichert werden.

##### Anmeldung mit beliebigen Benutzernamen und Passworten
Dafür muss <code>UserDetailsService</code> implementiert und als <code>@Service</code>-Bean 
annotiert werden. Dies bewirkt:
+ Spring Security konsultiert bei jedem Anmeldeversuch dieses <code>UserDetailsService</code>.
Das gespeicherte Passwort muss als BCrypt-Hash vorliegen.
+ Der Adapter <code>at.rennweg.htl.sew.autoconfig.UserInfo</code> erspart es, für das 
<code>UserDetailsService</code> auch das umfangreiche <code>UserDetails</code>-Interface
implementieren zu müssen. Er reduziert diese Arbeit auf das Implementieren von höchstens zwei Methoden.
+ Entity-Properties können mit <code>@CreatedBy</code> und <code>@LastModifiedBy</code> 
annotiert werden. Dazu muss die Entity zusätzlich mit 
<code>@EntityListeners(AuditingEntityListener.class)</code> annotiert werden.

##### Anmeldung über OAuth2
Die OAuth2-Provider <code>github</code>, <code>google</code>, <code>facebook</code>
<code>okta</code> und <code>microsoft</code> (Azure Active Directory) werden unterstützt.
Zumindest diese Properties (und für Microsoft einige weitere) müssen definiert werden:
+ <code>spring.security.oauth2.client.registration.{provider}.client-id</code>
+ <code>spring.security.oauth2.client.registration.{provider}.client-secret</code>

Dies hat zur Folge:
+ Den Authentifizierungsvorgang startet man durch ein <code>GET</code> auf 
<code>/oauth2/authorization/{provider}</code>.
+ Nach erfolgreicher Authentifizierung wird dieser <code>GET</code>-Request schließlich
auf den im Property <code>sew.oauth2-login-success</code> angegebenen Pfad umgeleitet.

### REST-API und CORS
Um CORS zu ermöglichen, gibt man die <code>Allowed-Origins</code> als Liste im Property 
<code>sew.allowed-origins</code> an. Andernfalls wird SOP erzwungen. 

### Profile
+ <code>logging</code> erzeugt ausführliche Debugging-Protokolle.
+ <code>tuning</code> optimiert Datenbankzugriffe; jede cachebare <code>@Entity</code> 
muss zusätzlich mit <code>@javax.persistence.Cacheable</code> annotiert werden.

