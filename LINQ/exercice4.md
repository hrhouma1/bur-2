
# Préparation (projet console)

* Ciblez .NET 6+ (fonctionnera aussi en .NET Framework avec `Program` classique).
* Ajoutez `using System; using System.Linq; using System.Collections.Generic;`
* Les exemples ci-dessous utilisent parfois un petit modèle de données.

```csharp
using System;
using System.Linq;
using System.Collections.Generic;

namespace DemoLinq
{
    class Etudiant
    {
        public int Id { get; set; }
        public string Nom { get; set; } = "";
        public int Age { get; set; }
        public string Ville { get; set; } = "";
        public List<int> Notes { get; set; } = new();
        public DateTime Inscription { get; set; }
    }

    class Cours
    {
        public int Id { get; set; }
        public string Titre { get; set; } = "";
        public int Credits { get; set; }
    }

    class Inscription
    {
        public int EtudiantId { get; set; }
        public int CoursId { get; set; }
        public double NoteFinale { get; set; }
    }
}
```

Jeu de données commun (à copier une fois et réutiliser) :

```csharp
// Listes simples
var entiers = new List<int> { 1, 2, 3, 4, 5, 3, 8, 10, 10, 11, 12 };
var prenoms = new List<string> { "Amine", "Sara", "Lina", "Omar", "Nora", "Adam", "Sami" };

// Modèle relationnel simple
var etudiants = new List<Etudiant>
{
    new() { Id=1, Nom="Amine", Age=21, Ville="Montréal",  Notes=new(){12,14,16}, Inscription=new DateTime(2025,01,10)},
    new() { Id=2, Nom="Sara",  Age=23, Ville="Laval",     Notes=new(){10,15,18}, Inscription=new DateTime(2025,02,05)},
    new() { Id=3, Nom="Lina",  Age=20, Ville="Montréal",  Notes=new(){17,17,19}, Inscription=new DateTime(2025,03,12)},
    new() { Id=4, Nom="Omar",  Age=22, Ville="Longueuil", Notes=new(){9,12,13},  Inscription=new DateTime(2025,01,25)},
    new() { Id=5, Nom="Nora",  Age=24, Ville="Montréal",  Notes=new(){18,18,19}, Inscription=new DateTime(2025,04,01)}
};

var cours = new List<Cours>
{
    new() { Id=101, Titre="Programmation C#",   Credits=3 },
    new() { Id=102, Titre="Bases de données",   Credits=3 },
    new() { Id=103, Titre="Réseaux",            Credits=2 },
};

var inscriptions = new List<Inscription>
{
    new() { EtudiantId=1, CoursId=101, NoteFinale=15.0 },
    new() { EtudiantId=1, CoursId=102, NoteFinale=16.0 },
    new() { EtudiantId=2, CoursId=101, NoteFinale=14.0 },
    new() { EtudiantId=3, CoursId=103, NoteFinale=18.5 },
    new() { EtudiantId=4, CoursId=102, NoteFinale=11.0 },
    new() { EtudiantId=5, CoursId=101, NoteFinale=19.0 },
    new() { EtudiantId=5, CoursId=103, NoteFinale=18.0 },
};
```

---

# Partie A – Bases LINQ et exécution (immédiate vs différée)

## Exercice 1 — Filtrer les pairs (immédiat)

**Objectif** : Comprendre comment `ToList()` force l’exécution.
**Énoncé** : À partir de `entiers`, sélectionnez uniquement les nombres pairs, **exécution immédiate**.
**Attendu** : Une `List<int>` contenant 2, 4, 8, 10, 10, 12.

**Correction (exemple)**

```csharp
List<int> pairs = (from n in entiers
                   where n % 2 == 0
                   select n).ToList();   // ToList => exécution immédiate
```

---

## Exercice 2 — Filtrer les pairs (différé)

**Objectif** : Voir l’évaluation différée.
**Énoncé** : Créez une requête différée qui sélectionne les pairs **sans** `ToList()`. Ajoutez ensuite `entiers.Add(100)` avant l’itération.
**Attendu** : L’itération doit inclure 100 (preuve de l’exécution différée).

**Correction (exemple)**

```csharp
var pairsQuery = from n in entiers
                 where n % 2 == 0
                 select n;               // différé

entiers.Add(100);                        // modifie la source après définition
foreach (var v in pairsQuery) Console.Write($"{v} "); // inclura 100
```

---

## Exercice 3 — Compter vs matérialiser

**Objectif** : Différence entre `Count()` (exécution immédiate) et `ToList().Count`.
**Énoncé** : Calculez le nombre de pairs directement avec `Count()` puis via `ToList().Count`.
**Attendu** : Même valeur, mais deux façons d’exécuter.

**Correction (exemple)**

```csharp
int c1 = entiers.Count(n => n % 2 == 0);         // exécution immédiate
int c2 = entiers.Where(n => n % 2 == 0).ToList().Count; // matérialise puis compte
```

---

## Exercice 4 — Projections simples

**Objectif** : Utiliser `Select`.
**Énoncé** : Projetez `entiers` en leurs carrés, version méthode et version requête.
**Attendu** : 1,4,9,16,…

**Correction (exemple)**

```csharp
var carres1 = entiers.Select(n => n * n);
var carres2 = from n in entiers select n * n;
```

---

## Exercice 5 — Distinct, tri et prise partielle

**Objectif** : Chaînage d’opérateurs.
**Énoncé** : Supprimez les doublons de `entiers`, triez par ordre croissant, puis prenez les 5 premiers.
**Attendu** : Séquence triée unique, 5 premiers éléments.

**Correction (exemple)**

```csharp
var top5 = entiers.Distinct()
                  .OrderBy(n => n)
                  .Take(5)
                  .ToList();
```

---

# Partie B – Opérations courantes sur objets

## Exercice 6 — Filtrer des objets et sélectionner des projections anonymes

**Objectif** : `Where`, `Select` avec types complexes.
**Énoncé** : À partir de `etudiants`, récupérer les noms et l’âge des étudiants de Montréal.
**Attendu** : Séquence d’objets anonymes `{ Nom, Age }`.

**Correction (exemple)**

```csharp
var montreal = etudiants
    .Where(e => e.Ville == "Montréal")
    .Select(e => new { e.Nom, e.Age });
```

---

## Exercice 7 — GroupBy (par ville)

**Objectif** : Regrouper et agréger.
**Énoncé** : Groupez les étudiants par `Ville` et affichez : `Ville`, `Effectif`.
**Attendu** : Une ligne par ville avec le nombre d’étudiants.

**Correction (exemple)**

```csharp
var groupes = etudiants
    .GroupBy(e => e.Ville)
    .Select(g => new { Ville = g.Key, Effectif = g.Count() });
```

---

## Exercice 8 — Moyenne des notes par étudiant

**Objectif** : `Select` sur sous-collections + agrégations.
**Énoncé** : Pour chaque étudiant, calculez la moyenne de `Notes` et triez par moyenne décroissante.
**Attendu** : `{ Nom, Moyenne }` triés.

**Correction (exemple)**

```csharp
var moyennes = etudiants
    .Select(e => new { e.Nom, Moyenne = e.Notes.Average() })
    .OrderByDescending(x => x.Moyenne);
```

---

## Exercice 9 — Aplatir avec SelectMany

**Objectif** : `SelectMany` pour dérouler des listes de listes.
**Énoncé** : Obtenez **toutes** les notes de **tous** les étudiants (séquence simple d’int). Filtrez les notes ≥ 15, triez décroissant, prenez 5.
**Attendu** : Top 5 meilleures notes globales.

**Correction (exemple)**

```csharp
var top5Notes = etudiants
    .SelectMany(e => e.Notes)
    .Where(n => n >= 15)
    .OrderByDescending(n => n)
    .Take(5)
    .ToList();
```

---

## Exercice 10 — Jointure interne Étudiants × Inscriptions × Cours

**Objectif** : `Join`/`GroupJoin`.
**Énoncé** : Construisez la liste `{ NomEtudiant, TitreCours, NoteFinale }` via jointure.
**Attendu** : Une ligne par inscription.

**Correction (exemple)**

```csharp
var bulletin = inscriptions
    .Join(etudiants,   i => i.EtudiantId, e => e.Id,   (i,e) => new { i, e })
    .Join(cours,       ie => ie.i.CoursId,  c => c.Id, (ie,c) => new
    {
        Etudiant = ie.e.Nom,
        Cours    = c.Titre,
        ie.i.NoteFinale
    });
```

---

## Exercice 11 — GroupJoin (cours par étudiant)

**Objectif** : Parent → enfants.
**Énoncé** : Pour chaque étudiant, lister ses cours suivis (titres).
**Attendu** : `{ Nom, Cours = [ ... ] }`.

**Correction (exemple)**

```csharp
var coursParEtudiant = etudiants
    .GroupJoin(inscriptions,
               e => e.Id,
               i => i.EtudiantId,
               (e, insc) => new
               {
                   e.Nom,
                   Cours = insc.Join(cours, x => x.CoursId, c => c.Id, (x,c) => c.Titre)
               });
```

---

## Exercice 12 — Any / All (prédicats)

**Objectif** : Tests d’existence et d’universalité.
**Énoncé** :

1. Y a-t-il **au moins un** étudiant à Montréal avec moyenne ≥ 18 ?
2. Tous les étudiants de Montréal ont-ils **au moins** une note ≥ 12 ?
   **Attendu** : booléens.

**Correction (exemple)**

```csharp
bool existeExcellentMTL = etudiants
    .Where(e => e.Ville == "Montréal")
    .Any(e => e.Notes.Average() >= 18);

bool tousOntNoteMin = etudiants
    .Where(e => e.Ville == "Montréal")
    .All(e => e.Notes.Any(n => n >= 12));
```

---

## Exercice 13 — First / FirstOrDefault / Single (sécurité)

**Objectif** : Savoir choisir l’opérateur adapté.
**Énoncé** : Récupérez l’étudiant nommé "Lina" de façon sûre. Puis tentez `Single` sur un prénom potentiellement dupliqué et gérez l’exception proprement.
**Attendu** : Utilisation de `FirstOrDefault` et `try/catch` pour `Single`.

**Correction (exemple)**

```csharp
var lina = etudiants.FirstOrDefault(e => e.Nom == "Lina");
if (lina != null) Console.WriteLine(lina.Nom);

try
{
    // Supposez des doublons de "Adam" dans d'autres jeux de données.
    var uniqueAdam = etudiants.Single(e => e.Nom == "Adam");
}
catch (InvalidOperationException ex)
{
    Console.WriteLine("Contrainte d’unicité violée: " + ex.Message);
}
```

---

## Exercice 14 — Pagination (Skip/Take)

**Objectif** : Découper un résultat trié.
**Énoncé** : Triez les étudiants par `Nom`, affichez la **page 2** de taille 2 (éléments 3 et 4).
**Attendu** : Deux étudiants.

**Correction (exemple)**

```csharp
int page = 2, pageSize = 2;
var page2 = etudiants
    .OrderBy(e => e.Nom)
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToList();
```

---

## Exercice 15 — ToDictionary (clés uniques)

**Objectif** : Matérialiser sous forme de dictionnaire.
**Énoncé** : Construisez un `Dictionary<int, string>` `{ Id -> Nom }`.
**Attendu** : Dictionnaire avec clés uniques; pensez aux collisions éventuelles.

**Correction (exemple)**

```csharp
var dict = etudiants.ToDictionary(e => e.Id, e => e.Nom);
```

---

# Partie C – Syntaxe requête vs méthodes, variables intermédiaires, pièges d’exécution

## Exercice 16 — Équivalence requête ↔ méthodes

**Objectif** : Passer de l’une à l’autre.
**Énoncé** : Convertissez la requête suivante en syntaxe méthodes :

```csharp
var q = from e in etudiants
        where e.Age >= 22
        orderby e.Nom
        select new { e.Nom, e.Age };
```

**Correction (exemple)**

```csharp
var q2 = etudiants
    .Where(e => e.Age >= 22)
    .OrderBy(e => e.Nom)
    .Select(e => new { e.Nom, e.Age });
```

---

## Exercice 17 — `let` et calculs intermédiaires (syntaxe requête)

**Objectif** : Lisibilité avec `let`.
**Énoncé** : Pour chaque étudiant, calculez `moy = moyenne des Notes`, puis filtrez `moy >= 16`, classez par `moy` décroissant et sélectionnez `{ Nom, moy }`.
**Attendu** : Utilisation de `let`.

**Correction (exemple)**

```csharp
var merites = from e in etudiants
              let moy = e.Notes.Average()
              where moy >= 16
              orderby moy descending
              select new { e.Nom, Moyenne = moy };
```

---

## Exercice 18 — Exécution différée et effets de bord

**Objectif** : Comprendre l’évaluation tardive.
**Énoncé** : Définissez `var q = entiers.Where(n => n % 2 == 0);`

1. Itérez et affichez.
2. Modifiez `entiers` (ajoutez/supprimez des valeurs).
3. Réitérez la **même** requête `q`.
   **Attendu** : Le second affichage change. Protégez-vous en matérialisant.

**Correction (exemple)**

```csharp
var q = entiers.Where(n => n % 2 == 0);
Console.WriteLine(string.Join(", ", q));

entiers.Add(100);
entiers.Remove(2);

Console.WriteLine(string.Join(", ", q)); // différent

var snapshot = q.ToList();                // fige
```

---

## Exercice 19 — GroupBy + agrégations multiples

**Objectif** : Calculs agrégés par groupe.
**Énoncé** : Par `Ville`, calculez : `Effectif`, `MoyenneGlobaleDesMoyennes`, `MaxNote` (meilleure note observée tous étudiants confondus).
**Attendu** : Une ligne par ville avec trois mesures.

**Correction (exemple)**

```csharp
var statsVille = etudiants
    .GroupBy(e => e.Ville)
    .Select(g => new
    {
        Ville       = g.Key,
        Effectif    = g.Count(),
        MoyenneMoy  = g.Average(e => e.Notes.Average()),
        MaxNote     = g.SelectMany(e => e.Notes).Max()
    });
```

---

## Exercice 20 — Requêtes composables avec `into`

**Objectif** : Enchaîner des requêtes en syntaxe *query*.
**Énoncé** :

1. Sélectionnez `{ Nom, Moy=e.Notes.Average() }`.
2. **into** tmp, filtrez `Moy >= 16`, triez par `Moy` décroissant.
   **Attendu** : Requête en deux temps via `into`.

**Correction (exemple)**

```csharp
var q = from e in etudiants
        select new { e.Nom, Moy = e.Notes.Average() }
        into t
        where t.Moy >= 16
        orderby t.Moy descending
        select t;
```
