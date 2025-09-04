# FICHE RÉVISION EXPRESS

## 1) Architecture & objectifs

* **Couches** : `METIER` (entités), `DAO` (interfaces + implémentations), `UTILEMETIER` (helpers), `EF` (DbContext, init/seed).
* **But du Repository/DAO** : découpler l’accès aux données (mémoire ↔ base) via `ICompteDao`.

## 2) Métier

```csharp
public class Compte {
  public long CompteId { get; set; }
  public string Numero { get; set; }
  public float Solde { get; set; }              // (prod : preferer decimal)
  public DateTime DateCreation { get; set; }
  public void Crediter(float m) => Solde += m;
  public bool Debiter(float m){ if(m<=Solde){Solde-=m; return true;} return false; }
  public override string ToString()=> $"Numero : {Numero} Solde : {Solde} DateCreation : {DateCreation}";
}
```

## 3) DAO – Interface

```csharp
public interface ICompteDao {
  Compte GetCompte(string numero);
  (string,float) GetNumeroAndSolde(string numero);
  Tuple<string,float> GetNumeroAndSoldeTuple(string numero);
  string GetInformationCompte();
}
```

## 4) DAO – Mémoire

* `CompteDao` : `ListeComptes = Utile.GetComptes();`
* LINQ in-memory (`Where`, `Select`, `FirstOrDefault`, `Count/Sum/Average/Min/Max`).

## 5) EF6 – Setup rapide

* **NuGet** : `EntityFramework` (6.x)
* **App.config**

```xml
<connectionStrings>
  <add name="C_BdMaBanque"
       connectionString="Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=BdComptes;Trusted_Connection=True"
       providerName="System.Data.SqlClient"/>
</connectionStrings>
```

* **Context**

```csharp
public class ContextEf : DbContext {
  public ContextEf() : base("C_BdMaBanque"){}
  public DbSet<Compte> CompteEntities { get; set; }
}
```

## 6) Création & Seed

* Création : `new CreateDatabaseIfNotExists<ContextEf>().InitializeDatabase(new ContextEf());`
* Seed :

```csharp
public class InitCompte : CreateDatabaseIfNotExists<ContextEf> {
  protected override void Seed(ContextEf ctx){
    ctx.CompteEntities.Add(new Compte("101", new DateTime(2016,10,10),100));
    // ... 102–107 ...
    ctx.SaveChanges();
    base.Seed(ctx);
  }
}
```

* `Util.InitTable()` appelle l’initializer `InitCompte`.

## 7) Mapping

* **Option Fluent** (Étape 8) : `OnModelCreating`, `ToTable("T_Comptes","dbo")`, `HasMaxLength(20)`, etc.
* **Option Annotations** (Étape 9) : `[Table("T_Comptes")]`, `[Column("C_Numero")]`, `[MaxLength(20)]`, etc.
* ⚠️ Ne pas mélanger (ou bien maîtriser collisions). Si tu changes : **Drop & Create** la base.

## 8) DAO – Base (EF) : `CompteDaoBD`

* **Read** (penser à `.AsNoTracking()` pour la lecture pure)

```csharp
public Compte GetCompte(string numero)
  => ctx.CompteEntities.AsNoTracking().FirstOrDefault(c=>c.Numero==numero);

public (string,float) GetNumeroAndSolde(string numero)
  => ctx.CompteEntities.AsNoTracking()
     .Where(c=>c.Numero==numero)
     .Select(c=>new {c.Numero,c.Solde})
     .Select(a=>(a.Numero,a.Solde))
     .FirstOrDefault();

public Tuple<string,float> GetNumeroAndSoldeTuple(string numero)
  => ctx.CompteEntities.AsNoTracking()
     .Where(c=>c.Numero==numero)
     .Select(c=>new {c.Numero,c.Solde}).AsEnumerable()
     .Select(a=>Tuple.Create(a.Numero,a.Solde))
     .FirstOrDefault();
```

* **Agrégations robustes**

```csharp
var soldes = ctx.CompteEntities.AsNoTracking().Select(c=>c.Solde);
int n = soldes.Count();
float total = n>0? soldes.Sum():0f; // idem Avg/Min/Max
```

* **CRUD**

```csharp
bool AjouterCompte(Compte c){ ctx.CompteEntities.Add(c); return ctx.SaveChanges()>0; }
bool SupprimerCompte(string num){ var c=ctx.CompteEntities.FirstOrDefault(x=>x.Numero==num);
  if(c==null) return false; ctx.Remove(c); return ctx.SaveChanges()>0; }
void ModifierCompte(Compte c,float m){ var tracked=ctx.CompteEntities.FirstOrDefault(x=>x.Numero==c.Numero);
  if(tracked==null) return; tracked.Solde+=m; ctx.SaveChanges(); }
```

* **SQL natif**

```csharp
string q="SELECT IdCompte as CompteId, C_Numero as Numero, C_Solde as Solde, C_DateCreation as DateCreation FROM dbo.T_Comptes WHERE C_Numero=@p0";
var bySql = ctx.Database.SqlQuery<Compte>(q, new[]{numero}).FirstOrDefault();
```

## 9) Bonnes pratiques

* **AsNoTracking** en lecture, **projections** ciblées, **index** sur `Numero`, gérer **null**.
* Si **schéma change** sans migrations : **Drop & Create**.
* Pour argent réel : **`decimal`** + précision.
* Transactions : `using (var tx = ctx.Database.BeginTransaction()) { ... tx.Commit(); }`



# EXERCICES GUIDÉS (avec corrigés succincts)

## Exercice 1 — Lister les comptes avec solde > X (tri décroissant)

**But** : pratiquer LINQ to Entities, projection minimale.

**Consigne** : ajoute une méthode dans `CompteDaoBD` :
`IEnumerable<(string Numero, float Solde)> GetComptesAvecSoldeMin(float min);`

**Corrigé :**

```csharp
public IEnumerable<(string Numero, float Solde)> GetComptesAvecSoldeMin(float min)
  => context.CompteEntities.AsNoTracking()
       .Where(c => c.Solde > min)
       .OrderByDescending(c => c.Solde)
       .Select(c => new { c.Numero, c.Solde })
       .AsEnumerable()
       .Select(a => (a.Numero, a.Solde));
```

## Exercice 2 — Pagination simple

**But** : `Skip/Take`.

**Consigne** : `IEnumerable<Compte> GetPage(int page, int pageSize)` (page 1-based).

**Corrigé :**

```csharp
public IEnumerable<Compte> GetPage(int page,int pageSize)
  => context.CompteEntities.AsNoTracking()
       .OrderBy(c => c.CompteId)
       .Skip((page-1)*pageSize)
       .Take(pageSize)
       .ToList();
```

## Exercice 3 — Recherche par préfixe de numéro (insensible à la casse)

**Consigne** : `IEnumerable<string> FindNumeros(string prefix);`

**Corrigé :**

```csharp
public IEnumerable<string> FindNumeros(string prefix)
{
  prefix = prefix?.ToLower() ?? "";
  return context.CompteEntities.AsNoTracking()
    .Where(c => c.Numero.ToLower().StartsWith(prefix))
    .OrderBy(c => c.Numero)
    .Select(c => c.Numero)
    .ToList();
}
```

## Exercice 4 — Passer `Solde` en `decimal(18,2)` (bonne pratique)

**Consigne** : changer le type, ajuster le mapping.

**Corrigé (Annotations) :**

```csharp
// Compte.cs
[Column("C_Solde", TypeName="decimal")]
public decimal Solde { get; set; }  // remplace float

// Drop & Create la base, puis Seed.
```

**Corrigé (Fluent)** :

```csharp
Property(c => c.Solde).HasColumnName("C_Solde").HasPrecision(18,2);
```

## Exercice 5 — Index unique sur `Numero`

**Consigne** : empêcher doublons.

**Corrigé (Annotations, EF6.1+) :**

```csharp
[Column("C_Numero")]
[Required, MaxLength(20)]
[Index("IX_T_Comptes_Numero", IsUnique = true)]
public string Numero { get; set; }
```

> Drop & Create la base.

**Corrigé (Fluent)** : utiliser `IndexAnnotation`.

## Exercice 6 — Contrainte métier : interdire `Debiter` au-delà du solde (test unitaire)

**Consigne** : écrire un test console simple.

**Corrigé (extrait) :**

```csharp
var c = new Compte("777", DateTime.Now, 50);
bool ok = c.Debiter(100);
Console.WriteLine(ok ? "KO (aurait dû refuser)" : "OK (refus correct)");
```

## Exercice 7 — Update “détaché”

**Consigne** : méthode qui attache puis modifie.

**Corrigé :**

```csharp
public void ModifierCompteDetache(Compte cpt, float montant)
{
  context.CompteEntities.Attach(cpt);
  cpt.Solde += montant;
  context.Entry(cpt).Property(x => x.Solde).IsModified = true;
  context.SaveChanges();
}
```

## Exercice 8 — Transactions : virement simple A→B

**Consigne** : si débit impossible, rollback.

**Corrigé (idée) :**

```csharp
public bool Virement(string numA, string numB, float montant)
{
  using (var tx = context.Database.BeginTransaction())
  {
    var A = context.CompteEntities.FirstOrDefault(c=>c.Numero==numA);
    var B = context.CompteEntities.FirstOrDefault(c=>c.Numero==numB);
    if (A==null || B==null || A.Solde < montant){ tx.Rollback(); return false; }
    A.Solde -= montant; B.Solde += montant;
    context.SaveChanges();
    tx.Commit(); return true;
  }
}
```

## Exercice 9 — Filtrer par **période** de création

**Consigne** : `IEnumerable<Compte> GetByDateRange(DateTime d1, DateTime d2)`.

**Corrigé :**

```csharp
public IEnumerable<Compte> GetByDateRange(DateTime d1, DateTime d2)
  => context.CompteEntities.AsNoTracking()
       .Where(c => c.DateCreation >= d1 && c.DateCreation < d2)
       .OrderBy(c => c.DateCreation)
       .ToList();
```

## Exercice 10 — SQL natif : total des soldes

**Consigne** : renvoyer un `decimal`/`double` (ou `float`) total.

**Corrigé :**

```csharp
public float GetTotalSoldesSql()
{
  var res = context.Database.SqlQuery<float>("SELECT CAST(SUM(C_Solde) AS real) FROM dbo.T_Comptes").FirstOrDefault();
  return res;
}
```

## Exercice 11 — Async (bonus)

**Consigne** : version async de `GetCompte`.

**Corrigé :**

```csharp
using System.Data.Entity; // ToListAsync, FirstOrDefaultAsync
public Task<Compte> GetCompteAsync(string numero)
  => context.CompteEntities.AsNoTracking().FirstOrDefaultAsync(c=>c.Numero==numero);
```

## Exercice 12 — DTO pour projection

**Consigne** : créer `CompteDto{ Numero, Solde }`, méthode qui renvoie `List<CompteDto>`.

**Corrigé :**

```csharp
public class CompteDto { public string Numero {get;set;} public float Solde {get;set;} }

public List<CompteDto> GetDtos()
  => context.CompteEntities.AsNoTracking()
       .Select(c => new CompteDto{ Numero=c.Numero, Solde=c.Solde })
       .ToList();
```
