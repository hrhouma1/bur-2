# 8. Mapping par **Fluent API** (Option 1) — **Étape 8 super exhaustive**

> **Objectif**
>
> * Remplacer les **conventions par défaut EF6** par un **mapping explicite** :
>
>   * Table `dbo.T_Comptes` (au lieu de `Comptes`)
>   * Colonnes renommées : `IdCompte`, `C_Numero`, `C_Solde`, `C_DateCreation`
>   * Longueurs / types / contraintes (ex. `Numero` ≤ 20, non null)
> * **Sans** utiliser d’annotations (on les verra en Étape 9).
> * **Attention** : si tu as déjà créé la base en Étape 7 avec des conventions par défaut, **supprime-la** avant d’appliquer le mapping Fluent (sinon conflit schéma ↔ modèle).



## 8.0 Rappel & stratégie

* **EF6** applique des **conventions** : nom de table pluriel, PK déduite, etc.
* **Fluent API** permet de **tout préciser** dans `OnModelCreating(DbModelBuilder)`.
* **Ne mélange pas** Fluent **et** Annotations au début : pour le cours, choisis **une seule** approche (on verra Annotations en Étape 9).



## 8.1 Préparer `ContextEf` pour le Fluent

**Fichier** : `EF/ContextEf.cs`
**Action** : ajouter/compléter `OnModelCreating` et (optionnel) retirer la pluralisation.

```csharp
using System.ComponentModel.DataAnnotations.Schema;
using System.Data.Entity;
using System.Data.Entity.ModelConfiguration.Conventions;
using DAODemoP1.METIER;

namespace DAODemoP1.EF
{
    public class ContextEf : DbContext
    {
        public ContextEf() : base("C_BdMaBanque") { }

        public DbSet<Compte> CompteEntities { get; set; }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            // (1) Facultatif : enlever la convention de pluralisation des noms de tables
            // (evite "Comptes" si tu utilises ToTable sans nom explicite)
            modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();

            // (2) Table + schéma
            modelBuilder.Entity<Compte>()
                        .ToTable("T_Comptes", "dbo");

            // (3) Clé primaire + identité (auto-incrément)
            modelBuilder.Entity<Compte>()
                        .HasKey(c => c.CompteId);

            modelBuilder.Entity<Compte>()
                        .Property(c => c.CompteId)
                        .HasColumnName("IdCompte")
                        .HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity);

            // (4) Numero : requis, long. max 20, colonne "C_Numero"
            modelBuilder.Entity<Compte>()
                        .Property(c => c.Numero)
                        .HasColumnName("C_Numero")
                        .IsRequired()
                        .HasMaxLength(20);

            // (5) Solde : mappé sur "C_Solde"
            // NOTE: le cours utilise float -> SQL "real". Pour une app bancaire réelle, préférer decimal.
            modelBuilder.Entity<Compte>()
                        .Property(c => c.Solde)
                        .HasColumnName("C_Solde");

            // (6) DateCreation : requis, colonne "C_DateCreation"
            modelBuilder.Entity<Compte>()
                        .Property(c => c.DateCreation)
                        .HasColumnName("C_DateCreation")
                        .IsRequired();

            base.OnModelCreating(modelBuilder);
        }
    }
}
```

> 🔍 **Ce que fait ce mapping**
>
> * Table `dbo.T_Comptes`
> * PK : `IdCompte` (Identity)
> * `Numero` requis, **NVARCHAR(20)** via `HasMaxLength(20)`
> * `Solde` (float → SQL `real`)
> * `DateCreation` requis (`datetime` par défaut)



## 8.2 (Option PRO) — Déporter le mapping dans une classe `EntityTypeConfiguration<>`

C’est plus **propre** si ton modèle grossit.

**Fichier** : `EF/Mapping/CompteConfiguration.cs`

```csharp
using System.ComponentModel.DataAnnotations.Schema;
using System.Data.Entity.ModelConfiguration;
using DAODemoP1.METIER;

namespace DAODemoP1.EF.Mapping
{
    public class CompteConfiguration : EntityTypeConfiguration<Compte>
    {
        public CompteConfiguration()
        {
            ToTable("T_Comptes", "dbo");

            HasKey(c => c.CompteId);

            Property(c => c.CompteId)
                .HasColumnName("IdCompte")
                .HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity);

            Property(c => c.Numero)
                .HasColumnName("C_Numero")
                .IsRequired()
                .HasMaxLength(20);

            Property(c => c.Solde)
                .HasColumnName("C_Solde");

            Property(c => c.DateCreation)
                .HasColumnName("C_DateCreation")
                .IsRequired();
        }
    }
}
```

**Puis** dans `ContextEf` :

```csharp
using System.Data.Entity;
using System.Data.Entity.ModelConfiguration.Conventions;
using DAODemoP1.EF.Mapping;

protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();

    // Enregistrer la configuration
    modelBuilder.Configurations.Add(new CompteConfiguration());

    base.OnModelCreating(modelBuilder);
}
```

> ✅ Avantages : chaque entité a sa **classe de mapping** → plus lisible, plus testable.



## 8.3 Indices & unicité (optionnel, recommandé)

* **Hypothèse** : `Numero` doit être **unique** (pas deux comptes avec le même numéro).
* En **Fluent** EF6, on peut poser un **index** unique via `HasColumnAnnotation` (+ `IndexAnnotation`) :

```csharp
using System.Data.Entity.Infrastructure.Annotations;
using System.ComponentModel.DataAnnotations.Schema;

// ...
Property(c => c.Numero)
    .HasColumnName("C_Numero")
    .IsRequired()
    .HasMaxLength(20)
    .HasColumnAnnotation(
        IndexAnnotation.AnnotationName,
        new IndexAnnotation(
            new IndexAttribute("IX_T_Comptes_Numero") { IsUnique = true }
        )
    );
```

> ⚠️ **Pré-req** : `using System.Data.Entity.Infrastructure.Annotations;`
> ⚠️ **Longueur requise** pour indexer une NVARCHAR, d’où le `.HasMaxLength(20)`.



## 8.4 Types : `float` vs `decimal` (important)

* Le cours utilise `float` (C#) → SQL `real`.
* Pour des montants **financiers**, **préfère `decimal`** en C# → `decimal(18,2)` en SQL.
* **Si tu veux passer à `decimal` maintenant** :

  1. Change le type dans `Compte.cs` : `public decimal Solde { get; set; }`
  2. En Fluent, précise la précision/échelle :

     ```csharp
     Property(c => c.Solde)
        .HasColumnName("C_Solde")
        .HasPrecision(18, 2);
     ```
  3. **Supprime** et **recrée** la base (sinon conflit de type).

> Pour rester strictement fidèle au cours, on garde `float`. Note pédagogique faite ✅



## 8.5 Réinitialiser la base après passage au Fluent

Si tu as déjà créé la base avec les conventions (Étape 7), **Drop & Create** :

### 8.5.1 Utilitaires (si tu ne les as pas déjà)

**Fichier** : `UTILEMETIER/Util.cs`

```csharp
using System.Data.Entity;
using DAODemoP1.EF;

namespace DAODemoP1.UTILEMETIER
{
    static class Util
    {
        public static void CreateTable()
        {
            var initializer = new CreateDatabaseIfNotExists<ContextEf>();
            initializer.InitializeDatabase(new ContextEf());
        }

        public static bool DropDatabase()
        {
            using (var ctx = new ContextEf())
            {
                if (ctx.Database.Exists())
                    return ctx.Database.Delete();
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

### 8.5.2 Programme de reset

```csharp
using System;
using DAODemoP1.UTILEMETIER;

class Program
{
    static void Main()
    {
        Console.WriteLine("Reset DB (drop + create)…");
        Util.ResetDatabase();
        Console.WriteLine("✅ Base recréée avec le mapping Fluent.");
        Console.ReadLine();
    }
}
```

**Après exécution** : tu dois voir la table **`dbo.T_Comptes`** avec les colonnes mappées.



## 8.6 Vérifier le mapping (SSMS / SQL)

Requête de contrôle :

```sql
USE BdComptes;
SELECT TOP 10 *
FROM dbo.T_Comptes;
```

Structure (SSMS → clic droit Table → Conception) attendue :

* `IdCompte` (PK, Identity, `bigint` car C# `long`)
* `C_Numero` (NVARCHAR 20, NOT NULL, index unique si tu l’as ajouté)
* `C_Solde` (`real` si `float` C# ; `decimal(18,2)` si tu as choisi `decimal`)
* `C_DateCreation` (`datetime`, NOT NULL)



## 8.7 Alternative : conventions globales (plus avancé)

Tu peux imposer des **conventions custom** (ex. `string` → `nvarchar(200)` par défaut), mais c’est **optionnel** :

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    // Exemple : toutes les strings limitées à 200 par défaut
    modelBuilder.Properties<string>()
                .Configure(p => p.HasMaxLength(200));

    // … puis mapping explicite pour les exceptions (Numero = 20)
}
```

> Attention : les conventions globales peuvent surprendre si on les oublie. À réserver à des équipes rodées.



## 8.8 Pièges & solutions (Fluent)

1. **Tu as oublié de supprimer la base** et EF crie au **conflit de schéma**
   → `Util.ResetDatabase()` puis relance.

2. **Tu as laissé les Annotations** en Étape 9 **ET** du Fluent ici
   → Choisis **une seule** approche. Si tu mixes, sois **cohérent** et conscient des collisions.

3. **Index unique** non pris en compte
   → Vérifie l’`IndexAnnotation` (EF ≥ 6.1), la longueur du champ, et **Drop/Create**.

4. **Pluralisation** revient alors que tu as `ToTable("T_Comptes")`
   → Si tu nommes la table explicitement, la pluralisation ne s’applique **pas**.
   La suppression de la convention est utile si tu relies d’autres entités sans `ToTable`.

5. **Type SQL inattendu** sur `Solde`
   → `float` C# → SQL `real` ; pour `decimal`, utilise `.HasPrecision(p,s)`.



## 8.9 Check-list de sortie (Étape 8)

* [ ] `OnModelCreating` **complété** (ou classe `CompteConfiguration` dédiée)
* [ ] Table mappée en **`dbo.T_Comptes`**
* [ ] Colonnes : `IdCompte` (PK Identity), `C_Numero` (NVARCHAR(20) NOT NULL), `C_Solde`, `C_DateCreation` (NOT NULL)
* [ ] (Optionnel) Index unique sur `C_Numero`
* [ ] Base **réinitialisée** (Drop+Create) après changement de mapping
* [ ] Vérification SSMS / requête SQL **OK**



> ✅ **Étape suivante (9)** : **Mapping par Annotations (Option 2)**
> On va reprendre les mêmes objectifs (table `T_Comptes`, colonnes renommées, contraintes), mais **avec des attributs** directement sur la classe `Compte`.
