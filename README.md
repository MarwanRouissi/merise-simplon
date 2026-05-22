# Sujet Groupe 2 — Gestion d'une compagnie aérienne
**Modélisation des données · Merise · Restitution collective**

## Marwan - Sebastien - Séverine - Walid

---

## Consignes générales

Vous travaillez en groupe de 4. À partir du cahier des charges ci-dessous, vous devez produire et présenter :

1. Le **dictionnaire des données** (toutes les informations identifiées, qualifiées)
2. Le **MCD complet** (entités, associations, cardinalités, associations ternaires si pertinentes, CIF, héritage avec contraintes, historisation si pertinente)
3. Le **MLD** (notation textuelle complète : clés primaires soulignées, clés étrangères précédées de #)
4. Une **analyse de normalisation** : vérifier que votre MLD est en 3FN/BCNF, identifier d'éventuelles violations de 4FN
5. Un **extrait MPD** : 3 à 5 tables en SQL DDL avec contraintes

**Outil de modélisation** : vous choisissez entre Looping, JMerise ou Mocodo.

**Restitution** : 15 minutes de présentation + 5 minutes de questions de la promo et du formateur.

---

## Contexte

La compagnie **AirLuc** opère des vols entre aéroports européens et souhaite refondre son système d'information.

Un **vol** est identifié par un numéro de vol (ex : AL205). Il relie un **aéroport de départ** à un **aéroport d'arrivée**. Un vol a une date et heure de départ prévues, une date et heure d'arrivée prévues, et un statut (programmé, en cours, atterri, annulé). Un même numéro de vol peut être opéré plusieurs jours différents — on considère que chaque opération d'un vol à une date donnée constitue une **rotation** distincte, identifiée par le numéro de vol et la date.

Les **aéroports** sont identifiés par leur code IATA (3 lettres, ex : CDG, LYS, MAD). On enregistre leur nom complet, leur ville et leur pays.

Chaque rotation est effectuée par exactement un **avion**. Un avion est identifié par son immatriculation. Il possède un modèle (ex : A320, B737), une capacité totale en sièges et une année de mise en service. Chaque avion appartient à un **type d'avion** (ex : monocouloir court-courrier, long-courrier gros-porteur). Un type d'avion a un code, un constructeur, un modèle générique et une capacité maximale théorique.

L'**équipage** d'une rotation est composé de **membres d'équipage**. Chaque membre est identifié par son matricule et possède un nom, un prénom, une date de naissance et une nationalité. Il existe deux catégories de membres : les **pilotes** et le **personnel de cabine**. Un pilote possède un numéro de licence, un nombre total d'heures de vol et une liste de **qualifications** sur des types d'avions (un pilote doit être qualifié sur le type d'avion de la rotation qu'il assure). Le personnel de cabine possède un niveau de certification sécurité et parle une ou plusieurs **langues** (pour l'accueil des passagers).

Un pilote ne peut pas être en même temps personnel de cabine. En revanche, un membre d'équipage peut changer de catégorie au cours de sa carrière — on ne souhaite pas historiser ce changement, seule la catégorie actuelle est mémorisée.

Les **passagers** sont identifiés par un numéro de passager. On enregistre leur nom, prénom, date de naissance et nationalité. Un passager peut effectuer des **réservations**. Une réservation concerne un passager sur une rotation précise. Elle a un numéro de réservation unique, une date de réservation, une classe de voyage (économique, affaires, première) et un prix payé. Un passager ne peut avoir qu'une seule réservation par rotation.

Certaines rotations comportent des **escales** intermédiaires. Une escale est un arrêt dans un aéroport entre le départ et l'arrivée finale. Pour chaque escale, on enregistre l'heure d'arrivée à l'aéroport d'escale, l'heure de départ et la durée au sol.

---

## Règles de gestion récapitulatives

1. Un vol est identifié par son numéro ; une rotation est identifiée par le numéro de vol et la date.
2. Chaque rotation est effectuée par exactement un avion.
3. Chaque avion est d'un seul type d'avion.
4. Un pilote doit être qualifié sur le type de l'avion qu'il pilote.
5. Un pilote peut être qualifié sur plusieurs types d'avions.
6. Le personnel de cabine parle une ou plusieurs langues (données indépendantes des affectations).
7. Tout membre d'équipage est soit pilote, soit personnel de cabine — jamais les deux simultanément.
8. Un passager ne peut avoir qu'une seule réservation par rotation.
9. Une rotation peut comporter zéro ou plusieurs escales dans des aéroports intermédiaires.
10. On ne souhaite pas historiser les affectations des membres d'équipage aux rotations passées — seules les rotations à venir et en cours sont gérées.

---

## Points de réflexion

- Un vol relie deux aéroports (départ et arrivée). Comment représenter ces **deux rôles distincts** du même aéroport dans le MCD ?
- La relation entre rotation, avion et équipage : peut-on la modéliser avec des associations binaires ? Ou faut-il une ternaire ? Quelle CIF peut-on identifier ?
- Les **qualifications** d'un pilote sur des types d'avions et les **langues** parlées par le personnel de cabine sont deux ensembles de valeurs multiples indépendants. Que se passe-t-il si on les met dans la même table ? Quelle forme normale est concernée ?
- L'escale implique un aéroport intermédiaire pour une rotation donnée. Comment modéliser l'**ordre des escales** si on en a plusieurs ?
- Votre MLD présente-t-il des dépendances transitives ? Vérifiez notamment les tables liées aux types d'avions et aux modèles.

## DICTIONNAIRE DE DONNEES : 

## NATIONALITE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | CHAR | 3 | ✓ | | ✓ | Code ISO-3166 (FRA, ESP, DEU) |
| libelle | VARCHAR | 50 | | | ✓ | Nom de la nationalité |

## PAYS
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | CHAR | 2 | ✓ | | ✓ | Code ISO (FR, ES, DE) |
| nom | VARCHAR | 100 | | | ✓ | Nom du pays |

## VILLE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | INT | | ✓ | | ✓ | Identifiant unique |
| nom | VARCHAR | 100 | | | ✓ | Nom de la ville |

## LANGUE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | INT | | ✓ | | ✓ | Identifiant unique |
| nom | VARCHAR | 50 | | | ✓ | Nom de la langue |

## CONSTRUCTEUR
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | INT | | ✓ | | ✓ | Identifiant unique |
| nom | VARCHAR | 100 | | | ✓ | Nom du constructeur (Airbus, Boeing) |

## MODELE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | INT | | ✓ | | ✓ | Identifiant unique |
| nom | VARCHAR | 50 | | | ✓ | Nom du modèle (A320, B737) |


## TYPE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| code | VARCHAR | 10 | ✓ | | ✓ | Code du type (A320-200, B737-800) |
| libelle | VARCHAR | 100 | | | ✓ | Description (Monocouloir court-courrier) |
| capacite_max_theorique | INT | | | | ✓ | Capacité maximale en sièges |


## STATUT
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | INT | | ✓ | | ✓ | Identifiant unique |
| libelle | VARCHAR | 50 | | | ✓ | État (Programmé, En cours, Atterri, Annulé) |

## CLASSE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | INT | | ✓ | | ✓ | Identifiant unique |
| libelle | VARCHAR | 50 | | | ✓ | Classe (Économique, Affaires, Première) |

## AÉROPORT
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| IATA | CHAR | 3 | ✓ | | ✓ | Code IATA (CDG, LYS, MAD) |
| nom | VARCHAR | 100 | | | ✓ | Nom de l'aéroport |

## VOL
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| numero | VARCHAR | 10 | ✓ | | ✓ | Numéro du vol (AL205) |
| depart_prevu | DATETIME | | | | ✓ | Date/heure départ prévu |
| arrivee_prevu | DATETIME | | | | ✓ | Date/heure arrivée prévu |

## ROTATION
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | INT | | ✓ | | ✓ | Identifiant unique |
| date | DATE | | | | ✓ | Date de la rotation |

## AVION
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| immatriculation | VARCHAR | 10 | ✓ | | ✓ | Immatriculation unique (F-GKAD) |
| capacite_sieges | INT | | | | ✓ | Nombre de sièges |
| annee_mise_en_service | YEAR | 4 | | | ✓ | Année de mise en service |

## ESCALE
| Attribut | Type | Taille | PK | FK | NN | Description |Calcul|
|----------|------|--------|----|----|----|----|----|
| arrivee | TIMESTAMP | | | | ✓ | Heure d'arrivée |
| depart | TIMESTAMP | | | | ✓ | Heure de départ |
| duree_au_sol | INT | | | | | Durée au sol (optionnel) | départ - arrivee

## MEMBRE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| matricule | VARCHAR | 20 | ✓ | | ✓ | Identifiant unique (EMP001234) |
| nom | VARCHAR | 100 | | | ✓ | Nom |
| prenom | VARCHAR | 100 | | | ✓ | Prénom |
| date naissance | DATE | | | | ✓ | Date de naissance |

## PILOTE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| numero licence | VARCHAR | 30 | | | ✓ | Numéro de licence |
| total_heures_vol | INT | | | | ✓ | Total d'heures de vol |

## PERSONNEL_CABINE
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| niveau certification | VARCHAR | 50 | | | ✓ | Niveau de certification (CCA, FDR) |

## PASSAGER
| Attribut | Type | Taille | PK | FK | NN | Description |
|----------|------|--------|----|----|----|----|
| id | INT | | ✓ | | ✓ | Identifiant unique |
| nom | VARCHAR | 100 | | | ✓ | Nom |
| prenom | VARCHAR | 100 | | | ✓ | Prénom |
| date de naissance | DATE | | | | ✓ | Date de naissance |

## RÉSERVATION
| Attribut | Type | Taille | PK | FK | NN | Description |Calcul|
|----------|------|--------|----|----|----|----|----|
| numero | VARCHAR | 20 | ✓ | | ✓ | Numéro unique (RES2025001234) |
| date_reservation | DATE | | | | ✓ | Date de réservation |
| prix_paye | DECIMAL | 10,2 | | | ✓ | Prix payé (150.50) | Inconnu ?


## MCD : 

<img width="1565" height="821" alt="image" src="https://github.com/user-attachments/assets/0c935348-d7bd-40d1-b450-d62ea40fe8e2" />

## MLD :

<img width="1505" height="827" alt="MLD_Aeroport" src="https://github.com/user-attachments/assets/52890c12-6a68-4576-b570-888f6f93fd2b" />

## MPD : 

**NATIONALITE** : <u>id</u> libelle
**ETRE_MEMBRE_NATIONALITE** : <u>#matricule</u> <u>#id_nationalite</u>
**ETRE_PASSAGER_NATIONALITE** : <u>#id_passager</u> <u>#id_nationalite</u>

**PAYS** : <u>id</u> nom
**VILLE** : <u>id</u> nom #id_pays
**AÉROPORT** : <u>IATA</u> nom #id_ville

**MEMBRE** : <u>matricule</u> nom prenom date_naissance type_membre
**AFFECTER** : <u>#matricule</u> <u>#id_rotation</u>

**PERSONNEL_CABINE** : <u>matricule</u> niveau_certification
**PARLER** : <u>#matricule</u> <u>#id_langue</u>
**LANGUE** : <u>id</u> nom

**PILOTE** : <u>matricule</u> numero_licence total_heures_vol
**QUALIFIER** : <u>#matricule</u> <u>#code_type</u>


**CONSTRUCTEUR** : <u>id</u> nom
**MODELE** : <u>id</u> nom #id_constructeur
**TYPE** : <u>code</u> libelle capacite max theorique #id_modele
**AVION** : <u>immatriculation</u> capacite_sieges annee_mise_en_service #code_type


**PASSAGER** : <u>id</u> nom prenom date_naissance #id_nationalite
**RESERVATION** : <u>numero</u> date #id_passager #id_rotation #id_classe
**CLASSE** : <u>id</u> libelle
**ROTATION** : <u>id</u> date #numero_vol #immatriculation_avion


**VOL** : <u>numero</u> #IATA_depart #IATA_arrivee depart_prevu arrivee_prevu #id_statut
**STATUT** : <u>id</u> libelle


**FAIRE ESCALE** : #id_rotation #IATA_aeroport arrivee depart

## 1. Rappel des formes normales
 
### 1NF (Première Forme Normale)
- Tous les attributs contiennent des valeurs atomiques (non décomposables)
- Aucun attribut multi-valué
### 2NF (Deuxième Forme Normale)
- Respecte 1NF
- Tous les attributs non-clés dépendent de la clé primaire complète (pas de dépendance partielle)
### 3NF (Troisième Forme Normale)
- Respecte 2NF
- Pas de dépendance fonctionnelle transitive (un attribut non-clé ne dépend que de la clé, pas d'un autre attribut non-clé)
### BCNF (Boyce-Codd Normal Form)
- Forme plus stricte que 3NF
- Chaque déterminant est une clé candidate
### 4FN (Quatrième Forme Normale)
- Respecte BCNF
- Pas de dépendances multivaluées non-triviales indépendantes

