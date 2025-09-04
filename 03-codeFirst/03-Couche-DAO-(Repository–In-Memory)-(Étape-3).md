# 3. Couche DAO (Repository – **In-Memory**) — **Étape 3 super exhaustive**

> **Objectif**
> Isoler l’accès aux données derrière une **interface** (contrat) et une **implémentation** en mémoire.
> On prépare ainsi la transition vers EF **sans toucher** au code de l’UI/console ou de la couche métier.



## 3.0 Pourquoi un Repository/DAO ?

* **Découplage** : la couche métier parle à une **interface** (`ICompteDao`), pas à une base SQL concrète.
* **Testabilité** : on injecte une implémentation “mémoire” pour les tests (rapide, déterministe).
* **Évolutivité** : demain on remplace par `CompteDaoBD` (EF) **sans casser** le reste.



## 3.1 Créer l’interface `ICompteDao` (dossier `DAO`)

**Fichier** : `DAO/ICompteDao.cs`

```csharp
using DAODemoP1.METIER;

namespace DAODemoP1.DAO
{
    public interface ICompteDao
    {
        /// <summary>
        /// Retourne le compte dont le Numéro correspond, ou null si introuvable.
        /// </summary>
        Compte GetCompte(string numero);
    }
}
```

**Notes importantes**

* **Contrat minimal** à ce stade : 1 seule méthode.
* On **autorise** `null` si non trouvé (c’est le comportement avec `FirstOrDefault()`).
* On ne dépend **que** de la couche METIER (normal).



## 3.2 Implémentation “mémoire” `CompteDao` (dossier `DAO`)

**Fichier** : `DAO/CompteDao.cs`

```csharp
using System.Collections.Generic;
using System.Linq;
using DAODemoP1.METIER;
using DAODemoP1.UTILEMETIER;

namespace DAODemoP1.DAO
{
    public class CompteDao : ICompteDao
    {
        public List<Compte> ListeComptes { get; }

        /// <summary>
        /// Remplit avec les données de démonstration (Utile.GetComptes()).
        /// </summary>
        public CompteDao()
        {
            ListeComptes = Utile.GetComptes();
        }

        /// <summary>
        /// Injection d'une liste personnalisée (utile pour tests).
        /// </summary>
        public CompteDao(List<Compte> listeComptes)
        {
            ListeComptes = listeComptes ?? new List<Compte>();
        }

        /// <inheritdoc />
        public Compte GetCompte(string numero)
        {
            // FirstOrDefault() : renvoie null si aucun élément
            return ListeComptes
                    .Where(cpt => cpt.Numero.Equals(numero))
                    .FirstOrDefault();
        }
    }
}
```

### 3.2.1 `First()` vs `FirstOrDefault()` (⚠️ à connaître)

* `First(predicate)` → **exception** si aucun élément.
* `FirstOrDefault(predicate)` → **null** (références) si aucun élément.
  Ici, on préfère **`FirstOrDefault`** et on gère `null` côté appelant.



## 3.3 Test rapide (dans `Program.cs`)

```csharp
using System;
using DAODemoP1.DAO;
using DAODemoP1.METIER;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            ICompteDao dao = new CompteDao();

            Compte c = dao.GetCompte("101");
            if (c != null)
                Console.WriteLine(c);
            else
                Console.WriteLine("Compte 101 introuvable");

            Console.WriteLine("Étape 3 OK");
            Console.ReadLine();
        }
    }
}
```

**Attendu** : l’objet `Compte` 101 est affiché.



## 3.4 Pièges fréquents

* **`NullReferenceException`** : si tu fais `dao.GetCompte("xxx").Numero` et que le compte n’existe pas → `null.Numero` **plante**.
  → **Vérifie `null`** avant d’accéder aux membres.
* **`Equals` vs `==`** sur `string` : les deux comparent la valeur, mais **`Equals`** explicite l’intention.
* **Casse/sensibilité** : `Equals` par défaut est sensible à la casse. Pour ignorer la casse :
  `cpt.Numero.Equals(numero, StringComparison.OrdinalIgnoreCase)`.



## 3.5 Check-list de sortie (Étape 3)

* [ ] `DAO/ICompteDao.cs` créé avec `GetCompte(string)`
* [ ] `DAO/CompteDao.cs` créé (mémoire, liste 101–107 via `Utile`)
* [ ] Test console OK (affiche le compte 101)



# 4. Extraction ciblée de champs (In-Memory) — **Étape 4 super exhaustive**

> **Objectif**
> Exposer des **vues** partielles de l’objet (Numéro/Solde) pour limiter le couplage et réduire les transferts.



## 4.0 Valeur tuple vs `Tuple<T1,T2>`

* **Value Tuple** `(string, float)` (C# 7+)

  * Déconstruction conviviale : `(numero, solde) = dao.GetNumeroAndSolde("101");`
  * Type léger, lisible.
* **Classe** `Tuple<string,float>`

  * API .NET historique (`Item1`, `Item2`)
  * Moins lisible mais connue.

> Le cours te fait pratiquer **les deux**.



## 4.1 Ajouter `(string,float) GetNumeroAndSolde(string)` à l’interface

**Fichier** : `DAO/ICompteDao.cs`

```csharp
public interface ICompteDao
{
    Compte GetCompte(string numero);

    // Étape 4 (P1) : extraction partielle via Value Tuple
    (string, float) GetNumeroAndSolde(string numero);
}
```



## 4.2 Implémentation `(string,float)` dans `CompteDao`

**Fichier** : `DAO/CompteDao.cs`

```csharp
public (string, float) GetNumeroAndSolde(string numero)
{
    var res = ListeComptes
                .Where(cpt => cpt.Numero.Equals(numero))
                .Select(cpt => new { cpt.Numero, cpt.Solde })
                .FirstOrDefault();

    // ⚠️ res peut être null si le compte n'existe pas !
    // La version "strictement conforme au cours" suppose une donnée existante.
    // Version plus robuste :
    if (res == null) return (null, 0f); // ou throw, ou pattern TryGet... au choix

    return (res.Numero, res.Solde);
}
```

> 🔒 **Robustesse** (optionnelle et conseillée)
> Exposer un **Try-pattern** pour éviter les `null` :
>
> ```csharp
> public bool TryGetNumeroAndSolde(string numero, out string num, out float solde)
> {
>     var res = ListeComptes
>         .Where(cpt => cpt.Numero.Equals(numero))
>         .Select(cpt => new { cpt.Numero, cpt.Solde })
>         .FirstOrDefault();
>     if (res == null) { num = null; solde = 0; return false; }
>     num = res.Numero; solde = res.Solde; return true;
> }
> ```



## 4.3 Test `(string,float)` dans `Program.cs`

```csharp
(string numero, float solde) = dao.GetNumeroAndSolde("101");
if (numero != null)
    Console.WriteLine($"Numero: {numero}  Solde: {solde}");
else
    Console.WriteLine("Compte introuvable (GetNumeroAndSolde).");
```


## 4.4 Ajouter `Tuple<string,float> GetNumeroAndSoldeTuple(string)` à l’interface

**Fichier** : `DAO/ICompteDao.cs`

```csharp
public interface ICompteDao
{
    Compte GetCompte(string numero);
    (string, float) GetNumeroAndSolde(string numero);

    // Étape 4 (P2) : extraction partielle via Tuple<T1,T2>
    Tuple<string, float> GetNumeroAndSoldeTuple(string numero);
}
```



## 4.5 Implémentation `Tuple<string,float>` dans `CompteDao`

```csharp
public Tuple<string, float> GetNumeroAndSoldeTuple(string numero)
{
    var res = ListeComptes
                .Where(cpt => cpt.Numero.Equals(numero))
                .Select(cpt => new Tuple<string, float>(cpt.Numero, cpt.Solde))
                .FirstOrDefault();

    // res peut être null si introuvable
    return res;
}
```



## 4.6 Test `Tuple<string,float>` dans `Program.cs`

```csharp
var t = dao.GetNumeroAndSoldeTuple("101");
if (t != null)
    Console.WriteLine($"Tuple -> Numero: {t.Item1}  Solde: {t.Item2}");
else
    Console.WriteLine("Compte introuvable (Tuple).");
```


## 4.7 Bonnes pratiques & variantes

* **Projection** `.Select` : on renvoie uniquement le nécessaire (`Numero`, `Solde`).
* **Erreurs d’absence** : définir une **convention** d’équipe (null, exception, Try-pattern…).
* **Nommer les elements du Value Tuple** :

  ```csharp
  public (string Numero, float Solde) GetNumeroAndSolde(string numero) { ... }
  // Appel : var (numero, solde) = dao.GetNumeroAndSolde("101");
  ```
* **Culture/format** : si tu affiches des montants, pense au formatage (`ToString("N2")` par ex.).



## 4.8 Check-list de sortie (Étape 4)

* [ ] `ICompteDao` enrichie avec 2 méthodes d’extraction
* [ ] `CompteDao` implémente ces 2 méthodes (Value Tuple + Tuple\<T1,T2>)
* [ ] Tests console OK (affichent Numéro/Solde)



# 5. Agrégations LINQ (In-Memory) — **Étape 5 super exhaustive**

> **Objectif**
> Produire un **résumé** des comptes (compte, somme, moyenne, min, max) via LINQ.


## 5.1 Ajouter `string GetInformationCompte()` à l’interface

**Fichier** : `DAO/ICompteDao.cs`

```csharp
public interface ICompteDao
{
    Compte GetCompte(string numero);
    (string, float) GetNumeroAndSolde(string numero);
    Tuple<string, float> GetNumeroAndSoldeTuple(string numero);

    // Étape 5 : agrégations
    string GetInformationCompte();
}
```



## 5.2 Implémentation `GetInformationCompte()` dans `CompteDao`

**Fichier** : `DAO/CompteDao.cs`

```csharp
using System.Text;

// ...

public string GetInformationCompte()
{
    var sb = new StringBuilder();
    sb.AppendLine("Information sur les comptes :");

    int count = ListeComptes.Count;
    sb.AppendLine("Nombre de comptes : " + count);

    // ⚠️ Average() sur séquence vide -> InvalidOperationException
    // Ici on a des données, mais soyez robustes si la liste est vide.
    float total = count > 0 ? ListeComptes.Sum(c => c.Solde) : 0f;
    sb.AppendLine("Total des soldes : " + total);

    float moyenne = count > 0 ? ListeComptes.Average(c => c.Solde) : 0f;
    sb.AppendLine("Moyenne des soldes : " + moyenne);

    float min = count > 0 ? ListeComptes.Min(c => c.Solde) : 0f;
    sb.AppendLine("Solde minimum : " + min);

    float max = count > 0 ? ListeComptes.Max(c => c.Solde) : 0f;
    sb.AppendLine("Solde maximum : " + max);

    return sb.ToString();
}
```

> 🔐 **Robustesse**
> Sur une liste **vide**, `Average`, `Min`, `Max` **lèvent** une exception.
> On protège en testant `count > 0` (ou via `DefaultIfEmpty(0f)` pour `Average`).

**Variante** (formatage soigné) :

```csharp
sb.AppendLine("Moyenne des soldes : " + moyenne.ToString("N2")); // 2 décimales
```


## 5.3 Test dans `Program.cs`

```csharp
Console.WriteLine(dao.GetInformationCompte());
```

**Exemple de sortie**

```
Information sur les comptes :
Nombre de comptes : 7
Total des soldes : 1300
Moyenne des soldes : 185,7143
Solde minimum : 0
Solde maximum : 500
```

> (Le séparateur `,`/`.` dépend de la culture système.)



## 5.4 Bonus : retourner un **objet** plutôt qu’une **string**

Pour un usage UI/API plus riche, crée un DTO :

```csharp
public class StatsComptes
{
    public int Nombre { get; set; }
    public float Total { get; set; }
    public float Moyenne { get; set; }
    public float Min { get; set; }
    public float Max { get; set; }
}
```

Et ajoute à l’interface :

```csharp
StatsComptes GetStats();
```

Implémentation :

```csharp
public StatsComptes GetStats()
{
    int count = ListeComptes.Count;
    return new StatsComptes
    {
        Nombre = count,
        Total = count > 0 ? ListeComptes.Sum(c => c.Solde) : 0f,
        Moyenne = count > 0 ? ListeComptes.Average(c => c.Solde) : 0f,
        Min = count > 0 ? ListeComptes.Min(c => c.Solde) : 0f,
        Max = count > 0 ? ListeComptes.Max(c => c.Solde) : 0f
    };
}
```



## 5.5 Programme de test **consolidé** (Étapes 3→5)

```csharp
using System;
using DAODemoP1.DAO;
using DAODemoP1.METIER;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            ICompteDao dao = new CompteDao(); // mémoire

            // --- Étape 3 : GetCompte
            var c101 = dao.GetCompte("101");
            Console.WriteLine(c101 != null ? c101.ToString() : "Compte 101 introuvable");

            // --- Étape 4 (P1) : Value Tuple
            var (numero, solde) = dao.GetNumeroAndSolde("101");
            Console.WriteLine(numero != null
                ? $"(tuple) Numero: {numero}  Solde: {solde}"
                : "Compte introuvable (ValueTuple)");

            // --- Étape 4 (P2) : Tuple<T1,T2>
            var t = dao.GetNumeroAndSoldeTuple("101");
            Console.WriteLine(t != null
                ? $"(Tuple) Numero: {t.Item1}  Solde: {t.Item2}"
                : "Compte introuvable (Tuple)");

            // --- Étape 5 : Agrégations
            Console.WriteLine(dao.GetInformationCompte());

            Console.WriteLine("Étapes 3→5 OK");
            Console.ReadLine();
        }
    }
}
```



## 5.6 Pièges & bonnes pratiques

* **Séquence vide** : protéger `Average/Min/Max`.
* **Nulls** : gérer les retours `null` des méthodes de recherche (logique d’erreur claire).
* **Performance** (in-memory) : `Where+FirstOrDefault` est OK ; inutile d’optimiser avant la vraie BD.
* **Clarté** : préfère `(string Numero, float Solde)` (tuples nommés) pour une API plus lisible.



## 5.7 Check-list de sortie (Étape 5)

* [ ] `ICompteDao` étendue avec `GetNumeroAndSolde`, `GetNumeroAndSoldeTuple`, `GetInformationCompte`
* [ ] `CompteDao` implémente ces méthodes (LINQ + `StringBuilder`)
* [ ] Tests console OK (affichent extraction + agrégations)



> ✅ **Étapes 3→5 terminées.**
> **Prochaine étape (6→10)** : **Entity Framework 6**
>
> * Installer EF6 (NuGet),
> * créer `ContextEf`,
> * configurer `App.config` (`connectionStrings`),
> * initialiser la BD et les tables (`CreateTable`, `InitCompte`),
> * mapping **Fluent API** *ou* **Annotations**.
