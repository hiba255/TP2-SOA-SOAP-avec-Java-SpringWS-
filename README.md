Reponses

```

#Service SOAP avec Java (Spring-WS)



---

# Partie A : Mise en place du projet

## A.1 Fork et clonage du dépôt

Le projet a été forké sur mon espace Git puis cloné en local.

Commandes utilisées :

```bash
git clone https://gitlab.com/hiba225/soap-bankk.git
cd soap-bankk


Le projet a ensuite été ouvert dans l’IDE ( VS Code).

---

## A.2 Compilation et démarrage

Compilation du projet :

```bash
mvn clean package


Lancement de l’application :

```bash
mvn spring-boot:run


Résultat :

- Build réussi
- Démarrage du serveur embarqué Tomcat
- Application accessible sur le port 8080

---

## A.3 Vérification du WSDL

URL testée :


http://localhost:8080/ws/bank.wsdl


Constat :

- Le WSDL est généré automatiquement par Spring-WS
- Le document est accessible depuis le navigateur
- La structure correspond au contrat défini dans le XSD

---

# Partie B : Analyse du contrat SOAP

## B.1 Fichier XSD et rôle

Fichier identifié :


src/main/resources/bank.xsd


### Rôle du XSD

Le fichier XSD définit le contrat du service web selon l’approche **contract-first**.

Il permet de :

- Définir la structure des messages SOAP
- Spécifier les types simples et complexes
- Fixer le namespace du service
- Générer automatiquement les classes Java via JAXB

### Chaîne de génération


XSD → Génération JAXB → Implémentation Java → WSDL généré automatiquement


---

## B.2 Structure des messages

### 1. AccountType (type complexe)

| Champ      | Type         |
|------------|-------------|
| accountId  | xsd:string  |
| owner      | xsd:string  |
| balance    | xsd:decimal |
| currency   | xsd:string  |

---

### 2. GetAccountRequest

- accountId : xsd:string

### 3. GetAccountResponse

- account : AccountType

---

### 4. DepositRequest

- accountId : xsd:string  
- amount : xsd:decimal  

### 5. DepositResponse

- newBalance : xsd:decimal  

---

## B.3 Analyse du WSDL

### Namespace


http://example.com/bank


Permet d’identifier de manière unique le service.

---

### PortType

Nom : `BankPort`

Opérations disponibles :

- GetAccount
- Deposit

---

### Binding

- Version : SOAP 1.1  
- Style : document  
- Encoding : literal  
- Transport : HTTP  

---

### Endpoint


http://localhost:8080/ws


Un point d’entrée unique pour toutes les opérations.

---

# Architecture du projet

## Fichiers principaux

### bank.xsd
Définit la structure des messages SOAP.

### WebServiceConfig.java
- Configuration Spring-WS  
- Enregistrement du servlet SOAP  
- Génération automatique du WSDL  

### BankEndpoint.java
- Exposition des opérations SOAP  
- Utilisation de l’annotation `@PayloadRoot`

### BankService.java
- Implémentation de la logique métier  
- Gestion des comptes en mémoire  

---

## Classes générées automatiquement

Situées dans :


target/generated-sources/jaxb/


Exemples :

- GetAccountRequest
- GetAccountResponse
- DepositRequest
- DepositResponse
- AccountType

---

# Comptes de test

| Account ID | Owner | Balance | Currency |
|------------|--------|----------|----------|
| A100 | Alice | 150.00 | TND |
| B200 | Bob | 80.50 | TND |

---

# Partie C : Tests avec Postman

## Configuration

- Method : POST  
- URL : http://localhost:8080/ws  
- Header : Content-Type: text/xml; charset=utf-8  
- Body : raw (XML)  

---

## C.1 Test GetAccount (A100)

Résultat :

- Les informations du compte sont correctement retournées  
- Structure conforme au XSD  
- Données cohérentes  

---

## C.2 Test Deposit (20.00 sur A100)

Résultat :

- Nouveau solde : 170.00  
- Élément newBalance présent dans la réponse  
- Traitement correct côté service  

---

## C.3 Tests SOAP Fault

### Cas 1 : Montant négatif

- faultcode : Client  
- faultstring : Amount must be > 0  

---

### Cas 2 : Compte inexistant

- faultcode : Client  
- faultstring : Unknown accountId  

Les exceptions Java sont automatiquement transformées en SOAP Fault par Spring-WS.

---

# Partie D : Ajout d’une nouvelle fonctionnalité

## Fonctionnalité choisie : Withdraw

Permet de retirer un montant d’un compte bancaire.

---

## D.1 Modification du XSD

Ajout des éléments :

- WithdrawRequest  
  - accountId  
  - amount  

- WithdrawResponse  
  - newBalance  

Après modification :

```bash
mvn clean package


Classes générées automatiquement :

- WithdrawRequest.java  
- WithdrawResponse.java  

---

## D.2 Implémentation

### Dans BankService

Ajout de la méthode `withdraw()` :

- Validation du montant (> 0)  
- Vérification de l’existence du compte  
- Vérification du solde suffisant  
- Mise à jour du solde  
- Retour du nouveau solde  

Création d’une exception personnalisée :

- InsufficientBalanceException  

---

### Dans BankEndpoint

Ajout de la méthode :

```java
@PayloadRoot(namespace = NAMESPACE_URI, localPart = "WithdrawRequest")


Cette méthode appelle le service métier et retourne un objet `WithdrawResponse`.

---

## D.3 Tests

### Cas valide

- Retrait de 10.00 sur A100  
- Nouveau solde correct  

### Cas erreur

- Montant supérieur au solde → SOAP Fault  
- Montant négatif → SOAP Fault  

---

## Vérification finale du WSDL

Le WSDL contient désormais 3 opérations :

- GetAccount  
- Deposit  
- Withdraw  

Accessible via :


http://localhost:8080/ws/bank.wsdl


---

# Conclusion

Ce TP m’a permis de :

- Comprendre l’approche contract-first  
- Utiliser JAXB pour générer automatiquement les classes Java  
- Implémenter un service SOAP avec Spring-WS  
- Gérer les erreurs via SOAP Fault  
- Tester un service SOAP avec Postman  
- Étendre le service en respectant le contrat défini  


```

