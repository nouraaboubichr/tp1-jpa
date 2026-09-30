# TP1 : Hibernate + JPA + H2

Projet Maven de découverte de JPA/Hibernate avec une base H2 en mémoire.
Ce TP met en place le strict minimum pour comprendre la chaîne complète : entité JPA → EntityManager → Hibernate → base H2, avec des opérations simples d'insertion et de lecture.

## Objectifs du TP

- Créer un projet Maven
- Configurer Hibernate avec une base de données H2
- Créer une entité JPA minimale
- Implémenter des opérations d'insertion et de lecture

## Prérequis

- JDK 8 ou supérieur installé
- Maven installé
- Un IDE (IntelliJ IDEA, Eclipse, NetBeans, etc.)

## Structure du projet

```
hibernate-demo/
├── pom.xml
└── src/
    └── main/
        ├── java/com/example/
        │   ├── App.java                # Classe principale (insertion + lecture)
        │   └── model/
        │       └── Produit.java        # Entité JPA minimale
        └── resources/
            └── META-INF/
                └── persistence.xml     # Configuration JPA / Hibernate / H2
```

## Stack technique

| Composant | Version |
|---|---|
| JPA API (javax.persistence) | 2.2 |
| Hibernate Core | 5.6.5.Final |
| Base de données H2 (en mémoire) | 2.1.214 |
| SLF4J API | 1.7.36 |
| SLF4J Simple (logs console) | 1.7.36 |

## Concepts clés

| Concept | Rôle |
|---|---|
| **JPA** | La spécification (le standard) : `@Entity`, `EntityManager`, `Persistence` |
| **Hibernate** | L'implémentation qui exécute réellement JPA (génère le SQL) |
| **H2** | Base de données légère qui vit en mémoire (aucune installation) |
| **EntityManagerFactory** | Objet coûteux, créé une seule fois pour toute l'application |
| **EntityManager** | Objet léger, ouvert et fermé pour chaque opération |
| **Transaction** | Obligatoire pour toute écriture (insert/update/delete) en `RESOURCE_LOCAL` |

## Configuration (persistence.xml)

- **Unité de persistance** : `hibernate-demo`
- **Base** : H2 en mémoire (`jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1`)
- **Utilisateur / mot de passe** : `sa` / *(vide)*
- **hibernate.hbm2ddl.auto = create-drop** : Hibernate crée les tables au démarrage et les supprime à l'arrêt (pratique en TP : chaque exécution repart d'une base vide)
- **hibernate.show_sql = true** : affiche les requêtes SQL générées dans la console
- **hibernate.format_sql = true** : formate le SQL affiché pour une meilleure lisibilité

## Modèle de données

### Produit
| Champ | Type | Description |
|---|---|---|
| id | Long | clé primaire, générée automatiquement (`GenerationType.IDENTITY`) |
| nom | String | nom du produit |
| prix | BigDecimal | prix du produit |

```java
@Entity
public class Produit {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nom;
    private BigDecimal prix;
}
```

## Installation et exécution

### 1. Récupérer les dépendances

```bash
mvn clean install
```

Dans IntelliJ : clic droit sur `pom.xml` → **Maven → Reload project**. Vérifier ensuite dans **External Libraries** que les jars Hibernate, JPA et H2 sont bien téléchargés.

### 2. Lancer l'application

```bash
mvn clean compile exec:java -Dexec.mainClass="com.example.App"
```

Ou directement dans l'IDE : clic droit sur `App.java` → **Run 'App.main()'**.

## Ce que fait `App.java`

Le programme se déroule en 3 étapes :

**1. Démarrage**
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hibernate-demo");
```
Lit `persistence.xml`, démarre Hibernate, crée automatiquement la table `Produit`.

**2. Insertion (`insererProduits`)**
```java
em.getTransaction().begin();
em.persist(new Produit("Laptop", new BigDecimal("999.99")));
em.persist(new Produit("Smartphone", new BigDecimal("499.99")));
em.persist(new Produit("Tablette", new BigDecimal("299.99")));
em.getTransaction().commit();
```
Trois produits sont créés et insérés dans une seule transaction (tout ou rien : en cas d'erreur, `rollback()` annule tout).

**3. Lecture (`lireProduits`)**
```java
List<Produit> produits = em.createQuery("SELECT p FROM Produit p", Produit.class).getResultList();
Produit p2 = em.find(Produit.class, 2L);
```
Une requête JPQL récupère tous les produits, puis `em.find()` recherche un produit précis par son identifiant.

## Résultat attendu
<img width="1270" height="674" alt="1" src="imagr/Capture d'écran 2026-09-30 002312.png" />

<img width="1270" height="674" alt="1" src="imagr/Capture d'écran 2026-09-30 002319.png" />

<img width="1270" height="674" alt="1" src="imagr/Capture d'écran 2026-09-30 002328.png" />
