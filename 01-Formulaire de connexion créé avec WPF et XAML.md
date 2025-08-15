# Cours : Création d’une fenêtre de connexion avec raccourcis clavier en WPF

<br/>

# 1. Introduction

Dans ce cours, nous allons étudier un formulaire de connexion créé avec WPF et XAML, intégrant :

* Des **raccourcis clavier** pour naviguer rapidement entre les champs.
* Une **organisation précise** via un `Grid` et un `WrapPanel`.
* Des **liens entre l’interface et le code C#** grâce aux événements.

L’objectif est de permettre à l’utilisateur d’interagir avec l’interface **sans utiliser la souris**, ce qui améliore l’accessibilité et la productivité.

<br/>

# 2. Aperçu visuel de l’interface

La fenêtre de connexion est composée de :

1. Une image représentant un personnage avec une clé.
2. Un champ **Nom d’utilisateur** (TextBox).
3. Un champ **Mot de passe** (PasswordBox).
4. Deux boutons : **OK** et **Annuler**.

Cette disposition est courante dans les systèmes d’authentification et peut servir de base pour de nombreuses applications.

<br/>

# 3. Les raccourcis clavier avec Alt

En WPF, un **underscore** (`_`) dans le texte d’un contrôle `Label` ou `Button` définit une **touche d’accès rapide**.
Exemples :

* `_Nom d’utilisateur` → Alt + N.
* `_Mot de passe` → Alt + M.
* `_OK` → Alt + O.
* `_Annuler` → Alt + A.

Pour que le raccourci d’un `Label` déplace le curseur vers un champ précis, il faut utiliser la propriété :

```xml
Target="{Binding ElementName=NomDuChamp}"
```

Ainsi :

```xml
<Label Content="_Nom d’utilisateur" Target="{Binding ElementName=txtUtilisateur}"/>
```

fera que Alt+N placera le focus sur `txtUtilisateur`.

<br/>

# 4. Structure en Grid

Le `Grid` est utilisé pour organiser l’interface en **colonnes** et **lignes**.

### Colonnes

```xml
<Grid.ColumnDefinitions>
    <ColumnDefinition Width="10"/>       <!-- Marge à gauche -->
    <ColumnDefinition Width="Auto"/>     <!-- Image -->
    <ColumnDefinition Width="Auto"/>     <!-- Labels -->
    <ColumnDefinition Width="Auto"/>     <!-- Zones de saisie -->
    <ColumnDefinition Width="50"/>       <!-- Marge à droite -->
</Grid.ColumnDefinitions>
```

### Lignes

```xml
<Grid.RowDefinitions>
    <RowDefinition Height="*"/>          <!-- Espace haut -->
    <RowDefinition Height="Auto"/>       <!-- Nom utilisateur -->
    <RowDefinition Height="Auto"/>       <!-- Mot de passe -->
    <RowDefinition Height="Auto"/>       <!-- Ligne intermédiaire -->
    <RowDefinition Height="Auto"/>       <!-- Boutons -->
    <RowDefinition Height="*"/>          <!-- Espace bas -->
</Grid.RowDefinitions>
```

<br/>

# 5. Placement des éléments

### Image

```xml
<Image Source="Resources/Clé.png"
       Grid.Column="1" Grid.Row="0" Grid.RowSpan="6"/>
```

* Placée en colonne 1.
* `RowSpan="6"` pour occuper toute la hauteur.

### Label + TextBox pour Nom d’utilisateur

```xml
<Label Content="_Nom d’utilisateur :" 
       Grid.Column="2" Grid.Row="1"
       Target="{Binding ElementName=txtUtilisateur}"/>
<TextBox x:Name="txtUtilisateur" 
         Grid.Column="3" Grid.Row="1" Width="150"/>
```

* Alt+N déplace le curseur vers `txtUtilisateur`.

### Label + PasswordBox pour Mot de passe

```xml
<Label Content="_Mot de passe :" 
       Grid.Column="2" Grid.Row="2"
       Target="{Binding ElementName=txtMotPasse}"/>
<PasswordBox x:Name="txtMotPasse" 
             Grid.Column="3" Grid.Row="2" Width="150"/>
```

* Alt+M déplace le curseur vers `txtMotPasse`.

### Boutons dans un WrapPanel

```xml
<WrapPanel Grid.Column="3" Grid.Row="4">
    <Button x:Name="btnOk" Content="_OK" Width="50" Click="btnOk_Click"/>
    <Button x:Name="btnAnnuler" Content="_Annuler" Width="50" Click="btnAnnuler_Click"/>
</WrapPanel>
```

* Alt+O équivaut à cliquer sur OK.
* Alt+A équivaut à cliquer sur Annuler.

<br/>

# 6. Code C# associé

Les événements `Click` permettent de définir la logique à exécuter lorsqu’un bouton est activé (par clic ou raccourci clavier).

Exemple :

```csharp
private void btnOk_Click(object sender, RoutedEventArgs e)
{
    string utilisateur = txtUtilisateur.Text;
    string motDePasse = txtMotPasse.Password;

    if (utilisateur == "admin" && motDePasse == "1234")
    {
        MessageBox.Show("Connexion réussie");
        // Code pour ouvrir la fenêtre principale
    }
    else
    {
        MessageBox.Show("Nom d'utilisateur ou mot de passe incorrect");
    }
}

private void btnAnnuler_Click(object sender, RoutedEventArgs e)
{
    this.Close(); // Ferme la fenêtre
}
```

<br/>

# 7. Points pédagogiques

1. **Accessibilité**

   * Les raccourcis Alt+lettre facilitent la navigation au clavier.
   * Les labels doivent toujours être liés à leurs champs via `Target`.

2. **Lisibilité du code XAML**

   * Utiliser `Grid` pour organiser clairement les éléments.
   * Séparer l’interface (XAML) de la logique (C#).

3. **Interaction utilisateur**

   * Un clic de souris et un raccourci clavier déclenchent le même événement `Click`.

4. **Sécurité basique**

   * Ne jamais stocker un mot de passe en clair dans le code.
   * Utiliser `PasswordBox` au lieu de `TextBox` pour masquer les caractères.

<br/>

# Annexe 1 - Glossaire des termes clés – Formulaire de connexion WPF

## 1.1. `_` (underscore dans un texte de Label ou Button)

* **Définition** : En WPF, lorsqu’on place un underscore (`_`) dans le texte (`Content`) d’un contrôle `Label` ou `Button`, la lettre qui suit devient une **touche d’accès rapide** (aussi appelée *mnemonic* ou *accelerator key*).
* **Exemple** :

  ```xml
  <Label Content="_Nom d’utilisateur" ... />
  ```

  Ici, **Alt+N** activera le raccourci.
* **Utilité** :

  * Améliorer l’accessibilité pour les personnes utilisant uniquement le clavier.
  * Permettre une navigation plus rapide.
* **Remarque** :

  * Pour que le raccourci d’un `Label` place le curseur dans un champ, il faut définir la propriété `Target`.



## 1.2. `Binding`

* **Définition** : En WPF, **Binding** est un mécanisme qui permet de relier (*lier*) une propriété d’un contrôle à une donnée ou à un autre contrôle, sans avoir à écrire de code manuel pour cette synchronisation.
* **Syntaxe dans ce contexte** :

  ```xml
  Target="{Binding ElementName=txtUtilisateur}"
  ```
* **Signification ici** :

  * `Target` indique au `Label` quel contrôle recevoir le focus lorsque l’utilisateur active le raccourci clavier.
  * `ElementName=txtUtilisateur` signifie que le label est lié à la zone de texte nommée `txtUtilisateur`.
* **Utilité** :

  * Réduire le code C# nécessaire pour relier des éléments.
  * Centraliser les relations entre composants directement dans le XAML.



## 1.3. `Content`

* **Définition** : Propriété commune aux contrôles comme `Label` ou `Button` en WPF qui définit le texte (ou contenu visuel) affiché.
* **Exemple** :

  ```xml
  <Button Content="_OK" ... />
  ```

  Affiche le texte "OK" avec un raccourci clavier Alt+O.



## 1.4. `Target`

* **Définition** : Propriété d’un `Label` qui permet d’associer ce label à un contrôle d’entrée (`TextBox`, `PasswordBox`, etc.).
* **Exemple** :

  ```xml
  Target="{Binding ElementName=txtMotPasse}"
  ```

  Associe le label au champ mot de passe.
* **Effet** :

  * Le raccourci clavier défini dans le `Content` du label positionnera le curseur dans le contrôle cible.



## 1.5. `TextBox`

* **Définition** : Contrôle WPF permettant à l’utilisateur de saisir du texte libre.
* **Caractéristiques** :

  * Affiche le texte en clair.
  * Idéal pour les noms, emails, etc.



## 1.6. `PasswordBox`

* **Définition** : Contrôle WPF similaire à `TextBox` mais conçu pour saisir un mot de passe.
* **Différences avec `TextBox`** :

  * Les caractères saisis sont masqués (généralement affichés sous forme de points).
  * La propriété contenant le mot de passe est `Password` et non `Text`.



## 1.7. `Click`

* **Définition** : Événement déclenché lorsqu’un bouton est activé (clic souris ou raccourci clavier).
* **Exemple** :

  ```xml
  Click="btnOk_Click"
  ```

  Appelle la méthode `btnOk_Click` dans le code C#.



## 1.8. `Grid`

* **Définition** : Conteneur WPF permettant de placer des contrôles dans un tableau de lignes et de colonnes.
* **Avantage** :

  * Organisation précise des éléments.
  * Adaptable à différentes tailles de fenêtre.



## 1.9. `WrapPanel`

* **Définition** : Conteneur WPF qui dispose ses enfants horizontalement ou verticalement, et passe automatiquement à la ligne si l’espace est insuffisant.
* **Utilisation ici** :

  * Placer les boutons OK et Annuler côte à côte.



<br/>

# Annexe 2


## 2.1. Explication du code C# (WPF)

```csharp
using System.Windows;

namespace College
{
    /// <summary>
    /// Logique d’interaction pour frmLoggin.xaml
    /// </summary>
    public partial class frmLoggin : Window
    {
        public frmLoggin()
        {
            InitializeComponent();
        }

        private void btnOk_Click(object sender, RoutedEventArgs e)
        {
            MessageBox.Show("Vous avez cliquez sur le bouton OK.", "Attention !",
                            MessageBoxButton.OK, MessageBoxImage.Exclamation);
        }

        private void btnAnnuler_Click(object sender, RoutedEventArgs e)
        {
            MessageBox.Show("Vous avez cliquez sur le bouton Annuler.", "Attention !",
                            MessageBoxButton.OK, MessageBoxImage.Exclamation);
        }
    }
}
```

Points à retenir (pédagogiques) :

* `using System.Windows;` : espace de noms qui expose les classes WPF (dont `Window` et `MessageBox`).
* `public partial class frmLoggin : Window` : classe fenêtre WPF associée au fichier XAML `frmLoggin.xaml`.
  `partial` indique que la définition de la classe est répartie entre XAML généré et code-behind.
* `InitializeComponent()` : instancie les contrôles définis en XAML et connecte les événements (`Click=...`).
* `btnOk_Click` / `btnAnnuler_Click` : gestionnaires d’évènements reliés aux boutons.
  `MessageBox.Show(...)` affiche une boîte de dialogue avec :

  * le **texte**,
  * le **titre** (caption),
  * les **boutons** (`MessageBoxButton.OK`),
  * l’**icône** (`MessageBoxImage.Exclamation`).

Remarque de qualité :

* Dans votre extrait initial, les guillemets typographiques `‘‘ ’’` doivent être remplacés par des guillemets standards `" "` pour compiler.
* Nom de classe : on préfère `frmLogin` (typo mineure).



## 2.2. Tableau ultra-détaillé sur `MessageBoxImage.Exclamation`

| Aspect                        | Détail                                                                                                                                                                        |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Nom de l’énumération          | `MessageBoxImage.Exclamation`                                                                                                                                                 |
| Synonyme WPF                  | `MessageBoxImage.Warning` (même valeur)                                                                                                                                       |
| Valeur numérique              | `48` (héritée de Win32 `MB_ICONEXCLAMATION` = 0x30)                                                                                                                           |
| Apparence visuelle            | Triangle jaune avec point d’exclamation noir, à gauche du message                                                                                                             |
| Sens/UI intent                | Avertissement, action risquée, conséquence potentielle, entrée invalide mais récupérable                                                                                      |
| Son système associé           | Son “Exclamation” de Windows (équivalent à `SystemSounds.Exclamation`)                                                                                                        |
| Boutons typiques              | `OK`, `OKCancel`, `YesNo`, `YesNoCancel` selon le scénario                                                                                                                    |
| Scénarios d’usage recommandés | Confirmation d’une action potentiellement dangereuse (supprimer, écraser un fichier), champs manquants, limites non critiques                                                 |
| À éviter pour                 | Erreurs fatales (utiliser `Error/Hand/Stop`), information neutre (`Information/Asterisk`), questions pures (`Question`)                                                       |
| Accessibilité                 | Icône et son aident la perception du niveau de gravité. Le texte doit rester clair et non alarmiste                                                                           |
| Comportement clavier          | `Enter` active le bouton par défaut, `Esc` le bouton d’annulation si présent                                                                                                  |
| DPI/Thème                     | Icône rendue par le système, adaptée au thème et aux échelles DPI                                                                                                             |
| Localisation                  | Le titre et le message doivent être localisés ; l’icône est standard système                                                                                                  |
| Exemple minimal               | `MessageBox.Show("Attention : action irréversible.", "Avertissement", MessageBoxButton.OK, MessageBoxImage.Exclamation);`                                                     |
| Exemple avec choix            | `var r = MessageBox.Show("Supprimer ce fichier ?", "Avertissement", MessageBoxButton.YesNo, MessageBoxImage.Exclamation); if (r == MessageBoxResult.Yes) { /* supprimer */ }` |
| Overload avec propriétaire    | `MessageBox.Show(this, "…", "Avertissement", MessageBoxButton.OK, MessageBoxImage.Exclamation);` centre la boîte sur la fenêtre                                               |
| Equivalence Win32             | `MB_ICONWARNING` / `MB_ICONEXCLAMATION`                                                                                                                                       |



# 2.3. Référence rapide : toutes les icônes possibles de `MessageBoxImage`

| Icône         | Valeur | Synonymes (même valeur) | Visuel                      | Usage recommandé                          |
| ------------- | -----: | ----------------------- | --------------------------- | ----------------------------------------- |
| `None`        |      0 | —                       | Aucune icône                | Messages neutres très simples             |
| `Hand`        |     16 | `Stop`, `Error`         | Rond rouge avec croix/paume | Erreur critique, opération impossible     |
| `Question`    |     32 | —                       | Point d’interrogation       | Question ouverte requérant un choix clair |
| `Exclamation` |     48 | `Warning`               | Triangle jaune “!”          | Avertissement, attention requise          |
| `Asterisk`    |     64 | `Information`           | Rond bleu “i”               | Information, confirmation non critique    |

Notes :

* Les synonymes partagent exactement la même valeur (comportement identique).
* La sélection de l’icône doit refléter la **gravité** et **l’intention** du message.



# 2.4. Bonnes pratiques

1. **Choisir l’icône selon la gravité**

   * Critique bloquant : `Error/Hand/Stop`
   * Avertissement récupérable : `Exclamation/Warning`
   * Simple information : `Information/Asterisk`
   * Question nécessitant décision : `Question`

2. **Toujours proposer un libellé clair et actionnable**

   * Préférer des phrases orientées action.
   * Exemple : “Supprimer définitivement ce document ?” plutôt que “Avertissement”.

3. **Adapter les boutons au contexte**

   * Confirmation binaire : `YesNo`
   * Validation/annulation : `OKCancel`
   * Simple notification : `OK`

4. **Traiter le retour**

   * `var result = MessageBox.Show(...);`
     Basculer selon `MessageBoxResult` (`OK`, `Cancel`, `Yes`, `No`).

5. **Centrez la boîte sur la fenêtre principale**

   * Utiliser la surcharge avec `owner` : `MessageBox.Show(this, ...)`.



# 2.5. Exemples prêts à copier

## A) Avertissement avec choix

```csharp
var res = MessageBox.Show(
    "Cette opération va écraser le fichier existant. Continuer ?",
    "Avertissement",
    MessageBoxButton.YesNo,
    MessageBoxImage.Exclamation);

if (res == MessageBoxResult.Yes)
{
    // Exécuter l’action
}
```

## B) Notification d’entrée invalide (récupérable)

```csharp
MessageBox.Show(
    "Le champ Nom d’utilisateur est requis.",
    "Avertissement",
    MessageBoxButton.OK,
    MessageBoxImage.Exclamation);
```

## C) Variante centrée sur la fenêtre

```csharp
MessageBox.Show(
    this,
    "Veuillez vérifier les informations saisies.",
    "Avertissement",
    MessageBoxButton.OK,
    MessageBoxImage.Exclamation);
```


