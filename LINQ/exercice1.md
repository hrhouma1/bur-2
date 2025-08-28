# Exercice : Écrire vos propres requêtes LINQ

## Contexte

On dispose d’une collection d’objets `Etudiant`. Chaque étudiant a les propriétés suivantes : `Id`, `Nom`, `Age`, `Note`.

```csharp
class Etudiant
{
    public int Id { get; set; }
    public string Nom { get; set; }
    public int Age { get; set; }
    public double Note { get; set; }
}

List<Etudiant> etudiants = new List<Etudiant>
{
    new Etudiant { Id=1, Nom="Alice", Age=20, Note=15.5 },
    new Etudiant { Id=2, Nom="Bob", Age=22, Note=12.0 },
    new Etudiant { Id=3, Nom="Charlie", Age=23, Note=9.5 },
    new Etudiant { Id=4, Nom="David", Age=21, Note=17.0 },
    new Etudiant { Id=5, Nom="Emma", Age=20, Note=18.0 }
};
```

---

## Questions

### Question 1

Écrivez une requête LINQ qui retourne **tous les étudiants ayant une note ≥ 15**.

### Question 2

Écrivez une requête qui retourne uniquement les **noms des étudiants** (projection sur la propriété `Nom`).

### Question 3

Écrivez une requête qui retourne les étudiants **triés par âge croissant**.

### Question 4

Écrivez une requête qui retourne le **nombre total d’étudiants** dans la liste.

### Question 5

Écrivez une requête qui retourne l’**étudiant le plus jeune**.

### Question 6

Écrivez une requête qui retourne la **moyenne des notes**.

### Question 7

Écrivez une requête qui retourne tous les étudiants dont le **nom commence par la lettre 'A'**.

### Question 8

Écrivez une requête qui retourne les étudiants avec une note ≥ 10, puis affichez-les **par ordre décroissant de note**.

### Question 9

Écrivez une requête qui retourne les étudiants groupés **par âge** (`GroupBy`).

### Question 10

Écrivez une requête qui vérifie si **au moins un étudiant a une note inférieure à 10**.

---

## Exemple de sortie attendue (Question 1)

```csharp
var excellents = from e in etudiants
                 where e.Note >= 15
                 select e;

foreach (var et in excellents)
    Console.WriteLine(et.Nom + " " + et.Note);
```

ou en syntaxe méthode :

```csharp
var excellents = etudiants.Where(e => e.Note >= 15);
```

