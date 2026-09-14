# 0001: Grafana als erste Client-Anwendung

## Status

Akzeptiert, 2026-09-14

## Kontext

Das Lab besteht bisher nur aus Keycloak und PostgreSQL. Zum Lernen fehlt eine
angebundene Anwendung, an der sich ein OIDC-Client, Rollen im Token und der
Unterschied zwischen Authentifizierung und Autorisierung beobachten lassen.
Lernziel ist nicht Betrieb einer Anwendung, sondern das Verstehen von
Autorisierung und Token-Auswertung.

## Entscheidung

Grafana wird als erste angebundene Client-Anwendung verwendet.

Begründung:

- Anbindung erfolgt vollständig über Umgebungsvariablen, also ohne
  Anwendungscode und schnell reproduzierbar.
- Grafana leitet eigene Rollen aus Claims des Access Tokens ab, damit ist
  Autorisierung sichtbar und nicht nur Anmeldung.
- Ein angemeldeter Benutzer ohne passende Rolle wird abgewiesen, das macht
  die Trennung von Authentifizierung und Autorisierung praktisch erfahrbar.
- Grafana kann später die Metriken des Keycloak Management-Endpunkts
  visualisieren, womit Anmeldungen und Token-Ausstellungen pro Client
  auswertbar werden.

## Betrachtete Alternativen

- **Nextcloud.** Anbindung einfach, deckt aber nur Single Sign-On ab und
  liefert kein Rollenmodell und keine Token-Auswertung. Bleibt als spätere
  Option für Federation- und Consent-Szenarien ausdrücklich offen und wird
  nicht ausgeschlossen.
- **API Gateway, etwa Apache APISIX.** Fachlich näher am Zielbild, weil die
  Token-Prüfung am Edge stattfindet. Als nächster Schritt nach Grafana
  vorgesehen, aber als Einstieg zu komplex.
- **Apache Fineract.** Open-Source-Kernbankensystem mit dokumentierter
  Keycloak-Anbindung und hohem fachlichem Bezug. Betriebsaufwand durch eigene
  Datenbank und JVM-Anwendung deutlich höher, daher optionaler Ausbau.

## Konsequenzen

- Der Compose-Stack wird um einen Grafana-Dienst erweitert, sobald Realm und
  ein Client mit Rollen-Mapper existieren.
- Es wird ein confidential Client für Grafana im Realm benötigt, dessen
  Secret über die nicht versionierte .env eingebunden wird.
- Das Rollenmodell muss so gestaltet sein, dass Rollen im Access Token
  erscheinen, was einen Protocol Mapper erfordert.
- Die Entscheidung ist umkehrbar, weitere Anwendungen können parallel
  angebunden werden.
