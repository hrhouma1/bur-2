# Glossaire des concepts couverts

## 1) Architecture en couches (Layered Architecture)

* **But** : séparer responsabilités (métier, accès aux données, utilitaires, EF) pour tester, faire évoluer et remplacer facilement une couche.
* **Dossiers** : `METIER` (entités), `DAO` (interfaces + implémentations), `UTILEMETIER` (helpers), `EF` (DbContext, init/seed).

## 2) DAO / Repository Pattern

* **Idée** : encapsuler l’accès aux données derrière une **interface** (`ICompteDao`) avec plusieurs implémentations interchangeables:

  * `CompteDao` (mémoire)
  * `CompteDaoBD` (EF / base)
* **Bénéfices** : découplage, testabilité, évolutivité (changer de stockage sans toucher au reste).

## 3) Entité métier (POCO)

* **Exemple** : `Compte` (Id, Numero, Solde, DateCreation) avec méthodes `Crediter`, `Debiter`, `ToString`, `Deconstruct`.
* **POCO** = classe simple, sans dépendance à un framework.

## 4) LINQ (Language Integrated Query)

* **Quoi** : langage de requêtes intégré à C# (sur listes, EF, etc.).
* **Syntaxes** : *method syntax* (`Where`, `Select`, `FirstOrDefault`, …) et *query syntax* (`from … in … select …`).
* **Clés** :

  * **Exécution différée** vs **immédiate** (`ToList`, `Count` déclenchent l’exécution).
  * **Projections** (ne renvoyer que ce qu’il faut), **agrégations** (`Count`, `Sum`, `Average`, `Min`, `Max`).
  * `First` lève si vide ; `FirstOrDefault` renvoie `null` (réf.) si vide.

## 5) Tuples : `ValueTuple` vs `Tuple<T1,T2>`

* **ValueTuple** `(string Numero, float Solde)` : moderne, lisible, déconstruction facile.
* **Tuple\<T1,T2>** : type .NET historique (`Item1`, `Item2`).
* **EF6** : pour créer un `Tuple<>` dans une requête EF, il faut **basculer en mémoire** via `.AsEnumerable()` avant `new Tuple`.

## 6) DTO (Data Transfer Object)

* **But** : objet de transport **différent** de l’entité (on ne renvoie que ce qui est utile à l’UI/API).
* **Avantages** : contrat stable, sécurité (on ne dévoile pas tout), perf (moins de données).
* **Exemple** : `CompteDto { string Numero; float Solde; }` + `Select` pour projeter.

## 7) Entity Framework 6 (EF6) – ORM

* **Objets clés** : `DbContext` (unité de travail), `DbSet<T>` (table), `SaveChanges()`.
* **Code First** : classes C# → schéma BD (création automatique via initializers).
* **Connection string** : `App.config`, `<connectionStrings>`, `name="C_BdMaBanque"`.

## 8) Data Annotations (Mapping par attributs)

* **But** : configurer le mapping **dans la classe** avec des attributs :

  * `[Table("T_Comptes")]`, `[Key]`, `[DatabaseGenerated(Identity)]`
  * `[Column("C_Numero")]`, `[Required]`, `[MaxLength(20)]`
  * (EF 6.1+) `[Index("IX_T_Comptes_Numero", IsUnique=true)]`
* **Pro** : lisible, proche du modèle. **Limites** : moins fin que Fluent pour certains cas (précision, clés composites).

## 9) Fluent API (Mapping par code dans `OnModelCreating`)

* **But** : config **riche et centralisée** dans `ContextEf` (ou classes `EntityTypeConfiguration<>`).
* **Exemples** :

  ```csharp
  modelBuilder.Entity<Compte>().ToTable("T_Comptes","dbo");
  modelBuilder.Entity<Compte>().Property(c=>c.Numero).HasColumnName("C_Numero").IsRequired().HasMaxLength(20);
  ```
* **Pro** : puissance, granularité (précision décimale, conventions, relations complexes).
* **Remarque** : ne pas mixer à l’aveugle Annotations & Fluent (risque de collisions).

## 10) Initializers EF6 & Seed (remplissage initial)

* **Initializers** :

  * `CreateDatabaseIfNotExists<T>()`
  * `DropCreateDatabaseIfModelChanges<T>()`
  * `DropCreateDatabaseAlways<T>()`
* **Seed** : méthode `Seed(ContextEf ctx)` pour **insérer des données** au moment de la **création** de la base (ex: comptes 101–107).
* **Idempotence** : si tu veux reseeder sans doublons, **teste** l’existence avant d’insérer.

## 11) Conventions & mapping par défaut

* EF déduit table, PK, types si rien n’est configuré :

  * Nom table pluriel (`Comptes`), PK = `Id` ou `{Nom}Id`, `string` → `nvarchar(max)`, etc.
* Tu as appris à **remplacer** ces conventions par **Fluent** ou **Annotations**.

## 12) Tracking vs `AsNoTracking()`

* **Tracking** : le contexte suit les entités lues pour détecter modifications → utile pour **Update**.
* **`AsNoTracking()`** : lecture **plus rapide** (read-only), pas d’overhead → recommandé pour requêtes de consultation.

## 13) CRUD avec EF

* **Create** : `Add` + `SaveChanges()`
* **Read** : LINQ to Entities (`Where`, `FirstOrDefault`), projections, agrégations
* **Update** : soit recharger entité suivie puis modifier, soit **Attach** + marquer modifié
* **Delete** : `Remove` + `SaveChanges()`
* **Gestion nulls** et **exceptions** (unicité, longueur, not null) à prévoir.

## 14) SQL natif (quand LINQ ne suffit pas)

* **Lecture** : `Database.SqlQuery<T>(sql, params…)` + **alias colonnes** vers propriétés C#.
* **Écriture** : `Database.ExecuteSqlCommand(...)`
* **Toujours** paramétrer (éviter injection SQL).

## 15) Transactions

* **But** : grouper des opérations atomiques (ex: virement A→B).
* **API** : `using (var tx = context.Database.BeginTransaction()) { … tx.Commit(); }`

## 16) Index & unicité

* **Pourquoi** : performance & contraintes métier (ex: `Numero` unique).
* **Moyens** :

  * Annotations (EF 6.1+) : `[Index("IX", IsUnique = true)]`
  * Fluent : `HasColumnAnnotation(IndexAnnotation.AnnotationName, …)`

## 17) Types numériques : `float` vs `decimal`

* **Cours** : `float` (→ SQL `real`) pour la simplicité.
* **Pratique métier** (finance) : **`decimal(18,2)`** pour éviter les erreurs d’arrondi.

## 18) Pagination & tri

* **LINQ** : `OrderBy`, `Skip`, `Take` (souvent côté base via EF).
* **Utilité** : limiter la charge réseau et mémoire.

## 19) Gestion des erreurs & null

* **Lecture introuvable** : `FirstOrDefault` ⇒ gérer `null` côté appelant.
* **Agrégations** sur séquence vide : protéger `Average/Min/Max` (sinon exception).
* **Changement de mapping sans migrations** : **Drop & Create** la base.

---

## Mini-aide-mémoire (extraits)

**Annotations**

```csharp
[Table("T_Comptes")]
public class Compte {
  [Key, Column("IdCompte"), DatabaseGenerated(DatabaseGeneratedOption.Identity)]
  public long CompteId { get; set; }

  [Column("C_Numero"), Required, MaxLength(20)]
  public string Numero { get; set; }

  [Column("C_Solde", TypeName="real")] // ou decimal
  public float Solde { get; set; }

  [Column("C_DateCreation"), Required]
  public DateTime DateCreation { get; set; }
}
```

**Fluent**

```csharp
protected override void OnModelCreating(DbModelBuilder mb){
  mb.Entity<Compte>().ToTable("T_Comptes","dbo")
    .HasKey(c=>c.CompteId);
  mb.Entity<Compte>().Property(c=>c.CompteId)
    .HasColumnName("IdCompte")
    .HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity);
  mb.Entity<Compte>().Property(c=>c.Numero)
    .HasColumnName("C_Numero").IsRequired().HasMaxLength(20);
  mb.Entity<Compte>().Property(c=>c.Solde)
    .HasColumnName("C_Solde");
  mb.Entity<Compte>().Property(c=>c.DateCreation)
    .HasColumnName("C_DateCreation").IsRequired();
}
```

**Seed (initializer)**

```csharp
public class InitCompte : CreateDatabaseIfNotExists<ContextEf> {
  protected override void Seed(ContextEf ctx){
    ctx.CompteEntities.Add(new Compte("101", new DateTime(2016,10,10),100));
    // … 102–107 …
    ctx.SaveChanges();
    base.Seed(ctx);
  }
}
```

**LINQ — projections & agrégations**

```csharp
var (num, solde) = ctx.CompteEntities
  .AsNoTracking()
  .Where(c=>c.Numero=="101")
  .Select(c=>new { c.Numero, c.Solde })
  .Select(a=>(a.Numero, a.Solde))
  .FirstOrDefault();

int n = ctx.CompteEntities.Count();
float total = n>0 ? ctx.CompteEntities.Sum(c=>c.Solde) : 0f;
```
