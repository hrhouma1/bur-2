# 2. Couche Métier (Business Layer) — **Étape 2 super exhaustive**

> **Objectif de l’étape**
> Mettre en place le **cœur métier** de l’application : une entité `Compte` qui représente un compte bancaire, avec ses propriétés, constructeurs et méthodes de base.
> Puis ajouter une petite **fabrique de données** (`Utile.GetComptes`) pour nos tests (liste 101–107).



## 2.0 Où créer les fichiers ?

* **Projet** : `DAODemoP1` (ou `DAODemoP1Console`)
* **Dossier** :

  * `METIER` → pour `Compte.cs`
  * `UTILEMETIER` → pour `Utile.cs`

> ℹ️ Les dossiers ont été créés à l’**Étape 1**. Si besoin : clic droit sur le projet → **Ajouter** → **Nouveau dossier**.



## 2.1 Classe `Compte.cs` (dans `METIER`)

### 2.1.1 Code complet (copier-coller)

```csharp
using System;

namespace DAODemoP1.METIER
{
    public class Compte
    {
        #region attributs
        // (Pas d'attributs privés explicites : on utilise des propriétés auto-implémentées)
        #endregion

        #region proprietes
        public long CompteId { get; set; }
        public string Numero { get; set; }
        public float Solde { get; set; }
        public DateTime DateCreation { get; set; }
        #endregion

        #region constructeurs
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

        // Constructeur par défaut : Numero = "Undefined", DateCreation = Now, Solde = 0
        public Compte() : this("Undefined") { }
        #endregion

        #region deconstruct
        // Permet l'assignation déstructurée : (var numero, var solde, var date) = compte;
        public void Deconstruct(out string numero, out float solde, out DateTime dateCreation)
        {
            numero = Numero;
            solde = Solde;
            dateCreation = DateCreation;
        }

        // Surcharge si on ne veut que (numero, solde)
        public void Deconstruct(out string numero, out float solde)
        {
            numero = Numero;
            solde = Solde;
        }
        #endregion

        #region methodes
        public void Crediter(float montant)
        {
            Solde += montant;
        }

        public bool Debiter(float montant)
        {
            bool res = false;
            if (montant <= Solde)
            {
                Solde -= montant;
                res = true;
            }
            return res;
        }

        public override string ToString()
            => $"Numero : {Numero} Solde : {Solde} DateCreation : {DateCreation}";
        #endregion
    }
}
```

### 2.1.2 Explications rapides

* **Propriétés** : `CompteId` (clé technique), `Numero`, `Solde`, `DateCreation`.

  > On utilise des **propriétés auto-implémentées** (simplicité).
* **Constructeurs** :

  * complet `(numero, dateCreation, solde)` — pratique pour reconstruire un état connu ;
  * partiel `(numero)` — initialise `DateCreation` à `Now` et `Solde` à `0` ;
  * par défaut — délègue au partiel avec `"Undefined"`.
* **Méthodes métier** :

  * `Crediter(montant)` : ajoute au solde ;
  * `Debiter(montant)` : débite si solde suffisant, renvoie `true`/`false` ;
  * `ToString()` : représentation lisible en console.
* **Deconstruct** : syntactic sugar pour écrire :

  ```csharp
  (string numero, float solde, DateTime date) = compte;
  ```

> 💡 **Argent & type numérique**
> Le cours utilise `float`. Pour un vrai logiciel bancaire, préférez **`decimal`** (meilleure précision pour les montants). On garde `float` ici pour rester fidèle à la progression, mais on pourra montrer la variante plus tard.

> 🧪 **Idées de validations (optionnel)**
>
> * Refuser `Numero` vide / null.
> * Empêcher `Crediter`/`Debiter` avec un **montant négatif**.
>   Ces gardes-fous ne sont **pas** dans la version de base du cours, mais tu peux les ajouter en exercice.



## 2.2 Classe utilitaire `Utile.cs` (dans `UTILEMETIER`)

### 2.2.1 Code complet (copier-coller)

```csharp
using System;
using System.Collections.Generic;
using DAODemoP1.METIER;

namespace DAODemoP1.UTILEMETIER
{
    static class Utile
    {
        /// <summary>
        /// Données de démonstration pour nos tests en mémoire (101–107).
        /// </summary>
        public static List<Compte> GetComptes()
        {
            return new List<Compte>
            {
                new Compte("101", new DateTime(2016, 10, 10), 100),
                new Compte("102", new DateTime(2017,  7, 18),   0),
                new Compte("103", new DateTime(2017,  3, 15), 100),
                new Compte("104", new DateTime(2016,  2,  5), 500),
                new Compte("105", new DateTime(2015,  8,  3), 200),
                new Compte("106", new DateTime(2014,  9, 24), 200),
                new Compte("107", new DateTime(2015,  1, 27), 200)
            };
        }
    }
}
```

### 2.2.2 Pourquoi une classe utilitaire ?

* Elle sert de **source de données en mémoire** pour les premières étapes (avant la base de données).
* Elle simplifie les tests : pas besoin d’EF ni de SQL pour commencer.



## 2.3 Mini-tests rapides (dans `Program.cs`)

> **But** : vérifier que `Compte` et `Utile` compilent et se comportent comme prévu.

```csharp
using System;
using DAODemoP1.METIER;
using DAODemoP1.UTILEMETIER;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            // 1) Créer un compte "manuel"
            var c = new Compte("999", DateTime.Today, 50);
            c.Crediter(25);      // Solde = 75
            bool ok = c.Debiter(100); // false, insuffisant
            Console.WriteLine($"{c} | Débit OK ? {ok}");

            // 2) Récupérer les comptes de démo
            var comptes = Utile.GetComptes();
            Console.WriteLine($"Nb comptes: {comptes.Count}");

            // 3) Déconstruction
            var c101 = comptes[0]; // "101"
            (string numero, float solde, DateTime date) = c101;
            Console.WriteLine($"Déconstruction -> {numero} | {solde} | {date:yyyy-MM-dd}");

            Console.WriteLine("Étape 2 OK");
            Console.ReadLine();
        }
    }
}
```

**Ce que tu dois voir** (exemple) :

```
Numero : 999 Solde : 75 DateCreation : 2025-09-03 00:00:00 | Débit OK ? False
Nb comptes: 7
Déconstruction -> 101 | 100 | 2016-10-10
Étape 2 OK
```



## 2.4 Check-list de sortie (Étape 2)

* [ ] `METIER/Compte.cs` créé, compile, avec : propriétés, 3 constructeurs, `Crediter`, `Debiter`, `ToString`, `Deconstruct` ×2
* [ ] `UTILEMETIER/Utile.cs` créé, `GetComptes()` retourne 7 comptes (101–107)
* [ ] **Mini-tests** dans `Program.cs` : **OK**
* [ ] (Optionnel) Règles métier supplémentaires envisagées (montants négatifs, etc.)



## 2.5 Pièges fréquents & solutions

* **Namespace incohérent** : assure-toi que `namespace DAODemoP1.METIER` et `namespace DAODemoP1.UTILEMETIER` correspondent bien à ton projet.

  > Si ton projet s’appelle `DAODemoP1Console`, Visual Studio peut générer `DAODemoP1Console.METIER`. Tu peux **ajuster** manuellement pour suivre le cours (`DAODemoP1.*`) ou garder la version générée, mais **reste cohérent partout**.
* **Dates invalides** : attention aux zéros de padding (ex. `new DateTime(2016, 2, 05)` est valide, mais `05` est inutile — `5` suffirait).
* **`float` & affichage** : les décimales peuvent s’afficher avec `,` ou `.` selon la culture. Ce n’est pas un bug, c’est la culture système.



> ✅ **Étape suivante** : **Étape 3–5 (DAO en mémoire)**
> On va créer l’interface `ICompteDao`, son implémentation `CompteDao` (en mémoire), puis ajouter des méthodes d’extraction ciblées et d’agrégation LINQ, avec des tests dans `Program.cs`.
