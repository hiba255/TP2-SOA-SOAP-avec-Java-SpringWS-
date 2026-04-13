# SOAP — soap-bank

> Service web SOAP simulant un système bancaire minimal, développé avec **Spring Boot** et **Spring-WS** selon l'approche **contract-first**.

---

##  Objectifs du TP

- Comprendre le principe **contract-first** en SOAP (XSD → WSDL → stubs/objets)
- Déployer et exécuter un service SOAP avec Spring Boot + Spring-WS
- Tester des opérations SOAP avec **Postman** (requêtes, réponses, erreurs Fault)
- Observer l'impact du contrat (XSD) sur les messages échangés
- Enrichir le service avec une nouvelle opération en respectant l'approche contract-first

---

##  Prérequis

| Outil | Version minimale |
|-------|-----------------|
| Java JDK | 17+ |
| Maven | 3.8+ |
| Postman | Desktop (dernière version) |
| Git | Toute version récente |

---

##  Installation et lancement

### 1. Forker le dépôt

Forker le dépôt GitLab suivant sur votre compte :

```
https://gitlab.com/tps-soa-microservices/soap-bank
```

### 2. Cloner votre fork

```bash
git clone https://gitlab.com/<votre-username>/soap-bank.git
cd soap-bank
```

### 3. Compiler le projet

```bash
mvn clean install
```

> **Note :** La phase de génération de code (via `jaxb2-maven-plugin`) s'exécute automatiquement à la compilation. Elle lit le fichier XSD et génère les classes Java correspondantes dans `target/generated-sources/`.

### 4. Lancer l'application

```bash
mvn spring-boot:run
```

L'application démarre sur le port **8080** par défaut.

### 5. Vérifier l'accès au WSDL

Ouvrir un navigateur et accéder à :

```
http://localhost:8080/ws/bank.wsdl
```

Le WSDL doit s'afficher correctement — cela confirme que le service est bien démarré.

---

##  Architecture du projet (contract-first)

```
soap-bank/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── ...endpoint/        ← Endpoint Spring-WS (logique métier)
│   │   └── resources/
│   │       └── bank.xsd            ← Contrat XSD (source de vérité)
│   └── test/
│       └── ...
├── pom.xml                         ← Config Maven + plugin JAXB
└── README.md
```

### Flux contract-first

```
bank.xsd  →  JAXB (mvn generate-sources)  →  Classes Java générées
    ↓
Spring-WS génère automatiquement le WSDL
    ↓
Endpoint implémente la logique métier
```

---

## Lecture du contrat (XSD & WSDL)

### Fichier XSD (`src/main/resources/bank.xsd`)

Le XSD est la **source de vérité** du service. Il définit :
- La structure de chaque message (requête et réponse)
- Les types de données et leurs contraintes (types primitifs, restrictions)
- Le namespace du service

Les classes Java sont **générées automatiquement** à partir de ce fichier lors de la compilation — toute modification du contrat nécessite une recompilation.

### Opérations disponibles

| Opération | Requête | Réponse |
|-----------|---------|---------|
| `GetAccount` | `accountId` (String) | `owner`, `balance`, `currency` |
| `Deposit` | `accountId` (String), `amount` (Decimal) | `newBalance` |
| `Withdraw` *(ajouté)* | `accountId` (String), `amount` (Decimal) | `newBalance` |

### Éléments WSDL importants

| Élément | Valeur |
|---------|--------|
| **Namespace** | `http://example.com/bank` |
| **portType** | `BankPort` |
| **Endpoint (URL)** | `http://localhost:8080/ws` |
| **Binding** | SOAP 1.1 / document-literal |

---

## Tests Postman

### Configuration

- **Méthode :** `POST`
- **URL :** `http://localhost:8080/ws`
- **Header :** `Content-Type: text/xml; charset=utf-8`

---

### Requête `GetAccount`

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:bank="http://example.com/bank">
  <soapenv:Header/>
  <soapenv:Body>
    <bank:getAccountRequest>
      <bank:accountId>A100</bank:accountId>
    </bank:getAccountRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

**Réponse attendue :**
```xml
<SOAP-ENV:Envelope ...>
  <SOAP-ENV:Body>
    <ns2:getAccountResponse ...>
      <ns2:owner>Alice</ns2:owner>
      <ns2:balance>1000.00</ns2:balance>
      <ns2:currency>EUR</ns2:currency>
    </ns2:getAccountResponse>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

---

### Requête `Deposit`

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:bank="http://example.com/bank">
  <soapenv:Header/>
  <soapenv:Body>
    <bank:depositRequest>
      <bank:accountId>A100</bank:accountId>
      <bank:amount>20.00</bank:amount>
    </bank:depositRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

**Réponse attendue :**
```xml
<SOAP-ENV:Envelope ...>
  <SOAP-ENV:Body>
    <ns2:depositResponse ...>
      <ns2:newBalance>1020.00</ns2:newBalance>
    </ns2:depositResponse>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

---

### Requête `Withdraw` *(opération ajoutée)*

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:bank="http://example.com/bank">
  <soapenv:Header/>
  <soapenv:Body>
    <bank:withdrawRequest>
      <bank:accountId>A100</bank:accountId>
      <bank:amount>50.00</bank:amount>
    </bank:withdrawRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

---

### Cas d'erreur — SOAP Fault

**Exemple : montant négatif**
```xml
<soapenv:Envelope ...>
  <soapenv:Body>
    <bank:depositRequest>
      <bank:accountId>A100</bank:accountId>
      <bank:amount>-10.00</bank:amount>
    </bank:depositRequest>
  </soapenv:Body>
</soapenv:Envelope>
```

**Réponse SOAP Fault :**
```xml
<SOAP-ENV:Envelope ...>
  <SOAP-ENV:Body>
    <SOAP-ENV:Fault>
      <faultcode>SOAP-ENV:Server</faultcode>
      <faultstring>Le montant doit être positif</faultstring>
    </SOAP-ENV:Fault>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

> D'autres cas d'erreur possibles : compte inexistant (`ACCOUNT_NOT_FOUND`), solde insuffisant lors d'un retrait (`INSUFFICIENT_FUNDS`).

---

##  Fonctionnalité ajoutée : `Withdraw`

### Description

L'opération `Withdraw` permet d'effectuer un **retrait** sur un compte bancaire existant. Elle vérifie que :
- Le compte existe
- Le montant est strictement positif
- Le solde est suffisant

### Fichiers modifiés

| Fichier | Modification |
|---------|-------------|
| `src/main/resources/bank.xsd` | Ajout de `withdrawRequest` et `withdrawResponse` |
| `BankEndpoint.java` | Ajout de la méthode `withdraw()` annotée `@PayloadRoot` |
| `BankService.java` | Implémentation de la logique de retrait avec validation |

### Éléments XSD ajoutés

```xml
<xs:element name="withdrawRequest">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="accountId" type="xs:string"/>
      <xs:element name="amount"    type="xs:decimal"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>

<xs:element name="withdrawResponse">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="newBalance" type="xs:decimal"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```

---

##  Livrables

```
livrables/
├── captures/
│   ├── getaccount_ok.png          ← GetAccount A100 — réponse OK
│   ├── deposit_ok.png             ← Deposit A100 +20.00 — nouveau solde
│   ├── fault_montant_negatif.png  ← Deposit montant négatif — SOAP Fault
│   ├── withdraw_ok.png            ← Withdraw A100 -50.00 — nouveau solde
│   └── fault_solde_insuffisant.png← Withdraw trop élevé — SOAP Fault
├── postman_collection.json        ← Export collection Postman
└── REPONSES.md                    ← Réponses à la partie Analyse
```

---

## Auteur

| Champ | Valeur |
|-------|--------|
| Nom | *[Votre nom]* |
| Groupe | *[Votre groupe]* |
| Dépôt GitLab | *[URL de votre fork]* |

---

##  Ressources

- [Documentation Spring-WS](https://docs.spring.io/spring-ws/docs/current/reference/)
- [W3C — SOAP 1.1](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/)
- [Tutoriel JAXB](https://jakarta.ee/specifications/xml-binding/)
- [Postman — SOAP requests](https://learning.postman.com/docs/sending-requests/soap/making-soap-requests/)
