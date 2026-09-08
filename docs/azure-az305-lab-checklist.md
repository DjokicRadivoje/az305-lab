# Azure AZ-305 Lab – Checklist

## Cilj lab-a

Primarni ciljevi:

- priprema za **AZ-305**
- praktično razumevanje Azure arhitekture
- postavljanje sopstvenog funkcionalnog Azure sistema
- fokus na **mikroservisnu arhitekturu**
- korišćenje **Azure Container Apps** kao glavne compute platforme
- praktičan CI/CD preko GitHub Actions
- bezbedan pristup servisima preko Managed Identity, RBAC i Key Vault-a
- razumevanje DEV / UAT / PROD strategije

---

## 1. Priprema naloga

- [ ] Napraviti poseban browser profil za privatni Azure lab
  - Predlog naziva: `Azure Personal`
  - Ne koristiti isti profil kao za firmin Azure nalog

- [ ] Izabrati privatni Microsoft nalog
  - Koristiti postojeći privatni Outlook/Hotmail nalog ili napraviti novi
  - Ne koristiti firminu email adresu
  - Uključiti MFA

- [ ] Otvoriti Azure Free Account
  - Prijaviti se privatnim Microsoft nalogom
  - Završiti verifikaciju telefona i kartice
  - Proveriti da je aktiviran početni Azure kredit

---

## 2. Provera Azure okruženja

- [ ] Otvoriti Azure Portal

- [ ] Proveriti Directory / Tenant
  - Zabeležiti naziv tenant-a:
    - `____________________________`

- [ ] Proveriti Subscription
  - Zabeležiti naziv subscription-a:
    - `____________________________`

- [ ] Proveriti da privatni subscription nije povezan sa firminim tenant-om

- [ ] Otvoriti Cost Management
  - Proveriti trenutno stanje kredita
  - Proveriti da nema neočekivanih troškova

---

## 3. Kontrola troškova

- [ ] Kreirati Budget

- [ ] Postaviti okvirni mesečni budget
  - Predlog: `5–10 €`

- [ ] Postaviti upozorenja
  - [ ] 50%
  - [ ] 80%
  - [ ] 100%

- [ ] Proveriti gde se vidi trenutna potrošnja

---

## 4. Resource Group

- [x] Kreirati Resource Group

- [x] Naziv:
  - `rg-az305-lab`

- [x] Region:
  - `West Europe`

- [ ] Razumeti odnos:
  - Subscription
  - Resource Group
  - Resource

---

## 5. GitHub i aplikacioni kod

- [ ] Prebaciti demo / očišćenu verziju postojećeg `.NET 8 Web API` projekta na privatni GitHub nalog

- [ ] Proveriti da repo ne sadrži:
  - secret-e
  - connection stringove
  - poslovne URL-ove
  - interne certifikate
  - produkcione podatke

- [ ] Napraviti osnovne grane:
  - `develop`
  - `uat`
  - `main`

- [ ] Proveriti lokalni build i testove

- [ ] Proveriti Swagger / OpenAPI

- [ ] Dodati Dockerfile

- [ ] Lokalno napraviti container image

- [ ] Lokalno pokrenuti container i proveriti API

---

## 6. Azure Container Registry (ACR)

- [ ] Kreirati Azure Container Registry

- [ ] Razumeti:
  - Registry
  - Repository
  - Image
  - Tag
  - Digest

- [ ] Push-ovati prvi image

- [ ] Proveriti image u ACR-u

- [ ] Ne koristiti registry admin credentials ako nije potrebno

- [ ] Kasnije omogućiti pristup preko Managed Identity + `AcrPull`

---

## 7. Azure Container Apps Environment

- [ ] Kreirati Azure Container Apps Environment

- [ ] Razumeti da Container Apps ne koristi App Service Plan

- [ ] Pregledati:
  - Environment
  - Container App
  - Revision
  - Replica
  - Ingress
  - Scaling

- [ ] Povezati Log Analytics workspace

- [ ] Razumeti odnos:
  - Container Apps Environment
  - više Container App mikroservisa unutar environment-a

---

## 8. Prvi Azure Container App

- [ ] Kreirati prvi Container App za API

- [ ] Povući image iz ACR-a

- [ ] Podesiti:
  - CPU
  - Memory
  - target port
  - external ingress

- [ ] Proveriti javni endpoint

- [ ] Proveriti Swagger / health endpoint

- [ ] Pregledati logove

- [ ] Razumeti razliku:
  - App Service
  - App Service for Containers
  - Azure Container Apps

---

## 9. Mikroservisna arhitektura

Cilj je da lab ne ostane samo jedan container.

Planirana minimalna arhitektura:

```text
Internet
   |
   v
Customer API
   |
   +------> Order API
   |
   +------> Storage / SQL

Order API
   |
   +------> Queue

Worker
   |
   +------> Queue
```

- [ ] Kreirati `customer-api`

- [ ] Kreirati `order-api`

- [ ] Kreirati `worker`

- [ ] Podesiti da samo potrebni servisi imaju external ingress

- [ ] Za interne mikroservise koristiti internal ingress gde ima smisla

- [ ] Proveriti komunikaciju između servisa

- [ ] Razumeti kada koristiti:
  - synchronous HTTP
  - asynchronous messaging

---

## 10. Revisions i deployment strategija

- [ ] Napraviti novu revision aplikacije

- [ ] Razumeti razliku:
  - Revision
  - Replica

- [ ] Uključiti multiple revisions mode

- [ ] Testirati traffic splitting

Primer:

```text
Revision v1 -> 90%
Revision v2 -> 10%
```

- [ ] Pratiti novu revision kroz logove i metrike

- [ ] Povećavati traffic ka novoj revision

- [ ] Vratiti traffic na staru revision kao rollback test

- [ ] Razumeti:
  - Canary deployment
  - Blue/Green deployment
  - Rollback

- [ ] Uporediti sa App Service deployment slots + swap

---

## 11. DEV / UAT / PROD strategija

Za lab prvo razumeti obe opcije.

### Opcija A – jedan Container Apps Environment

```text
cae-az305-lab
├── customer-api-dev
├── customer-api-uat
└── customer-api-prod
```

Prednost:
- jeftinije za privatni lab

Mana:
- manja izolacija

### Opcija B – odvojena okruženja

```text
cae-dev
cae-uat
cae-prod
```

Prednost:
- jača izolacija
- bliže enterprise arhitekturi

- [ ] Za lab izabrati ekonomičniju varijantu

- [ ] Dokumentovati kako bi izgledala enterprise varijanta

- [ ] Podesiti:
  - Development
  - UAT
  - Production

- [ ] Razdvojiti konfiguraciju po environment-u

---

## 12. GitHub Actions CI/CD

- [ ] Napraviti GitHub Actions workflow

### CI

- [ ] Checkout
- [ ] Restore
- [ ] Build
- [ ] Test
- [ ] Docker build
- [ ] Push image u ACR

### CD

- [ ] `develop` -> DEV
- [ ] `uat` -> UAT
- [ ] `main` -> PROD

- [ ] Koristiti GitHub Environments

- [ ] Dodati approval za PROD ako je dostupno

- [ ] Razumeti princip:
  - build once
  - promote same image through environments

- [ ] Ne buildovati različit image samo zbog environment konfiguracije

- [ ] Koristiti immutable tag / commit SHA

---

## 13. Application Insights i observability

- [ ] Povezati aplikacije sa Application Insights / Azure Monitor pristupom

- [ ] Generisati API pozive

- [ ] Proveriti:
  - Requests
  - Failures
  - Performance
  - Dependencies
  - Traces
  - Logs

- [ ] Probati KQL query

- [ ] Dodati Correlation ID kroz mikroservisni poziv

- [ ] Pratiti isti request kroz više servisa

- [ ] Razumeti:
  - logs
  - metrics
  - traces
  - distributed tracing

---

## 14. Key Vault

- [ ] Kreirati Azure Key Vault

- [ ] Dodati testni secret

- [ ] Omogućiti Managed Identity za Container App

- [ ] Dodeliti odgovarajući Key Vault RBAC pristup

- [ ] Iz aplikacije pročitati secret bez čuvanja credential-a u kodu

- [ ] Razumeti razliku:
  - Managed Identity
  - RBAC
  - Secret
  - Key
  - Certificate

- [ ] Razmotriti DEV / UAT / PROD Key Vault strategiju

---

## 15. Managed Identity i ACR pristup

- [ ] Omogućiti Managed Identity na Container App-u

- [ ] Dodeliti `AcrPull` rolu za ACR

- [ ] Proveriti da aplikacija može povući image bez registry password-a

- [ ] Razumeti:
  - System-assigned Managed Identity
  - User-assigned Managed Identity

- [ ] Vežbati izbor između njih kroz AZ-305 scenario

---

## 16. Storage Account

- [ ] Kreirati Storage Account

- [ ] Kreirati Blob Container

- [ ] Uploadovati testni dokument

- [ ] Povezati mikroservis sa Storage Account-om

- [ ] Probati pristup preko Managed Identity

- [ ] Razumeti:
  - Blob
  - Container
  - Queue
  - Table
  - File Share

- [ ] Kasnije ograničiti public access

---

## 17. Azure SQL

- [ ] Kreirati malu Azure SQL bazu

- [ ] Napraviti test tabelu `Customers`

- [ ] Povezati `.NET API` sa bazom

- [ ] Implementirati:
  - [ ] GET customers
  - [ ] POST customer

- [ ] Razmotriti autentikaciju preko Managed Identity

- [ ] Pregledati:
  - Firewall
  - Networking
  - Connection strings
  - Serverless / compute model
  - High availability opcije

---

## 18. Messaging / async komunikacija

Za mikroservisnu arhitekturu dodati bar jedan async scenario.

- [ ] Izabrati Azure Service Bus ili Storage Queue za prvi lab

- [ ] API šalje poruku

- [ ] Worker obrađuje poruku

- [ ] Proveriti retry ponašanje

- [ ] Razumeti:
  - Queue
  - Topic
  - Subscription
  - Dead-letter queue
  - At-least-once delivery

- [ ] Uporediti:
  - Azure Service Bus
  - Storage Queue
  - Event Grid
  - Event Hubs

---

## 19. Azure Functions

Azure Functions ostaje u lab-u zbog AZ-305, iako Container Apps postaje glavna compute platforma.

- [ ] Kreirati Azure Function

- [ ] Napraviti jednostavnu HTTP Function

- [ ] Napraviti Blob Trigger ili Timer Trigger

- [ ] Povezati Function sa Storage Account-om

- [ ] Pregledati logove

- [ ] Razumeti kada izabrati:
  - Container App worker
  - Azure Function

---

## 20. Entra ID

- [ ] Otvoriti Microsoft Entra ID

- [ ] Pregledati:
  - Users
  - Groups
  - App registrations
  - Enterprise applications
  - Roles and administrators

- [ ] Registrovati test aplikaciju

- [ ] Registrovati / expose-ovati API

- [ ] Zaštititi jedan API endpoint autentikacijom

- [ ] Testirati access token

- [ ] Razumeti:
  - Authentication
  - Authorization
  - OAuth 2.0
  - OpenID Connect
  - Access token
  - ID token
  - Scope
  - App role
  - Audience
  - Issuer

---

## 21. RBAC vežba

- [ ] Pregledati IAM na Resource Group-u

- [ ] Dodati testnu RBAC rolu

- [ ] Razumeti:
  - Owner
  - Contributor
  - Reader

- [ ] Proveriti inheritance:
  - Subscription
  - Resource Group
  - Resource

- [ ] Razlikovati:
  - Azure RBAC
  - aplikacionu autorizaciju preko Entra ID

---

## 22. Networking

- [ ] Kreirati VNet

- [ ] Kreirati subnet

- [ ] Pregledati Network Security Group

- [ ] Razumeti:
  - Public endpoint
  - Private endpoint
  - Service endpoint
  - VNet integration

- [ ] Povezati Container Apps Environment sa VNet-om kada dođe vreme

- [ ] Razmotriti privatni pristup:
  - Key Vault
  - Storage
  - SQL

- [ ] Razumeti DNS posledice Private Endpoint-a

---

## 23. Availability, resiliency i scaling

- [ ] Testirati HTTP-based scaling

- [ ] Testirati min/max replicas

- [ ] Testirati scale-to-zero gde ima smisla

- [ ] Razumeti retry / timeout / circuit breaker koncept

- [ ] Razumeti:
  - Availability Zones
  - Region pairs
  - RTO
  - RPO
  - Backup
  - Disaster Recovery

- [ ] Napraviti jednostavan AZ-305 DR scenario za arhitekturu laba

---

## 24. Infrastructure as Code

- [ ] Kada ručno postavljena arhitektura proradi, opisati je kroz IaC

- [ ] Izabrati Bicep kao primarni Azure IaC alat

- [ ] Kreirati:
  - Resource Group parametre
  - ACR
  - Container Apps Environment
  - Container Apps
  - Key Vault
  - Storage
  - monitoring resurse

- [ ] Napraviti parametre za DEV / UAT / PROD

- [ ] Razumeti:
  - Bicep
  - ARM
  - Terraform konceptualno

---

## 25. Ciljna arhitektura glavnog lab-a

```text
                         GitHub
                           |
                    GitHub Actions
                           |
                           v
                  Azure Container Registry
                           |
                           v
              Azure Container Apps Environment
                 /          |          \
                /           |           \
               v            v            v
        Customer API    Order API      Worker
             |              |             |
             |              v             |
             |          Service Bus -------+
             |
             +------> Azure SQL
             |
             +------> Storage Account
             |
             +------> Key Vault
             |
             +------> Application Insights

Authentication / Authorization:
Microsoft Entra ID

Access between Azure resources:
Managed Identity + RBAC
```

---

## 26. AZ-305 fokus tokom svakog koraka

Kod svakog servisa odgovoriti na pitanja:

- Zašto sam izabrao baš ovaj servis?
- Koja je alternativa?
- Kada bi alternativa bila bolja?
- Kako servis skalira?
- Kako se štiti?
- Kako se monitoriše?
- Kako rešava HA / DR?
- Koliko košta?
- Koje su glavne granice / ograničenja?
- Kako se uklapa u Well-Architected Framework?

---

## 27. Kasniji RAG lab

Ne raditi odmah.

- [ ] Blob Storage za dokumente

- [ ] Azure Function ili Container App worker za obradu dokumenata

- [ ] Chunking

- [ ] Embeddings

- [ ] Azure AI Search

- [ ] Azure OpenAI

- [ ] Retrieval

- [ ] Reranking

- [ ] LLM odgovor sa citatima

Ciljna arhitektura:

```text
Documents
   |
   v
Blob Storage
   |
   v
Function / Container App Worker
   |
   v
Chunking + Embeddings
   |
   v
Azure AI Search
   ^
   |
.NET API / Container App
   |
   v
Azure OpenAI
```

---

## 28. Napomena – šta ova checklista trenutno ne pokriva dovoljno od AZ-305 ispita

Ovaj praktični lab pokriva veliki deo AZ-305 tema kroz stvarnu implementaciju, ali ne pokriva kompletan ispit. Sledeće oblasti treba dodatno obraditi kroz posebne scenarije, poređenja i teorijsku pripremu:

### Governance i organizacija Azure okruženja

- Management Groups
- Subscription design
- Azure Policy
- Resource Locks
- Tagging strategija
- Governance na nivou više subscription-a
- Delegation i separation of duties

### Cost management i optimizacija

- Azure Reservations
- Savings Plans
- Cost Management + Billing
- TCO / cost comparison scenariji
- izbor odgovarajućeg SKU-a
- optimizacija troška između servisa
- trade-off između performansi, dostupnosti i cene

### Business continuity i disaster recovery

- Azure Backup
- Azure Site Recovery
- regionalni failover
- active-active vs active-passive
- geo-redundancy
- recovery planovi
- detaljnije RTO / RPO odluke
- backup retention strategije
- cross-region arhitekture

### Storage dizajn – dublje teme

- LRS
- ZRS
- GRS
- RA-GRS
- GZRS
- RA-GZRS
- Access tiers
- Lifecycle Management
- object replication
- izbor Storage servisa prema zahtevu
- performanse, redundancy i cost trade-off

### Database izbor i arhitektura

- Azure SQL Database
- Azure SQL Managed Instance
- SQL Server on Azure VM
- Azure Cosmos DB
- Azure Database for PostgreSQL
- izbor relacione vs NoSQL baze
- consistency modeli
- globalna distribucija podataka
- HA / DR opcije baza
- migration scenariji za baze

### Hybrid i enterprise networking

- VPN Gateway
- ExpressRoute
- Hub-Spoke topologija
- Virtual WAN
- peering strategije
- DNS arhitektura
- on-premises povezivanje
- network segmentation
- multi-region networking

### Load balancing i globalni routing

- Azure Front Door
- Application Gateway
- Azure Load Balancer
- Traffic Manager
- izbor odgovarajućeg servisa prema Layer 4 / Layer 7 i globalnim zahtevima
- WAF scenariji
- globalni failover
- session affinity i routing strategije

### Širi izbor compute platforme

- Azure Virtual Machines
- Virtual Machine Scale Sets
- Azure Functions
- App Service
- Azure Container Apps
- AKS
- izbor compute servisa prema workload-u
- serverless vs PaaS vs container orchestration vs IaaS

### Migration i modernization scenariji

- rehost
- refactor
- rearchitect
- rebuild
- Azure Migrate
- database migration
- workload modernization
- izbor ciljne Azure platforme prema postojećem sistemu

### Architecture decision practice

Pored praktičnog lab-a potrebno je dodatno raditi AZ-305 scenario pitanja gde je cilj da se izabere najbolji servis na osnovu zahteva kao što su:

- availability
- scalability
- security
- governance
- latency
- compliance
- cost
- RTO / RPO
- operational complexity

Ove oblasti ne treba nužno sve implementirati u privatnom Azure lab-u. Deo njih je efikasnije obraditi kroz arhitektonske scenarije i poređenje Azure servisa.

# Napredak

## Trenutno sam stigao do:

`Resource Group rg-az305-lab je kreiran; odlučeno je da glavni lab ide ka mikroservisnoj arhitekturi na Azure Container Apps.`

## Sledeći korak:

`Prebaciti očišćen .NET API na privatni GitHub repo, dodati Dockerfile i zatim kreirati Azure Container Registry.`

## Beleške

- App Service je već praktično obrađen kroz poslovni projekat.
- U ovom lab-u App Service ostaje važan za poređenje i AZ-305 scenario pitanja.
- Glavna compute platforma za praktični deo postaje Azure Container Apps.
- Fokus nije samo polaganje ispita, već i postavljanje funkcionalnog sistema koji može kasnije da se proširuje.

---

## Pitanja za proveru

- Kada izabrati App Service, a kada Azure Container Apps?
- Kada koristiti revisions, a kada deployment slots?
- Kada koristiti system-assigned, a kada user-assigned Managed Identity?
- Kada koristiti Service Bus, Storage Queue, Event Grid ili Event Hubs?
- Kada koristiti Private Endpoint, a kada Service Endpoint?
- Kako dizajnirati DEV / UAT / PROD izolaciju?
- Kako izvesti canary deployment i rollback?
