---
title: "Verbesserung der ISO 20022-Abwicklung im internationalen Handel durch dezentrale Escrow- und Identitätsschichten"
subtitle: "Colabonate-Stack, ICP-Kanister, Nostr, Lightning und XID"
author: "Deniz Yilmaz, Colabonate"
status: "Arbeitsgrundlage für Fachjury-Vorschlag"
date: 2026-08-06
target_body: "[Name des Fachgremiums]"
target_repo: "[Zielrepository: bei Integration eintragen]"
language: de
---

<div class="title-page">

# Verbesserung der ISO 20022-Abwicklung im internationalen Handel durch dezentrale Escrow- und Identitätsschichten

### Colabonate-Stack, ICP-Kanister, Nostr, Lightning und XID

**Autor:** Deniz Yilmaz, Colabonate

**Status:** Arbeitsgrundlage für Fachjury-Vorschlag

**Datum:** 06. August 2026

**Adressat:** [Name des Fachgremiums]

**Zielrepository:** [Zielrepository: bei Integration eintragen]

> *Offener Punkt:* Die Bezeichnung des adressierten Fachgremiums sowie das Zielrepository sind dem Autor nicht bekannt und daher als Platzhalter markiert. Diese beiden Felder können erst bei der Integration in das Ziel-Repository final benannt werden.

</div>

---

## Inhaltsverzeichnis

0. Executive Summary
1. Ausgangslage und Zielsetzung
2. ISO 20022: Status quo und strukturelle Lücken
3. Der Colabonate-Stack im Überblick
4. Escrow-Strategie über ICP-Kanister
5. Identitätsschicht: Nostr, XID und Vertrauensmodell
6. Lightning als Zahlungsschicht
7. Vergleichende Gegenüberstellung für die Fachjury
8. Proof-of-Concept-Skizze und ISO-20022-Adapter-Blueprint
9. Fazit und Empfehlung an das Komitee
- Quellenverzeichnis

---

## 0. Executive Summary

> Diese Zusammenfassung liegt in zwei Fassungen vor: Abschnitt 0.1 als allgemeinverständliche Einführung für ein breites Gremium ohne tieferen Payments-/Technik-Hintergrund, Abschnitt 0.2 als vertiefende Fassung für das technische Fachpublikum.

### 0.1 Allgemeinverständliche Zusammenfassung (für ein breites Gremium)

Der internationale Zahlungsverkehr stellt derzeit verbindlich auf den neuen Nachrichtenstandard ISO 20022 um. Dieser Standard regelt jedoch nur, wie Zahlungsinformationen in einer Nachricht beschrieben und verschickt werden — er beantwortet drei andere, für die Sicherheit eines internationalen Handelsgeschäfts mindestens ebenso wichtige Fragen nicht: Wie stellt man fälschungssicher fest, wer der andere Vertragspartner wirklich ist? Was passiert, wenn beide Seiten sich über die Erfüllung eines Geschäfts nicht einig werden? Und: Wer hält das Geld, bis beide Seiten ihren Teil erfüllt haben — und wie ist ausgeschlossen, dass diese Stelle das Geld selbst einbehält? Diese drei Fragen (Identität, Schlichtung, Treuhand) bleiben nach der ISO-20022-Umstellung unbeantwortet, weil ein reiner Nachrichtenstandard sie gar nicht regeln kann.

Die zentrale These dieses Papiers: Der Colabonate-Stack löst ISO 20022 nicht ab, sondern **ergänzt** die Norm genau um diese drei fehlenden Antworten. Ein Treuhand-Mechanismus per Programmcode — ein sogenannter ICP-Kanister, technisch abgesichert über natives Bitcoin — hält das Geld, ohne dass irgendeine Partei — auch nicht Colabonate selbst — jemals darüber verfügen kann; eine schlüsselbasierte Identitätsschicht — Nostr, ein dezentrales Protokoll für digitale Identität, ergänzt um den vorgeschlagenen Baustein XID — macht Handelspartner eindeutig und überprüfbar; ein mehrstufiges, von Menschen geführtes Schlichtungsverfahren klärt Streitfälle. Der technische Kerngedanke beruht auf zwei Prinzipien. Erstens: Jede Freigabe des hinterlegten Geldes ist an eine digitale Unterschrift gebunden, die ausschließlich für genau dieses eine Geschäft gültig ist. Zweitens: Im Streitfall entscheiden Menschen, wer im Recht ist — das System setzt diese Entscheidung anschließend nur noch technisch um, ohne dass die Schlichtungsstelle selbst je Zugriff auf das Geld erhält. Damit löst der Stack ein klassisches Entweder-Oder auf: Entweder vertraut man einer dritten Partei das Geld an — mit dem Risiko, dass sie es einbehält —, oder man verzichtet auf jede Schlichtungsmöglichkeit. Hier gilt beides zugleich: unwiderruflich sicheres Settlement *und* ein bindendes Schlichtungsverfahren. Alle drei Ergänzungen setzen dabei über ISO 20022 an — die Norm selbst bleibt unangetastet und wird durch die zusätzliche Absicherung sogar gestärkt.

Dieses Papier unterscheidet durchgängig ehrlich zwischen dem, was bereits nachgewiesen ist, und dem, was noch Entwurf oder Vorschlag ist. **Bereits nachgewiesen:** In einer kontrollierten Testumgebung wurde ein vollständiger Handel mit stufenweiser Geldfreigabe (in drei Schritten: 25 %, 50 %, 25 %) bereits erfolgreich durchgespielt [Canister-README] — der Treuhand-Mechanismus funktioniert nachweislich. **Als Entwurf beschlossen, technisch noch nicht vollständig umgesetzt:** einzelne Bausteine der Schlichtungslogik [ADR-253]. **Reiner Vorschlag, im heutigen System nicht vorhanden:** der XID-Baustein für die Identitätsschicht — eine gezielte Suche im gesamten Programmcode ergab dazu keinen einzigen echten Treffer [Canister-Code-Audit] — sowie die biometrisch gestützte Human-Identity-Komponente, die bislang nicht einmal als erste Testversion existiert. Diese drei Kategorien werden nie vermischt: Ein Vorschlag wird an keiner Stelle als fertige Komponente dargestellt. Belegt ist dagegen das wirtschaftliche Versprechen: Die zentralen Vorgänge — Kauf, Verkauf, Treuhandnutzung — bleiben im Colabonate-Geschäftsmodell dauerhaft gebührenfrei, im Unterschied zu den 15–30 % Provision, die klassische Online-Marktplätze verlangen [WP7 §4.1].

### 0.2 Vertiefende Zusammenfassung (für das Fachpublikum)

Der internationale Zahlungsverkehr befindet sich in einer regulatorisch verbindlichen Umstellung auf den Nachrichtenstandard ISO 20022. Diese Arbeit argumentiert, dass ISO 20022 eine notwendige, aber nicht hinreichende Standardisierung darstellt: Die Norm regelt ausschließlich die *Nachrichtenschicht* (also wie Zahlungsdaten beschrieben und übertragen werden) und klammert bewusst drei Bereiche aus, die für die wirtschaftliche Abwicklungssicherheit im internationalen Handel entscheidend sind: die *kryptografische Verifizierbarkeit der Gegenparteien*, die *Streit- und Schlichtungslogik* und das *technische Settlement* mitsamt Treuhandfunktion.

Die zentrale These dieses Papiers lautet: Der Colabonate-Stack löst ISO 20022 nicht ab, sondern **ergänzt** die Norm um genau diese drei Schichten. Ein ICP-basierter Escrow-Kanister übernimmt auf nativem Bitcoin die nicht-custodiale Treuhand- und Freigabelogik (Settlement-Lücke); eine Nostr-basierte Identitätsschicht plus ein vorgeschlagener XID-Anschluss (Blockchain Commons) trägt verifizierbare Partei-Identifikatoren (Identitäts-Lücke); ein mehrstufiges, menschlich geführtes Dispute-Resolution-Protocol schließt die Schlichtungs-Lücke. Technischer Kern und Neuheit ist dabei die *Integration* aus **hash-basierter Zahlungsabwicklung** (jede Zustandsänderung autorisiert durch eine BIP-340-Signatur über einen domänen-separierten Hash) und **nicht-custodialem Schlichtungsmechanismus** (Deliberation off-chain durch ein humanes Verfahren, Exekution on-chain rein via Signaturverifikation). Diese Integration löst das klassische Dilemma zwischen Custody-Risiko und Streitrekurs: eine grenzüberschreitende Transaktion erhält irreversible Settlement-Garantien *und* bindenden Streitrekurs, ohne dass jemals eine Partei Custody ausübt. Alle drei Ergänzungen sind als *Schichten über* ISO 20022 konzipiert. Die Norm bleibt unangetastet und wird operativ sogar gestärkt.

Das Papier ist in seinen Behauptungen redlich im Implementierungsstatus: Wo der Stack bereits gegen `bitcoind -regtest` end-to-end verifiziert ist (3-Milestone-Escrow-Trade 25/50/25), wird dies als erbrachter Nachweis dargestellt; wo Bausteine noch als `accepted`-Designentscheidung oder als ausdrücklicher Vorschlag vorliegen, insbesondere der XID-Anschluss (in der Referenzimplementierung nicht vorhanden) und Human Identity (Konzept, PoC noch offen), wird dies klar als Entwurf/Vorschlag gekennzeichnet und nicht als fertige Komponente ausgegeben. Die Wirtschaftlichkeitsbehauptung (dauerhaft gebührenfreie Kern-Transaktionen gegenüber 15–30 % Provision bei Web2-Marktplätzen) ist belegter Bestandteil des Colabonate-Geschäftsmodells.

---

## 1. Ausgangslage und Zielsetzung

### 1.1 Kontext: die ISO 20022-Migration

Der internationale Zahlungsverkehr vollzieht gegenwärtig den Wechsel vom klassischen MT-Nachrichtenformat (SWIFT) auf den reichhaltigeren, XML-basierten ISO 20022-Standard. Diese Migration ist nicht optional, sondern regulatorisch verankert und größtenteils bereits vollzogen: Das US-amerikanische Fedwire Funds Service hat das ISO 20022-Format am **14. Juli 2025** erfolgreich in Betrieb genommen [FRB], nachdem der ursprünglich für den 10. März 2025 geplante Cutover verschoben worden war; das Clearing-House-Interbank-Payments-System (CHIPS) war bereits im **April 2024** auf ISO 20022 migriert [TCH]. Für den SWIFT-Cross-Border-Verkehr endet die Branchenmehrheit nach übereinstimmenden Branchenangaben die Koexistenz von MT- und MX-Nachrichten im **November 2026** mit dem Rückzug der Kategorie-1/2-MT-Nachrichten (u. a. MT103, MT202) *[SWIFT-Daten branchenbekannt, in dieser Recherche nicht gegen eine erreichbare Primärquelle verifiziert; siehe Abschnitt 2.2]*. ISO 20022 liefert damit strukturierte Nachrichtenformate mit erheblich reichhaltigeren Datenfeldern und verbesserten Compliance-Prüfmöglichkeiten als die Vorgängernorm.

### 1.2 Zielsetzung des Papiers

Ziel dieser Arbeit ist aufzuzeigen, wie *strukturelle* Lücken der reinen Nachrichtenstandardisierung durch einen komplementären, dezentralen Stack geschlossen werden können. Die Argumentation richtet sich auf den wirtschaftlichen Mehrwert für den internationalen Handel, nicht primär auf technische Interoperabilität. Technische Interoperabilität ist ein Mittel, nicht der Endzweck; Endzweck ist die Reduktion von Abwicklungsrisiko und -kosten bei gleichzeitiger Erhöhung der Vertrauenswürdigkeit zwischen Handelsparteien, die einander nicht kennen.

### 1.3 Zielgruppe

Adressiert ist ein Fachgremium mit Bezug zu ISO 20022 *[Platzhalter für die konkrete Bezeichnung des Komitees]*. Die Ausführungen setzen Vertrautheit mit Zahlungs-Nachrichtenformaten voraus, erläutern aber die dezentralen Komponenten (ICP-Kanister, Nostr, Lightning, XID) soweit, dass ihre *komplementäre* Rolle zur Norm nachvollziehbar wird.

### 1.4 Nutzenversprechen

Der wirtschaftliche Mehrwert für den internationalen Handel ergibt sich aus drei Hebeln: erstens der Reduktion von Gegenparteirisiko durch eine nicht-custodiale, code-basierte Treuhand statt eines institutionellen Treuhänders; zweitens der Kostensenkung durch ein Geschäftsmodell, dessen Kern-Transaktionen einschließlich Escrow-Nutzung dauerhaft gebührenfrei sind (im Gegensatz zu 15–30 % Provision bei klassischen Web2-Marktplätzen [WP7 §4.1]); drittens der Nachvollziehbarkeit und Eindeutigkeit der Handelsparteien über eine kryptografische Identitätsschicht.

### 1.5 Zentrale Positionierung: die Leitplanke des gesamten Papiers

Der Colabonate-Stack wird durchgängig als **komplementäre Schicht zu ISO 20022** dargestellt: nicht als Ersatz und nicht als Ablösung. Diese Positionierung ist nicht rhetorisch, sondern sachlich geboten: ISO 20022 ist regulatorisch verankert und wird ab November 2026 branchenweit verbindlich; ein Ablösungs-Framing wäre für ein Komitee nicht konsensfähig und sachlich falsch, da die Norm eine andere Ebene standardisiert als der Stack adressiert (die drei konkreten Lücken leitet Abschnitt 2.4 systematisch her). Durchgängige Sprachregelung ist daher »ergänzt«, »schließt Lücke« und »Ergänzungsschicht« statt »ersetzt« oder »löst ab«. Ein Hinweis zur Redlichkeit: ISO 20022 wird in der Colabonate-Referenzimplementierung selbst nicht erwähnt [Canister-Code-Audit]; die Komplementaritäts-These ist folglich eine *Leistung dieses Papiers*, keine im Quellcode dokumentierte Aussage.

---

## 2. ISO 20022: Status quo und strukturelle Lücken

### 2.1 Was ISO 20022 leistet

ISO 20022 ist ein Nachrichtenstandard für den finanziellen Nachrichtenaustausch. Sein Beitrag ist substanziell und soll hier nicht kleingeredet werden: Die Norm liefert strukturierte, XML-basierte Nachrichtenformate mit erheblich reichhaltigeren Datenfeldern als die Vorgänger-MT-Nachrichten und ermöglicht dadurch eine feingranularere, maschinell prüfbare Beschreibung von Zahlungen, Parteien, Konten und Statusinformationen. Die verbesserte Datenqualität unterstützt Compliance-Prüfungen (Geldwäscheprävention, Sanktionsprüfung, Regulierungsreporting) deutlich besser als die knappen, freitextlastigen MT-Felder. Wer den vollen Umfang der Norm an einem Referenzmodell nachvollziehen will, findet ihn in der Java-Referenzimplementierung *prowide-iso20022* [prowide], die die ISO-20022-Dictionary-Typen über mehr als 60 Nachrichtenfamilien und rund 20 Business-Areas hinaus modelliert. Das belegt, dass die Norm breit angelegt ist und bewusst weit mehr standardisiert, als einzelne Marktteilnehmer in der Praxis ausreizen.

### 2.2 Zeitplan und regulatorischer Druck bis November 2026

Die Migration auf ISO 20022 ist nicht eine Empfehlung, sondern ein verbindlicher, bereits weitgehend vollzogener Prozess. Gegen Primärquellen belegbar sind:

- **CHIPS** (The Clearing House, USA) ist im **April 2024** auf ISO 20022 migriert [TCH]. Der exakte Stichtag (häufig als 19. April 2024 angegeben) ließ sich in dieser Recherche gegen die abgerufene TCH-Primärquelle auf Monatsebene bestätigen, auf Tagesebene nicht.
- **Fedwire Funds Service** (Federal Reserve, USA) hat das ISO 20022-Format am **14. Juli 2025** in Betrieb genommen [FRB], nachdem der ursprünglich für den 10. März 2025 geplante Cutover verschoben worden war. Fedwire-Zahlungen sind nach Quellenangaben unmittelbar, final und unwiderruflich (RTGS) [FRB].
- **SWIFT-Cross-Border-Verkehr (CBPR+):** Nach übereinstimmenden Branchenangaben ging CBPR+ am 20. März 2023 live; das Ende der MT/MX-Koexistenz mit Rückzug der Kategorie-1/2-MT-Nachrichten (u. a. MT103, MT202) wird für **November 2026** erwartet. *Diese beiden SWIFT-Termine sind branchenbekannt, konnten jedoch in dieser Recherche nicht gegen eine erreichbare Primärquelle (swift.com und iso20022.org waren über automatisierten Abruf nicht erreichbar) verifiziert werden und sind daher als nicht primärquellenbelegt gekennzeichnet.* [SWIFT-Vorbehalt]

Der regulatorische Druck ist damit real: Wer im internationalen Zahlungsverkehr teilnehmen will, muss spätestens mit Ablauf der Koexistenzfrist ISO 20022 sprechen. Genau dieser Druck macht die Frage drängend, *was* die Norm leistet und *was* sie bewusst offenlässt.

### 2.3 Was ISO 20022 *nicht* leistet

So substanziell der Beitrag der Nachrichtenschicht ist, so klar sind ihre Grenzen. ISO 20022 beschreibt, *wie* eine Zahlung und ihre Parteien dargestellt werden, sie definiert aber *nicht*:

- ein **technisches Settlement-Escrow**: die Norm spezifiziert Zahlungsnachrichten, aber keinen Treuhandschlitz, der eine Gegenleistung bis zur bestätigten Lieferung zurückhält und automatisch freigibt. Das Settlement-Geschäft bleibt externen, institutionellen Treuhändern oder bilateralen Vertrauensannahmen überlassen.
- eine **dezentrale Identitätsschicht**: die Norm sieht in ihren Nachrichten Partei- und Kontoidentifikatoren vor (Debtor/Creditor-Blöcke mit BIC, IBAN, LEI, Organisation-IDs), definiert aber keine kryptografische Methode, um die *Identität hinter* diesen Feldern selbst zu verifizieren oder deren Historie nachvollziehbar zu machen. Die Identifikatoren sind beschreibend, nicht verifizierend.
- eine **automatisierte Schlichtung**: die Norm kennt Statuscodes (u. a. `RJCT`, `PDNG`, `ACSC` [prowide: `TransactionIndividualStatus3Code`]), die den *Nachrichtenzustand* einer Transaktion abbilden, aber keine Logik, einen inhaltlichen Streit zwischen den Parteien über die Lieferung oder Leistung verbindlich zu entscheiden.

### 2.4 Ableitung der drei Kernlücken

Aus den Grenzen in Abschnitt 2.3 leiten sich die drei strukturellen Lücken ab, die den Ansatzpunkt für den Colabonate-Stack bilden:

- **Lücke (a): Identitätsverifikation der Gegenpartei:** die Norm beschreibt Identifikatoren, verifiziert aber nicht, wer dahinter steht.
- **Lücke (b): native Schlichtungs-/Streitfalllogik:** die Norm meldet Status, entscheidet aber keine inhaltlichen Streitigkeiten.
- **Lücke (c): technisches Settlement/Escrow:** die Norm überträgt den Zahlungsauftrag, verwahrt aber keine Gegenleistung treuhänderisch bis zur Erfüllung.

Diese drei Lücken strukturieren die Abschnitte 4 bis 6 dieses Papiers: Abschnitt 4 (Escrow) schließt Lücke (c), Abschnitt 5 (Identitätsschicht mit Nostr und XID) schließt Lücke (a), und das in Abschnitt 4.4.1 beschriebene Dispute-Resolution-Protocol schließt Lücke (b).

### 2.5 Ergänzender Hinweis zu Lücke (a): die kryptografische Tiefe

Innerhalb der Identitätslücke (a) liegt ein spezieller, für den internationalen Handel hochrelevanter Teilaspekt: die *fehlende kryptografische Verifizierbarkeit und Provenienz* der Partei-Identifikatoren. Die Norm kann einen LEI oder eine Organisation-ID transportieren, aber sie kann nicht belegen, dass diese Zuordnung zu einer realen, schlüsselkontrollierenden Entität gehört, noch die Historie dieser Zuordnung kryptografisch nachvollziehbar machen. Genau hier, an der kryptografischen Tiefe der Identitätslücke, setzt der vorgeschlagene XID-Anschluss (Abschnitt 5.5) mit der größten Hebelwirkung an. Es ist wichtig, diese Teil-Lücke von den *anderen* Identitätsaspekten (Eindeutigkeit/Sybil-Resistenz, Vertrauensmetrik) zu trennen, die Colabonate konzeptionell durch Human Identity (HID) und das Reputation Scoring Framework adressiert (Abschnitt 5.1.1). XID ergänzt diese, überschneidet sich aber nicht mit ihnen.

---

## 3. Der Colabonate-Stack im Überblick

### 3.1 Grundkonzept

Colabonate ist ein offenes Protokoll für *Decentralized Collaborative Commerce (DCC)*, gesteuert durch den Colabonate Codex und die Colabonate DAO [WP7 §1.1, §2.1]. Die Plattform läuft als dezentrale Commerce- und Kollaborationsplattform auf Bitcoin L2; ihre Kernprinzipien sind Dezentralisierung, Transparenz, Sicherheit (Bitcoin L2, Smart Contracts, Human Identities), Nachhaltigkeit und Autonomie/Selbstbestimmung [WP7 §2.1]. Für die wirtschaftliche Nutzenargumentation des Komitees (vgl. Abschnitt 1.4 und 7.2) ist ein konkreter, zitierfähiger Befund zentral: Die Kern-Transaktionen Colabonates, *einschließlich* der Escrow-Nutzung, sind laut Geschäftsmodell **dauerhaft gebührenfrei**: Es gibt keine prozentuale Provision und keine Einstell- oder Auszahlungsgebühr [WP7 §4.1]. Eine Schlichtungsgebühr (1 % für Mediation, 2 % für das Schiedsverfahren) fällt ausschließlich im tatsächlichen Streitfall an. Verglichen mit 15–30 % Provision bei klassischen Web2-Marktplätzen ist dies ein konkreter, belegter Wirtschaftlichkeits-Kontrast, der in Abschnitt 7.2 aufgegriffen wird.

### 3.1.1 Hinweis zur versionsübergreifenden Redlichkeit

Das Colabonate Whitepaper in der Fassung v7 (01.08.2026) beschreibt in §3.3 noch eine Drei-Layer-Architektur mit RSK als Contract-Layer für Escrow/Governance (geplant für »Phase 4«) und in §5.1 Lightning-Hold-Invoice-Escrow als bereits umgesetzten Mechanismus. Diese Beschreibung ist durch die drei Tage später verfassten Architekturentscheidungen ADR-253 und ADR-254 (02.–04.08.2026) überholt: RSK als Contract-Layer wurde als v2-Ansatz verworfen, und der custodiale Lightning-Hold-Invoice-Pfad bleibt hinter einem Feature-Flag und geht nach Founder-Entscheidung nie in Produktion.

Aus dieser zeitlichen Staffelung ergibt sich für dieses Papier eine bewusste methodische Quellen-Hierarchie: Für die *technische Beschreibung des Escrow-Mechanismus* sind **ADR-253/254 und der referenzierte Code die allein maßgebliche Quelle der Wahrheit**, weil sie den aktuell gebauten und verifizierten Stand abbilden. Das Whitepaper v7 hingegen behält seine Gültigkeit für *Kontext, Vision, Werte und Geschäftsmodell*, also für alles, was keine Aussage über den konkreten Treuhand-Mechanismus trifft. Diese Trennung wird im Papier konsequent angewandt: Jede technische Behauptung zu Escrow, Abwicklung und Schlichtung ist gegen ADR-Status und Code belegt, jede wirtschaftliche oder visionäre Aussage gegen das Whitepaper. Die offensichtlich benannte Diskrepanz ist somit kein Inkonsistenz-Vorwurf, sondern der nachvollziehbare Beleg einer aktiven, innerhalb weniger Tage dokumentierten Weiterentwicklung: Das Whitepaper ist schlicht noch nicht auf den ADR-253/254-Stand nachgezogen.

### 3.1.2 Referenzimplementierung und Kernentscheidung

Die kernarchitektonische Entscheidung ist in ADR-253 festgehalten (Status: *accepted*, 02.–03.08.2026) und lautet pointiert: *»Zwei Pfade, null Custody, eine neue Technologie«* [ADR-253]:

1. **Direktzahlungs-Pfad** (Lightning oder Cashu, nicht-custodial ab Tag 1) für Micro-Trades ohne Escrow-Bedarf.
2. **Escrow-Pfad** über einen ICP-Kanister auf nativem Bitcoin (für höherwertige Trades: Kooperationen, Milestones, mehrtägige Lieferungen).

Custodiale Vorläufer-Architekturen, Plattform-LND-Hold-Invoices (ADR-124/184/193) und das RSK-Swap-Sandwich (v2), wurden zugunsten des ICP-Ansatzes explizit verworfen. Der entscheidende technische Grund: ICP-Kanister bieten native Bitcoin-Threshold-Kryptografie (t-ECDSA) und kommen dadurch ohne ckBTC, ohne ICP-Wallet-Zwang für die Nutzer und ohne Plattform-Custody aus [ADR-253].

### 3.2 Architekturbausteine im Überblick

Der Stack setzt sich aus drei Architekturbausteinen zusammen:

- **(a) ICP-Escrow-Kanister** (`packages/icp-escrow-canister`): ein Rust/Candid-Kanister für Settlement und Schlichtung; das Detail-Design ist in ADR-254 dokumentiert. Dieser Baustein ist der Gegenstand von Abschnitt 4.
- **(b) Nostr als Identitäts- und Transportschicht**: App- und serverseitiger Code (`apps/colabonate-app/src/lib/nostr*.ts`, `apps/server/services/nostr-*.ts`) sowie ein eigenes Schema-Package (`packages/nostr-schema`). Dieser Baustein ist die reale Basis der Identitätsschicht (Abschnitt 5.1) und des Schlichtungs-Transports.
- **(c) Lightning/Cashu als Zahlungsschicht**: eine Payment-Provider-Abstraktion (`packages/payments` mit LNBits, LND-Hold-Invoices, Lightspark, Mock) plus ein Cashu-Wallet (`packages/cashu-wallet`) für nicht-custodiales eCash (NIP-60/61). Dieser Baustein bedient den Direktzahlungspfad (Abschnitt 6).

### 3.3 Positionierung gegenüber klassischer Treuhand

Anders als bei klassischem Escrow im internationalen Handel gibt es bei Colabonate keinen menschlichen oder institutionellen Treuhänder. Die Verfügungsgewalt liegt bei einem reproduzierbar gebauten, *blackholed* oder DAO-kontrollierten Kanister-Code [ADR-253, Abschnitt »Verifizierbarkeit«]. Auszahlungen sind On-Chain nachvollziehbar statt intern gebucht. Diese Eigenschaft (treuhänderische Funktion ohne Treuhänder) ist der Kern des Beitrags zur Settlement-Lücke (c) und wird in Abschnitt 4 technisch ausgefaltet.

### 3.4 Explizite Abgrenzung: Ergänzungsschicht *über* ISO 20022

Der Stack konkurriert nicht mit ISO 20022, sondern tritt als Ergänzungsschicht *über* der Norm auf, in der in Abschnitt 1.5 begründeten Positionierung und Sprachregelung. Die folgenden drei Abschnitte zeigen, wie diese Ergänzungsschicht die in Abschnitt 2.4 abgeleiteten Lücken (a) bis (c) im Einzelnen schließt.

---

## 4. Escrow-Strategie über ICP-Kanister

### 4.1 Grundidee und erbrachter Nachweis

Die Escrow-Lösung ist lokal entwickelt und bildet Settlement und Schlichtung *aktiv* über ICP-Kanister und die Bitcoin-Blockchain ab, anstatt eine zentrale Treuhandstelle einzuschalten. Quelle ist das Package `packages/icp-escrow-canister` (Rust/Candid), dessen Detail-Design in ADR-254 dokumentiert ist (Status: *accepted*, 04.08.2026) [ADR-254].

Der erbrachte technische Nachweis ist konkret und reproduzierbar: Der Kanister wurde gegen `bitcoind -regtest` end-to-end verifiziert, einschließlich eines **3-Milestone-Testtrades im Split 25/50/25** mit unabhängig geprüften On-Chain-Salden [Canister-README]. Die Milestone-Anteile werden in Basispunkten ausgedrückt (der Vektor `milestone_bps` muss in Summe 10.000 ergeben; die Beispielkonfiguration lautet `vec{2500, 5000, 2500}` [Canister: `state_machine.rs`]). Bei dem dokumentierten Testlauf wurde der Trade mit 150.000 Sats finanziert; nach Freigabe aller drei Milestones ergaben sich kumulierte Verkäufer-Salden von 37.500 / 112.500 / 140.042 Sats, eine Rest-Escrow-Change von 109.116 / 30.732 / 0 Sats und insgesamt 9.958 Sats Miner-Gebühren; die vom Kanister gemeldeten `funded_utxos` stimmten zu jedem Schritt mit der unabhängig über `bitcoin-cli scantxoutset` geprüften Chain überein [Canister-README]. Das ist ein echter, replizierbarer Beweis der Settlement-Funktionalität, kein Konzeptpapier.

### 4.2 Ablauf eines Escrow-Vorgangs

Der Escrow-Ablauf beruht auf zwei durchgängigen Prinzipien, die ihn wissenschaftlich präzise als *nicht-custodial* und *hash-basiert* ausweisen. **Erstens** wird jeder zustandsändernde Schritt durch eine kryptografische Signatur *über einen deterministischen, domänen-separierten Hash* autorisiert, niemals durch eine Plattform-Identität (kein `msg.caller`-Auth). Die Autorisierung ist also ein Beweis über die Kenntnis eines privaten Schlüssels zu einem berechneten Hash, on-chain vom Kanister verifizierbar. **Zweitens** vollzieht sich die eigentliche Wertbewegung (das Settlement) über Threshold-ECDSA: der Kanister leitet einen trade-individuellen Bitcoin-Schlüssel ab und signiert Auszahlungstransaktionen *ausschließlich* nach erfolgreichem Output-Check. Aus diesen beiden Prinzipien folgt, dass zu keinem Zeitpunkt eine Partei, auch Colabonate nicht, Verfügungsgewalt über die hinterlegten Mittel ausübt. Konkret gliedert sich der Ablauf in folgende autorisierte Zustandsübergänge:

**4.2.1 Trade-Erstellung.** Der Aufruf `create_trade(buyer_pubkey, seller_pubkey, buyer_refund_address, seller_payout_address, expected_amount_sats, milestones, timeout_at)` wird vom Colabonate-Server als ICP-Principal-authentifizierter Call ausgelöst; die Trade-Terms stammen aus einer vorherigen Nostr-Verhandlung (Offer/Accept, Event-Kinds 16/17). Der Kanister leitet daraufhin per Threshold-ECDSA eine eigene, trade-individuelle native P2WPKH-Bitcoin-Adresse ab (`derivation_path = [trade_id]`) und geht in den State `PendingFunding`.

**4.2.2 Bedingungsprüfung / Einzahlung.** Der Buyer zahlt On-Chain BTC an die Trade-Adresse. Der Aufruf `check_funding(trade_id)` fragt die native `bitcoin_get_utxos`-Schnittstelle des IC ab (*kein* externes Oracle) und setzt bei ausreichender Summe und Bestätigungstiefe den State auf `Funded`.

**4.2.3 Freigabe.** `release_milestone(trade_id, milestone_index, buyer_signature)` ist ausschließlich Buyer-autorisiert (der Buyer bestätigt eine Teil-Lieferung bzw. Teil-Leistung). Die Milestone-Anteile in Basispunkten müssen in Summe 10.000 ergeben; der letzte Milestone setzt den State auf `Released`.

**4.2.4 Schlichtung.** `open_dispute(trade_id, opener_signature)` ist Buyer- oder Seller-autorisiert, aus `Funded` erreichbar und sperrt die regulären Release-/Refund-Calls. `submit_dao_verdict(trade_id, buyer_bps, seller_bps, dao_signature)` prüft die Signatur gegen einen zur Kompilierzeit fixierten DAO-Public-Key und triggert die finale Split-Auszahlung.

**4.2.5 Timeout-Pfad.** `claim_timeout_refund(trade_id)` ist *permissionless*, von jedem Aufrufer (z. B. einem Keeper-Skript) ausführbar, und refundet den vollen UTXO-Rest an die Buyer-Adresse, solange der State noch `Funded` ist und `now() > timeout_at` gilt.

**4.2.6 Autorisierung ohne ICP-Wallet-Zwang.** Buyer und Seller besitzen ausschließlich Nostr-Schlüssel (secp256k1/BIP-340), keine ICP-Principals. Zustandsändernde Calls, die eine Partei autorisieren (`open_dispute`, `release_milestone`), werden vom Kanister selbst gegen eine BIP-340-Schnorr-Signatur über einen domain-separierten Message-Hash (`"colabonate-escrow-v1:<action>:<trade_id>[...]"`) verifiziert; es gibt *kein* `msg.caller`-Auth [ADR-254 §2; Canister: `src/sig.rs`].

### 4.3 Die neuartige Integration: hash-basierte Abwicklung und nicht-custodiale Invariante

Klassisches Escrow steht vor einem strukturellen Dilemma: Custodiale Treuhand *ermöglicht* Streitbeilegung, erzeugt aber Custody-Risiko (Gegenparteirisiko des Treuhänders); reine On-Chain-Escrow *vermeidet* Custody, bietet aber keinen Streitrekurs. Der Colabonate-Ansatz löst dieses Dilemma, indem er *Verwahrung* und *Urteilsfindung* sauber trennt und beides kryptografisch absichert. Die Integration ruht auf zwei Säulen.

**Säule 1: Hash-basierte Zahlungsabwicklung.** Jede zustandsändernde Operation (`release_milestone`, `open_dispute`, die DAO-Verdict-Annahme) wird über eine BIP-340-Schnorr-Signatur autorisiert, die *über einen domänen-separierten Message-Hash* (`"colabonate-escrow-v1:<action>:<trade_id>:<payload>"`) gebildet wird [ADR-254 §2; Canister: `src/sig.rs`]. Der Domain-Separator bindet jede Signatur an *eine* Aktion auf *einem* Trade und schließt damit Replay-Angriffe und die Wiederverwendung einer Signatur für eine andere Aktion aus. Die Autorisierung ist folglich *hash-basiert* und nicht *identitätsbasiert*: statt einer Plattform-Identität (`msg.caller`) verlangt der Kanister den Nachweis, dass der Aufrufer den zu einem öffentlichen Schlüssel gehörenden privaten Schlüssel kennt. Buyer und Seller authentifizieren sich ausschließlich über ihre Nostr-Schlüssel (secp256k1/BIP-340), ohne ICP-Wallet oder Plattform-Account. Das eigentliche Settlement vollzieht sich dann über Threshold-ECDSA: der Kanister leitet pro Trade einen eigenen Bitcoin-Schlüssel ab und signiert die Auszahlungstransaktion erst, nachdem die Output-Menge programmatisch geprüft wurde.

**Säule 2: Nicht-custodiale Invariante.** Die formal gefasste Sicherheitseigenschaft lautet: *kein zur Laufzeit veränderlicher dritter Empfänger*. Jede vom Kanister signierte Auszahlung darf ausschließlich Outputs an drei bei Trade-Erstellung fixierte Adressen enthalten: Buyer-Refund, Seller-Payout und Fee (die Fee ist eine zur Kompilierzeit konstante Größe, im Pilot auf 0 gesetzt). Diese dritte Adresse ist zur Laufzeit *nicht* veränderbar, und es existiert *keine* Admin-Funktion, die dies aufhebt [ADR-254 §5; Canister: `validate_payout_outputs` in `src/state_machine.rs`, vor jedem Signieren geprüft]. Präzise heißt das: die Invariante ist *»kein zur Laufzeit veränderlicher dritter Empfänger«*, nicht *»überhaupt keine dritte Adresse«*. In Kombination mit Säule 1 bedeutet dies, dass die Mittel zu jedem Zeitpunkt *mathematisch gebunden* sind: verwahrt von einem Code, der sie nicht umleiten kann, autorisiert von Signaturen, die nur die berechtigte Partei erzeugen kann.

**Warum die Integration neuartig ist.** Weil Verwahrung (nicht-custodialer Kanister) und Streitbeilegung (vierstufiges, humanes Verfahren, Abschnitt 4.4.1) entkoppelt sind, entsteht eine grenzüberschreitende Transaktion mit *beidem* zugleich: irreversiblen Settlement-Garantien *und* bindendem Streitrekurs, und das, ohne dass irgendeine Partei jemals Custody ausübt. Vertrauen wird nur noch für die *Qualität* der Streitbeilegung verlangt, niemals für die Verwahrung der Mittel. Gegenüber klassischem Escrow ergeben sich daraus die drei Vorteile Automatisierung, drastisch reduziertes Gegenparteirisiko und volle On-Chain-Transparenz.

**Verifizierbarkeit und dezentrale Kontrolle.** Die Invariante ist nicht nur behauptet, sondern nachprüfbar: reproduzierbarer Build (`Dockerfile` + `verify-reproducible-build.sh`), ein On-Chain abrufbarer Modul-Hash sowie ein *blackholed* oder DAO-Multisig-gesteuerter Controller garantieren, dass keine Partei, auch Colabonate nicht, den Code einseitig nachträglich ändern kann [ADR-253]. Diese dezentrale Kontrolle ist der philosophische Kern: Verfügungsgewalt liegt nicht bei einer Institution, sondern bei reproduzierbar gebautem, DAO- oder Community-kontrolliertem Code.

### 4.3.1 Umgang mit Fiat-Crypto-Volatilität und Währungskonvertierung

Im internationalen Warenhandel werden Verträge überwiegend in staatlichen Recheneinheiten (EUR, USD) denominiert und bilanziert. Da der ICP-Escrow-Kanister nativ in ganzzahligen Satoshis rechnet und mehrwöchige Liefer- oder Leistungsfristen üblich sind, stellt die Wechselkursvolatilität zwischen Bitcoin und Fiat-Währungen eine praktische Herausforderung dar. Colabonate adressiert dieses Problem über ein dreistufiges Modell:

1. **Fixed-Satoshi-Baseline (Krypto-native Abrechnung):** Für Handelspartner, die Bitcoin als eigene Verrechnungseinheit führen, gilt die direkte Satoshi-Fixierung bei Trade-Erstellung. Es tritt kein Währungsrisiko auf; 100 % der vereinbarten Sats fließen gemäß Meilensteinplan.
2. **Dual-Denominated Escrow mit Puffer (Ziel-Fiat-Hedging):** Für in EUR/USD denominierte Handelsverträge hinterlegt der Käufer bei Trade-Erstellung einen Satoshi-Gegenwert, der einem Puffer von 110–115 % des aktuellen Wechselkurses entspricht. Bei jeder Meilensteinfreigabe (`release_milestone`) berechnet der Kanister über ein dezentrales Exchange-Rate-Oracle den exakten Satoshi-Betrag, der dem fälligen Fiat-Teilbetrag entspricht, und zahlt diesen an den Verkäufer aus. Nach Erfüllung des letzten Meilensteins wird der unverbrauchte Restpuffer im selben Transaktionsschritt automatisch an die `buyer_refund_address` zurückerstattet (`validate_payout_outputs`).
3. **Multi-Token- und Stablecoin-Roadmap (ckUSDC / ckEUR):** Durch die native Multi-Token-Unterstützung des Internet Computers kann die Kanister-Architektur perspektivisch auf Chain-Key-Tokens (ckUSDC, ckEUR) erweitert werden. Damit wird eine vollwertige Fiat-Parität ohne Krypto-Volatilität erreicht, während die Nicht-Custodialität und die BIP-340-Signaturverifikation vollständig erhalten bleiben.

### 4.4 Schnittstelle zu ISO 20022: ein ehrliches Mapping

Wo im Zahlungsfluss setzt die Kanister-Logik an? Präzise und redlich formuliert: Der Kanister ersetzt *keine* ISO-20022-Nachricht, sondern die *Verwahrungs- und Freigabelogik*, die auf eine ISO-20022-konform beschriebene Zahlung folgt. Ein 1:1-Mapping zwischen Kanister-States und ISO-20022-Statuscodes gibt es *nicht*, und zwar aus einem sachlichen Grund: Kanister-States sind technische Escrow-Zustände einer Treuhand-UTXO, während ISO-20022-Statuscodes Nachrichtenzustände einer Zahlungstransaktion abbilden. Es handelt sich um verschiedene Achsen. Die angemessene Entsprechung ist daher eine *zeitliche Abfolge mit wirtschaftlicher Korrespondenz*, kein tabellarischer Gleichklang:

| Kanister-State | Wirtschaftliche Entsprechung | ISO-20022-Korrespondenz (konzeptionell) |
|---|---|---|
| `PendingFunding` | Trade eröffnet, Einzahlung ausstehend | ausgelöst durch eine pacs.008-Kreditübertragung (Customer Credit Transfer); Kanister erwartet die Funding-UTXO |
| `Funded` | Einzahlung bestätigt, Treuhand aktiv | wirtschaftlich nahe `ACSP`/`ACSC` (accepted for settlement / settlement completed auf Debtor-Seite) |
| `Released` | Gegenleistung erbracht, Auszahlung an Seller | wirtschaftlich `ACSC` (settlement completed) auf Creditor-Seite |
| `Disputed` | Streitfall, reguläre Freigabe gesperrt | *kein direktes normatives Analogon*: die Norm kennt keinen Schlichtungs-Zustand (Lücke b); das ist genau die Ergänzung des Stacks |
| `Refunded` | Timeout- oder Verdict-bedingte Rückzahlung | wirtschaftlich einer Rückbuchung/`RJCT`-Richtung verwandt |

Die normativen Statuscodes entstammen der Enum `TransactionIndividualStatus3Code` der pacs.002-Familie (Werte u. a. `ACTC`, `RJCT`, `PDNG`, `ACCP`, `ACSP`, `ACSC`, `ACWC`) [prowide: `model-common-types/.../TransactionIndividualStatus3Code.java`]. Zwei Beobachtungen schärfen das Bild: Erstens ist die ISO-20022-Norm deutlich breiter angelegt als ihre praktische Nutzung. Eine verbreitete JS-Implementierung, *iso20022.js*, deckt lediglich etwa sieben Nachrichtenfamilien in zwei Business-Areas ab (rund 12 % der am prowide-Modell sichtbaren Normbreite) und implementiert die FI-zu-FI-Statusberichte pacs.002/pacs.008 *gar nicht*: nur die Customer-to-Bank-Seite pain.002 mit sechs Codes [iso20022.js: `src/pain/002/types.ts`]. Zweitens (und das ist der entscheidende, redlich zu benennende Befund) referenziert die Colabonate-Referenzimplementierung ISO 20022 an keiner Stelle: Eine Suche über das gesamte Kanister-Package ergibt null Treffer, und die Candid-Schnittstelle trägt ausschließlich krypto-native Felder (Publickeys, Adressen, Sats, Basispunkte, Timeout), keine Remittance-, Debtor-/Creditor- oder Währungsfelder [Canister: `icp_escrow_canister.did`]. Die obige Tabelle ist also eine *konzeptionelle Korrespondenz, die dieses Papier herstellt*, nicht eine im Code vorhandene ISO-20022-Bindung. Was für eine vollständige, ISO-20022-bezogene PoC noch fehlt, ist ein Nachrichten-Adapter, der pacs.008-Felder in `create_trade`-Parameter übersetzt und umgekehrt Kanister-States in pacs.002-Statusberichte zurückmeldet (Abschnitt 8.3 und 8.4).

### 4.4.1 Das Schlichtungsverfahren: nicht-custodiale Deliberation und kryptografische Exekution

Der Schlüssel zum nicht-custodialen Schlichtungsmechanismus liegt in einer strengen Trennung von *Deliberation* und *Exekution*. Der Kanister prüft bei `submit_dao_verdict` ausschließlich die *Signatur* eines Verdicts. Er vollzieht **keine** inhaltliche Bewertung. Das Zustandekommen der Entscheidung wird vollständig *off-chain* in einem **vierstufigen Dispute-Resolution-Protocol** [WP7 §3.5.5] erarbeitet:

1. strukturierte, On-Chain dokumentierte Kommunikation zwischen den Parteien;
2. Peer-Mediation durch 2–3 zufällig ausgewählte Nutzer mit *gemäß Design* verifizierter HID und Reputations-Mindestschwelle;
3. Fach-Schiedsrichter mit Kategorie-Expertise bei Scheitern von Stufe 2;
4. DAO-Appeal als letzte Instanz per Abstimmung mit Mindestquorum.

Das *Ergebnis* dieser off-chain-Deliberation (ein prozentualer Split aus `buyer_bps`/`seller_bps`) wird auf dem Kanister *allein* durch Signaturverifikation gegen einen zur Kompilierzeit fixierten DAO-Public-Key durchsetzt. Die *Exekution* ist damit deterministisch und kryptografisch: der Kanister trifft keine subjektive Entscheidung, er validiert nur, dass ein autorisierter Schlüssel das Verdict unterschrieben hat, und löst dann die durch Säule 2 (Abschnitt 4.3) ohnehin fixierte Auszahlung aus. Genau daraus ergibt sich die Nicht-Custodialität des Schlichtungsmechanismus: die Streitbeilegung *braucht keine Verwahrungsgewalt*. Die Mittel bleiben während des gesamten Verfahrens mathematisch gebunden im nicht-custodialen Kanister, und das Verdict bewegt sie lediglich innerhalb der vorgegebenen Adressen, ohne eine neue Empfängermöglichkeit zu eröffnen.

Ausdrücklich *ohne* automatisierte KI-Bot-Mediation: die Whitepaper-Begründung lautet Nachvollziehbarkeit und Fairness [WP7 §3.5.5]. Für die Fachjury ist entscheidend: Die Norm-Lücke »fehlende Schlichtungslogik« wird also nicht durch reine Code-Automatik geschlossen, sondern durch ein mehrstufiges, *humangestütztes* Verfahren, dessen Resultat kryptografisch durchsetzbar in den Kanister einfließt. Den Ausgang des Dispute-Verfahrens verknüpft das Reputation Scoring Framework unmittelbar mit der Reputation der beteiligten Parteien (Abschnitt 5.1.1), ein Compliance-relevanter Rückkopplungseffekt.

**Redlicher Status-Hinweis zur HID-Anbindung:** Die HID-Verifiziertheit der Mediatoren (Stufe 2) ist *Designziel*, kein aktueller Ist-Stand; HID selbst ist als Proof-of-Concept noch nicht validiert (Status und Vorarbeiten im Detail: Abschnitt 5.1.1). Bis zur HID-Validierung ist die Mediatorenauswahl über das Reputations- und Review-System abgesichert, die HID-Kopplung bleibt ein noch zu erbringender Entwicklungsschritt.

### 4.4.2 Rechtliche Einordnung und Schiedsfähigkeit nach UNCITRAL

Die Streitbeilegung im internationalen B2B-Handel unterliegt strengen rechtlichen Formvorschriften. Um sicherzustellen, dass Entscheidungen des vierstufigen Schlichtungsverfahrens nicht nur technisch exekutiert werden, sondern auch einer rechtlichen Anfechtung vor staatlichen Gerichten standhalten, fügt sich der Stack in bestehende Rechtsrahmen ein:

- **Elektronische Schiedsvereinbarung (Art. 7 UNCITRAL-Modellgesetz):** Mit dem gegenseitigen Signieren der Trade-Terms via Nostr (Event-Kinds 16/17) schließen Käufer und Verkäufer eine rechtsgültige Schiedsabrede in elektronischer Form gem. Art. 7 Abs. 4 des UNCITRAL Model Law on International Commercial Arbitration ab. Der im Kanister hinterlegte Hash der Vertragsbedingungen (`trade_terms_hash`) belegt das beiderseitige Einverständnis in den Schlichtungsablauf.
- **Qualifikation als Schiedsgutachten (Expert Determination):** Die Entscheidung der Mediatoren bzw. der DAO über den prozentualen Auszahlungssplit (`buyer_bps`/`seller_bps`) qualifiziert sich zivilrechtlich als vorab vereinbarte vertragliche Leistungsbestimmung bzw. Schiedsgutachten.
- **Deterministische Vollstreckung ohne staatliche Zwangsvollstreckung:** Da die Mittel bereits im Kanister hinterlegt sind und die Auszahlung rein rechnerisch nach Signaturprüfung erfolgt, entfällt das Erfordernis eines langwierigen Vollstreckbarerklärungsverfahrens nach dem New Yorker Schiedsübereinkommen von 1958. Das Urteil setzt sich unmittelbar vermögenswirksam um.

### 4.5 Redlicher Implementierungsstatus

Redlichkeit vor der Jury verlangt, dass keine Designentscheidung als fertige Implementierung ausgegeben wird. ADR-253 selbst führt einen »Implementation Reality Check« mit offenen Punkten aus, die hier ungeschönt benannt werden [ADR-253]:

- Der **DAO-Governance-Key ist noch nicht konfiguriert**: `submit_dao_verdict` liefert bewusst einen Fehler statt einer Stub-Erfolgsantwort (`sig::DAO_PUBKEY_HEX = None`). Der Dispute-Pfad ist damit im Code vorhanden, aber *derzeit nicht vollständig nutzbar*, bis der DAO-Key gesetzt ist.
- Der **Kanister-State liegt noch in Heap- statt in Stable-Memory**, ein Upgrade-Risiko vor dem Mainnet.
- Die **Blackhole-vs-DAO-Controller-Entscheidung ist noch offen** (Follow-up FU-253-H).
- Der bestehende **custodiale Hold-Invoice-Pfad (Lightning)** bleibt im Code hinter einem Feature-Flag, wird aber nach Founder-Entscheidung *nie* öffentlich aktiviert.

Diese Punkte sind keine Schwäche des Ansatzes, sondern der Beleg eines ehrlich geführten Implementierungsstatus: sie werden in Abschnitt 7.1 als jeweiliger Implementierungsstatus (`partial`) pro Baustein mitgeführt.

### 4.6 Rechtliche und regulatorische Schutzmaßnahmen

Die Architektur ist von vornherein so angelegt, dass dezentrale Philosophie und rechtliche Absicherung sich nicht ausschließen, sondern gegenseitig tragen. Die folgenden Schutzmaßnahmen ergeben sich *strukturell* aus dem Design; sie sind keine nachträglichen Zusagen, sondern Eigenschaften des gebauten Codes.

- **Nicht-Custodialität als Schutz vor Geldemittenten-Exposition.** Weil die Colabonate-Entität zu keinem Zeitpunkt Verfügungsgewalt über die hinterlegten Mittel ausübt und den dritten Ausgabe-Empfänger zur Laufzeit nicht verändern kann (Säule 2, Abschnitt 4.3), ist die Architektur so konzipiert, dass im Escrow-Bein *keine* custodiale Geldemittenten-Rolle der Plattform entsteht. Technischer Settlement-Agent ist der nicht-custodiale Kanister, nicht Colabonate.
- **Nachprüfbarer Code als Rechenschaftsbasis.** Reproduzierbarer Build, ein On-Chain abrufbarer Modul-Hash und ein *blackholed* oder DAO-Multisig-gesteuerter Controller [ADR-253] machen den exakt die Mittel regelnden Code für jede Partei verifizierbar und schützen vor nachträglicher, einseitiger Manipulation. Verantwortung wird dadurch verteilt und transparent, nicht zentralisiert.
- **Permissionless Timeout als Schutz vor Mittel-Blockade.** Der Timeout-Pfad (`claim_timeout_refund`, Abschnitt 4.2.5) ist von jedem Aufrufer ausführbar. Auch bei vollständigem Plattform-Ausfall können die Mittel nicht dauerhaft blockiert werden, eine operationelle und rechtliche Sicherheitsventil-Funktion gegen unbefristete Verwahrung.
- **Due-Process-gerechte, On-Chain dokumentierte Schlichtung.** Das vierstufige Dispute-Resolution-Protocol (Abschnitt 4.4.1) wird On-Chain dokumentiert geführt; die Mediatorenauswahl ist *gemäß Design* HID-verifizierbar und reputationsgebunden, das Verdict fließt deterministisch in die Reputation ein. Dies stützt eine Nachvollziehbarkeits- und Fairness-Argumentation und macht den Streitverlauf auditierbar.
- **Datenschutz durch Design.** Die Zero-Knowledge-Auslegung von HID (keine Rohdatenspeicherung, Konzeptstand; siehe Abschnitt 5.1.1) und die XID-Elision (selektive Offenlegung ohne Signaturbruch, Abschnitt 5.5.2) sind auf Datenminimierung angelegt und damit für grenzüberschreitende KYC-/Datenschutz-Anforderungen relevant.
- **Keine Identitätsnötigung.** Nutzer transagieren ausschließlich mit eigenen Nostr-Schlüsseln; es besteht kein ICP-Wallet-Zwang und keine custodiale Onboarding-Pflicht. Die Selbstbestimmung der Handelnden über ihre Identität und ihr Geld bleibt gewahrt.

**Einschränkende Klarstellung:** Die genannten Punkte sind *architektonische Eigenschaften*, keine juristischen Einschätzungen. Die regulatorische Einordnung (insbesondere die Frage, ob und wie eine jeweilige Jurisdiktion den Kanister, die DAO oder die Colabonate-Entität qualifiziert) ist rechtsgebietsabhängig und muss vor einem produktiven Rollout durch qualifizierte Rechtsberatung bestätigt werden. Dieses Papier stellt die *technische* Schutzlage dar, nicht ihre rechtliche Anerkennung.

---

## 5. Identitätsschicht: Nostr, XID und Vertrauensmodell

### 5.1 Nostr als Basisschicht (implementiert)

Die reale Basis der Identitätsschicht ist Nostr, ein schlüsselbasiertes, zensurresistentes Protokoll für Kommunikation und Identität. Im Gegensatz zu den vorgeschlagenen Bausteinen weiter unten in diesem Abschnitt ist die Nostr-Integration in der Colabonate-Referenzimplementierung **real und breit ausgebaut**, nicht nur konzeptionell: App-seitig existieren Module für Event-Building/Publishing, kind:0-Profile, NIP-59-Gift-Wrap (kind:1059), NIP-17 private Direktnachrichten (kind:14), Identität, Handshake, NIP-44-Verschlüsselung, NIP-09 und NIP-89 (`apps/colabonate-app/src/lib/nostr*.ts`, `nostr-identity.ts`, `nostr-handshake.ts`, `nip44.ts`, `nip09.ts`, `nip89.ts`); serverseitig ein Relay-Service, Nostr-Auth, DAO-Nostr sowie entsprechende Routen (`apps/server/services/nostr-relay.ts`, `nostr-auth.ts`, `dao-nostr.ts`; Routen `auth-nostr.ts`, `identity.ts`, `dao.ts`).

Genutzte Standard-NIPs sind 01 (Basisprotokoll), 42 (Relay-Auth), 44 (Verschlüsselung v2), 59 (Gift Wrap), 46/07 (Remote-Signer/Bunker), 98 (HTTP-Auth, 60s-Replay-Fenster), 60/61 (Cashu-Wallet) und 17 (private DMs). Darüber hinaus definiert Colabonate eigene addressierbare Event-Kinds 30017–30027 für protokollspezifische Objekte (ADR-009, Status: *implemented*): 30017 Offer, 30018 Ticket erstellt, 30019 Ticket-Statusupdate, 30020 Dispute eröffnet, 30021 Verification Credential (»Soulbound«), 30022 Governance Vote, 30023 HID/Humanode-Attestation, 30024 Reputationsbewertung, 30025 Token-Stake, 30026 Proximity Proof, 30027 Company Profile [ADR-009], dokumentiert als interne Konvention pending offizieller NIP-Registrierung.

### 5.1.1 Ergänzende Identitäts- und Vertrauensbausteine (Konzept- bzw. Entwurfsstand)

Zwei Bausteine ergänzen die Nostr-Basis und sind für die Jury-Argumentation von Belang, weil sie *andere* Dimensionen der Identitätslücke (a) schließen als XID und nicht mit diesem verwechselt werden dürfen. Beide sind aktuell *Konzepte* und keine gebauten Komponenten: ihre Nennung hier ist daher ein Entwurfsstand, kein Implementierungsnachweis.

- **Human Identity (HID)** mit optionaler biometrischer Kopplung über den Humanode Biomapper [WP7 §3.4]: ein als Zero-Knowledge-Einzigartigkeitsnachweis *konzipierter* Mechanismus ohne Rohdatenspeicherung, mit dem Designziel *»eine reale Person = eine Identität«*. **Redlicher Status:** HID ist als Proof-of-Concept *noch nicht entwickelt und nicht erprobt* und muss für die verlässliche Feststellung der Echtheit einer menschlichen Identität erst noch validiert werden. Die maßgeblichen *Vorarbeiten* liegen jedoch bereit: Spezifikation im Whitepaper [WP7 §3.4], das addressierbare Nostr-Event-Kind 30023 (»HID/Humanode-Attestation«, ADR-009) sowie die Integrations-Schnittstelle zum Humanode Biomapper. HID adressiert bei gelungener Validierung das *Sybil-/Eindeutigkeits-Problem*.
- **Reputation Scoring Framework (RSF)** [WP7 §3.5]: bewusst *deterministisch und regelbasiert* statt einer KI-Black-Box *konzipiert*, mit sechs auditierbaren Dimensionen (Zuverlässigkeit, Qualität, Kommunikation, Compliance, Fairness, Community-Beitrag) und DAO-gesteuerten Gewichten per 1P1V-Abstimmung. RSF ist als Konzept zur Lieferung einer *nachvollziehbaren Vertrauensmetrik* zwischen Handelspartnern angelegt; der heutige reale Vertrauensanker im Stack ist das Nostr-basierte Reputations-/Review-System (kind 30024).

Beide Bausteine ergänzen sich konzeptionell, überschneiden sich aber *nicht* mit dem, was XID leisten würde (kryptografische Provenienz und Historie einzelner Identifikator-Attestierungen, Abschnitt 5.5). In der Vergleichstabelle Abschnitt 7.1 treten sie als eigene Dimensionen neben XID auf.

### 5.2 XID (Blockchain Commons) als vorgeschlagene Erweiterung

XID (*eXtensible IDentifier*, Blockchain Commons) ist ein selbstsouveräner, provenienzbasierter Identifikator mit selektiver Offenlegung über Gordian Envelopes. Der Status ist hier mit aller Deutlichkeit zu markieren: **XID ist in der Colabonate-Referenzimplementierung derzeit weder implementiert noch referenziert.** Eine gezielte Suche nach *»XID«*, *»Gordian Envelope«*, *»Provenance Mark«*, *»bc-envelope«* und *»did:key«* über das gesamte Repo ergab keine echten Treffer (nur False Positives wie `taxId`/`txid`) [Canister-Code-Audit]. Die reale Identitätsschicht der App ist heute Nostr-Schlüssel plus Reputations-/Review-Events (kind 30024), *kein* W3C-DID- oder XID-Format. Dieser Abschnitt ist folglich als **Ergänzungsvorschlag für den Colabonate-Stack** zu lesen, nicht als Beschreibung einer gebauten Komponente. Die Argumentation an die Jury bleibt dennoch gültig (XID schließt eine Lücke, die weder ISO 20022 noch der heutige Colabonate-Stack schließt), nur muss die Formulierung den Unterschied zwischen *vorgeschlagen* und *implementiert* sauber halten. Primärquellen sind die Blockchain-Commons-Spezifikationen BCR-2024-010 (XID) und BCR-2026-003 (XID Edges) [BCR-2024-010; BCR-2026-003].

### 5.3 Komplementarität von Nostr-Schlüsseln und XID

Die Kombination beider ist stärker als jede Komponente einzeln, weil sie auf *derselben* kryptografischen Substanz aufsetzt: Im heutigen Stack vorhanden ist nur die Nostr-Schlüsselseite (secp256k1/BIP-340, siehe Abschnitt 5.1 und 4.2.6 zur kanisterseitigen Signaturverifikation). Die XID-Seite wäre eine *additive Erweiterung auf denselben Schlüsseln*, kein Ersatz. Konkret entsteht ein XID deterministisch als SHA-256-Hash der CBOR-Kodierung eines `PublicSigningKey` (Inception Key) und ist damit an dasselbe Schlüsselpaar gebunden, bleibt aber über seinen Lebenszyklus stabil, da Verifikationsschlüssel rotiert werden können, ohne den XID zu ändern [BCR-2024-010].

### 5.4 Anwendungsfall im internationalen Handel

Der Anwendungsfall ist die Verifikation von Handelspartnern ohne zentrale Ausstellerinstanz. Der heutige Ist-Zustand in der Referenzimplementierung ist ein pragmatischer Zwischenschritt: Relay-Ownership-Verifikation via kind:0/kind:30021/kind:30405-Credential (*»L1-Upgrade«*) plus NIP-98-Auth (`apps/server/routes/identity.ts`). Dieser Schritt ließe sich durch XID vertiefen (Provenienz-Nachweis, selektive Offenlegung), *ohne* dass das heutige System dafür ersetzt werden müsste.

### 5.5 XID als eigenständiger Ergänzungsbaustein zu ISO 20022

ISO 20022 sieht in seinen Nachrichten Partei- und Kontoidentifikatoren vor, definiert aber, wie in Abschnitt 2.3 dargelegt, keine kryptografische Methode, um die Identität hinter diesen Feldern zu verifizieren oder deren Historie nachvollziehbar zu machen. Genau diese Lücke *würde* der vorgeschlagene XID-Anschluss (vgl. Status in Abschnitt 5.2) über drei Mechanismen schließen:

- **5.5.1 Provenance Marks:** eine nachvollziehbare, hashkettenbasierte Historie einer Identität über die Zeit, inklusive Nachweis, welche Version die aktuell gültige ist. Relevant für Compliance-Prüfungen im Zahlungsverkehr.
- **5.5.2 Gordian Envelope / selektive Offenlegung (Elision):** Handelspartner können einzelne Attestierungen gezielt offenlegen oder verschleiern, *ohne* die Signatur des Gesamtdokuments zu brechen. Relevant für den Datenschutz bei grenzüberschreitenden KYC-Anforderungen.
- **5.5.3 Schlüsselbasierte Selbstsouveränität ohne zentrale Ausstellerinstanz:** strukturell passend zu den dezentralen Kanister- und Nostr-Identitäten im Stack, im Gegensatz zu klassischen zentral ausgestellten Identifikatoren.

### 5.6 Konkreter Ergänzungsvorschlag für ISO-20022-Nachrichtenfelder

Auf Basis der prowide-Feldanalyse [prowide] und der XID-Spezifikation [BCR-2024-010] lässt sich ein konkreter, *minimal-invasiver* Mapping-Vorschlag formulieren. ISO 20022 besitzt bereits einen normativ vorgesehenen Erweiterungsmechanismus für proprietäre Identifikationsschemata: der generische Slot `Othr` (z. B. `GenericOrganisationIdentification1`) innerhalb der Debtor-/Creditor-Party-Blöcke, mit den Freitext-Subelementen `Id`, `SchmeNm` (wahlweise ein offizieller Code `Cd` oder ein *proprietärer* String `Prtry`) und `Issr`. Eine XID-Referenz lässt sich dort *additiv* einbetten, ohne bestehende Feldsemantik zu verändern:

```xml
<Dbtr>
  <Nm>Acme Cooperative</Nm>
  <Id>
    <OrgId>
      <Othr>
        <Id>529900T8BM49AURSDO55</Id>
        <SchmeNm><Prtry>LEI</Prtry></SchmeNm>
        <Issr>GLEIF</Issr>
      </Othr>
      <Othr>
        <Id>71274df1</Id>
        <SchmeNm><Prtry>XID</Prtry></SchmeNm>
        <Issr>Blockchain Commons</Issr>
      </Othr>
    </OrgId>
  </Id>
</Dbtr>
```

Dies ist *keine* Normänderung, sondern die bestimmungsgemäße Verwendung des normativen `Othr`/`Prtry`-Erweiterungspunkts: `Othr` ist ein Array, der XID-Eintrag wird *neben* bestehenden Identifikatoren (LEI, BIC, interne IDs) gesetzt, und alle Felder bleiben optional: ein Empfänger, der das Scheme *»XID«* nicht kennt, ignoriert den Eintrag schlicht (forward-kompatibel). Analoge Pfade existieren für Personen (`PrvtId/Othr`), Konten (`AccountIdentification4Choice.Othr`) und Finanzinstitute (`FinInstnId/Othr`) [prowide].

**Ehrliche Einschränkung (Längen-/Kodierungs-Constraint).** Ein vollständiger, maschinenverifizierbarer XID (32 Byte) passt in *keiner* standardisierten Kodierung in ein `Max35Text`-Feld: als Hex sind es 64 Zeichen, als `ur:xid/...` rund 85 Zeichen, als vollständiges XID-Dokument (Gordian Envelope) mehrere hundert Zeichen [BCR-2024-010]. Das `Othr.Id`-Feld kann daher *nur einen verifizierbaren Zeiger* halten, nicht den vollständigen Beweis. Der empfohlene Ansatz für einen Prototyp ist, das 4-Byte-Anerkennungspräfix des XID (z. B. `71274df1`, 8 Zeichen) als Erkennungshinweis in `Othr.Id` zu führen und die *vollständige* Verifikation (über alle 32 Byte) sowie die Auflösung über eine `dereferenceVia`-URL *out-of-band* vorzunehmen [BCR-2024-010]. Die Spezifikation stellt ausdrücklich klar, dass der Maschinenvergleich stets über alle 32 Byte erfolgt und das 4-Byte-Präfix allein kollisionsanfällig und nicht beweisend ist. Die ISO-20022-Nachricht trägt also den *Hinweis*, der kryptografische *Beweis* bleibt eine additive, dezentral geführte Schicht, passend zur Komplementaritäts-These des gesamten Papiers.

### 5.7 End-to-End-Auflösungsprozess von XID und Nostr-Identitäten

Für Unternehmens-ERP-Systeme und Banken-Gateways ergibt sich daraus ein klarer, vierstufiger Verifikationsablauf:

1. **Extraktion des Identifikator-Zeigers:** Das Empfängersystem empfängt die ISO-20022-XML-Nachricht und extrahiert aus `Dbtr/Id/OrgId/Othr[SchmeNm/Prtry='XID']/Id` das 4-Byte-Präfix (z. B. `71274df1`).
2. **Out-of-Band-Auflösung:** Über das Nostr-Netzwerk (Abfrage von kind:30027 Company Profile Events) oder einen dezentralen Resolver wird das vollständige 32-Byte XID Inception Document (Gordian Envelope) abgerufen.
3. **Kryptografische Signatur- und Provenienzprüfung:** Das System prüft die BIP-340-Schnorr-Signatur des Inception Keys und validiert die Hashkette (Provenance Marks) bis zum aktuellen Zustand.
4. **Selektive Offenlegung (Elision):** Durch die Gordian-Envelope-Technologie validiert das System die offengelegten Identitätsattribute (z. B. LEI `529900T8BM49AURSDO55` gegen das GLEIF-Register), während private Metadaten kryptografisch verschleiert bleiben, ohne die Gesamtsignatur zu brechen.

---

## 6. Lightning als Zahlungsschicht

### 6.1 Rolle im Stack (implementiert, teilweise legacy)

Lightning fungiert im Stack als schnelle, kostengünstige Bitcoin-L2-Zahlungsschiene *ergänzend* zum Escrow-Layer. Die Referenzimplementierung abstrahiert die Zahlungs-Provider über das Interface `PaymentProvider` (`packages/payments/src/PaymentProvider.ts`) mit den Operationen `createInvoice`, `createHoldInvoice`/`settleHoldInvoice`/`cancelHoldInvoice`, `payInvoice`, `createSubWallet` und konkreten Implementierungen `LNBitsProvider`, `LndHoldClient` (native LND-Hold-Invoices, ADR-193) und `LightsparkProvider`.

Klarzustellen ist der Status des Hold-Invoice-Pfads: Er ist der **legacy/custodiale** Escrow-Rail und bleibt hinter einem Feature-Flag; nach Founder-Entscheidung wird er *nie* öffentlich aktiviert, sobald der ICP-Kanister produktiv ist [ADR-253]. Direkte Lightning-Zahlungen *ohne* Escrow (Buyer zahlt die Invoice des Sellers direkt, via LNURL-Adresse/NWC) bleiben dagegen der Launch-Pfad *»Stufe 1«* für Micro-Trades ohne Treuhandbedarf. Als zweite nicht-custodiale Zahlungsschiene für Direktzahlungen kommt Cashu/eCash (NIP-60/61, `packages/cashu-wallet`) hinzu.

### 6.2 Zusammenspiel mit dem ICP-Kanister: zwei Rails, keine Fernsteuerung

Ein Lightning-*basiertes* Kanister-Escrow wurde in ADR-253 explizit geprüft und verworfen [ADR-253]. Der technische Grund ist struktureller Natur: Lightning-Fonds existieren ausschließlich in Channels einer laufend online betriebenen Node; ein Kanister kann aber strukturell *keine* Node sein (keine persistenten P2P-Verbindungen, rund 1–2 Sekunden Konsens-Latenz). Jede Konstruktion der Art *»Kanister steuert eine LND fern«* beließe die tatsächliche Verfügungsmacht beim Node-Betreiber und wäre damit custodial. Die Konsequenz ist eine saubere Trennung der Rails: Lightning und Cashu bedienen den *Direktzahlungspfad*, der ICP-Kanister mit nativer Bitcoin-Threshold-Kryptografie bedient den *Escrow-Pfad*. Beide Rails koexistieren; Lightning löst die Escrow-Freigabe *nicht* technisch aus.

---

## 7. Vergleichende Gegenüberstellung für die Fachjury

### 7.1 Vergleichstabelle (mit Implementierungsstatus je Zeile)

Die folgende Tabelle vergleicht »ISO 20022 allein« mit »ISO 20022 + Colabonate-Stack«. In der rechten Spalte wird pro Zeile der *Implementierungsstatus* mitgeführt (`implementiert` / `partial` / `vorgeschlagen`), damit die Tabelle nicht mehr verspricht als der Code hält.

| Kategorie | ISO 20022 allein | ISO 20022 + Colabonate-Stack | Status |
|---|---|---|---|
| Settlement-Sicherheit / Treuhand | kein natives Escrow; Treuhand extern/institutionell | nicht-custodiale Escrow-Invariante (kein zur Laufzeit veränderlicher dritter Empfänger) + hash-basierte Autorisierung (BIP-340 über Domain-Separator), On-Chain nachvollziehbar | `partial` (gegen `bitcoind -regtest` verifiziert [ADR-254]) |
| Sybil-Resistenz / Identitäts-Eindeutigkeit | Identifikatoren nur beschreibend | Human Identity (HID), Designziel: 1 Person = 1 Identität, ZK-Einzigartigkeit | `vorgeschlagen` (Konzept, PoC offen; Vorarbeiten bereit [WP7 §3.4]) |
| Vertrauensmetrik | keine | Reputation Scoring Framework (RSF), 6 auditierbare Dimensionen, deterministisch | `vorgeschlagen` [WP7 §3.5] |
| Identitätsverifikation & -provenienz | keine kryptografische Verifikation der Felder | XID (Provenance Marks, selektive Offenlegung) als Ergänzung | `vorgeschlagen` (im Code nicht vorhanden [BCR-2024-010]) |
| Schlichtung / Streitfalllogik | Statuscodes, keine inhaltliche Entscheidung | vierstufiges Dispute-Resolution-Protocol, nicht-custodial (Deliberation off-chain, Exekution via DAO-Verdict-Signatur) | `partial` (DAO-Key noch nicht konfiguriert [ADR-253; WP7 §3.5.5]) |
| Rechtliche Schiedsfähigkeit | Formelle Gerichts- oder Schiedsverfahren (T+Monate) | UNCITRAL-konforme elektronische Schiedsabrede + automatische Ausführung als Schiedsgutachten | `partial` (Konzeptionell geklärt, Pilot offen) |
| Währungs- & Volatilitätsmanagement | Klassisches FX-Hedging der Banken | Dual-Denominated Escrow mit Puffer + Multi-Token-Roadmap (ckUSDC/ckEUR) | `vorgeschlagen` (Architekturmodell) |
| Geschwindigkeit (Settlement) | klassisch branchenüblich T+1 bis T+3 *\[nicht primärquellenbelegt\]* | On-Chain-Bestätigung + sofortige Freigabe bei Meilenstein | `partial` |
| Kosten | klassisch: World-Bank-Remittance-Durchschnitt **6,36 %** *\[niedrigwertiger P2P-Korridor\]*; Wholesale-Korrespondenzbank nicht belegbar | Kern-Transaktionen **0 %** Provision; Schlichtung nur 1–2 % im Streitfall; On-Chain-Fee-Overhead ~200–1.000 Sats/Trade (Hypothese) | `implementiert` (Geschäftsmodell) / `partial` (Ökonomie-Gate offen) [WP7 §4.1; ADR-253] |
| Transparenz / Datenschutz | transparente Nachrichten, kein Elisions-Mechanismus | XID-Gordian-Envelope-Elision (selektive Offenlegung ohne Signaturbruch) | `vorgeschlagen` [BCR-2024-010] |
| grenzüberschreitende Eignung | vorhanden (Nachrichtenebene) | nativ grenzüberschreitend (Bitcoin L2, kein Korrespondenzbanken-Aufbau) | `partial` |

### 7.2 Wirtschaftliches Nutzenversprechen

Der wirtschaftliche Kernvorteil ist die Reduktion von Abwicklungsrisiko und -kosten im internationalen Handel. Wie in Abschnitt 3.1 belegt, sind Colabonates Kern-Transaktionen einschließlich Escrow-Nutzung dauerhaft gebührenfrei, gegenüber 15–30 % Provision bei klassischen Web2-Marktplätzen [WP7 §4.1]. Zum Vergleich klassischer Korrespondenzbankverkehr: Der World-Bank-Remittance-Durchschnitt liegt bei 6,36 % (Q3 2025) [WB-RPW], wobei ausdrücklich einzuschränken ist, dass dies ein *niedrigwertiger P2P-Remittance*-Benchmark ist (typische Testbeträge USD 200–500) und *keine* Kennzahl für hochwertige Wholesale-Korrespondenzbankzahlungen, für die keine öffentliche Primärzahl existiert.

Ergänzend aus ADR-253: Der On-Chain-Fee-Overhead des ICP-Kanister-Escrows (rund 200–1.000 Sats pro Trade bei ruhiger Fee-Lage, zwei oder mehr Transaktionen) ist die noch zu messende Restgröße und Gegenstand des Ökonomie-Gates (FU-253-H) [ADR-253]. Für die Mindest-Escrow-Größe (Arbeitshypothese rund 100–250k Sats) ist dieser Overhead real relevant; sobald Messwerte vorliegen, sollten sie in der finalen Tabelle die Arbeitshypothese ersetzen. Im dokumentierten 3-Milestone-Test betrugen die gesamten Miner-Gebühren 9.958 Sats bei 150.000 Sats Finanzierung.

### 7.3 Einordnung als Ergänzung, nicht als Ersatz

Für die Akzeptanz im Komitee ist diese Einordnung entscheidend: ISO 20022 bleibt als Nachrichtenstandard unangetastet und wird operativ sogar *gestärkt*, weil Identität, Schlichtung und Settlement erstmals strukturell abgesichert werden. Keine Änderung an ISO 20022 selbst ist nötig; der XID-Anschluss nutzt ausdrücklich den *bereits normativ vorgesehenen* `Othr`/`Prtry`-Erweiterungspunkt (Abschnitt 5.6), und das Escrow operiert *oberhalb* der Nachrichtenschicht.

---

## 8. Proof-of-Concept-Skizze und ISO-20022-Adapter-Blueprint

### 8.1 Minimaler Testfall und erbrachter Nachweis

Der minimale Testfall ist ein grenzüberschreitender Handelsvorgang mit Escrow über den Kanister, Identität über Nostr (und perspektivisch XID), Zahlung über Lightning sowie einer ISO-20022-konformen Nachrichtenschicht.

Redlich getrennt werden müssen zwei Dinge: **(1) Was bereits erbracht ist:** der in Abschnitt 4.1 dokumentierte 3-Milestone-Escrow-Trade im Split 25/50/25, reproduzierbar verifiziert gegen einen isolierten `bitcoind -regtest`-Node [Canister-README]. Dieser Testlauf ist der technische Kern-Nachweis, dass die nicht-custodiale Treuhand- und Freigabelogik funktioniert. **(2) Was für eine ISO-20022-bezogene PoC noch entwickelt werden muss:** der Test ist ein *reiner* Kanister-/Bitcoin-Test *ohne* ISO-20022-Bezug. Ein vollständiger, ISO-20022-bezogener PoC benötigt den nachfolgend spezifizierten Nachrichten-Adapter.

![Abbildung 1: Escrow-Zustandsautomat des ICP-Kanisters](assets/escrow-state-diagram.svg)

*Abbildung 1: Zustandsautomat des ICP-Escrow-Kanisters. Rot hervorgehoben ist der Dispute-Pfad als Risiko-Kante; gestrichelt der permissionless Timeout-Refund. Quelle: ADR-253/254, `packages/icp-escrow-canister`.*

### 8.2 Erfolgskriterien (Pilot-Gates aus ADR-253)

Als Erfolgskriterien für den PoC übernimmt dieses Papier die in ADR-253 definierten Pilot-Gates [ADR-253]:

- **Ökonomie-Gate:** Messung des On-Chain-Fee-Overheads (zwei oder mehr Transaktionen pro Trade) und Festlegung einer Mindest-Escrow-Größe.
- **Security-Gate:** externes Audit (t-ECDSA-Key-Handling, UTXO-Edge-Cases, Reorg-Tiefe).
- **Governance-Gate:** reproduzierbare Build-Dokumentation sowie Controller-Status (Blackhole oder DAO-Multisig).
- **Pilot-Gate:** Testnet-/Regtest-End-to-End *einschließlich* Dispute und Timeout-Refund, gefolgt von einem Mainnet-Pilot (≥20 Trades, ≥1 Dispute, ≥1 Timeout).

### 8.3 Konkreter pacs.008-zu-Candid-Adapter-Blueprint

Um die Lücke zwischen traditionellen Banknachrichten und dem ICP-Kanister technisch exakt zu schließen, übersetzt ein schlanker Adapter eingehende `pacs.008.001.10`-Überweisungsaufträge deterministisch in die Candid-Schnittstelle `create_trade` (`packages/icp-escrow-canister/src/types.rs`):

| ISO 20022 `pacs.008` XML-Pfad | ICP-Escrow-Canister Candid Feld | Typ & Konvertierungsregel |
|---|---|---|
| `CdtTrfTxInf/PmtId/EndToEndId` | `trade_id` (im Kontext/Mapping) | Eindeutige Transaktions-ID |
| `CdtTrfTxInf/Dbtr/Id/OrgId/Othr[Prtry='XID']/Id` | `buyer_pubkey` | 32-Byte x-only Nostr/XID Inception Key (Hex) |
| `CdtTrfTxInf/Cdtr/Id/OrgId/Othr[Prtry='XID']/Id` | `seller_pubkey` | 32-Byte x-only Nostr/XID Inception Key (Hex) |
| `CdtTrfTxInf/DbtrAcct/Id/Othr/Id` | `buyer_refund_address` | Native SegWit Bitcoin-Adresse (`bc1q...`) |
| `CdtTrfTxInf/CdtrAcct/Id/Othr/Id` | `seller_payout_address` | Native SegWit Bitcoin-Adresse (`bc1q...`) |
| `CdtTrfTxInf/IntrBkSttlmAmt` | `expected_amount_sats` | Umrechnung des Zahlbetrags in Satoshis (`u64`) |
| `CdtTrfTxInf/RmtInf/Ustrd` | `milestone_bps` | Parsing des Meilenstein-Musters (z. B. `MS:2500,5000,2500`) |
| `CdtTrfTxInf/SttlmTmReq/TillTm` | `timeout_at` | Ablauf-Timestamp in Nanosekunden (`u64`) |

**Beispiel eines eingehenden pacs.008-Nachrichtenfragments:**

```xml
<CdtTrfTxInf>
  <PmtId>
    <EndToEndId>TRADE-2026-08-001</EndToEndId>
  </PmtId>
  <IntrBkSttlmAmt Ccy="EUR">1500.00</IntrBkSttlmAmt>
  <Dbtr>
    <Nm>Buyer Corp</Nm>
    <Id><OrgId><Othr><Id>71274df1</Id><SchmeNm><Prtry>XID</Prtry></SchmeNm></Othr></OrgId></Id>
  </Dbtr>
  <DbtrAcct>
    <Id><Othr><Id>bc1qbuyerrefundaddressxxxxxxxxx</Id></Othr></Id>
  </DbtrAcct>
  <Cdtr>
    <Nm>Seller Global Ltd</Nm>
    <Id><OrgId><Othr><Id>a839f201</Id><SchmeNm><Prtry>XID</Prtry></SchmeNm></Othr></OrgId></Id>
  </Cdtr>
  <CdtrAcct>
    <Id><Othr><Id>bc1qsellerpayoutaddressxxxxxxxx</Id></Othr></Id>
  </CdtrAcct>
  <RmtInf>
    <Ustrd>/MS/2500,5000,2500/EXP/1787123456</Ustrd>
  </RmtInf>
</CdtTrfTxInf>
```

### 8.4 Status-Rückmeldung via pacs.002

Sobald der Kanister-Zustand durch On-Chain-Ereignisse fortschreitet, generiert der Adapter einen standardisierten `pacs.002.001.12`-Statusbericht (Payment Status Report) zurück an die Banksysteme:

```xml
<FIToFIPmtStsRpt>
  <GrpHdr>
    <MsgId>STAT-2026-08-001-01</MsgId>
    <CreDtTm>2026-08-06T14:30:00Z</CreDtTm>
  </GrpHdr>
  <TxInfAndSts>
    <OrgnlEndToEndId>TRADE-2026-08-001</OrgnlEndToEndId>
    <TxSts>ACSP</TxSts>
    <StsRsnInf>
      <Rsn><Prtry>ESCROW_FUNDED_ONCHAIN</Prtry></Rsn>
      <AddtlInf>BTC UTXO confirmed at block height 892140</AddtlInf>
    </StsRsnInf>
  </TxInfAndSts>
</FIToFIPmtStsRpt>
```

### 8.5 Offene technische Fragen und Entwicklungs-Roadmap

Aus dem Code-Audit und der Spezifikation ergeben sich die folgenden priorisierten Entwicklungsaufgaben:

1. **Standalone-Adapter-Modul:** Implementierung des in Abschnitt 8.3 und 8.4 spezifizierten ISO-20022-XML-zu-Candid-Adapters als eigenständiges Microservice-Repository.
2. **Migration auf Stable Memory (FU-254-G):** Überführung des Kanister-Zustands von Heap Memory auf `ic-stable-structures`, um Upgradesicherheit im Mainnet zu garantieren.
3. **Provisionierung des DAO-Governance-Schlüssels (FU-253-H):** Konfiguration von `sig::DAO_PUBKEY_HEX` im Kanister für den produktiven Abschluss des Schlichtungspfads.
4. **End-to-End-Test-Harness:** Automatisierte Testsuite, die einen `pacs.008`-Auftrag generiert, den Kanister auf Bitcoin Regtest finanziert, Meilensteine via BIP-340 signiert freigibt und die korrespondierenden `pacs.002`-Berichte validiert.

---

## 9. Fazit und Empfehlung an das Komitee

### 9.1 Zusammenfassung der Kernthese

Der Colabonate-Stack löst ISO 20022 nicht ab, sondern schließt die drei strukturellen Lücken, die die Norm bewusst offenlässt: die *Identitätsverifikation* der Gegenpartei, die *Schlichtung* von Streitfällen und das *technische Settlement* mitsamt Treuhand. Diese Ergänzung geschieht als Schicht *über* der Nachrichtennorm, ohne diese zu verändern, im XID-Fall sogar unter Nutzung des bereits normativ vorgesehenen `Othr`/`Prtry`-Erweiterungspunkts. ISO 20022 bleibt unangetastet und wird operativ gestärkt.

### 9.2 Konkrete Handlungsempfehlung

Die Empfehlung an das Komitee lautet: Prüfung des Colabonate-Stacks als optionale Ergänzungsschicht beziehungsweise Referenzarchitektur *oberhalb* von ISO 20022. Es ist keine Änderung an ISO 20022 selbst erforderlich. Wo der Stack bereits erbracht ist (nicht-custodiale Escrow-Invariante, gegen Regtest verifiziert; gebührenfreies Geschäftsmodell), ist der Nachweis konkret; wo er noch Vorschlag ist (XID-Provenienz, RSF, vollständige Pilot-Gates), wird dies offen ausgewiesen und als Weiterentwicklungs-Roadmap dargestellt. Der nächste konkrete Schritt ist der ISO-20022-Nachrichtenadapter (Abschnitt 8.3 und 8.4), der den bereits funktionierenden Treuhand-Kanal nahtlos an die Norm anbindet.

---

## Quellenverzeichnis

Die Quellen sind als Kurztags im Fließtext referenziert und hier aufgelöst.

- **[WP7]** Colabonate Whitepaper v7, `docs/colabonate_whitepaper_de_v7.md` (Colabonate-App-Repo, 01.08.2026). Hinweis: Whitepaper v7 ist älter als ADR-253/254 und beschreibt in §3.3/§5.1 die *verworfene* Vorgänger-Architektur (RSK, Lightning-Hold-Invoice-Escrow); für den technischen Escrow-Mechanismus gilt ADR-253/254 + Code als Quelle der Wahrheit.
- **[ADR-253]** ADR-253: Non-Custodial P2P Escrow Strategy, `docs/decisions/253-non-custodial-p2p-escrow-strategy.md` (Status: accepted, 02.–03.08.2026). Quelle für: Zwei-Pfade-/Null-Custody-Entscheidung, Implementierungs-Reality-Check, Pilot-Gates, FU-253-H, Verwerfung von RSK/Lightning-Hold-Invoice-Escrow.
- **[ADR-254]** ADR-254: ICP Escrow Canister Detail Design, `docs/decisions/254-icp-escrow-canister-detail-design.md` (Status: accepted, 04.08.2026). Quelle für: BIP-340-Autorisierung (§2), Ausgabe-Invariante (§5), Teststrategie (§12).
- **[ADR-009]** ADR-009: Nostr Event Kind Range, `docs/decisions/009-nostr-event-kind-range.md` (Status: implemented). Quelle für Event-Kinds 30017–30027.
- **[Canister-README]** `packages/icp-escrow-canister/README.md`: Quelle für 3-Milestone-Testtrade 25/50/25, 150.000 Sats Finanzierung, 9.958 Sats Gebühren, `scantxoutset`-Verifikation.
- **[Canister-Code-Audit]** Code-Audit vom 05.08.2026 über `packages/icp-escrow-canister`: keine Treffer für ISO 20022 / XID / Gordian / Provenance Mark / did:key (nur False Positives `taxId`/`txid`); Candid-Schnittstelle trägt ausschließlich krypto-native Felder.
- **[prowide]** prowide-iso20022 (Java-Referenzimplementierung der ISO-20022-Dictionary-Typen), `Repos/prowide-iso20022`. Quelle für Statuscode-Enum `TransactionIndividualStatus3Code` (`model-common-types/.../TransactionIndividualStatus3Code.java`, 7 Werte) und die Party-/Account-Block-Feldstruktur inkl. generischem `Othr`/`SchmeNm.Prtry`-Slot. Detaillierte Analyse: `Research/2026-08-05-prowide-statuscode-analyse.md`.
- **[iso20022.js]** iso20022.js (JS-Bibliothek), `Repos/iso20022.js`. Quelle für das Praxis-Subset (7 Familien, 2 Business-Areas, ~12 % der Normbreite; pacs.002/pacs.008 nicht implementiert). Detaillierte Analyse: `Research/2026-08-05-iso20022js-praxis-subset.md`.
- **[BCR-2024-010]** Blockchain Commons, BCR-2024-010: XID (eXtensible IDentifier), `Repos/Research/papers/bcr-2024-010-xid.md`. Quelle für: XID als 32-Byte-Wert, SHA-256-des-Inception-Key, `dereferenceVia`-Resolution, 4-Byte-Präfix als reines Erkennungsmittel, Maschinenvergleich über alle 32 Byte.
- **[BCR-2026-003]** Blockchain Commons, BCR-2026-003: XID Edges, `Repos/Research/papers/bcr-2026-003-xid-edges.md`. Quelle für: signierte Edges als verifizierbare Claims.
- **[FRB]** Federal Reserve / FRBServices: Fedwire Funds Service ISO 20022 FAQ. URL: https://www.frbservices.org/resources/financial-services/wires/faq/iso-20022/overview-implementation-details (Cutover 14. Juli 2025; RTGS-Eigenschaft). Abgerufen 05.08.2026.
- **[TCH]** The Clearing House: CHIPS. URL: https://www.theclearinghouse.org/payment-systems/CHIPS (Migration auf ISO 20022 im April 2024). Abgerufen 05.08.2026.
- **[WB-RPW]** World Bank: Remittance Prices Worldwide (Q3 2025). URL: https://remittanceprices.worldbank.org/ (globaler Durchschnitt 6,36 %; Niedrigwert-P2P-Remittance-Benchmark, kein Wholesale-Maß). Abgerufen 05.08.2026.
- **[SWIFT-Vorbehalt]** SWIFT CBPR+ Go-Live (20.03.2023) und MT-Rückzug (November 2026): branchenbekannt, in dieser Recherche *nicht* gegen eine erreichbare Primärquelle verifiziert (swift.com/iso20022.org über automatisierten Abruf nicht erreichbar). Zur manuellen Nachverifikierung freigehalten.

*Stand: 06. August 2026. Dieses Papier ist eine Arbeitsgrundlage für einen Fachjury-Vorschlag; offene Punkte (Bezeichnung des Fachgremiums, Zielrepository, manuelle SWIFT-Nachverifikation, ISO-20022-Nachrichtenadapter) sind im Text ausgewiesen.*
