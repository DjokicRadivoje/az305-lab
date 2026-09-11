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

- [x] Koristiti odvojenu browser sesiju za privatni Azure lab
  - Koristi se **Private/Incognito** prozor umesto posebnog browser profila
  - Ne koristiti istu aktivnu sesiju kao za firmin Azure nalog

- [x] Izabrati privatni Microsoft nalog
  - Koristiti postojeći privatni Outlook/Hotmail nalog ili napraviti novi
  - Ne koristiti firminu email adresu
  - Uključiti MFA

- [x] Otvoriti Azure Free Account
  - Prijaviti se privatnim Microsoft nalogom
  - Završiti verifikaciju telefona i kartice
  - Proveriti da je aktiviran početni Azure kredit

---

## 2. Provera Azure okruženja

- [x] Otvoriti Azure Portal

- [x] Proveriti Directory / Tenant
  - Directory / tenant: `Default Directory`
  - Domain: `radivojedjokiccloudoutlook.onmicrosoft.com`

- [x] Proveriti Subscription
  - Naziv subscription-a: `AZ305-Personal-Lab`

- [x] Proveriti da privatni subscription nije povezan sa firminim tenant-om
  - Potvrđeno: prikazan je samo privatni `Default Directory` za ovaj lab

- [x] Otvoriti Cost Management
  - [x] Proveriti trenutno stanje kredita
  - [x] Proveriti da nema neočekivanih troškova

---

## 3. Kontrola troškova

- [x] Kreirati Budget

- [x] Postaviti okvirni mesečni budget
  - Naziv: `AZ305_Sept2026-Aug2028`
  - Iznos: `5 USD` mesečno
  - Period: septembar 2026 – avgust 2028

- [x] Postaviti upozorenja
  - [x] 20%
  - [x] 50%
  - [x] 100%
  - [x] 150%
  - Alert recipients: privatni email nalozi

- [x] Proveriti gde se vidi trenutna potrošnja

---

## 4. Resource Group

- [x] Kreirati Resource Group

- [x] Naziv:
  - `rg-az305-lab`

- [x] Region:
  - `West Europe`

- [x] Razumeti odnos:
  - Subscription
  - Resource Group
  - Resource

### Napomena o App Service Plan-u

- Prethodni lab je bio zaustavljen neposredno pre kreiranja **App Service Plan-a**.
- App Service Plan **nije kreiran**.
- Pošto je praktični pravac promenjen ka **Azure Container Apps** i mikroservisnoj arhitekturi, App Service Plan se u ovom lab-u **neće kreirati**.
- App Service i App Service Plan ostaju teme za AZ-305 poređenje i scenario pitanja.

---

## 5. GitHub i aplikacioni kod

- [x] Prebaciti demo / očišćenu verziju postojećeg `.NET 10 Web API` projekta na privatni GitHub nalog

- [x] Proveriti da repo ne sadrži:
  - secret-e
  - connection stringove
  - poslovne URL-ove
  - interne certifikate
  - produkcione podatke

- [x] Napraviti osnovne grane:
  - `develop`
  - `uat`
  - `main`

- [x] Proveriti lokalni build i testove

- [x] Proveriti Swagger / OpenAPI

- [x] Dodati Dockerfile

- [x] Lokalno napraviti container image
  - Image: `az305-template-service-api:1.0`

- [x] Lokalno pokrenuti container i proveriti API
  - Container: `az305-template-api`
  - Lokalni port: `8080`
  - Swagger potvrđen u `Development` okruženju

---

## 6. Azure Container Registry (ACR)

- [x] Kreirati Azure Container Registry
  - Registry: `acraz305lab`
  - Login server: `acraz305lab.azurecr.io`
  - Region: `North Europe`
  - Pricing plan: `Basic`
  - Public network access: enabled za početni lab
  - RBAC Registry Permissions

- [x] Razumeti:
  - Registry
  - Repository
  - Image
  - Tag
  - Digest

- [x] Push-ovati prvi image
  - `acraz305lab.azurecr.io/az305-template-service-api:1.0`

- [x] Proveriti image u ACR-u
  - Repository: `az305-template-service-api`
  - Tag: `1.0`
  - Digest potvrđen u ACR-u

- [x] Ne koristiti registry admin credentials ako nije potrebno
  - Login urađen preko Azure CLI (`az acr login`)

- [x] Omogućiti pristup preko Managed Identity + `AcrPull`
  - Container Apps Environment koristi system-assigned Managed Identity
  - Identity je dobio `AcrPull` nad registry-jem `acraz305lab`
  - Image je uspešno povučen bez registry admin username/password-a

---

## 7. Azure Container Apps Environment

- [x] Kreirati Azure Container Apps Environment
  - Environment: `cae-az305-lab`
  - Region: `North Europe`
  - Workload profile: `Consumption`

- [x] Razumeti da Container Apps ne koristi App Service Plan

- [x] Pregledati:
  - Environment
  - Container App
  - Revision
  - Replica
  - Ingress
  - Scaling

- [x] Povezati Log Analytics workspace
  - Workspace kreiran zajedno sa Container Apps Environment-om
  - Potvrđeno da se logovi vide u Azure Portalu

- [x] Razumeti odnos:
  - Container Apps Environment predstavlja zajedničko okruženje za više Container App resursa
  - jedan Environment može sadržati više mikroservisa / Container App aplikacija
  - trenutno `cae-az305-lab` sadrži `az305-template-api`

### Koncepti potvrđeni u praksi

- **Revision** = verzija konfiguracije/deployment-a jednog Container App-a
- **Replica** = konkretna pokrenuta instanca određene revision verzije
- **Ingress** = kontroliše kako saobraćaj ulazi u Container App
  - external ingress za javno dostupne API-je
  - internal ingress za servise dostupne samo unutar Container Apps Environment-a
  - worker može raditi bez ingress-a
- **Scaling** = automatsko povećavanje/smanjivanje broja replica; kod Consumption modela moguće je `minReplicas = 0` i scale-to-zero

---

## 8. Prvi Azure Container App

- [x] Kreirati prvi Container App za API
  - Container App: `az305-template-api`

- [x] Povući image iz ACR-a
  - Registry: `acraz305lab.azurecr.io`
  - Repository/image: `az305-template-service-api`
  - Tag: `1.0`
  - pristup preko Managed Identity + `AcrPull`

- [x] Podesiti:
  - CPU: `0.5`
  - Memory: `1 GiB`
  - target port: `8080`
  - external ingress: enabled / accepting traffic from anywhere
  - `ASPNETCORE_ENVIRONMENT=Development` radi Swagger-a u lab okruženju

- [x] Proveriti javni endpoint
  - javni HTTPS endpoint radi

- [ ] Proveriti Swagger / health endpoint
  - [x] Swagger javno dostupan i potvrđen
  - [ ] Health endpoint proveriti / dodati ako je potrebno

- [x] Pregledati logove
  - logovi su dostupni kroz Container Apps Environment / Log Analytics

- [x] Razumeti razliku:
  - **App Service** = PaaS za direktno hostovanje aplikacionog koda/runtime-a; koristi App Service Plan
  - **App Service for Containers** = App Service model, ali kao deployment artifact koristi Docker/container image; i dalje koristi App Service Plan
  - **Azure Container Apps** = container-native PaaS za mikroservise i event-driven workload-e; koristi Container Apps Environment, revisions, replicas, ingress i autoscaling/scale-to-zero; ne koristi App Service Plan

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

`Sekcije 1–7 su završene. U sekciji 8 prvi Azure Container App je uspešno podignut i javno dostupan. Razumljena je razlika između App Service, App Service for Containers i Azure Container Apps. Jedino je health endpoint ostavljen kao opcioni/proverni korak. Sledeća velika praktična tema je revision management i traffic splitting.`

## Sledeći korak:

`Po povratku sa odmora prva tačka je ponovno kreiranje Azure Container Registry-ja acraz305lab ako je pre odmora obrisan radi uštede troška. Zatim ponovo build/tag/push image-a az305-template-service-api:1.0 i provera Managed Identity + AcrPull veze. Posle kratke obnove nastaviti sa health endpoint-om po potrebi i sekcijom 10: revision management, multiple revisions mode, traffic splitting, canary/blue-green i rollback.`

## Beleške

- App Service je već praktično obrađen kroz poslovni projekat.
- App Service Plan nije kreiran i nije potreban za Azure Container Apps tok.
- U ovom lab-u App Service ostaje važan za poređenje i AZ-305 scenario pitanja.
- Glavna compute platforma za praktični deo postaje Azure Container Apps.
- ACR je kreiran u `North Europe` zato što `West Europe` trenutno nije prihvatao nove ACR korisnike za ovaj subscription.
- Za pristup ACR-u nije korišćen registry admin user; prvo je korišćen Azure CLI login, a Container Apps sada koristi Managed Identity + `AcrPull`.
- Container App koristi external ingress i javni HTTPS endpoint; interni servisi će kasnije koristiti internal ingress ili rad bez ingress-a, zavisno od namene.
- `minReplicas = 0` omogućava scale-to-zero u Consumption modelu.
- Cost Management trenutno pokazuje mali trošak; dogovoreno je da se nakon jednog dana proveri realan trošak ACR-a, Container Apps-a i Log Analytics-a.
- ACR može biti obrisan pre odmora radi izbegavanja baseline troška. Source code i Dockerfile ostaju u GitHub-u, pa se image može ponovo izgraditi i push-ovati po povratku.
- Fokus nije samo polaganje ispita, već i postavljanje funkcionalnog sistema koji može kasnije da se proširuje.

---

# Azure objekti kreirani do ove tačke

- **Subscription:** `AZ305-Personal-Lab`
- **Glavni Resource Group:** `rg-az305-lab`
  - region RG-a: `West Europe`
- **Dodatni Resource Group:** `DefaultResourceGroup-NEU`
  - region: `North Europe`
  - trenutno prazan
  - pojavio se automatski tokom rada u North Europe; tačno poreklo nije potvrđeno i nema funkcionalnu ulogu u trenutnom lab-u
- **Azure Container Registry:** `acraz305lab`
  - region: `North Europe`
  - SKU: `Basic`
  - login server: `acraz305lab.azurecr.io`
  - repository: `az305-template-service-api`
  - tag: `1.0`
  - napomena: može biti obrisan pre odmora radi uštede; po povratku je njegovo ponovno kreiranje prva tačka
- **Azure Container Apps Environment:** `cae-az305-lab`
  - region: `North Europe`
  - workload profile: `Consumption`
  - system-assigned Managed Identity
  - `AcrPull` pristup prema `acraz305lab`
- **Log Analytics Workspace:** automatski kreiran uz Container Apps Environment
  - koristi se za Container Apps logove
  - potvrđeno da logovi stižu i vide se u Azure Portalu
- **Container App:** `az305-template-api`
  - image: `acraz305lab.azurecr.io/az305-template-service-api:1.0`
  - CPU: `0.5`
  - memorija: `1 GiB`
  - target port: `8080`
  - external HTTP ingress
  - javni HTTPS endpoint i Swagger rade
  - `ASPNETCORE_ENVIRONMENT=Development` za trenutni lab
  - `minReplicas = 0`
- **Budget:** `AZ305_Sept2026-Aug2028`
  - `5 USD` mesečno
  - upozorenja: 20%, 50%, 100%, 150%

---

# Podsetnik / obnova posle odmora

Planirana pauza: oko **9 dana**. Pre nastavka ne kretati odmah na novu temu; prvo uraditi kratku obnovu da se ponovo uspostavi ceo mentalni model.

## Prva tačka po povratku

1. **Ponovo kreirati Azure Container Registry `acraz305lab`** ako je pre odmora obrisan radi uštede troška.
   - Region: `North Europe`
   - SKU: `Basic`
   - Public network access: enabled za početni lab
   - Registry permissions: RBAC
   - zatim ponovo uraditi build/tag/push image-a `az305-template-service-api:1.0`
   - proveriti da Container Apps Environment Managed Identity ponovo ima potreban `AcrPull` pristup

## Brza obnova – 15 do 30 minuta

2. Otvoriti `rg-az305-lab` i pregledati resurse koji postoje.
3. Proveriti Cost Analysis i koliko su do tada koštali ACR, Container Apps i Log Analytics.
4. Ponoviti Docker tok:

```text
Dockerfile
  -> docker build
  -> image
  -> docker tag
  -> docker push
  -> ACR
  -> Container App pull
  -> replica
```

5. Ponoviti ACR pojmove:
   - Registry
   - Repository
   - Image
   - Tag
   - Digest
6. Ponoviti Container Apps hijerarhiju:

```text
Container Apps Environment
└── Container App
    └── Revision
        └── 0..N Replica
```

7. Ponoviti:
   - external ingress = javni pristup
   - internal ingress = pristup unutar Container Apps Environment-a
   - no ingress = npr. worker koji ne prima HTTP zahteve
   - scaling = promena broja replica
   - `minReplicas = 0` = scale-to-zero
8. Otvoriti javni Swagger `az305-template-api` i potvrditi da aplikacija i dalje radi.
9. Ponoviti razliku:
   - App Service
   - App Service for Containers
   - Azure Container Apps
10. Podsetiti se bezbednosnog toka:
   - ACR admin credentials nisu korišćeni
   - Managed Identity + `AcrPull` omogućava Container Apps platformi da povuče image
11. Tek nakon obnove nastaviti sa **revision management + traffic splitting**.

## Mini pitanja za povratak u temu

- Koja je razlika između image-a i container-a?
- Zašto ACR nije runtime servis?
- Šta je digest i po čemu se razlikuje od taga?
- Šta je Container Apps Environment, a šta Container App?
- Šta je Revision, a šta Replica?
- Kada koristiti external, internal ili bez ingress-a?
- Kako scale-to-zero utiče na broj replica i trošak?
- Zašto Container App i dalje treba pristup ACR-u?
- Koja je razlika između App Service for Containers i Azure Container Apps?
- Kako bi izgledao javni API + interni API + worker u istom Environment-u?

---

## Pitanja za proveru

- Kada izabrati App Service, a kada Azure Container Apps?
- Kada koristiti revisions, a kada deployment slots?
- Kada koristiti system-assigned, a kada user-assigned Managed Identity?
- Kada koristiti Service Bus, Storage Queue, Event Grid ili Event Hubs?
- Kada koristiti Private Endpoint, a kada Service Endpoint?
- Kako dizajnirati DEV / UAT / PROD izolaciju?
- Kako izvesti canary deployment i rollback?
