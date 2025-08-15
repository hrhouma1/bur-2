## Ligne complète à analyser

```csharp
MessageBox.Show(
    $"Bonjour !{Environment.NewLine}Utilisateur : {utilisateur + Environment.NewLine}Mot de passe : {motPasse + Environment.NewLine}", 
    "Bienvenue !", 
    MessageBoxButton.OK
);
```

---

## 1. **MessageBox.Show**

* **Type** : Méthode statique de la classe `MessageBox` (namespace `System.Windows` en WPF).
* **Rôle** : Affiche une petite boîte de dialogue modale pour montrer un message à l’utilisateur.
* **Blocage** : Tant que l’utilisateur n’a pas cliqué sur un bouton de la boîte, le programme reste en pause sur cette ligne.

---

## 2. **Premier paramètre** – *Le message à afficher*

```csharp
$"Bonjour !{Environment.NewLine}Utilisateur : {utilisateur + Environment.NewLine}Mot de passe : {motPasse + Environment.NewLine}"
```

* Le **`$`** devant la chaîne indique qu’on utilise **l’interpolation de chaînes**.
  → Cela permet d’intégrer directement des variables dans le texte avec `{variable}`.
* `Bonjour !` : Texte fixe.
* `{Environment.NewLine}` :

  * Code spécial qui ajoute un **retour à la ligne** compatible avec le système d’exploitation (équivalent de appuyer sur "Entrée").
  * Permet d’écrire sur plusieurs lignes dans la boîte de message.
* `Utilisateur : {utilisateur + Environment.NewLine}`

  * On affiche le mot "Utilisateur :" suivi de la valeur contenue dans la variable `utilisateur`.
  * Puis un retour à la ligne pour passer à l’info suivante.
* `Mot de passe : {motPasse + Environment.NewLine}`

  * Même principe : affichage de l’étiquette et de la valeur contenue dans la variable `motPasse`, suivie d’un retour à la ligne.

💡 **Remarque** :
Ici tu as écrit `utilisateur + Environment.NewLine` dans l’interpolation, mais en C# on préfère plutôt écrire :

```csharp
$"Utilisateur : {utilisateur}{Environment.NewLine}"
```

Cela évite des parenthèses inutiles et rend le code plus clair.

---

## 3. **Deuxième paramètre** – *Le titre de la boîte*

```csharp
"Bienvenue !"
```

* C’est le texte qui s’affiche **dans la barre de titre** de la fenêtre de message.
* Ici, il sera affiché en haut de la boîte de dialogue.

---

## 4. **Troisième paramètre** – *Le bouton à afficher*

```csharp
MessageBoxButton.OK
```

* C’est une **énumération** (`enum`) qui indique quels boutons la boîte doit contenir.
* `OK` signifie qu’il y aura **un seul bouton "OK"** pour fermer la boîte.
* Autres valeurs possibles :

  * `MessageBoxButton.OKCancel`
  * `MessageBoxButton.YesNo`
  * `MessageBoxButton.YesNoCancel`

---

## 5. **Comportement final**

Quand cette ligne s’exécute :

* Une petite fenêtre apparaît avec :

  * **Titre** : "Bienvenue !"
  * **Texte** :

    ```
    Bonjour !
    Utilisateur : (valeur saisie)
    Mot de passe : (valeur saisie)
    ```
  * **Bouton** : OK
* Tant que l’utilisateur n’a pas cliqué sur OK, la suite du code n’est pas exécutée.

