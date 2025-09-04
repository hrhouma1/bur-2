# 7. Base de données & tables — **Étape 7 super exhaustive**

> **Objectif**
> Créer **effectivement** la base de données **et** la/les table(s) à partir du modèle `Compte` à l’aide d’**EF6**.
> On utilise un **initializer** EF6 (sans migrations) pour créer la base si elle n’existe pas.



## 7.0 Principe (sans migrations, via Initializer)

* EF6 possède des **initializers** (stratégies) qui **s’exécutent au premier accès** au `DbContext` :

  * `CreateDatabaseIfNotExists<TContext>` : crée la base **si elle n’existe pas** (notre cas).
  * `DropCreateDatabaseIfModelChanges<TContext>` : supprime et recrée si le **modèle change**.
  * `DropCreateDatabaseAlways<TContext>` : supprime et recrée **à chaque exécution**.
* Nous allons encapsuler l’appel dans un utilitaire `Util.CreateTable()` pour **contrôler quand** la base est créée.

> ⚠️ À ce stade, nous n’avons pas encore **de mapping personnalisé** (Étape 8/9).
> EF appliquera ses **conventions par défaut** :
>
> * Nom de table (par défaut) : **`Comptes`** (dérivé de `DbSet<Compte>` – *pluralization*).
> * Clé primaire : propriété nommée `Id` ou `CompteId` → **`CompteId`** ✔️
> * Types : `string` → `nvarchar(max)`, `float` → `real`, `DateTime` → `datetime`, etc.



## 7.1 Méthode utilitaire `CreateTable()` (dans `UTILEMETIER`)

**Fichier** : `UTILEMETIER/Util.cs`

> Si tu as déjà une classe `Utile.cs` pour `GetComptes()`, ce n’est pas un problème :
> tu peux **coexister** avec une classe `Util` (ou fusionner dans la même classe statique).

```csharp
using System.Data.Entity;
using DAODemoP1.EF;

namespace DAODemoP1.UTILEMETIER
{
    static class Util
    {
        /// <summary>
        /// Crée la base et les tables si elles n'existent pas encore,
        /// selon le modèle (ContextEf + conventions EF).
        /// </summary>
        public static void CreateTable()
        {
            var initializer = new CreateDatabaseIfNotExists<ContextEf>();
            initializer.InitializeDatabase(new ContextEf());
        }
    }
}
```

> Cette implémentation **force** explicitement l’exécution de la stratégie et évite d’attendre un “premier accès” implicite (comme un `ctx.CompteEntities.ToList()`).


## 7.2 Appel (unique) depuis `Program.cs`

**But** : exécuter **une seule fois** pour créer la base (puis commenter cet appel).

```csharp
using System;
using DAODemoP1.UTILEMETIER;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            // 1) Création de la base (à faire UNE fois)
            Util.CreateTable();
            Console.WriteLine("Si elle n'existait pas, la base et la table ont été créées.");

            // 2) (Optionnel) Lancer un petit accès pour vérifier
            //    (nécessite d'avoir DAO/CompteDaoBD en Étape 11, sinon saute cette partie)
            // var dao = new DAODemoP1.DAO.CompteDaoBD();
            // Console.WriteLine("Existe Compte 101 ? " + (dao.GetCompte("101") != null ? "Oui" : "Non"));

            Console.ReadLine();
        }
    }
}
```

> ✔️ Après exécution, la **base** `BdComptes` et la **table** (par défaut `Comptes`) doivent exister sur ton serveur SQL (selon ta chaîne de connexion : LocalDB, SQLEXPRESS, etc.).



## 7.3 Vérifier la création (SSMS / SQL / Visual Studio)

* **SQL Server Management Studio (SSMS)** → connecter le serveur (`(localdb)\MSSQLLocalDB` ou `.\SQLEXPRESS`) :

  * Déployer **Bases de données** → voir **`BdComptes`**.
  * Déployer **Tables** → voir **`dbo.Comptes`**.
  * Clic droit → **Sélectionner les 1000 premières lignes**.
* **Requête SQL** :

  ```sql
  USE BdComptes;
  SELECT TOP 10 * FROM dbo.Comptes;
  ```
* **Explorateur de serveurs** (Visual Studio) → **Connexions de données** → ajouter une connexion SQL Server.

**Schéma attendu (conventions EF)** :

* Table **`dbo.Comptes`**

  * `CompteId` (PK, `int`/`bigint` selon mapping ; par défaut `int identity` si non configuré → ici `long` => `bigint` + identity)
  * `Numero` (`nvarchar(max)` par défaut)
  * `Solde` (`real`)
  * `DateCreation` (`datetime`)

> 📝 Nous affinerons (noms de colonnes, longueur, types) en **Étape 8 (Fluent)** ou **Étape 9 (Annotations)**.
> Le cours vise `T_Comptes`, `C_Numero`, etc. → mapping personnalisé à venir.



## 7.4 Alternative : initializers “globaux” (au démarrage)

Au lieu d’appeler `InitializeDatabase` manuellement, on peut **déclarer** la stratégie **une fois** au démarrage :

```csharp
using System;
using System.Data.Entity;
using DAODemoP1.EF;

class Program
{
    static void Main()
    {
        Database.SetInitializer(new CreateDatabaseIfNotExists<ContextEf>());

        using (var ctx = new ContextEf())
        {
            // Le premier accès déclenche la création si besoin
            ctx.Database.Initialize(false);
            Console.WriteLine("Base créée/existante : " + ctx.Database.Exists());
        }
        Console.ReadLine();
    }
}
```

> ✅ Pratique si tu préfères laisser EF “piloter” au premier accès.
> ❗️ Pour un **contrôle fin** (cours/labo), la version `Util.CreateTable()` est plus explicite.



## 7.5 (Optionnel) Méthodes utilitaires **Drop** & **Reset**

Quand tu changes de mapping (Étape 8/9), sans activer les **migrations**, il faut souvent **recréer** la base :

```csharp
using System.Data.Entity;
using DAODemoP1.EF;

namespace DAODemoP1.UTILEMETIER
{
    static partial class Util
    {
        public static bool DropDatabase()
        {
            using (var ctx = new ContextEf())
            {
                if (ctx.Database.Exists())
                    return ctx.Database.Delete(); // DROP DATABASE
                return false;
            }
        }

        public static void ResetDatabase()
        {
            DropDatabase();
            CreateTable();
        }
    }
}
```

> **Quand l’utiliser ?**
>
> * Tu passes des **conventions** au **mapping Fluent** (Étape 8) → **drop & create**.
> * Tu passes au **mapping Annotations** (Étape 9) → **drop & create**.
> * Tu modifies la **forme** de `Compte` (colonnes) sans migrations → **drop & create**.



## 7.6 Journaliser les requêtes EF (debug)

Pour voir ce que fait EF au moment de la création / des requêtes :

```csharp
using (var ctx = new ContextEf())
{
    ctx.Database.Log = s => Console.WriteLine(s); // log vers la console
    ctx.Database.Initialize(false);
}
```

> Utile pour **comprendre** la séquence SQL générée et détecter des soucis de connexion/mapping.



## 7.7 Erreurs courantes & solutions

1. **`A network-related or instance-specific error`**
   → Mauvais `Data Source` (serveur/instance).

   * LocalDB : `Data Source=(localdb)\MSSQLLocalDB`
   * SQL Express local : `Data Source=.\SQLEXPRESS`
   * Serveur nommé : `Data Source=SERVEUR\INSTANCE`

2. **`Login failed`**
   → Problème d’authentification.

   * En **Trusted** : ton utilisateur Windows n’a pas les droits → utiliser SSMS pour créer la base ou basculer en **SQL Auth**.
   * En **SQL Auth** : mauvais **user/password** → corriger.

3. **`providerName` ou `name` incorrects** dans `App.config`
   → `name="C_BdMaBanque"` doit **matcher** `base("C_BdMaBanque")`.
   → `providerName="System.Data.SqlClient"` requis.

4. **La base ne se crée pas**
   → Tu as peut-être **laissé** `Database.SetInitializer<ContextEf>(null)` (Étape 6 Ping).

   * Retire cette ligne
   * ou force la création via `Util.CreateTable()`.

5. **Pluralisation du nom de table** inattendue
   → EF a créé `Comptes`.

   * Ce sera aligné en **Étape 8 (Fluent)** ou **Étape 9 (Annotations)** sur `T_Comptes`.
   * Sinon, désactive la convention :

     ```csharp
     modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
     ```

6. **Schema non `dbo`** (rare)
   → Si ton utilisateur n’est pas dans `dbo`, la table peut être créée sous un autre schéma.

   * Solution : mapping explicite `ToTable("T_Comptes", "dbo")` (Étape 8) ou **donner les droits**.



## 7.8 Check-list de sortie (Étape 7)

* [ ] `UTILEMETIER/Util.cs` contient **`CreateTable()`** (initializer `CreateDatabaseIfNotExists<ContextEf>`)
* [ ] `Program.cs` appelle `Util.CreateTable()` **une fois** pour créer la base
* [ ] Base **`BdComptes`** visible dans SQL (LocalDB / SQLEXPRESS / serveur choisi)
* [ ] Table **`Comptes`** présente avec colonnes **`CompteId`**, **`Numero`**, **`Solde`**, **`DateCreation`**
* [ ] (Optionnel) Méthodes **Drop/Reset** disponibles pour réinitialiser entre Étapes 8/9



> ✅ **Étape suivante (8)** : **Mapping par Fluent API**
>
> * Renommer proprement la table en **`T_Comptes`**,
> * Renommer les colonnes (`C_Numero`, etc.),
> * Définir des **longueurs** (ex. `Numero` ≤ 20),
> * (Optionnel) Désactiver la **pluralisation**.

