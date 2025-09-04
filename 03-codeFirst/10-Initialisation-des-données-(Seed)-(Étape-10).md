# 10. Initialisation des données (Seed) — **Étape 10 super exhaustive**

> **Objectif**
> Remplir automatiquement la table `T_Comptes` **au moment de la création** de la base avec les **comptes 101–107** (mêmes données que l’in-memory).
> On utilise un **initializer EF6** qui surcharge `Seed(...)`.


## 10.0 Ce qu’il faut savoir (avant de coder)

* Le **Seed** s’exécute **uniquement** quand l’initializer **crée** la base (pas à chaque run).
* On va :

  1. créer un **initializer** `InitCompte : CreateDatabaseIfNotExists<ContextEf>` ;
  2. y **surcharger** `Seed(ContextEf context)` pour **insérer 101–107** ;
  3. exposer une méthode utilitaire `Util.InitTable()` qui **déclenche** cet initializer ;
  4. **tester** via un petit programme.
* Si tu changes la forme des données (ou si tu veux reseeder), tu devras **Drop + Create** la base (cf. Étapes 7–9).



## 10.1 Créer l’initializer `InitCompte`

**Fichier** : `EF/InitCompte.cs`

```csharp
using System;
using System.Data.Entity;
using DAODemoP1.METIER;

namespace DAODemoP1.EF
{
    /// <summary>
    /// Initializer EF6 : crée la base si elle n'existe pas, puis l'initialise (Seed).
    /// </summary>
    public class InitCompte : CreateDatabaseIfNotExists<ContextEf>
    {
        protected override void Seed(ContextEf context)
        {
            // 1) Remplir la table avec nos comptes de démo
            context.CompteEntities.Add(new Compte("101", new DateTime(2016, 10, 10), 100));
            context.CompteEntities.Add(new Compte("102", new DateTime(2017,  7, 18),   0));
            context.CompteEntities.Add(new Compte("103", new DateTime(2017,  3, 15), 100));
            context.CompteEntities.Add(new Compte("104", new DateTime(2016,  2,  5), 500));
            context.CompteEntities.Add(new Compte("105", new DateTime(2015,  8,  3), 200));
            context.CompteEntities.Add(new Compte("106", new DateTime(2014,  9, 24), 200));
            context.CompteEntities.Add(new Compte("107", new DateTime(2015,  1, 27), 200));

            // 2) Persister
            context.SaveChanges();

            // 3) Appeler toujours le Seed de base (bonne pratique)
            base.Seed(context);
        }
    }
}
```

> 📌 **Remarques**
>
> * Tu peux utiliser `AddRange(new[]{ ... })` si tu préfères ajouter en lot.
> * Si tu as choisi **Annotations (Étape 9)**, le mapping de `Compte` (table/colonnes) est déjà pris en compte au moment du Seed.
> * Si tu as choisi **Fluent (Étape 8)**, assure-toi que `OnModelCreating` est bien en place **avant** de seeder (sinon la table peut ne pas correspondre).



## 10.2 Exposer `Util.InitTable()` pour déclencher le Seed

**Fichier** : `UTILEMETIER/Util.cs`

> On ajoute **à côté** de `CreateTable()`, `DropDatabase()`, `ResetDatabase()` (vus avant).

```csharp
using System.Data.Entity;
using DAODemoP1.EF;

namespace DAODemoP1.UTILEMETIER
{
    static class Util
    {
        // … (CreateTable, DropDatabase, ResetDatabase déjà vus) …

        /// <summary>
        /// Crée la base si absente et exécute le Seed (InitCompte).
        /// </summary>
        public static void InitTable()
        {
            // Déclarer l'initializer de façon explicite
            var initializer = new InitCompte();

            // Déclencher la création + Seed si la base n'existe pas
            initializer.InitializeDatabase(new ContextEf());
        }
    }
}
```

> 🔁 **Quand l’appeler ?**
>
> * La **première fois** que tu veux une base **créée + seedée**.
> * Si tu dois **reseed** → fais d’abord `ResetDatabase()` puis `InitTable()`.


## 10.3 Programme de test (création + seed)

**Fichier** : `Program.cs` (exécution unitaire pour valider l’étape)

```csharp
using System;
using System.Linq;
using DAODemoP1.EF;
using DAODemoP1.UTILEMETIER;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            // 1) (Facultatif) tout remettre à zéro pour démonstration :
            // Util.ResetDatabase();

            // 2) Créer et seeder si la base n'existe pas
            Util.InitTable();
            Console.WriteLine("InitTable() exécuté.");

            // 3) Vérifier
            using (var ctx = new ContextEf())
            {
                var nb = ctx.CompteEntities.Count();
                Console.WriteLine("Nombre de comptes en base : " + nb);

                var c101 = ctx.CompteEntities.FirstOrDefault(c => c.Numero == "101");
                Console.WriteLine(c101 != null
                    ? $"OK -> {c101}"
                    : "ERREUR : compte 101 introuvable");
            }

            Console.WriteLine("Étape 10 - Seed terminé.");
            Console.ReadLine();
        }
    }
}
```

**Sortie attendue** (exemple) :

```
InitTable() exécuté.
Nombre de comptes en base : 7
OK -> Numero : 101 Solde : 100 DateCreation : 2016-10-10 00:00:00
Étape 10 - Seed terminé.
```



## 10.4 Variantes d’initializers (selon le besoin)

Pour le **développement**, tu peux préférer d’autres stratégies :

* `DropCreateDatabaseIfModelChanges<ContextEf>`
  → si **le modèle change**, EF **drop** et **recrée** la base puis **reseede** (pratique en cours/labo).
* `DropCreateDatabaseAlways<ContextEf>`
  → **toujours** drop + create + seed (chaque exécution).

**Exemple** (à la place de `InitCompte`) :

```csharp
public class InitCompteDev : DropCreateDatabaseIfModelChanges<ContextEf>
{
    protected override void Seed(ContextEf context)
    {
        // mêmes insertions que plus haut
        base.Seed(context);
    }
}
```

**Utilisation** :

```csharp
public static void InitTableDev()
{
    Database.SetInitializer(new InitCompteDev());

    using (var ctx = new ContextEf())
    {
        // Le premier Initialize va drop+create si modèle a changé
        ctx.Database.Initialize(false);
    }
}
```

> ⚠️ **Attention** en équipe : ces initializers **destructifs** ne s’utilisent **pas** en prod.



## 10.5 Seed **idempotent** (si tu veux rendre InitTable “safe”)

Le Seed n’est appelé **qu’à la création**.
Si tu crées une méthode d’**initialisation métier** que tu veux rejouer **sans doublons**, protège-la :

```csharp
protected override void Seed(ContextEf context)
{
    // Exemple d'idempotence : n'ajouter 101–107 que s'ils n'existent pas
    if (!context.CompteEntities.Any(c => c.Numero == "101"))
        context.CompteEntities.Add(new Compte("101", new DateTime(2016, 10, 10), 100));

    // … répéter pour 102–107 (ou boucler sur une liste) …

    context.SaveChanges();
    base.Seed(context);
}
```

> ✅ Avec cette approche, rejouer un **ResetDatabase + InitTable** donne un état propre et **prévisible**.



## 10.6 Journaliser ce que fait EF (utile pour comprendre)

```csharp
using (var ctx = new ContextEf())
{
    ctx.Database.Log = s => Console.WriteLine(s);
    // toute action suivante (Initialize, requêtes) écrira le SQL généré
    ctx.Database.Initialize(false);
}
```

> Très utile pour voir les `INSERT` émis par le Seed et vérifier la connexion.



## 10.7 Pièges & solutions

1. **Le Seed ne s’exécute pas**
   → L’initializer **ne se déclenche** que si la base est **créée**.

   * Si la base existe déjà, fais `Util.DropDatabase()` ou `ResetDatabase()` puis `InitTable()`.

2. **Doublons** après re-seed\*\*
   → Le Seed n’est **pas rejoué** (sauf recréation).
   → Si tu appelles un **seed “perso”** en dehors du lifecycle EF, **rends-le idempotent** (teste existence avant insertion).

3. **Erreur de mapping** (table/colonnes manquantes)
   → Ton mapping Fluent/Annotations a changé après création.
   → **Drop & Create** la base pour repartir propre.

4. **Problème de connexion**
   → Revois **`App.config`** (`name="C_BdMaBanque"`, `providerName="System.Data.SqlClient"`, `Data Source`, droits).
   → Teste un **“ping EF”** (Étape 6.4).

5. **Culture/date**
   → Les **dates** (`DateTime`) sont **invariantes** en code (OK), mais l’affichage peut changer selon la culture. Ce n’est **pas** un bug.



## 10.8 Check-list de sortie (Étape 10)

* [ ] `EF/InitCompte.cs` créé (hérite d’un initializer EF6, `Seed` insère 101–107)
* [ ] `UTILEMETIER/Util.cs` expose **`InitTable()`** (et, si utile, `ResetDatabase()`)
* [ ] `Program.cs` lance `InitTable()` et **vérifie** (`Count`, lecture de `101`)
* [ ] (Optionnel) Initializer de dev **DropIfModelChanges** disponible
* [ ] Seed **idempotent** envisagé si tu as besoin de rejouer des initialisations



> ✅ **Étape suivante (11)** : **DAO (Repository) côté Base de données**
> On va coder `CompteDaoBD : ICompteDao` utilisant **LINQ to Entities** (via `ContextEf`) et préparer les **opérations CRUD** (Étapes 12–16) + **SQL natif** (Étape 17).

