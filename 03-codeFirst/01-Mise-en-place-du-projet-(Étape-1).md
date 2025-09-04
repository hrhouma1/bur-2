# 1. Mise en place du projet (Étape 1)

> **Objectif de cette étape**
> Démarrer une solution **Console (.NET Framework)** propre, prête à accueillir nos 4 couches (**METIER**, **DAO**, **UTILEMETIER**, **EF**), conformément au cours.



## 1.0 Pré-requis & choix de version

* **Visual Studio (Windows)** : Community 2019/2022 (ou édition supérieure).
* **.NET ciblé pour ce cours** : **.NET Framework 4.7.2** (ou ≥ 4.6.1).

  > Le code des étapes suivantes utilise **Entity Framework 6 (EF6)** (`System.Data.Entity`) et **App.config**. C’est le chemin le plus simple : **Console (.NET Framework)** + **EF6**.
* **Alternative possible** : .NET 6/7/8 + **EF Core** (*différent* : namespaces `Microsoft.EntityFrameworkCore`, `appsettings.json`…), **non couvert** par ce cours.

  > Si tu veux EF Core plus tard, on fera une variante dédiée.



## 1.1 Créer la solution et le projet Console (.NET Framework)

### A. Avec l’interface Visual Studio (recommandé)

1. **Fichier → Nouveau → Projet…**
2. Rechercher **“Console App (.NET Framework)”** (*Application console (.NET Framework)*).

   > ⚠️ Ne pas confondre avec *Console App (.NET)* qui crée un projet .NET moderne (non Framework).
3. **Nom du projet** : `DAODemoP1Console` (ou `DAODemoP1`)
   **Nom de la solution** : `DAODemoP1`
4. **Cadre cible** : **.NET Framework 4.7.2** (ou 4.8 si dispo).
5. Créer le projet. Visual Studio génère un `Program.cs` minimal.

**Vérification rapide**

* Menu **Déboguer → Démarrer sans débogage** : la console s’ouvre, le projet compile → ✅

### B. (Optionnel) En ligne de commande (CLI .NET Framework)

> Le Workflow CLI natif cible plutôt .NET moderne. Pour un projet **.NET Framework**, reste sur Visual Studio GUI (A).
> Si tu insistes pour du CLI, il faut des templates spécifiques ou partir d’un `.csproj` modèle — moins simple pour débuter.



## 1.2 Créer l’architecture en couches (dossiers)

Nous gardons une **solution simple** avec **un seul projet Console** et **4 dossiers** à l’intérieur :

* `METIER` — Couche métier (entités, logique domaine simple)
* `DAO` — Accès aux données (interfaces + implémentations)
* `UTILEMETIER` — Aides/Utilitaires pour la couche métier (ex. générer des données de démo)
* `EF` — Fichiers **Entity Framework 6** (Contexte, Initializer…)

### Pas à pas (Visual Studio)

1. Dans **l’Explorateur de solutions**, clic droit sur le **projet** → **Ajouter** → **Nouveau dossier**.
2. Créer **exactement** ces 4 dossiers :

   * `METIER`
   * `DAO`
   * `UTILEMETIER`
   * `EF`

> 💡 **Convention de nommage & namespaces**
>
> * Le namespace racine dépend du nom de **projet** (p. ex. `DAODemoP1Console`).
> * Les fichiers dans `DAO` auront un namespace logique comme `DAODemoP1.DAO` (tu pourras l’ajuster manuellement pour correspondre au cours si besoin).



## 1.3 Préparer le squelette minimal

Tu peux déjà mettre un **Program.cs** minimal qui nous servira à tester chaque étape :

```csharp
using System;

namespace DAODemoP1
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("DAODemoP1 - Étape 1 OK");
            Console.ReadLine();
        }
    }
}
```

**Compile & Run** → tu dois voir `DAODemoP1 - Étape 1 OK`.



## 1.4 Résultat attendu (arborescence)

```
DAODemoP1
└─ DAODemoP1Console  (projet)
   ├─ DAO
   ├─ EF
   ├─ METIER
   ├─ UTILEMETIER
   ├─ Program.cs
   └─ App.config        (sera ajouté plus tard par NuGet EF6)
```

> 🧭 Tu peux comparer visuellement avec les figures partagées (arborescences VS).



## 1.5 (Optionnel) Initialisation Git & qualité de vie

* **Git** :

  * Clic droit solution → **Ajouter la solution au contrôle de code source**
  * Ou en shell dans le dossier solution :

    ```bash
    git init
    echo "bin/"        >> .gitignore
    echo "obj/"        >> .gitignore
    echo ".vs/"        >> .gitignore
    echo "packages/"   >> .gitignore
    git add .
    git commit -m "Étape 1 - projet console + dossiers"
    ```
* **.editorconfig** (facultatif mais utile) :

  * Indenter avec 4 espaces, encodage UTF-8, fin de ligne LF/CRLF selon ton équipe.
* **LangVersion / Nullable** (facultatif) :

  * Pour profiter des dernières features C#, tu pourras régler `LangVersion=latest` plus tard dans le `.csproj` (non obligatoire pour EF6).



## 1.6 Pièges fréquents & solutions

* **Erreur de modèle EF plus tard** : tu as choisi “Console App (.NET)” au lieu de “Console App (.NET Framework)”.
  → Recrée un projet **.NET Framework** (EF6 utilise `System.Data.Entity` et `App.config`).
* **Namespaces hétérogènes** après déplacements de fichiers :
  → Adapte le `namespace` en haut de chaque `.cs` pour garder la cohérence **`DAODemoP1.*`**.
* **Compilation OK mais rien ne s’affiche** :
  → Vérifie que **`Main`** écrit bien quelque chose et que tu appelles `Console.ReadLine()` (sinon la fenêtre se ferme trop vite en “Démarrer sans débogage”).



## 1.7 Check-list de sortie (Étape 1)

* [ ] Projet **Console (.NET Framework)** créé, compile et s’exécute
* [ ] Dossiers **METIER**, **DAO**, **UTILEMETIER**, **EF** présents
* [ ] `Program.cs` minimal affichant un message
* [ ] (Optionnel) Repo **Git** initialisé avec `.gitignore`



> **Prochaine étape** : **Étape 2 – Couche métier**
> On va créer la classe **`Compte`** (propriétés, constructeurs, méthodes `Crediter`/`Debiter`, `Deconstruct`, `ToString`) puis la classe utilitaire **`Utile`** avec des données de test.
