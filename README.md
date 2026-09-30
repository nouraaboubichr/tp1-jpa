# Gestion Salles — TP JPA/Hibernate

Projet Maven de démonstration JPA/Hibernate avec base H2 en mémoire.
Il implémente une architecture en couches (entités, services CRUD génériques) pour gérer des **Utilisateurs** et des **Salles**, avec validations (Bean Validation).

## Objectifs du TP

- Créer un projet Maven
- Configurer Hibernate avec une base de données H2
- Créer les entités `Salle` et `Utilisateur` avec validations (`@NotBlank`, `@Email`, `@Min`, etc.)
- Configurer la génération automatique du schéma (`hibernate.hbm2ddl.auto`)
- Implémenter et tester les opérations CRUD (Create, Read, Update, Delete)

## Prérequis

- JDK 8 ou supérieur
- Maven 3.6+
- Un IDE (IntelliJ IDEA, Eclipse, NetBeans...)

## Structure du projet

```
gestion-salles/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/example/
    │   │   ├── App.java                       # Classe principale (tests CRUD)
    │   │   ├── model/
    │   │   │   ├── Utilisateur.java           # Entité JPA + validations
    │   │   │   └── Salle.java                 # Entité JPA + validations
    │   │   └── service/
    │   │       ├── CrudService.java           # Interface CRUD générique
    │   │       ├── AbstractCrudService.java   # Implémentation CRUD générique
    │   │       ├── UtilisateurService.java    # Service spécifique (findByEmail)
    │   │       └── SalleService.java          # Service spécifique (findByDisponible, findByCapaciteMinimum)
    │   └── resources/
    │       └── META-INF/
    │           └── persistence.xml            # Configuration JPA / Hibernate / H2
    └── test/
        └── java/com/example/service/
            ├── UtilisateurServiceTest.java    # Tests JUnit
            └── SalleServiceTest.java          # Tests JUnit
```

## Stack technique

| Composant | Version |
|---|---|
| JPA API (javax.persistence) | 2.2 |
| Hibernate Core | 5.6.5.Final |
| Hibernate Validator | 6.2.0.Final |
| Jakarta EL (requis par Hibernate Validator) | 3.0.4 |
| Base de données H2 (en mémoire) | 2.1.214 |
| SLF4J (logs) | 1.7.36 |
| JUnit | 4.13.2 |

## Configuration (persistence.xml)

- **Base** : H2 en mémoire (`jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1`)
- **Utilisateur / mot de passe** : `sa` / *(vide)*
- **hibernate.hbm2ddl.auto = create-drop** : le schéma est créé au démarrage et supprimé à l'arrêt (idéal pour un TP, aucune donnée persistante entre deux exécutions)
- **hibernate.show_sql = true** : affiche les requêtes SQL générées dans la console

## Modèle de données

### Utilisateur
| Champ | Type | Contraintes |
|---|---|---|
| id | Long | généré automatiquement (IDENTITY) |
| nom | String | obligatoire, 2 à 50 caractères |
| prenom | String | obligatoire, 2 à 50 caractères |
| email | String | obligatoire, format email valide, unique |
| dateNaissance | LocalDate | doit être dans le passé |
| telephone | String | format `+33612345678` (10 à 15 chiffres) |

### Salle
| Champ | Type | Contraintes |
|---|---|---|
| id | Long | généré automatiquement (IDENTITY) |
| nom | String | obligatoire, 2 à 100 caractères |
| capacite | Integer | obligatoire, entre 1 et 1000 |
| description | String | max 500 caractères |
| disponible | Boolean | obligatoire (défaut : true) |
| etage | Integer | doit être ≥ 0 |

## Installation et exécution

### 1. Récupérer les dépendances

```bash
mvn clean install
```

Dans IntelliJ : clic droit sur `pom.xml` → **Maven → Reload project**.

### 2. Lancer l'application

```bash
mvn clean compile exec:java -Dexec.mainClass="com.example.App"
```

Ou directement dans l'IDE : clic droit sur `App.java` → **Run**.

### 3. Lancer les tests

```bash
mvn test
```

Ou dans l'IDE : clic droit sur `src/test/java` → **Run 'All Tests'**.

## Ce que fait `App.java`

Le programme exécute une démonstration complète du CRUD :

**Pour les Utilisateurs :**
1. Création de 2 utilisateurs
2. Lecture de tous les utilisateurs
3. Recherche par ID et par email
4. Mise à jour d'un utilisateur (téléphone)
5. Suppression d'un utilisateur par ID

**Pour les Salles :**
1. Création de 3 salles (dont une indisponible)
2. Lecture de toutes les salles
3. Recherche par ID, par disponibilité, par capacité minimum
4. Mise à jour d'une salle (capacité)
5. Suppression d'une salle par ID

## Architecture des services

```
CrudService<T, ID>              (interface : save, findById, findAll, update, delete, deleteById)
        ▲
        │ implémente
AbstractCrudService<T, ID>      (logique CRUD générique, réutilisable pour n'importe quelle entité)
        ▲                ▲
        │ étend          │ étend
UtilisateurService   SalleService
(+ findByEmail)      (+ findByDisponible, findByCapaciteMinimum)
```

`AbstractCrudService` utilise la réflexion Java (`ParameterizedType`) pour connaître dynamiquement la classe de l'entité gérée, ce qui permet d'écrire le code CRUD **une seule fois** pour toutes les entités.

## Résultat attendu (extrait console)


<img width="1270" height="674" alt="1" src="image/Capture d'écran 2026-09-30 000924.png" />

<img width="1270" height="674" alt="1" src="image/Capture d'écran 2026-09-30 000934.png" />

<img width="1270" height="674" alt="1" src="image/Capture d'écran 2026-09-30 000950.png" />

<img width="1270" height="674" alt="1" src="image/Capture d'écran 2026-09-30 000958.png" />
