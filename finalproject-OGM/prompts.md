> Detalla en esta sección los prompts principales utilizados durante la creación del proyecto, que justifiquen el uso de asistentes de código en todas las fases del ciclo de vida del desarrollo. Esperamos un máximo de 3 por sección, principalmente los de creación inicial o  los de corrección o adición de funcionalidades que consideres más relevantes.
Puedes añadir adicionalmente la conversación completa como link o archivo adjunto si así lo consideras


## Índice

1. [Descripción general del producto](#1-descripción-general-del-producto)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Modelo de datos](#3-modelo-de-datos)
4. [Especificación de la API](#4-especificación-de-la-api)
5. [Historias de usuario](#5-historias-de-usuario)
6. [Tickets de trabajo](#6-tickets-de-trabajo)
7. [Pull requests](#7-pull-requests)

---

## 1. Descripción general del producto

### Herramientas: Deepseek R1

**Prompt 1:**

You are an expert project manager. I want to design and document a software system following investigation and analysis phases, defining use cases data models and high level design for a finance management system called Welz. 

Welz will integrate with as many financial institutions as possible to aggregate accounts, transactions, and investments in one place. It will also calculate total net worth by categorizing assets, debts, and liquid funds, converting them into a unified view. Finally, it will offer advanced analytics, using AI to categorize transactions and provide users with deep financial insights.

I need to generate a PRD containing:

* Small description of Welz's software, added value and competitive advantages in front of state-of-the-art competitors.
* Explanation of the main functionality, and a Lean Canvas diagram to better understand the business model.
* Description of the 3 main use cases, with an associated diagram for each of them.
* Data model that will cover entities, attributes and relations.
* High level design, with both a text explanation and a diagram.
* In-depth C4 diagrams for each of the components.

We will proceed step by step, and I will tell you once we can continue with the next one. 

Do you have any questions before proceeding?

**Prompt 2: Responder a preguntas**

One important aspect is that the initial phase will be a PoC for an AI-course I'm currently attending, so the scope will have to be limited, and things like monetization will not be the main focus. This will affect the scope of things we tackle further down, but I will make notes as we progress. Nevertheless, we can still define the whole scope for documentation sake.

1. **Target Audience & Scope: Is Welz focused on individual consumers (e.g., personal finance) or businesses (e.g., SMBs)? This affects use cases (e.g., tax planning vs. payroll integration) and compliance requirements.**

    Yes, we will focus on individual consumers in the PoC, though the final project will allow users to have shared budgeting (e.g. families, flatmates, potentially SMBs in the very long term).

2. **Monetization Strategy: How will Welz generate revenue? Options include subscriptions, freemium models, or white-labeling for institutions. This is critical for the Lean Canvas (Revenue Streams/Cost Structure).**

    We will have a free trial that will allow users experiment with the platform, and then charge a subscription fee.

3. **Compliance & Security: Are there specific regulations to prioritize (e.g., GDPR, PSD2, Open Banking standards)? Security requirements (e.g., encryption, audit trails) will shape the high-level design.**
    
    Yes, we will operate in the EU, so we need to follow all standard regulations.

4. **Integration Approach: Will Welz use existing financial aggregators (e.g., Plaid, MX, Yodlee) or build direct integrations? This impacts the data model and system complexity.**
    
    Correct, we will use an aggregator, though for the PoC, we will be using a fake integration to focus on the main value-generating features.

5. **AI/ML Scope: What specific analytics are planned? For example:**

    * **Automated transaction categorization (e.g., "Food" vs. "Travel")?**
    * **Predictive insights (e.g., cash flow forecasting)?**
    * **Anomaly detection (e.g., fraud alerts)?**
    
    We can focus on automated transaction categorization for the PoC, but the final product will also have the other two features you mentioned.

6. **Deployment Model: Cloud-native (AWS/Azure/GCP) or hybrid? This influences scalability, infrastructure costs, and high-level architecture.**

    The PoC will be deployed on-premise on a small server, but should be transferrable to a cloud-based solution, so we will be using docker.

7. **Competitive Differentiation: What are the top 3 competitors (e.g., Mint, Personal Capital)? What unique advantages will Welz emphasize? (e.g., broader institution coverage, better AI insights?)**
    
    Our main competitors would be Fintonic, Monefy and Bilance. If you know of any other similar ones, you can add them as well.

**Prompt 3: Ajustes**

Documentation must be complete, but we will only refine use cases that are relevant for the PoC.

Please use markdown, as that is the format I will be storing the documentation in.

Welz will also support having cash accounts, to allow managing physical transactions.
We will also be focusing on the EU market only, just like Fintonic, but also adding stock market institutions.

---

## 2. Arquitectura del Sistema

### **2.1. Diagrama de arquitectura:**

#### Herramientas: Deepseek R1 + ChatGPT 4o

**Prompt 1:**

You are an expert Systems Architect.

Create the high level design with the following technical requirements:
- Language: TypeScript (both frontend and backend)
- Frameworks: Deno, EffectTS
- Database: PostgreSQL
- API Integration: TrueLayer
- Auth: OAuth + RBAC for profiles
- CI: GithubActions

Do not answer yet. Ask any questions you require first to give a thorough answer.

**Prompt 2: Respuesta a preguntas**

1. **Deployment Architecture:**
    - **Should the design assume **Docker containers** for on-prem PoC, with a future shift to Kubernetes in the cloud?** 

        Correct

    - **Are there specific **EU cloud regions** (e.g., AWS Frankfurt) to prioritize for GDPR compliance?**
    
        We are in Spain, so we will need to follow regulations there.

2. **RBAC Granularity:**
    - **Are predefined roles (`VIEWER`, `EDITOR`, `OWNER`) sufficient, or do you need custom roles (e.g., "BUDGET_EDITOR")?** 
    
        Yes, those are ok.

3. **EffectTS Scope**:  
    - **Should EffectTS handle domain logic (e.g., net worth calculations) and error handling, or is it primarily for API layer validation?**

        I'm not an expert, so use whatever solution you consider best. Using multiple features from the same library sounds like a good choice.

4. **Security Extensions**:  
    - **Is client-side encryption (e.g., encrypting sensitive fields like balances) required, or is server-side encryption sufficient?**

        Use encryption on both sides.

    - **Are audit logs needed for profile sharing or transaction edits?**

        Audit logs will not be implemented in the PoC, but you can include them in the documentation.

5. **Frontend Framework**:  
    - **Should the frontend use React (with Next.js) or a lightweight framework like Fresh (Deno-native)?**

        Use Deno Fresh

6. **Monitoring**:  
    - **Are OpenTelemetry integrations or third-party tools (e.g., Datadog) expected for observability?**

        Do the same as with audit logging.

**Prompt 3: Ajustes**

- I would prefer to only have two artifacts, one for the frontend and another one for the backend.
- The frontend will have a web client as well as a mobile app

### **2.2. Descripción de componentes principales:**

#### Herramientas: GithubCopilot + Claude 3.5 Sonnet

**Prompt 1:**

Let's move to filling the C4 model, starting with the first two levels.
Include a small description of the elements for each of the two diagrams.

**Prompt 2:**

You're missing the functionality to manually include transactions in the Container diagram, as well letting the user change transaction categories. Which modules do you think would be in charge?

Focus on the use cases defined in the product requirements. Do you think the modules we have currently defined would be able to fulfill them, or do we need to make any adjustments?

**Prompt 3:**

Seeing that the diagrams are using events and an event bus, let's embrace Event Sourcing and CQRS. Update the documentation accordingly.

### **2.3. Descripción de alto nivel del proyecto y estructura de ficheros**

**Prompt 1:**

What file structure would you suggest for the project?
Can you provide a script to create the necessary folders?

**Prompt 2:**

**Prompt 3:**

### **2.4. Infraestructura y despliegue**

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

### **2.5. Seguridad**

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

### **2.6. Tests**

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

---

### 3. Modelo de Datos

#### Herramientas: Deepseek R1

**Prompt 1:**

Let's move to the data model next. Please include names, type, description for attributes, and primary/foreign keys.

**Prompt 2:**

- Each financial entity will be able to contain multiple accounts (that will be synced together).
- The user connects to financial entities. The relation to financial account is indirect only
- Rename NetWorthSnapshot to BalanceSnapshot and attach it to each account instead of the user. The net worth history can be calculated as the sum of account balance snapshots.
- A user can own multiple profiles, which they can then share with other users

**Prompt 3:**

Let's define all the events required for Event Sourcing so that we can better define the components in our system.

---

### 4. Especificación de la API

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

---

### 5. Historias de Usuario

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

---

### 6. Tickets de Trabajo

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**

---

### 7. Pull Requests

**Prompt 1:**

**Prompt 2:**

**Prompt 3:**
