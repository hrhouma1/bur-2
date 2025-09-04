# 11. DAO (Repository) **côté Base de données** — **Étape 11 super exhaustive**

> **Objectif**
> Fournir une implémentation de `ICompteDao` branchée sur **Entity Framework 6** (LINQ to Entities) :
>
> * Recherches (lecture) : `GetCompte`, `(string,float) GetNumeroAndSolde`, `Tuple<,> GetNumeroAndSoldeTuple`
> * Agrégations : `GetInformationCompte()`
> * (+ déjà prêt pour les prochaines étapes) **CRUD** (`Ajouter`, `Supprimer`, `Modifier`) et **SQL natif** (on les détaillera aux Étapes 12–17)



## 11.0 Rappel : interface (contrat)

**Fichier** : `DAO/ICompteDao.cs` (déjà créé aux Étapes 3–5)

```csharp
using DAODemoP1.METIER;
using System;

namespace DAODemoP1.DAO
{
    public interface ICompteDao
    {
        Compte GetCompte(string numero);
        (string, float) GetNumeroAndSolde(string numero);
        Tuple<string, float> GetNumeroAndSoldeTuple(string numero);
        string GetInformationCompte();
    }
}
```

> On garde l’interface **inchangée** pour que l’appelant puisse **switcher** entre mémoire (`CompteDao`) et base (`CompteDaoBD`) **sans toucher** au code appelant.



## 11.1 Implémentation EF : `CompteDaoBD`

**Fichier** : `DAO/CompteDaoBD.cs`

```csharp
using System;
using System.Linq;
using System.Text;
using System.Collections.Generic;
using System.Data.Entity;              // EF6
using DAODemoP1.METIER;
using DAODemoP1.EF;

namespace DAODemoP1.DAO
{
    /// <summary>
    /// Repository EF6 : accès à la base via ContextEf.
    /// Implémente les méthodes de l'interface + CRUD (Étapes 12–16) + SQL natif (Étape 17).
    /// </summary>
    public class CompteDaoBD : ICompteDao, IDisposable
    {
        private readonly ContextEf context;

        public CompteDaoBD()
        {
            context = new ContextEf();
        }

        /// <summary>
        /// Recherche d’un compte par numéro. Retourne null si introuvable.
        /// </summary>
        public Compte GetCompte(string numero)
        {
            // Read-only ? -> AsNoTracking() pour éviter le coût du change-tracker
            return context.CompteEntities
                          .AsNoTracking()
                          .Where(cpt => cpt.Numero.Equals(numero))
                          .FirstOrDefault();
        }

        /// <summary>
        /// Extraction partielle (ValueTuple) : (Numero, Solde). Numero = null si introuvable.
        /// </summary>
        public (string, float) GetNumeroAndSolde(string numero)
        {
            var res = context.CompteEntities
                             .AsNoTracking()
                             .Where(cpt => cpt.Numero.Equals(numero))
                             .Select(cpt => new { cpt.Numero, cpt.Solde })
                             .FirstOrDefault();
            return res == null ? (null, 0f) : (res.Numero, res.Solde);
        }

        /// <summary>
        /// Extraction partielle (Tuple class). Attention : Tuple<> non traduisible par EF -> AsEnumerable().
        /// </summary>
        public Tuple<string, float> GetNumeroAndSoldeTuple(string numero)
        {
            var res = context.CompteEntities
                             .AsNoTracking()
                             .Where(cpt => cpt.Numero.Equals(numero))
                             .Select(cpt => new { cpt.Numero, cpt.Solde })
                             .AsEnumerable() // bascule LINQ to Objects (EF ne sait pas construire Tuple<> en SQL)
                             .Select(x => new Tuple<string, float>(x.Numero, x.Solde))
                             .FirstOrDefault();
            return res; // peut être null
        }

        /// <summary>
        /// Agrégations sur les comptes (Count, Sum, Avg, Min, Max).
        /// Protège contre les séquences vides.
        /// </summary>
        public string GetInformationCompte()
        {
            var sb = new StringBuilder();
            sb.AppendLine("Information sur les comptes :");

            // Pour éviter plusieurs allers-retours, on peut charger les soldes seulement
            var soldes = context.CompteEntities.AsNoTracking().Select(c => c.Solde);

            int count = soldes.Count();
            sb.AppendLine("Nombre de comptes : " + count);

            float total = count > 0 ? soldes.Sum() : 0f;
            sb.AppendLine("Total des soldes : " + total);

            float moyenne = count > 0 ? soldes.Average() : 0f;
            sb.AppendLine("Moyenne des soldes : " + moyenne);

            float min = count > 0 ? soldes.Min() : 0f;
            sb.AppendLine("Solde minimum : " + min);

            float max = count > 0 ? soldes.Max() : 0f;
            sb.AppendLine("Solde maximum : " + max);

            return sb.ToString();
        }

        // ==============================
        // CRUD (détaillés aux Étapes 12–16)
        // ==============================

        /// <summary>Insertion d’un compte.</summary>
        public bool AjouterCompte(Compte cpt)
        {
            context.CompteEntities.Add(cpt);
            return context.SaveChanges() > 0;
        }

        /// <summary>Suppression par numéro. Retourne false si le compte n’existe pas.</summary>
        public bool SupprimerCompte(string numero)
        {
            var cpt = context.CompteEntities.Where(c => c.Numero == numero).FirstOrDefault();
            if (cpt == null) return false;
            context.CompteEntities.Remove(cpt);
            return context.SaveChanges() > 0;
        }

        /// <summary>Modification simple : incrémente le solde d’un compte existant.</summary>
        public void ModifierCompte(Compte cpt, float montant)
        {
            // Si cpt vient de GetCompte(AsNoTracking), il n’est pas attaché : on peut l’attacher.
            // Ici, on récupère l’entité suivie pour être sûrs :
            var tracked = context.CompteEntities.Where(x => x.Numero == cpt.Numero).FirstOrDefault();
            if (tracked == null) return;
            tracked.Solde += montant;
            context.SaveChanges();
        }

        // ==============================
        // SQL natif (Étape 17)
        // ==============================

        /// <summary>
        /// Lecture via SQL natif paramétré (alias sur colonnes pour binder sur Compte).
        /// </summary>
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

        // ==============================
        // Gestion du cycle de vie
        // ==============================

        public void Dispose()
        {
            context?.Dispose();
        }
    }
}
```

### Points clés de cette implémentation

* **AsNoTracking()** pour les **lectures** (pas de modification prévue) → **moins de surcharge**.
  Si tu veux **modifier** l’entité retournée plus tard, **enlève** `AsNoTracking()` ou **rattache** l’entité (cf. `ModifierCompte`).
* **Tuple<> et EF6** : EF6 **ne sait pas** traduire `new Tuple<...>(...)` **en SQL** → il faut **basculer en mémoire** avec `.AsEnumerable()` **avant** de construire le `Tuple`.
* **Agrégations robustes** : protège `Average/Min/Max` quand la séquence est **vide** (sinon exception).
* **Dispose** : le `DbContext` tient des connexions et un change-tracker → implémente **`IDisposable`** pour libérer proprement.



## 11.2 Utilisation dans `Program.cs` (switch mémoire / base)

```csharp
using System;
using DAODemoP1.DAO;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            // Choix du backend :
            // ICompteDao dao = new CompteDao();      // In-memory
            using (var daoBD = new CompteDaoBD())     // EF (base)
            {
                ICompteDao dao = daoBD;

                var c = dao.GetCompte("101");
                Console.WriteLine(c != null ? c.ToString() : "Compte 101 introuvable");

                var (num, solde) = dao.GetNumeroAndSolde("101");
                Console.WriteLine(num != null
                    ? $"(ValueTuple) Numero={num} Solde={solde}"
                    : "Introuvable (ValueTuple)");

                var t = dao.GetNumeroAndSoldeTuple("101");
                Console.WriteLine(t != null
                    ? $"(Tuple) Numero={t.Item1} Solde={t.Item2}"
                    : "Introuvable (Tuple)");

                Console.WriteLine(dao.GetInformationCompte());
            }

            Console.WriteLine("Étape 11 OK");
            Console.ReadLine();
        }
    }
}
```

> Si tu lances ça **après** l’Étape 10 (seed), tu dois **retrouver** les données 101–107.



## 11.3 Bonnes pratiques (EF6 Repository)

* **Durée de vie du `DbContext`** : court et maîtrisé (ici : par instance de DAO).
  Tu peux aussi **ouvrir/fermer** un `ContextEf` **par méthode** si besoin.
* **AsNoTracking** pour **toutes les lectures** *read-only*.
* **Projections** (`Select`) pour **ne renvoyer** que ce qui est utile (ex. `(Numero, Solde)`).
* **Exceptions SQL** : enveloppe éventuellement dans tes propres exceptions métiers.
* **Transactions** (opérations multi-rows) : utilise `context.Database.BeginTransaction()` quand on y sera (Étapes CRUD).
* **Index** sur `Numero` si requêtes fréquentes (vu Étapes 8/9).



## 11.4 Erreurs fréquentes & solutions

1. **`InvalidOperationException: The method 'New Tuple' is not supported…`**
   → Tu as construit un `Tuple<>` **avant** `.AsEnumerable()` → déplace `.AsEnumerable()` **avant** la création du `Tuple`.
2. **`NullReferenceException`** côté appelant
   → Gère le cas **introuvable** (retours `null` / `Numero==null`), ne fais pas `dao.GetCompte("xxx").Numero`.
3. **`DbUpdateException`** à l’insertion
   → Violations de contraintes (clé unique si tu l’as ajoutée, longueur max, NOT NULL…).
4. **`MultipleActiveResultSets`** (rare ici)
   → Si tu enchaînes des lectures simultanées sur la **même** connexion sans MARS activé → close/read avant nouvelle requête.
5. **Perf**
   → Ajoute **index** (`Numero`), utilise **AsNoTracking**, **Select** ciblés.



## 11.5 Check-list de sortie (Étape 11)

* [ ] `DAO/CompteDaoBD.cs` créé et **compile**
* [ ] Méthodes **lecture** OK : `GetCompte`, `GetNumeroAndSolde`, `GetNumeroAndSoldeTuple`
* [ ] **Agrégations** OK : `GetInformationCompte()` (robuste si vide)
* [ ] **CRUD** déjà codé (on les pratiquera en Étapes 12–16)
* [ ] **SQL natif** prêt (`GetCompteDirectSql`) pour Étape 17
* [ ] Test `Program.cs` basculant **mémoire** ⇄ **base** OK



> ✅ **Étape suivante (12)** : **CRUD – Introduction & scénarios (Read)**
> On va détailler et tester **pas à pas** :
> 12.1 Sélection du backend & scénarios de lecture
> 13\. **Read** approfondi (filtres, NoTracking, pagination)
> 14\. **Create** (insertion), 15. **Delete** (suppression), 16. **Update** (modification)
> 17\. **SQL natif** avec `Database.SqlQuery<T>`
>

