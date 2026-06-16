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

graph TD
    %% Stili Generali
    classDef user fill:#2ecc71,stroke:#27ae60,stroke-width:2px,color:#fff;
    classDef dev fill:#e67e22,stroke:#d35400,stroke-width:2px,color:#fff;
    classDef aws fill:#232F3E,stroke:#111,stroke-width:2px,color:#fff;
    classDef external fill:#7f8c8d,stroke:#34495e,stroke-width:2px,color:#fff;

    %% Nodi Principali
    Developer([💻 Sviluppatore]) :::dev
    GitHub[🐙 GitHub Repository] :::external
    Amplify[🚀 AWS Amplify Hosting] :::aws
    Route53[🌐 AWS Route 53] :::aws
    User([👥 Utente Finale]) :::user
    Cognito[🔐 AWS Cognito Hosted UI] :::aws
    IdentityPool[🔑 Identity Pools] :::aws
    S3[(🗄️ AWS S3 Buckets)] :::aws
    Firebase[(🔥 Firebase DB)] :::external

    %% Flusso DevOps CI/CD
    subgraph Pipeline CI/CD
        Developer -->|1. git push| GitHub
        GitHub -->|2. Webhook / Auto-Deploy| Amplify
    end

    %% Flusso Utente e Traffico Rete
    subgraph Routing e Connessione
        User -->|3. Naviga al dominio| Route53
        Route53 -->|4. Risolve DNS| Amplify
    end

    %% Flusso Autenticazione (PKCE) e Autorizzazione
    subgraph Flusso Sicurezza & Dati (OAuth 2.0 PKCE)
        User -->|5. Richiede Login| Cognito
        Cognito -->|6. Scambia codice per token JWT| User
        User -->|7. Invia ID Token| IdentityPool
        IdentityPool -->|8. Rilascia credenziali temporanee IAM| User
        User -->|9. Richiede Pre-signed URL| S3
        User -->|10. Sincronizzazione Calendario| Firebase
    end

    %% Note di stile visive per i blocchi
    style Pipeline CI/CD fill:#f9f9f9,stroke:#e67e22,stroke-dasharray: 5 5
    style Routing e Connessione fill:#f9f9f9,stroke:#2980b9,stroke-dasharray: 5 5
    style Flusso Sicurezza & Dati fill:#f9f9f9,stroke:#2ecc71,stroke-dasharray: 5 5

