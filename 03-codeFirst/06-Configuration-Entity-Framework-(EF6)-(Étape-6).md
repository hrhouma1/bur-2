# 6. Configuration Entity Framework (EF6) — **Étape 6 super exhaustive**

> **Objectif**
>
> * Ajouter **Entity Framework 6** (EF6) à notre **Console (.NET Framework)**.
> * Créer le **contexte** `ContextEf` branché sur la chaîne de connexion **`"C_BdMaBanque"`**.
> * Préparer **`App.config`** avec `<connectionStrings>`.
> * Valider la configuration par un **mini “ping EF”** sans encore créer de tables (la création arrive en Étape 7).



## 6.0 Pré-requis (SQL & .NET)

* **Projet** : *Console (.NET Framework)* (4.7.2/4.8 recommandé).

  > ⚠️ Pas *Console (.NET)* (Core/5/6/7/8). EF6 vise .NET Framework et utilise `App.config`.
* **Moteur SQL** (au choix) :

  * **SQL Server Express LocalDB** (local, simple) : `MSSQLLocalDB`
  * **SQL Server Express** : `.\SQLEXPRESS`
  * **SQL Server** (instance nommée), ou un **serveur distant**
* **Compte d’accès** :

  * **Trusted connection** (Windows Auth) → `trusted_connection=true`
  * OU **SQL Auth** (`user id=...; password=...`)

> 💡 Si tu n’as pas SQL installé, **LocalDB** suffit pour le cours. Installe **SQL Server Express LocalDB** (via Visual Studio Installer ou MS docs).



## 6.1 Installer EF6 via NuGet

### 6.1.1 Interface Visual Studio

1. **Clic droit** sur le **projet** → **Gérer les packages NuGet…**
2. Onglet **Parcourir** → rechercher **EntityFramework** (éditeur Microsoft).
3. Installer **EntityFramework 6.x** (ex. 6.4.4).
4. Visual Studio ajoute la référence et crée/maj `packages.config` (selon la solution).

### 6.1.2 Vérifications

* Dans **Références**, tu dois voir **EntityFramework** et **EntityFramework.SqlServer**.
* Le `using` à ajouter dans le code sera **`using System.Data.Entity;`** (EF6).

> 🧪 **Test de compilation rapide**
> Ajoute temporairement `using System.Data.Entity;` en haut d’un fichier : s’il n’est **pas** surligné, le package est OK.



## 6.2 Créer le contexte EF : `ContextEf`

**Fichier** : `EF/ContextEf.cs`

> **But** : Définir le **DbContext** de l’application et exposer la table **Comptes** via `DbSet<Compte>`.
> **Convention** : le constructeur appelle `base("C_BdMaBanque")` pour pointer la chaîne de connexion de **`App.config`**.

```csharp
using System.Data.Entity;
using DAODemoP1.METIER;

namespace DAODemoP1.EF
{
    public class ContextEf : DbContext
    {
        // Le nom correspond à la clé dans <connectionStrings> d'App.config
        public ContextEf() : base("C_BdMaBanque") { }

        // DbSet = table "Comptes" (nom exact dépendra du mapping Étape 8/9)
        public DbSet<Compte> CompteEntities { get; set; }

        // ⚙️ Étape 8 (Fluent API) : on y reviendra.
        // protected override void OnModelCreating(DbModelBuilder modelBuilder)
        // {
        //     base.OnModelCreating(modelBuilder);
        //     modelBuilder.Entity<Compte>().ToTable("T_Comptes", "dbo");
        //     modelBuilder.Entity<Compte>().Property(c => c.Numero).HasColumnName("C_Numero");
        //     modelBuilder.Entity<Compte>().Property(c => c.Numero).HasMaxLength(20);
        // }

        // 🪵 (Optionnel) Journaliser les requêtes SQL EF en console
        // protected override void OnContextCreated()
        // {
        //     this.Database.Log = s => System.Diagnostics.Debug.WriteLine(s);
        // }
    }
}
```

> ℹ️ `DbSet<Compte> CompteEntities` : le nom **“CompteEntities”** est arbitraire. On peut l’appeler `Comptes`.
> ℹ️ Nous mapperons finement la table en **Étape 8 (Fluent)** ou **Étape 9 (Annotations)**.



## 6.3 Ajouter `<connectionStrings>` dans **App.config**

EF6 lit par défaut la **chaîne de connexion** dans `App.config` (projet Console).
Crée/ouvre **`App.config`** à la racine du projet et ajoute ceci **dans** `<configuration>` :

### 6.3.1 Exemple **LocalDB** (recommandé débutants)

```xml
<configuration>
  <connectionStrings>
    <add name="C_BdMaBanque"
         connectionString="Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=BdComptes;Trusted_Connection=True"
         providerName="System.Data.SqlClient" />
  </connectionStrings>
</configuration>
```

### 6.3.2 Exemple **SQL Express** (instance locale)

```xml
<configuration>
  <connectionStrings>
    <add name="C_BdMaBanque"
         connectionString="Data Source=.\SQLEXPRESS;Initial Catalog=BdComptes;Trusted_Connection=True"
         providerName="System.Data.SqlClient" />
  </connectionStrings>
</configuration>
```

### 6.3.3 Exemple **SQL Auth** (utilisateur/mot de passe)

```xml
<configuration>
  <connectionStrings>
    <add name="C_BdMaBanque"
         connectionString="Data Source=SERVEUR\INSTANCE;Initial Catalog=BdComptes;User ID=MonUser;Password=MonMotDePasse"
         providerName="System.Data.SqlClient" />
  </connectionStrings>
</configuration>
```

> ✅ **Clés essentielles** :
>
> * **name** = `C_BdMaBanque` → doit **matcher** `base("C_BdMaBanque")`
> * **providerName** = `System.Data.SqlClient`
> * **Initial Catalog** = nom de base (ici `BdComptes`)
> * **Data Source** = nom du serveur/instance SQL

### 6.3.4 Script T-SQL pour générer une chaîne (optionnel)

Dans SQL Server Management Studio (SSMS), exécute :

```sql
SELECT
    'data source=' + @@SERVERNAME +
    ';initial catalog=' + DB_NAME() +
    CASE type_desc
        WHEN 'WINDOWS_LOGIN' THEN ';trusted_connection=true'
        ELSE ';user id=' + SUSER_NAME() + ';password=<<YourPassword>>'
    END AS ConnectionString
FROM sys.server_principals
WHERE name = SUSER_NAME();
```

Copie/colle la chaîne puis **adapte `initial catalog=`** (ex. `BdComptes`).



## 6.4 “Ping EF” sans créer la base (vérification douce)

Avant l’Étape 7 (création de tables), faisons un **test de connexion**.
On **désactive temporairement** les initializers pour que rien ne soit créé :

```csharp
using System;
using System.Data.Entity;
using DAODemoP1.EF;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            // ⚠️ Empêche EF de créer automatiquement la base au premier accès
            Database.SetInitializer<ContextEf>(null);

            try
            {
                using (var ctx = new ContextEf())
                {
                    // Force EF à initialiser les services/connexion
                    ctx.Database.Initialize(false);

                    bool exists = ctx.Database.Exists();
                    Console.WriteLine("Connexion EF OK. Base existe ? " + (exists ? "Oui" : "Non"));

                    // (Optionnel) Afficher la chaîne de connexion effective
                    Console.WriteLine("ConnectionString : " + ctx.Database.Connection.ConnectionString);
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Erreur EF/SQL : " + ex.Message);
            }

            Console.WriteLine("Étape 6 - Ping terminé");
            Console.ReadLine();
        }
    }
}
```

**Résultat attendu** :

* Si la base n’existe pas encore → `Base existe ? Non` (c’est normal, on la créera en Étape 7).
* Aucune exception → **chaîne et provider** sont **corrects**.

> 💡 **Réactiver les initializers** : on les remettra en jeu dès **l’Étape 7** (création) et **l’Étape 10** (seed).



## 6.5 Où placer quoi ? (récap dossiers & namespaces)

* `EF/ContextEf.cs` → `namespace DAODemoP1.EF` (ou `DAODemoP1.Ef` selon ton choix, **reste cohérent**).
* `App.config` → à **la racine** du projet Console.
* `METIER/Compte.cs` → `namespace DAODemoP1.METIER` (créé à l’Étape 2).

> 🧭 Si Visual Studio génère `DAODemoP1Console.*`, tu peux renommer le **namespace** manuellement pour suivre le cours (`DAODemoP1.*`), mais assure-toi de le faire **partout** (usings & fichiers).

---

## 6.6 Dépannage (les 10 pièges classiques)

1. **`The type or namespace name 'DbContext' could not be found`**
   → EF6 non installé **ou** projet .NET Core au lieu de .NET Framework.
   **Action** : Installer **EntityFramework 6.x** via NuGet + vérifier le **type de projet**.

2. **`providerName` manquant / incorrect**
   → Ajoute `providerName="System.Data.SqlClient"` dans la `<connectionStrings>`.

3. **Mauvais `name`** dans `App.config`
   → Le `name="C_BdMaBanque"` doit **matcher** exactement `base("C_BdMaBanque")`.

4. **Erreur de serveur** : `A network-related or instance-specific error…`
   → `Data Source` incorrect. Essaie `.\SQLEXPRESS` ou `(localdb)\MSSQLLocalDB`.
   Vérifie l’existence de l’instance dans **SQL Server Configuration Manager**.

5. **Droits d’accès** (Trusted vs SQL Auth)
   → Si Windows Auth ne marche pas, bascule en **SQL Auth** (user/password valides).

6. **App.config pas pris en compte**
   → Le fichier **doit** s’appeler **`App.config`** dans le projet Console.

7. **Chevauchement de namespaces**
   → Unifier `DAODemoP1.*` (ou `DAODemoP1Console.*`) dans tous les fichiers.

8. **Conflit futur Fluent vs Annotations**
   → On ne mélange **pas** les deux (sauf maîtrise). Choisis **Étape 8** *ou* **Étape 9**.
   Si tu changes de stratégie **sans migrations**, supprime la DB d’abord.

9. **x86/x64** (rare en EF6)
   → Si DLLs natives (pilotes) : fais correspondre l’architecture **CPU** & **driver**.

10. **Journalisation SQL** (debug)
    → `ctx.Database.Log = s => Console.WriteLine(s);` (EF6) pour voir les requêtes générées.



## 6.7 Check-list de sortie (Étape 6)

* [ ] Package **EntityFramework 6.x** installé (NuGet)
* [ ] Fichier **`EF/ContextEf.cs`** créé (hérite de `DbContext`, `DbSet<Compte>`)
* [ ] **`App.config`** contient `<connectionStrings>` avec **`name="C_BdMaBanque"`** et **`providerName="System.Data.SqlClient"`**
* [ ] **Ping EF** (Initialize/Exists) fonctionne **sans créer** la base (initializers désactivés)



> ✅ **Étape suivante (7)** : **Création de la base & des tables**
> On va **réactiver/configurer** un **initializer** (`CreateDatabaseIfNotExists<ContextEf>`) et appeler notre utilitaire `Util.CreateTable()` depuis `Program.cs` pour créer le schéma. Ensuite (Étape 8/9) on verra **Fluent API** *ou* **Annotations** pour mapper proprement `Compte → T_Comptes`.
