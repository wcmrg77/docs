---
title: "AI Receptionist"
description: "Set up an AI agent as a front-desk receptionist"
---

Set up an AI agent that acts as a front-desk receptionist — greeting callers, answering common questions, and routing calls to the right department.

## Scenario

A law firm, medical practice, or office wants their phone answered professionally 24/7 without hiring additional reception staff. The AI agent:

- Greets callers warmly
- Identifies the reason for the call
- Answers common questions (opening hours, directions, services)
- Transfers calls to the right employee based on the topic
- Captures caller name and contact details for follow-up

## Setup steps

### 1. Create the agent

Create a new agent or use an existing one. Set the language and formality:

- **Language:** `de` (German)
- **Formality:** `formal` (Sie)

### 2. Write the prompt

```
Du bist die freundliche Telefonrezeption der Kanzlei Mustermann & Partner.

Deine Aufgaben:
- Begruesse jeden Anrufer hoeflich und professionell
- Erfasse den Namen des Anrufers
- Frage nach dem Anliegen
- Beantworte Fragen zu Oeffnungszeiten und Anfahrt aus der Wissensdatenbank
- Leite Anrufer je nach Anliegen an den richtigen Ansprechpartner weiter:
  - Slot 01: Familienrecht (Frau Dr. Mueller)
  - Slot 02: Arbeitsrecht (Herr Schmidt)
  - Slot 03: Vertragsrecht (Frau Weber)
- Wenn kein passender Ansprechpartner verfuegbar ist, nimm eine Nachricht auf und biete einen Rueckruf an
- Verabschiede dich stets freundlich

Verhalte dich hoeflich, professionell und geduldig. Sprich den Anrufer mit "Sie" an.
```

### 3. Set the greeting

```
Guten Tag, Sie sprechen mit der Kanzlei Mustermann und Partner. Mein Name ist Lisa, wie kann ich Ihnen behilflich sein?
```

### 4. Add knowledge base documents

Upload or create documents with information the agent needs:

**Document: "Oeffnungszeiten und Kontakt"**
```
Kanzlei Mustermann & Partner
Adresse: Berliner Strasse 42, 10115 Berlin
Telefon: +49 30 123 456 78

Oeffnungszeiten:
Montag - Donnerstag: 08:30 - 17:30 Uhr
Freitag: 08:30 - 15:00 Uhr
Samstag & Sonntag: geschlossen

Anfahrt: U-Bahn Station Friedrichstrasse, Ausgang Nord, 5 Minuten Fussweg.
Parkplaetze im Hof verfuegbar.
```

**Document: "Leistungen"**
```
Unsere Fachgebiete:
- Familienrecht: Scheidung, Sorgerecht, Unterhalt
- Arbeitsrecht: Kuendigung, Arbeitsvertraege, Abfindung
- Vertragsrecht: Vertragspruefung, AGB, Streitfaelle

Erstberatung: 30 Minuten fuer 90 EUR (inkl. MwSt.)
Terminvereinbarung telefonisch oder per E-Mail an info@kanzlei-mustermann.de
```

### 5. Configure forwarding slots

| Slot | Cases | Assignment | Priority 1 |
|------|-------|------------|------------|
| 01 | Familienrecht, Scheidung, Sorgerecht | Dr. Mueller | +491701111111 |
| 02 | Arbeitsrecht, Kuendigung, Arbeitsvertrag | Herr Schmidt | +491702222222 |
| 03 | Vertragsrecht, AGB, Vertragspruefung | Frau Weber | +491703333333 |

### 6. Add tools

- **Knowledge Base Search** — To answer questions from uploaded documents
- **Transfer Call** — To route calls to employees
- **Extract Variable** — To capture the caller's name and email
- **End Call** — To hang up politely after the conversation

### 7. Set business hours

Configure the schedule to match office hours:
- Mon-Thu: 08:30 - 17:30
- Fri: 08:30 - 15:00

Set a **backup agent** (e.g., an after-hours voicemail agent) for calls outside business hours.

## Result

Callers experience a professional, consistent reception. Common questions are answered instantly from the knowledge base. Complex issues are routed to the right lawyer with the caller's name and context already captured.
