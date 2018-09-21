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
+ Test
+ Configuration Processor

Als weitere Abhängigkeiten sind enthalten:
+ Spring Session
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
+ Der Server horcht auf <code>127.0.0.1:8080</code>.
+ Auf <code>http://127.0.0.1:8080/index.html</code> wird ein Hinweistext ausgeliefert.

### Sicherheit
+ Bei jedem Start wird ein Zufallspasswort für den Benutzer <code>user</code> erzeugt 
und protokolliert.
+ Die Anmeldung erfolgt durch <code>POST</code> auf <code>/login</code>
(Property <code>sew.login</code> in <code>application.properties</code>).
+ Zum Abmelden dient ein <code>POST</code> auf <code>/logout</code> (Property <code>sew.logout</code>).
Nach 600 Sekunden Inaktivität (Property <code>sew.session-timeout</code>) wird man automatisch abgemeldet.
+ Zugriffstoken werden in <code>x-auth-token</code>-Headern geliefert und erwartet.
+ Alle Pfade sind stets ohne Authentifizierung zugänglich, d.h. nur <i>Methoden</i> 
können abgesichert werden.
+ Es werden keine Cookies gesetzt.

Ist ein Spring Bean definiert, der das <code>UserDetailsService</code> implementiert, so bewirkt dies:
+ Es werden kein Benutzer <code>user</code> und kein Zufallspasswort erzeugt.
+ Stattdessen konsultiert Spring Security bei jedem Anmeldeversuch das <code>UserDetailsService</code>. Das
gespeicherte Passwort muss als bcrypt-Hash vorliegen.
+ Unter `/api/me` werden Benutzername und Rolle der angemeldeten Benutzerin ausgeliefert.
+ Der Adapter <code>at.rennweg.htl.sew.autoconfig.UserInfo</code> erspart es, für das 
<code>UserDetailsService</code> auch das umfangreiche <code>UserDetails</code>-Interface
implementieren zu müssen. Er reduziert diese Arbeit auf das Implementieren von zwei Methoden.
+ Entity-Properties können mit <code>@CreatedBy</code> und <code>@LastModifiedBy</code> 
annotiert werden. Dazu muss die Entity zusätzlich mit 
<code>@EntityListeners(AuditingEntityListener.class)</code> annotiert werden.

### Profile
+ <code>logging</code> erzeugt ausführliche Debugging-Protokolle.
+ <code>tuning</code> optimiert Datenbankzugriffe; cachebare Entities müssen zusätzlich mit 
<code>@javax.persistence.Cacheable</code> annotiert werden.

