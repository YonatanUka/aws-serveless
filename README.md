# Alacant-Nordeste

A serverless web app I built for a private organization to manage documents, meeting schedules, and announcements replacing a WhatsApp group and scattered PDF files.

# What it does

Users log in with their credentials and get access to:
- Announcements and upcoming events
- Monthly schedules (editable by assigned members)
- Group assignments
- Protected PDF documents

Access is role-based regular users see documents, admins can upload and edit content, and specific roles can edit only their section.

# Stack

The frontend is plain HTML, CSS and JavaScript. Authentication and infrastructure are handled entirely on AWS:

- Cognito with PKCE flow for authentication and group-based permissions
- S3 for document storage, accessed through short-lived pre-signed URLs so files are never publicly accessible
- Amplify for hosting and CI/CD — connected to GitHub so every push to main deploys automatically
- Route 53 for the custom domain

# How auth works

Login uses the Cognito Hosted UI with PKCE (no client secret exposed). After authentication, the app exchanges the authorization code for tokens, uses the ID token to retrieve temporary AWS credentials from an Identity Pool, and uses those credentials to access S3 directly from the browser.

Users are assigned to Cognito groups (admin, programas, audio, acomodadores, etc.) and the app reads group membership from the decoded JWT to show or hide functionality.

# Work in progress

The monthly schedule editor currently runs on Firebase Realtime Database. I'm migrating it to DynamoDB to keep the whole stack on AWS.

# Live

[alacant-nordeste.com](https://alacant-nordeste.com) — login required

## 📊 Architecture Diagram

```mermaid
graph TD
    %% Nodi Principali con testo protetto da virgolette
    Developer(["💻 Developer"])
    GitHub["🐙 GitHub Repository"]
    Amplify["🚀 AWS Amplify Hosting"]
    Route53["🌐 AWS Route 53"]
    User(["👥 End User"])
    Cognito["🔐 AWS Cognito Hosted UI"]
    IdentityPool["🔑 Cognito Identity Pools"]
    S3[("🗄️ AWS S3 Buckets")]
    Firebase[("🔥 Firebase Realtime DB")]

    %% Flusso DevOps CI/CD
    subgraph Pipeline_CICD ["Pipeline CI/CD"]
        Developer -->|1. git push| GitHub
        GitHub -->|2. Auto-Deploy| Amplify
    end

    %% Flusso Utente e Traffico Rete
    subgraph Routing ["Routing & Connessione"]
        User -->|3. Accesses domain| Route53
        Route53 -->|4. Resolves DNS| Amplify
    end

    %% Flusso Autenticazione (PKCE) e Autorizzazione
    subgraph Security_Data ["Security & Data (OAuth 2.0 PKCE)"]
        User -->|5. Login request| Cognito
        Cognito -->|6. Exchanges PKCE code for JWT| User
        User -->|7. Sends ID Token| IdentityPool
        IdentityPool -->|8. Grants temporary IAM keys| User
        User -->|9. Requests Pre-signed URL| S3
        User -->|10. Syncs schedule data| Firebase
    end

    %% Collegamento degli stili base di Mermaid
    style Pipeline_CICD fill:#f9f9f9,stroke:#e67e22,stroke-width:1px,stroke-dasharray: 5 5
    style Routing fill:#f9f9f9,stroke:#2980b9,stroke-width:1px,stroke-dasharray: 5 5
    style Security_Data fill:#f9f9f9,stroke:#2ecc71,stroke-width:1px,stroke-dasharray: 5 5
```
