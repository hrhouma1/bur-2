# 12. CRUD — Introduction & scénarios (EF6) **super exhaustif**

> **Objectif**
> Comprendre **comment et quand** utiliser chaque opération CRUD via `CompteDaoBD` (EF6), en gardant la même **interface** que notre DAO mémoire. On formalise aussi des **bonnes pratiques** (NoTracking, gestion des nulls, validations simples, transactions).



## 12.1 Rappel des deux backends interchangeables

* **Mémoire** : `new CompteDao()` → rapide pour prototyper.
* **Base EF6** : `new CompteDaoBD()` → persistant, utilise `ContextEf` et `App.config`.

Grâce à l’**interface** `ICompteDao`, on peut **switcher** sans toucher au code appelant :

```csharp
ICompteDao dao;
// dao = new CompteDao();         // in-memory
dao = new CompteDaoBD();           // EF6
```



## 12.2 Les 4 opérations CRUD (vue d’ensemble)

| Opération                 | Méthode `CompteDaoBD` (déjà codée)                                                 | Points clés                                                           |
| ------------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **Read** (lecture)        | `GetCompte`, `GetNumeroAndSolde`, `GetNumeroAndSoldeTuple`, `GetInformationCompte` | `AsNoTracking()` pour lecture pure ; projections                      |
| **Create** (insertion)    | `AjouterCompte(Compte cpt)`                                                        | `Add` → `SaveChanges()` ; gérer contrainte `Numero` unique si ajoutée |
| **Delete** (suppression)  | `SupprimerCompte(string numero)`                                                   | Charger, `Remove`, `SaveChanges()` ; gérer inexistant                 |
| **Update** (modification) | `ModifierCompte(Compte cpt, float montant)`                                        | Tracked vs détaché ; recharger avant MAJ ; `SaveChanges()`            |



## 12.3 Stratégie de tests

* **Étape 10** a seedé **101–107**. On peut tester **Read** immédiatement.
* Pour **Create/Delete/Update**, on travaille sur un **compte “temp”** (ex. `110`), pour ne pas casser le seed.
* On découpe le programme de test en **blocs** activables/désactivables (commentaires).



# 13. CRUD — **Read (lecture)** super exhaustif

> **Objectif**
> Lire un compte ou des champs ciblés ; faire des agrégations ; bonnes pratiques de performance et de robustesse.



## 13.1 Lecture d’un compte par numéro

```csharp
var cpt = dao.GetCompte("101");
if (cpt != null)
    Console.WriteLine(cpt);
else
    Console.WriteLine("Compte 101 introuvable");
```

### Détails d’implémentation (rappel)

```csharp
public Compte GetCompte(string numero)
{
    return context.CompteEntities
                  .AsNoTracking()                      // lecture pure
                  .Where(c => c.Numero.Equals(numero))
                  .FirstOrDefault();                   // null si introuvable
}
```

**Pourquoi `AsNoTracking()` ?**

* Moins de surcharge (EF ne suit pas l’objet) → **plus rapide** en lecture pure.
* Si on veut **modifier** ensuite, on retirera `AsNoTracking()` **ou** on **rechargera** l’entité.



## 13.2 Lecture partielle (projection)

### `(string, float) GetNumeroAndSolde(string numero)`

```csharp
var (num, solde) = dao.GetNumeroAndSolde("101");
Console.WriteLine(num != null
    ? $"Numero={num}, Solde={solde}"
    : "Introuvable (ValueTuple)");
```

**Implémentation :**

```csharp
public (string, float) GetNumeroAndSolde(string numero)
{
    var res = context.CompteEntities
        .AsNoTracking()
        .Where(c => c.Numero.Equals(numero))
        .Select(c => new { c.Numero, c.Solde })
        .FirstOrDefault();
    return res == null ? (null, 0f) : (res.Numero, res.Solde);
}
```

### `Tuple<string,float> GetNumeroAndSoldeTuple(string numero)`

```csharp
var t = dao.GetNumeroAndSoldeTuple("101");
Console.WriteLine(t != null
    ? $"Tuple -> Numero={t.Item1}, Solde={t.Item2}"
    : "Introuvable (Tuple)");
```

**Implémentation (⚠️ `.AsEnumerable()` avant `new Tuple`)** :

```csharp
public Tuple<string, float> GetNumeroAndSoldeTuple(string numero)
{
    var res = context.CompteEntities
        .AsNoTracking()
        .Where(c => c.Numero.Equals(numero))
        .Select(c => new { c.Numero, c.Solde })
        .AsEnumerable()                              // bascule LINQ-to-Objects
        .Select(x => new Tuple<string, float>(x.Numero, x.Solde))
        .FirstOrDefault();
    return res;
}
```



## 13.3 Agrégations (stats globales)

```csharp
Console.WriteLine(dao.GetInformationCompte());
```

**Implémentation (robuste si vide)** :

```csharp
public string GetInformationCompte()
{
    var sb = new StringBuilder();
    var soldes = context.CompteEntities.AsNoTracking().Select(c => c.Solde);

    int count = soldes.Count();
    sb.AppendLine("Information sur les comptes :");
    sb.AppendLine("Nombre de comptes : " + count);
    sb.AppendLine("Total des soldes : " + (count > 0 ? soldes.Sum() : 0f));
    sb.AppendLine("Moyenne des soldes : " + (count > 0 ? soldes.Average() : 0f));
    sb.AppendLine("Solde minimum : " + (count > 0 ? soldes.Min() : 0f));
    sb.AppendLine("Solde maximum : " + (count > 0 ? soldes.Max() : 0f));
    return sb.ToString();
}
```



## 13.4 Bonnes pratiques Lecture

* **Toujours** gérer le cas **introuvable** (`null`).
* **Projections** (`Select`) pour ne tirer que **ce qui est utile**.
* **AsNoTracking** pour lecture pure (grands volumes, APIs read-mostly).
* **Index** sur les colonnes de recherche (ex. `Numero`) si volumétrie importante.



# 14. CRUD — **Create (insertion)** super exhaustif

> **Objectif**
> Insérer un nouveau `Compte` et persister avec `SaveChanges()` ; voir les erreurs fréquentes (unicité, NOT NULL, longueurs).



## 14.1 API

```csharp
public bool AjouterCompte(Compte cpt)
{
    context.CompteEntities.Add(cpt);
    int res = context.SaveChanges();
    return res > 0;
}
```

## 14.2 Test d’insertion dans `Program.cs`

```csharp
// --- Insertion ---
var nouveau = new Compte("110", DateTime.Now, 100);
bool okInsert = daoBD.AjouterCompte(nouveau);
Console.WriteLine(okInsert ? "Insertion OK" : "Insertion KO");

// Vérification
var lu = daoBD.GetCompte("110");
Console.WriteLine(lu != null ? $"Lu après insert: {lu}" : "Non trouvé après insert");
```

### Résultats attendus

* `Insertion OK`
* `Lu après insert: Numero : 110 Solde : 100 ...`

## 14.3 Erreurs fréquentes (Create)

* **Violation d’unicité** (si index unique sur `Numero`) → `DbUpdateException`.
  ➜ Gérer *en amont* (vérifier existence) **ou** capturer et expliquer.
* **Longueur** (ex. `Numero` > 20) → exception côté SQL.
  ➜ Valider côté C# avant `Add`.
* **Champs requis** (`[Required]`) vides → exception.



# 15. CRUD — **Delete (suppression)** super exhaustif

> **Objectif**
> Supprimer un compte par son `Numero`. Gérer le cas où il n’existe pas.


## 15.1 API

```csharp
public bool SupprimerCompte(string numero)
{
    var cpt = context.CompteEntities.FirstOrDefault(c => c.Numero == numero);
    if (cpt == null) return false;
    context.CompteEntities.Remove(cpt);
    return context.SaveChanges() > 0;
}
```

## 15.2 Test de suppression

```csharp
// --- Suppression ---
bool okDel = daoBD.SupprimerCompte("110");
Console.WriteLine(okDel ? "Suppression OK" : "Suppression KO / compte inexistant");

// Vérification
var afterDel = daoBD.GetCompte("110");
Console.WriteLine(afterDel == null ? "Supprimé (confirmé)" : "Toujours présent !");
```

### Notes

* Si **relations** (FK) existaient, la suppression pourrait échouer (contrainte FK).
  ➜ Configurer **cascade** ou supprimer enfants d’abord (non concerné ici).



# 16. CRUD — **Update (modification)** super exhaustif

> **Objectif**
> Modifier le solde d’un compte. Gérer les entités **détachées** vs **suivies**.



## 16.1 API (rappel)

```csharp
public void ModifierCompte(Compte cpt, float montant)
{
    // Sécuriser : recharger l’entité “suivie” depuis le contexte
    var tracked = context.CompteEntities
                         .FirstOrDefault(x => x.Numero == cpt.Numero);
    if (tracked == null) return;

    tracked.Solde += montant;
    context.SaveChanges();
}
```

> **Pourquoi recharger ?**
>
> * Si `cpt` vient d’une lecture avec **`AsNoTracking()`**, EF **ne le suit pas**.
> * Recharger garantit que l’entité est **attachée** et traçable.

### Variante (si on sait que `cpt` est détaché) :

```csharp
public void ModifierCompteDetache(Compte cpt, float montant)
{
    context.CompteEntities.Attach(cpt);            // attache l’entité
    cpt.Solde += montant;
    context.Entry(cpt).State = EntityState.Modified; // marque comme modifiée
    context.SaveChanges();
}
```

> ⚠️ Attention : `Attach` suppose que `cpt` représente l’**état actuel** en base.
> Avec une **clé** seulement, on met plutôt à jour champ par champ après **Find**/`FirstOrDefault`.



## 16.2 Test de modification

```csharp
// --- Update ---
var c101 = daoBD.GetCompte("101");       // lecture (AsNoTracking)
Console.WriteLine("Avant modif -> " + c101);

daoBD.ModifierCompte(c101, 200);         // +200

var c101b = daoBD.GetCompte("101");
Console.WriteLine("Après modif -> " + c101b);
```

### Résultat attendu

* Le `Solde` de `101` a augmenté de `200`.



## 16.3 Transactions (bonus)

Pour grouper plusieurs opérations :

```csharp
using (var tx = daoBD.BeginTransaction()) // => expose une méthode wrapper si besoin
{
    try
    {
        daoBD.AjouterCompte(new Compte("120", DateTime.Now, 50));
        // … autres opérations …
        tx.Commit();
    }
    catch
    {
        tx.Rollback();
        throw;
    }
}
```

Dans `CompteDaoBD`, tu peux exposer :

```csharp
public DbContextTransaction BeginTransaction()
{
    return context.Database.BeginTransaction();
}
```



# 17. **SQL natif** — `Database.SqlQuery<T>` super exhaustif

> **Objectif**
> Exécuter une requête SQL **paramétrée** et **hydrater** nos entités (ou DTOs).



## 17.1 Lecture d’un compte via SQL

```csharp
public Compte GetCompteDirectSql(string numero)
{
    string[] p = { numero };
    string query =
        "SELECT IdCompte as CompteId, " +
        "       C_Numero as Numero, " +
        "       C_Solde as Solde, " +
        "       C_DateCreation as DateCreation " +
        "FROM dbo.T_Comptes WHERE C_Numero = @p0";

    return context.Database.SqlQuery<Compte>(query, p).FirstOrDefault();
}
```

### Points clés

* **Paramètres** → `@p0`, `@p1`, … (évite l’injection SQL).
* **Alias** des colonnes vers les **noms des propriétés C#** (`as CompteId`, etc.).
* Retourne des `Compte` **détachés** (non suivis par le change-tracker).

## 17.2 Test

```csharp
var viaSql = daoBD.GetCompteDirectSql("101");
Console.WriteLine(viaSql != null ? $"SQL natif -> {viaSql}" : "Introuvable (SQL)");
```



## 17.3 Quand préférer le SQL natif ?

* **Requêtes complexes** (CTE, window functions, hints) difficiles à exprimer en LINQ.
* **Procédures stockées** (utiliser `Database.SqlQuery<T>` / `ExecuteSqlCommand`).
* **Optimisations** fines (index hints, option recompile…).
* **Rapports**/aggrégations multi-tables lourdes.

> Pour le **quotidien** : **LINQ** d’abord (lisible, typé, portable), SQL natif **quand nécessaire**.



# 18. Programme de test consolidé (CRUD + SQL)

> Active/désactive les blocs selon ce que tu veux tester.

```csharp
using System;
using DAODemoP1.DAO;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            using (var daoBD = new CompteDaoBD())
            {
                ICompteDao dao = daoBD;

                // ===== READ
                Console.WriteLine("=== READ ===");
                var c = dao.GetCompte("101");
                Console.WriteLine(c != null ? c.ToString() : "Introuvable 101");

                var (num, solde) = dao.GetNumeroAndSolde("101");
                Console.WriteLine(num != null
                    ? $"(ValueTuple) Numero={num} Solde={solde}"
                    : "Introuvable (ValueTuple)");

                var t = dao.GetNumeroAndSoldeTuple("101");
                Console.WriteLine(t != null
                    ? $"(Tuple) Numero={t.Item1} Solde={t.Item2}"
                    : "Introuvable (Tuple)");

                Console.WriteLine(dao.GetInformationCompte());

                // ===== CREATE
                Console.WriteLine("\n=== CREATE ===");
                var nouveau = new DAODemoP1.METIER.Compte("110", DateTime.Now, 100);
                Console.WriteLine(daoBD.AjouterCompte(nouveau) ? "Insertion OK" : "Insertion KO");

                // Vérif
                Console.WriteLine(dao.GetCompte("110") ?? (object)"Non trouvé après insert");

                // ===== UPDATE
                Console.WriteLine("\n=== UPDATE (+200) ===");
                var c110 = dao.GetCompte("110");
                if (c110 != null)
                {
                    daoBD.ModifierCompte(c110, 200);
                    Console.WriteLine("Après modif -> " + dao.GetCompte("110"));
                }

                // ===== SQL Natif
                Console.WriteLine("\n=== SQL natif ===");
                Console.WriteLine(daoBD.GetCompteDirectSql("110") ?? (object)"Introuvable (SQL)");

                // ===== DELETE
                Console.WriteLine("\n=== DELETE ===");
                Console.WriteLine(daoBD.SupprimerCompte("110") ? "Suppression OK" : "Suppression KO");
                Console.WriteLine(dao.GetCompte("110") == null ? "Supprimé (confirmé)" : "Toujours présent !");
            }

            Console.WriteLine("\nFIN CRUD/SQL");
            Console.ReadLine();
        }
    }
}
```



# 19. Check-lists & anti-bugs

## 19.1 Check-list CRUD (EF6)

* [ ] **Read** : `AsNoTracking()` utilisé pour lecture pure, projections OK, null géré
* [ ] **Create** : validations basiques (longueur, required), gestion unicité (si index unique)
* [ ] **Delete** : OK si inexistant, pas d’enfant/PK-FK bloquante
* [ ] **Update** : recharger entité suivie OU `Attach` + `Modified` (en connaissance de cause)
* [ ] **SaveChanges()** vérifié, exceptions catchées au besoin

## 19.2 Anti-bugs courants

* **Tuple EF** : `.AsEnumerable()` **avant** `new Tuple`
* **Séquence vide** : `Average/Min/Max` → protéger
* **Changements de schéma** : **Drop & Create** si tu n’utilises pas les migrations
* **Type `float`** : pour du vrai financier, passer à **`decimal`** + précision
* **Transactions** : grouper les opérations sensibles



# 20. Où on en est & suite

* Tu as un **Repository EF6** complet : lecture, agrégations, insertion, suppression, modification, et **SQL natif**.
* Tu peux **switcher** entre in-memory et base **à la volée**.
* **Prochaines améliorations possibles** :

  * **Mise en page**/formatage (affichages console),
  * **DTOs** dédiés pour les projections,
  * **Unit tests** (xUnit/NUnit) avec un **provider en mémoire** ou une base locale jetable,
  * **Concurrence optimiste** (timestamp / RowVersion),
  * **Migrations EF** pour gérer l’évolution du schéma.

