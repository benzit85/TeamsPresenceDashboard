# Teams Presence Dashboard - Deployment Anleitung

Diese Anleitung beschreibt, wie Sie das Teams Presence Dashboard in Ihrer Azure Subscription bereitstellen.

---

## Inhaltsverzeichnis

1. [Voraussetzungen](#1-voraussetzungen)
2. [Entra ID App Registration erstellen](#2-entra-id-app-registration-erstellen)
3. [Azure Static Web App erstellen](#3-azure-static-web-app-erstellen)
4. [Konfiguration abschliessen](#4-konfiguration-abschliessen)
5. [Testen](#5-testen)
6. [Fehlerbehebung](#6-fehlerbehebung)

---

## 1. Voraussetzungen

### Benoetigte Berechtigungen
- **Azure Subscription**: Contributor-Rechte oder hoeher
- **Entra ID (Azure AD)**: Application Administrator oder Global Administrator

### Benoetigte Informationen
Nach Abschluss der Einrichtung benoetigen wir von Ihnen:
- [ ] Client ID (Application ID)
- [ ] Tenant ID (Directory ID)
- [ ] Azure Static Web App URL

---

## 2. Entra ID App Registration erstellen

### 2.1 App Registration anlegen

1. Navigieren Sie zu [Entra ID Portal](https://entra.microsoft.com)
2. Gehen Sie zu **Identity** → **Applications** → **App registrations**
3. Klicken Sie auf **+ New registration**
4. Fuellen Sie aus:
   - **Name**: `Teams Presence Dashboard`
   - **Supported account types**: `Accounts in this organizational directory only`
   - **Redirect URI**: Leer lassen (wird spaeter konfiguriert)
5. Klicken Sie auf **Register**

### 2.2 Wichtige IDs notieren

Nach der Registrierung sehen Sie die Uebersichtsseite. Notieren Sie:

| Feld | Wert |
|------|------|
| **Application (client) ID** | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| **Directory (tenant) ID** | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |

### 2.3 API Permissions hinzufuegen

1. Klicken Sie im linken Menue auf **API permissions**
2. Klicken Sie auf **+ Add a permission**
3. Waehlen Sie **Microsoft Graph**
4. Waehlen Sie **Delegated permissions**
5. Suchen und aktivieren Sie folgende Berechtigungen:

| Permission | Beschreibung |
|------------|--------------|
| `User.Read` | Angemeldeten Benutzer lesen |
| `User.Read.All` | Alle Benutzerprofile lesen |
| `User.ReadBasic.All` | Basis-Benutzerinfos lesen |
| `Presence.Read.All` | Praesenzstatus aller Benutzer lesen |
| `Directory.Read.All` | Verzeichnisdaten lesen |
| `ProfilePhoto.ReadWrite.All` | Profilfotos lesen/schreiben |

6. Klicken Sie auf **Add permissions**

### 2.4 Admin Consent erteilen

1. Klicken Sie auf **Grant admin consent for [Ihre Organisation]**
2. Bestaetigen Sie mit **Yes**
3. Alle Berechtigungen sollten nun einen gruenen Haken zeigen

---

## 3. Azure Static Web App erstellen

### 3.1 Im Azure Portal

1. Navigieren Sie zu [Azure Portal](https://portal.azure.com)
2. Klicken Sie auf **+ Create a resource**
3. Suchen Sie nach **Static Web App** und waehlen Sie es aus
4. Klicken Sie auf **Create**

### 3.2 Grundeinstellungen

| Feld | Wert |
|------|------|
| **Subscription** | Ihre Subscription |
| **Resource Group** | Neu erstellen: `rg-teams-presence` |
| **Name** | `teams-presence-dashboard` |
| **Plan type** | `Free` |
| **Region** | `West Europe` |

### 3.3 Deployment Details

| Feld | Wert |
|------|------|
| **Source** | `GitHub` |
| **GitHub Account** | Mit GitHub anmelden |
| **Organization** | `benzit85` |
| **Repository** | `TeamsPresenceDashboard` |
| **Branch** | `main` |

### 3.4 Build Details

| Feld | Wert |
|------|------|
| **Build Presets** | `Next.js` |
| **App location** | `/` |
| **Api location** | (leer lassen) |
| **Output location** | `.next` |

5. Klicken Sie auf **Review + create**
6. Klicken Sie auf **Create**

### 3.5 URL notieren

Nach dem Deployment (ca. 5 Minuten):
1. Gehen Sie zur erstellten Static Web App
2. Kopieren Sie die **URL** (z.B. `https://happy-sky-abc123.azurestaticapps.net`)

---

## 4. Konfiguration abschliessen

### 4.1 Redirect URI in Entra ID hinzufuegen

1. Gehen Sie zurueck zur App Registration in Entra ID
2. Klicken Sie auf **Authentication**
3. Klicken Sie auf **+ Add a platform**
4. Waehlen Sie **Single-page application**
5. Fuegen Sie die URL hinzu: `https://[ihre-swa-url].azurestaticapps.net`
6. Klicken Sie auf **Configure**
7. Klicken Sie auf **Save**

> **Wichtig**: Die URL muss als "Single-page application" konfiguriert sein, NICHT als "Web"!

### 4.2 Environment Variables setzen

1. Gehen Sie zur Static Web App im Azure Portal
2. Klicken Sie im linken Menue auf **Environment variables** (unter Settings)
3. Fuegen Sie folgende Variablen hinzu:

| Name | Wert |
|------|------|
| `NEXT_PUBLIC_AZURE_AD_CLIENT_ID` | Ihre Client ID aus Schritt 2.2 |
| `NEXT_PUBLIC_AZURE_AD_TENANT_ID` | Ihre Tenant ID aus Schritt 2.2 |

4. Klicken Sie auf **Save**

### 4.3 GitHub Secrets aktualisieren (nur bei erstmaliger Einrichtung)

Falls Sie Zugriff auf das GitHub Repository haben:

1. Gehen Sie zu **Settings** → **Secrets and variables** → **Actions**
2. Aktualisieren oder erstellen Sie:
   - `NEXT_PUBLIC_AZURE_AD_CLIENT_ID`: Ihre Client ID
   - `NEXT_PUBLIC_AZURE_AD_TENANT_ID`: Ihre Tenant ID

3. Loesen Sie einen neuen Build aus (Push oder manuell unter Actions)

---

## 5. Testen

1. Oeffnen Sie die Static Web App URL in einem Browser
2. Klicken Sie auf **Sign in with Microsoft**
3. Melden Sie sich mit einem Benutzer aus Ihrem Tenant an
4. Bei der ersten Anmeldung: Berechtigungen akzeptieren
5. Das Dashboard sollte nun den Praesenzstatus anzeigen

---

## 6. Fehlerbehebung

### "Client ID is not configured"
- Environment Variables sind nicht gesetzt
- Loesung: Schritt 4.2 wiederholen und Build neu ausloesen

### "AADSTS9002326: Cross-origin token redemption..."
- Redirect URI ist als "Web" statt "Single-page application" konfiguriert
- Loesung: In Authentication die URL loeschen und als SPA neu hinzufuegen

### "AADSTS50011: Reply URL does not match"
- Die Static Web App URL stimmt nicht mit der Redirect URI ueberein
- Loesung: Exakte URL in Entra ID Authentication hinzufuegen

### "Insufficient privileges"
- Admin Consent wurde nicht erteilt
- Loesung: Schritt 2.4 wiederholen

### Benutzer sieht keine anderen Praesenzstatus
- `Presence.Read.All` Permission fehlt oder kein Admin Consent
- Loesung: API Permissions pruefen und Admin Consent erteilen

---

## Checkliste

- [ ] App Registration erstellt
- [ ] Client ID notiert
- [ ] Tenant ID notiert
- [ ] API Permissions hinzugefuegt
- [ ] Admin Consent erteilt
- [ ] Static Web App erstellt
- [ ] Static Web App URL notiert
- [ ] Redirect URI als SPA konfiguriert
- [ ] Environment Variables gesetzt
- [ ] Login getestet

---

## Kontakt

Bei Fragen wenden Sie sich an: [Ihre Kontaktdaten hier einfuegen]

---

*Letzte Aktualisierung: Januar 2026*
