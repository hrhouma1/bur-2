# Examen 1 – Suivi des étudiants (WPF C# + BD)

## Contexte

On vous fournit une solution **WPF C# (.NET/EF6 ou ADO.NET)** comportant :

* un **écran de connexion** `frmLoggin` ;
* une fenêtre principale **SuiviEtudiantsUI** avec un **ComboBox** « Liste des étudiants », un **onglet Étudiant** (champs texte, statut radio, boutons **Enregistrer** / **Supprimer**) et un onglet **Cours**.
* Le chargement de la fiche d’un étudiant depuis la BD est déjà fonctionnel (ID, prénom, nom, adresse, etc.).
* **Manquent volontairement** : la récupération automatique **Instructeur** et **Programme**, la **mise à jour** (bouton **Enregistrer**) et la **suppression** (bouton **Supprimer** avec **confirmation**).

Votre travail consiste à **compléter** ces fonctionnalités en respectant les exigences ci-dessous.


> Note importante:

**« Vous avez 3 possibilités d’implémentation au choix : (1) *ADO.NET (SqlClient)* avec requêtes paramétrées et **chaîne de connexion** que **vous définissez** dans `App.config`; (2) *Entity Framework 6 — Database First* (modèle `.edmx` généré depuis la BD); (3) *Entity Framework 6 — Code First* (classes POCO + migrations). Indiquez l’option retenue dans votre .docx. »**




## Objectifs pédagogiques

1. Lier des contrôles WinForms à des données relationnelles.
2. Mettre à jour et supprimer de façon sûre (requêtes paramétrées / EF6, transactions, gestion d’erreurs).
3. Valider les entrées et améliorer l’UX (messages clairs, confirmations).



## Modèle de données (minimal requis)

Vous pouvez utiliser EF6 (Code First ou Database First) **ou** ADO.NET `SqlClient`. Le schéma attendu (ou équivalent) :

* **Etudiants**(
  `Id` (PK, string ou int), `Prenom`, `Nom`, `Adresse`, `Ville`, `Province`, `CodePostal`, `Telephone`,
  `Statut` (enum/string: Actif/Arret/Gradue), `ProgrammeId` (FK), `InstructeurId` (FK) )
* **Programmes**(`ProgrammeId` PK, `NomProgramme`)
* **Instructeurs**(`InstructeurId` PK, `NomComplet`)
* **Utilisateurs**(`UserId` PK, `Username`, `Password` \[plaintext autorisé pour l’examen], `Role`)

> Données minimales de test exigées :
> – au moins **5 étudiants**, **2 programmes**, **2 instructeurs**, **1 utilisateur admin** (ex. `admin`/`admin`) et **1 utilisateur instructeur**.



## Tâches à réaliser

### 1) Récupération Instructeur & Programme (ComboBox) – 20 pts

* Au chargement de la fenêtre, **remplir** les ComboBox **Instructeur** et **Programme** depuis la base (ordre alphabétique).
* Lorsqu’un étudiant est sélectionné, **pré-sélectionner** automatiquement son instructeur et son programme dans les ComboBox.
* Les champs **ID** non éditables.

**Critères d’acceptation**
A1. Les listes affichent toutes les valeurs de la BD.
A2. La fiche d’un étudiant reflète correctement ses FK (sélections visibles).
A3. Aucun SQL concaténé : **requêtes paramétrées** ou **EF**.



### 2) Mise à jour (bouton **Enregistrer**) – 25 pts

* Autoriser l’édition des champs (adresse, téléphone, ville, province, code postal, statut, instructeur, programme).
* Au clic sur **Enregistrer** :

  * **Valider** : champs obligatoires (`Prenom`, `Nom`, `Telephone`, `Programme`, `Instructeur`), format téléphone et code postal (tolérance raisonnable), statut présent.
  * **Persister** l’étudiant via **UPDATE** paramétré ou **SaveChanges()** EF.
  * **Transaction** recommandée (ou `SaveChanges()` atomique).
  * Afficher un **message de succès** et **rafraîchir** l’interface (la sélection reste sur l’étudiant).

**Critères d’acceptation**
B1. Les validations bloquent les entrées invalides avec messages clairs.
B2. Les modifications sont visibles après re-ouverture de l’app ou rechargement.
B3. Aucune injection SQL possible.



### 3) Suppression (bouton **Supprimer**) **avec confirmation** – 25 pts

* Au clic sur **Supprimer**, afficher une **boîte de dialogue de confirmation** indiquant clairement :
  « Voulez-vous vraiment supprimer l’étudiant **\[Nom Prénom]** (ID **\[Id]**)? »
  **Boutons : Oui / Non**.
* En cas de **Oui** :

  * Gérer les **contraintes de clé étrangère** (ex. inscriptions/cours). Deux options acceptées :

    1. **Cascade** : supprimer d’abord les enregistrements dépendants de l’étudiant, puis l’étudiant (transaction).
    2. **Blocage** : si des dépendances existent, **annuler** et afficher un message explicite.
  * Retirer l’étudiant de la liste et **vider** la fiche.

**Critères d’acceptation**
C1. La suppression n’a lieu **que** après confirmation « Oui ».
C2. En cas d’échec (FK, connexion, etc.), message d’erreur clair sans crasher l’app.
C3. Après suppression, l’étudiant **disparaît** du ComboBox et la fiche est propre.

**Bonus (jusqu’à +5 pts)** : seconde confirmation par **saisie du matricule** (dialogue personnalisé) avant de permettre la suppression.



### 4) Onglet **Cours** (lecture seule) – 10 pts

* Afficher la liste des cours suivis par l’étudiant (si table disponible).
* Après suppression de l’étudiant, cet onglet doit afficher **aucune donnée** et l’UI rester stable.


### 5) Qualité, sécurité, UX, exceptions – 10 pts

* `using`/`Dispose` sur connexions/contexts, messages d’erreur conviviaux, **try/catch** au bon niveau.
* Pas de duplication de code obvious (utiliser méthodes/Repository).
* Chaîne de connexion centralisée dans `App.config`.



### 6) Livraison & preuves – 10 pts

À remettre dans un **.zip** :

1. Le **projet Visual Studio** complet.
2. Le **script SQL** (création + insertion de données de test) **ou** le fichier MDF/LocalDB + instructions.
3. Un **document .docx** contenant les **captures d’écran** suivantes avec légendes :

   * Connexion réussie (admin ou instructeur)
   * Fiche étudiant **avant** modification
   * ComboBox Instructeur & Programme **pré-sélectionnés**
   * Message de **succès** après **Enregistrer** + fiche **après** modification
   * **Boîte de confirmation** de suppression
   * Fiche/liste **après suppression** (l’étudiant n’est plus présent)
   * (optionnel) Onglet Cours
4. Une page **explicative (10–15 lignes)** : choix techniques (EF/ADO.NET), validations, gestion FK, transactions.



## Barème (100 points)

* (20) Récupération et binding **Instructeur/Programme**
* (25) **Enregistrer** : validations + persistance + rafraîchissement
* (25) **Supprimer** : confirmation + gestion FK + rafraîchissement
* (10) Onglet **Cours** (lecture seule, cohérent après actions)
* (10) Qualité du code, sécurité (paramétrage/EF), gestion exceptions
* (10) Livraison (projet, script, .docx avec captures et explications)
* Bonus jusqu’à **+5** : confirmation par saisie du matricule, recherche par nom, export CSV, tests unitaires basiques



## Contraintes techniques

* **Interdits** : SQL concaténé, mots de passe codés en dur dans le code de production (autorisé uniquement pour `admin/admin` de test).
* **Requis** : requêtes **paramétrées** ou **EF6**, messages d’erreur/confirmation en **français**, champs **ID** non modifiables.
* L’UI fournie ne doit pas être cassée (vous pouvez convertir **Instructeur** et **Programme** en **ComboBox** si ce n’est pas déjà le cas).



## Scénarios de test (enseignant)

1. Connexion `admin/admin` → OK.
2. Sélection d’un étudiant → tous les champs + **instructeur/programme** s’affichent.
3. Modifier téléphone et statut → **Enregistrer** → relancer l’app → changements persistants.
4. **Supprimer** l’étudiant → confirmation **Oui** → l’étudiant n’apparaît plus (et dépendances gérées).
5. Entrées invalides (téléphone vide) → message de validation, aucun enregistrement BD.



### Remarques

* Vous pouvez choisir EF6 **ou** ADO.NET, mais la **sécurité** (paramètres/transactions) et la **clarté** priment.
* Commentez brièvement les méthodes clés (`UpdateEtudiant`, `DeleteEtudiant`, chargement des ComboBox).

Bon travail.
