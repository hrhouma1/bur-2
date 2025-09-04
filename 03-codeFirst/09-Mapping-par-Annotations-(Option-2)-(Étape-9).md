# 9. Mapping par **Annotations** (Option 2) — **Étape 9 super exhaustive**

> **Objectif**
> Mapper l’entité **`Compte`** directement **par attributs** (Data Annotations) — **sans Fluent API** — pour obtenir :
>
> * Table `dbo.T_Comptes`
> * Colonnes : `IdCompte`, `C_Numero`, `C_Solde`, `C_DateCreation`
> * Contraintes : `Numero` requis, longueur maximale 20 (et *optionnellement* index unique)
>
> **Important** : si tu as déjà appliqué le **mapping Fluent (Étape 8)** et créé la base, **supprime la base** (ou la table) avant de passer aux Annotations (sinon conflit de schéma).
> Ne **mélange pas** Fluent et Annotations pendant l’apprentissage — choisis l’un **ou** l’autre.


## 9.0 Préparer le terrain

1. **Désactiver/retirer le Fluent** dans `ContextEf` :

   * Commente **tout** le contenu de `OnModelCreating` (ou supprime-le).
   * Si tu avais une classe de config dédiée (`CompteConfiguration`), **ne l’enregistre plus**.
2. **Réinitialiser la base** :

   * Utilise `Util.DropDatabase()` puis `Util.CreateTable()` (vu en Étape 8).
   * Ou exécute ton petit programme **Reset DB** si tu l’as créé.

> ⚠️ Tant que la base existante ne correspond pas au mapping par Annotations, EF lèvera des erreurs (colonnes manquantes / types différents).



## 9.1 Ajouter les Data Annotations dans `METIER/Compte.cs`

**Fichier** : `METIER/Compte.cs`
**Namespaces requis** :

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
```

### 9.1.1 Version fidèle au cours (float + DataType)

```csharp
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace DAODemoP1.METIER
{
    [Table("T_Comptes")] // Table cible
    public class Compte
    {
        #region proprietes

        [Key]                                                  // Clé primaire
        [Column("IdCompte")]                                    // Nom de colonne
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]   // Identity (auto-incrément)
        public long CompteId { get; set; }

        [Column("C_Numero")]
        [Required]                  // NOT NULL
        [MaxLength(20)]             // NVARCHAR(20)
        public string Numero { get; set; }

        [Column("C_Solde")]
        [DataType("Real")]          // NOTE: vu en cours ; voir remarque 9.1.2
        public float Solde { get; set; }

        [Column("C_DateCreation")]
        [Required]
        public DateTime DateCreation { get; set; }

        #endregion

        #region constructeurs (identiques à l'étape 2)

        public Compte(string numero, DateTime dateCreation, float solde)
        {
            Numero = numero;
            DateCreation = dateCreation;
            Solde = solde;
        }

        public Compte(string numero)
        {
            Numero = numero;
            DateCreation = DateTime.Now;
            Solde = 0;
        }

        public Compte() : this("Undefined") { }

        #endregion

        #region deconstruct & méthodes (identiques à l'étape 2)

        public void Deconstruct(out string numero, out float solde, out DateTime dateCreation)
        { numero = Numero; solde = Solde; dateCreation = DateCreation; }

        public void Deconstruct(out string numero, out float solde)
        { numero = Numero; solde = Solde; }

        public void Crediter(float montant) => Solde += montant;

        public bool Debiter(float montant)
        {
            if (montant <= Solde) { Solde -= montant; return true; }
            return false;
        }

        public override string ToString()
            => $"Numero : {Numero} Solde : {Solde} DateCreation : {DateCreation}";

        #endregion
    }
}
```

### 9.1.2 Remarque importante (**DataType** vs **Column(TypeName=)**)

* `[DataType("Real")]` est **pédagogique** mais **n’influence pas** le type SQL pour EF6.
  Pour imposer le **type SQL**, utilise plutôt :

  ```csharp
  [Column("C_Solde", TypeName = "real")] // si tu gardes float côté C#
  public float Solde { get; set; }
  ```
* Pour un usage **financier**, préfère **`decimal`** en C# et mappe en SQL :

  ```csharp
  [Column("C_Solde", TypeName = "decimal")]
  [Range(typeof(decimal), "0", "79228162514264337593543950335")] // optionnel
  public decimal Solde { get; set; }

  // + dans Fluent tu pourrais forcer la précision ; en annotations EF6 ne fournit pas [Precision]
  // mais tu peux la régler en Fluent si mixte (ou via migration).
  ```

> Dans ce cours on garde `float` pour coller au fil conducteur, mais tu **connais la bonne pratique** pour la prod.



## 9.2 (Optionnel) Index **unique** par Data Annotations (EF 6.1+)

Si tu veux imposer l’unicité de `Numero` (pas deux comptes avec le même `Numero`) **et** que tu es en **EF 6.1+**, tu peux utiliser **`IndexAttribute`** :

```csharp
using System.ComponentModel.DataAnnotations.Schema;

[Column("C_Numero")]
[Required]
[MaxLength(20)]
[Index("IX_T_Comptes_Numero", IsUnique = true)]
public string Numero { get; set; }
```

> ⚠️ Nécessite EF 6.1+ (attribut dans l’assembly EntityFramework, namespace `System.ComponentModel.DataAnnotations.Schema`).
> ⚠️ L’attribut `[Index]` respecte la **longueur** — assure-toi que `MaxLength(20)` est défini (index sur NVARCHAR(MAX) interdit).



## 9.3 Laisser `ContextEf` **sans Fluent**

**Fichier** : `EF/ContextEf.cs` — Version **sans `OnModelCreating`** ou avec un `OnModelCreating` **vide**.

```csharp
using System.Data.Entity;
using DAODemoP1.METIER;

namespace DAODemoP1.EF
{
    public class ContextEf : DbContext
    {
        public ContextEf() : base("C_BdMaBanque") { }

        public DbSet<Compte> CompteEntities { get; set; }

        // Pas de mapping Fluent ici : on s'en remet aux Annotations
        // protected override void OnModelCreating(DbModelBuilder modelBuilder) { base.OnModelCreating(modelBuilder); }
    }
}
```

> **Astuce** : Si tu avais retiré la pluralisation en Fluent, elle n’est plus supprimée ici — **mais** tu nommes explicitement la **table** avec `[Table("T_Comptes")]`, donc EF respectera ce nom.



## 9.4 Recréer la base avec Annotations

### 9.4.1 `Util.ResetDatabase()` (si pas déjà codé)

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

### 9.4.2 Programme minimal de reset

```csharp
using System;
using DAODemoP1.UTILEMETIER;

class Program
{
    static void Main()
    {
        Console.WriteLine("Reset (Annotations) …");
        Util.ResetDatabase();
        Console.WriteLine("✅ Base recréée selon les Annotations.");
        Console.ReadLine();
    }
}
```

**Après exécution** :

* Base `BdComptes` créée
* Table **`dbo.T_Comptes`** créée avec colonnes et contraintes des Annotations



## 9.5 Vérifier dans SSMS / SQL

```sql
USE BdComptes;
SELECT TOP 10 * FROM dbo.T_Comptes;
```

**Structure attendue** :

* `IdCompte` : **PK**, Identity, type SQL `bigint` (car C# `long`)
* `C_Numero` : **NVARCHAR(20)**, **NOT NULL** (+ **index unique** si ajouté)
* `C_Solde` : par défaut `real` si `float`, ou `decimal` si tu as changé le type + `TypeName`
* `C_DateCreation` : **NOT NULL**



## 9.6 Comparatif : **Fluent** (Étape 8) vs **Annotations** (Étape 9)

| Aspect                | Fluent API                                     | Data Annotations                               |
| --------------------- | ---------------------------------------------- | ---------------------------------------------- |
| **Localisation**      | `ContextEf.OnModelCreating` (+ classes config) | Directement sur la classe `Compte`             |
| **Lisibilité modèle** | Modèle **pur** (sans attributs)                | Modèle **auto-documenté** (attributs visibles) |
| **Granularité**       | Très fine, APIs riches (conventions, etc.)     | Limitée aux attributs fournis par EF6          |
| **Types/Précision**   | Très simple (`HasPrecision`, `HasColumnType`)  | Plus limité (pas d’attribut `Precision`)       |
| **Index**             | `IndexAnnotation` (EF 6.1+)                    | `[Index]` (EF 6.1+)                            |
| **Mix possible ?**    | Oui, mais à manier avec précaution             | Oui, mais éviter les **collisions**            |

> Pour les **projets réels**, beaucoup d’équipes utilisent **Fluent** pour tout ce qui est “technique” (types/précisions/index/composite keys) et laissent quelques **Annotations** “sémantiques” (`[Required]`, `[MaxLength]`, `[Table]`) — avec une **discipline** d’équipe claire.



## 9.7 Pièges & solutions (Annotations)

1. **Tu as laissé du Fluent actif** → conflits de mapping
   → Commente/supprime **tout** mapping Fluent, réinitialise la base.

2. **`[DataType("Real")]` n’a pas l’effet attendu**
   → Utilise plutôt `[Column(TypeName = "real")]` pour le type SQL.
   (ou `decimal` + mapping dédié)

3. **Index unique non créé**
   → Vérifie EF **6.1+**, l’attribut `[Index]`, et `MaxLength`.
   → **Reset** la base (Drop/Create).

4. **Pluralisation**
   → Avec `[Table("T_Comptes")]`, EF respecte ce nom **exact** (pluralisation ignorée).

5. **Problèmes de droits / connectionStrings**
   → Voir Étape 6 (providerName, name, Data Source).



## 9.8 Check-list de sortie (Étape 9)

* [ ] Fluent **désactivé** (aucun mapping dans `OnModelCreating`)
* [ ] `METIER/Compte.cs` annoté : `[Table("T_Comptes")]`, `[Key]`, `[DatabaseGenerated(Identity)]`, `[Column(...)]`, `[Required]`, `[MaxLength(20)]`
* [ ] (Optionnel) `[Index(..., IsUnique = true)]` sur `Numero` (EF 6.1+)
* [ ] Base réinitialisée (Drop + Create)
* [ ] Vérification SSMS : table **`dbo.T_Comptes`** conforme



> ✅ **Étape suivante (10)** : **Initialisation des données (Seed)**
> On crée un `Initializer` (`InitCompte`) qui remplit la table `T_Comptes` avec les **comptes 101–107**, puis on ajoute `Util.InitTable()` et on **teste**.

