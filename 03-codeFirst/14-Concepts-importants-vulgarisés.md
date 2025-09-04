# Concepts du cours DAO / EF en vulgarisation

## 1) **Annotations**

Les annotations sont des **petits mots-clés** que tu écris directement au-dessus de ta classe ou de tes propriétés pour donner des instructions à Entity Framework (EF).
Imagine que tu colles un **Post-it** sur un tiroir pour dire « Ceci est le tiroir principal » ou « Ce tiroir doit contenir au maximum 20 caractères ».
Dans le code, ça donne par exemple :

* `[Key]` → ça indique la clé primaire (identifiant unique du compte).
* `[Required]` → ça dit que la valeur ne peut pas être vide.
* `[MaxLength(20)]` → ça fixe une limite de longueur pour un champ texte.
* `[Column("C_Numero")]` → ça précise le vrai nom de la colonne en base.

Avec les annotations, pas besoin d’aller configurer ta base à la main : le code dit à EF comment créer la table SQL. C’est pratique, mais attention : si tu veux des réglages très complexes (plusieurs clés, relations compliquées), ça devient limité. Là, on passera au **Fluent API**.



## 2) **LINQ (Language Integrated Query)**

LINQ, c’est un langage qui permet de **poser des questions** à tes données directement dans C#.
Au lieu d’écrire des boucles `foreach` à rallonge, tu peux dire :

* « Donne-moi tous les comptes dont le solde est supérieur à 100 ».
* « Calcule la moyenne de tous les soldes ».

Exemple :

```csharp
var comptesRiches = comptes.Where(c => c.Solde > 100);
```

C’est comme si C# intégrait un **mini-Excel** dans ton code : tu peux filtrer, trier, compter, faire des sommes.
Ce qu’il faut retenir :

* **Différé** = la requête n’est pas exécutée tant que tu n’as pas demandé les résultats (ex: `ToList()` déclenche l’exécution).
* **Immédiat** = tu forces l’exécution (ex: `Count()`, `Sum()`).
  LINQ marche à la fois sur des **listes en mémoire** et sur des **tables en base via EF** (on appelle ça LINQ to Entities).



## 3) **DTO (Data Transfer Object)**

Un DTO, c’est une **boîte simplifiée** qui contient uniquement les informations dont tu as besoin pour transmettre des données, par exemple vers une interface utilisateur ou une API.
Imagine que tu as un gros classeur de comptes avec 50 colonnes, mais que tu veux juste donner à ton collègue le **numéro et le solde**. Tu ne vas pas lui photocopier les 50 pages, tu fais une **fiche résumée**.
C’est ça, un DTO.

Exemple :

```csharp
public class CompteDto {
   public string Numero { get; set; }
   public float Solde { get; set; }
}
```

Ensuite, dans ton code :

```csharp
var dto = new CompteDto { Numero = compte.Numero, Solde = compte.Solde };
```

C’est utile car :

* ça protège tes données sensibles (tu ne donnes pas tout),
* ça évite de trop charger la mémoire,
* ça fait une couche de séparation claire entre le **modèle interne** et ce que tu montres à l’extérieur.



## 4) **Fluent API**

Le Fluent API, c’est une **façon plus puissante et flexible** de configurer ta base que les annotations.
Au lieu d’écrire les règles directement dans ta classe avec des Post-it (annotations), tu as un **bureau de contrôle central** (dans `ContextEf`, méthode `OnModelCreating`) où tu définis toutes les règles.
Exemple :

```csharp
modelBuilder.Entity<Compte>()
    .ToTable("T_Comptes", "dbo")
    .Property(c => c.Numero)
    .HasColumnName("C_Numero")
    .HasMaxLength(20)
    .IsRequired();
```

C’est comme si tu avais un **manuel de construction** pour ta maison, séparé des pièces elles-mêmes. Tu peux dire :

* quel est le nom exact de la table,
* quelles colonnes sont obligatoires,
* la longueur maximale,
* et même les relations entre plusieurs tables.

Avantage : tout est centralisé, et c’est plus complet. Inconvénient : c’est un peu plus verbeux à lire.



## 5) **Seed (initialisation de données)**

Le “seed” (graine en anglais), c’est l’action de **planter des données de départ** dans ta base quand tu la crées.
Exemple : quand tu ouvres une nouvelle application bancaire, tu veux avoir déjà **quelques comptes de test** pour jouer.
Avec EF, tu peux créer une classe `InitCompte` qui insère automatiquement 7 comptes (101 à 107) dans ta table.
C’est comme un **kit de démarrage** pour ta base : tu n’as pas besoin de saisir manuellement les données après la création.

Exemple :

```csharp
protected override void Seed(ContextEf ctx){
    ctx.CompteEntities.Add(new Compte("101", DateTime.Now, 100));
    ctx.SaveChanges();
}
```

Ça permet de tester immédiatement ton code CRUD (Create, Read, Update, Delete) sans perdre du temps à remplir à la main.


















## 6) **ContextEf (le “cerveau” qui parle à la base de données)**

En Entity Framework, on a besoin d’un **intermédiaire** entre ton code C# et la base SQL. Cet intermédiaire, c’est une classe qu’on appelle **DbContext**.

Tu peux l’imaginer comme un **traducteur officiel** :

* D’un côté, tu lui parles en C# avec tes classes (`Compte`, `Client`, etc.).
* De l’autre côté, il traduit en **SQL** pour discuter avec la base de données.

Dans ton projet, cette classe s’appelle **ContextEf**.
Elle hérite de `DbContext`, ce qui veut dire : *“je suis une spécialisation de DbContext, je sais parler avec une base bien précise”*.

Exemple simplifié de ton code :

```csharp
public class ContextEf : DbContext
{
    public ContextEf() : base("C_BdMaBanque") { }

    public DbSet<Compte> CompteEntities { get; set; }
}
```

Quelques points à retenir :

1. `base("C_BdMaBanque")` → ça dit au traducteur quelle **connexion** utiliser (elle est définie dans `App.config`).
2. `DbSet<Compte>` → c’est comme une **table virtuelle** en C#, qui correspond à une vraie table SQL (ici `Comptes`).
3. Chaque fois que tu écris `context.CompteEntities.Add(...)` puis `context.SaveChanges()`, EF traduit ça en `INSERT INTO` en SQL et l’envoie à la base.

En résumé : **ContextEf, c’est le chef d’orchestre qui connecte ton monde C# avec la base SQL**.



## 7) **OnModelCreating (le bureau des règles spéciales)**

La méthode **OnModelCreating** se trouve dans ta classe `ContextEf`.
Son rôle ? Permettre de **poser des règles précises** pour dire comment tes classes doivent être traduites en tables SQL.

Imagine que tu sois dans une école. Par défaut, l’école décide que :

* tous les élèves auront un casier avec un numéro automatique,
* tous les noms seront inscrits avec 255 caractères maximum,
* et la liste des élèves sera dans une table qui s’appelle “Eleves”.

Mais toi, tu veux changer ces règles par défaut. Tu vas donc au **bureau du directeur** (c’est `OnModelCreating`) et tu déposes ton règlement spécial :

* “Non, je veux que la table s’appelle `T_Comptes` et pas `Comptes`.”
* “Le champ `Numero` doit être limité à 20 caractères.”
* “Le champ `Numero` s’appelle en réalité `C_Numero` en base.”

Exemple :

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    modelBuilder.Entity<Compte>()
        .ToTable("T_Comptes", "dbo")
        .Property(c => c.Numero)
        .HasColumnName("C_Numero")
        .HasMaxLength(20)
        .IsRequired();
}
```

Traduction vulgarisée :

* `ToTable("T_Comptes", "dbo")` → je veux que cette classe soit liée à la table SQL `T_Comptes`.
* `HasColumnName("C_Numero")` → en base, cette propriété `Numero` sera appelée `C_Numero`.
* `HasMaxLength(20)` → limite la taille à 20 caractères.
* `IsRequired()` → impossible de laisser ce champ vide (équivalent d’un `NOT NULL`).

Donc, **OnModelCreating, c’est le bureau des exceptions** : tu y définis toutes les règles spéciales que tu veux appliquer à ta base, au-delà des conventions automatiques d’Entity Framework.
















## 8) **CRUD (Create, Read, Update, Delete)**

Le CRUD, c’est un acronyme anglais qui signifie **Créer – Lire – Mettre à jour – Supprimer**.
En gros, ce sont les **4 opérations de base** que tu fais avec n’importe quelle donnée, que ce soit dans une base de données, dans un fichier, ou même sur un papier dans ton bureau.

* **Create (Créer)** : tu ajoutes une nouvelle donnée. Exemple : ouvrir un nouveau compte bancaire. En SQL, c’est un `INSERT`.
* **Read (Lire)** : tu recherches ou consultes une donnée. Exemple : afficher le solde d’un compte. En SQL, c’est un `SELECT`.
* **Update (Mettre à jour)** : tu modifies une donnée existante. Exemple : déposer 100 \$ sur un compte. En SQL, c’est un `UPDATE`.
* **Delete (Supprimer)** : tu effaces une donnée. Exemple : fermer un compte bancaire. En SQL, c’est un `DELETE`.

Dans ton code avec Entity Framework :

* **Create** → `context.CompteEntities.Add(cpt); context.SaveChanges();`
* **Read** → `context.CompteEntities.FirstOrDefault(c => c.Numero == "101");`
* **Update** → tu modifies la propriété, puis `context.SaveChanges();`
* **Delete** → `context.CompteEntities.Remove(cpt); context.SaveChanges();`

👉 **Image simple** : imagine une armoire à dossiers.

* `Create` → tu ajoutes un nouveau dossier.
* `Read` → tu prends un dossier pour le lire.
* `Update` → tu ajoutes une feuille ou corriges une info dans un dossier existant.
* `Delete` → tu retires un dossier et tu le mets à la poubelle.

C’est aussi simple que ça : **CRUD = le cycle de vie de tes données**.



## 9) **Seed (ensemencement des données)**

Le mot “Seed” veut dire **graine** en anglais. Dans le contexte de bases de données, ça veut dire **remplir automatiquement la base avec quelques données de départ**.

Pourquoi ?

* Parce qu’une base vide, c’est comme un jardin sans fleurs : difficile de tester ou de montrer quelque chose.
* Le seed te permet de dire : “Dès que ma base est créée, mets-moi déjà ces comptes-là pour que je puisse commencer à travailler dessus”.

Exemple dans ton projet :

```csharp
protected override void Seed(ContextEf context)
{
    context.CompteEntities.Add(new Compte("101", new DateTime(2016,10,10), 100));
    context.CompteEntities.Add(new Compte("102", new DateTime(2017,7,18), 0));
    // … et ainsi de suite jusqu’au compte 107
    context.SaveChanges();
}
```

👉 **Image simple** :

* Quand un supermarché ouvre, les rayons ne sont pas vides. On y met déjà du pain, du lait, des fruits.
* C’est pareil avec une base : le seed remplit les rayons pour que tout soit prêt à l’usage.













## 10) **Tracking vs AsNoTracking**

Quand tu utilises Entity Framework pour aller chercher des données, EF se comporte comme un **assistant personnel** :

* Il garde en mémoire ce que tu as chargé.
* Il surveille si tu fais des changements dessus.
* Et quand tu dis `SaveChanges()`, il va automatiquement appliquer les modifications dans la base.

C’est ce qu’on appelle le **tracking** (le suivi).
Exemple :

```csharp
var cpt = context.CompteEntities.First(c => c.Numero == "101");
cpt.Solde += 100;
context.SaveChanges(); // EF voit le changement et génère un UPDATE SQL
```

Mais parfois, tu veux juste **lire les données sans rien modifier** (par exemple afficher une liste). Dans ce cas, le tracking est inutile et consomme de la mémoire.
Tu peux alors dire à EF : **“Ne surveille pas, je veux juste regarder”** avec `AsNoTracking()`.
Exemple :

```csharp
var comptes = context.CompteEntities.AsNoTracking().ToList();
```

👉 **Image simple** :

* Tracking = comme un vigile qui suit chaque client dans un magasin pour voir ce qu’il prend et le facturer.
* AsNoTracking = tu regardes juste les rayons, personne ne te suit.

---

## 11) **Transactions**

Une transaction, c’est comme un **contrat indivisible** avec ta base de données :

* soit **toutes** les actions réussissent,
* soit **aucune** n’est appliquée.

Imagine que tu transfères 100 \$ d’un compte A vers un compte B :

1. Tu débites A de 100 \$.
2. Tu crédite B de 100 \$.

Si la première opération réussit mais que la deuxième échoue (panne, bug, etc.), tu te retrouves avec un solde faux.
La transaction garantit que soit **les deux opérations passent ensemble**, soit **rien ne passe** (comme si on annulait tout).

En EF, tu écris :

```csharp
using (var transaction = context.Database.BeginTransaction())
{
    try {
        // opérations multiples
        context.SaveChanges();
        transaction.Commit(); // valider
    } catch {
        transaction.Rollback(); // annuler
    }
}
```

👉 **Image simple** :
C’est comme un distributeur automatique : si tu tapes ton code, mais que le billet reste bloqué dans la machine, la transaction annule le retrait pour ne pas te voler ton argent.

---

## 12) **SQL natif**

Entity Framework génère automatiquement du SQL pour toi (quand tu écris du LINQ, il traduit en SQL).
Mais parfois, tu veux écrire **toi-même ta requête SQL** (plus complexe, plus rapide, ou spécifique).
C’est ça, le **SQL natif** : tu donnes directement la commande SQL à EF.

Exemple dans ton projet :

```csharp
string query = "SELECT IdCompte as CompteId, C_Numero as Numero, C_Solde as Solde, C_DateCreation as DateCreation FROM T_Comptes WHERE C_Numero = @p0";
return context.Database.SqlQuery<Compte>(query, numero).FirstOrDefault();
```

👉 **Image simple** :

* Normalement, EF est comme **Google Translate** : tu parles en LINQ, il traduit en SQL.
* Avec SQL natif, tu dis : “Laisse tomber, je parle directement en SQL”.

C’est puissant, mais il faut faire attention : tu perds la sécurité et la simplicité de LINQ. Il faut bien protéger tes requêtes contre les injections SQL.



Sans seed, tu devrais **tout insérer à la main** à chaque fois que tu crées ta base. Avec seed, tout est prêt automatiquement dès le premier lancement.














## 13) **DTO vs Modèle métier**

On a déjà évoqué le **DTO (Data Transfer Object)** comme une version simplifiée d’un objet.
Le **modèle métier**, lui, c’est la **vraie classe complète** qui représente un objet du monde réel (ex: un `Compte` avec son ID, son numéro, son solde, sa date de création, et plein d’autres choses).

👉 Différence vulgarisée :

* Le **modèle métier**, c’est ton **passeport complet** : il contient ton nom, ta date de naissance, ton adresse, ta photo, etc.
* Le **DTO**, c’est juste une **photocopie simplifiée** que tu donnes pour prouver ton âge (par exemple, juste ton nom et ta date de naissance).

Pourquoi on sépare les deux ?

* **Sécurité** → éviter de transmettre des données sensibles.
* **Performance** → ne charger que ce qui est utile.
* **Clarté** → séparer ce qu’on stocke en base (modèle métier) et ce qu’on échange avec le monde extérieur (DTO).

Exemple :

* Modèle métier : `Compte { Id, Numero, Solde, DateCreation, HistoriqueTransactions }`
* DTO : `CompteDto { Numero, Solde }`



## 14) **Repository Pattern (DAO)**

Le **Repository Pattern**, c’est le design pattern qui dit :
“Sépare complètement la **logique métier** (ce que ton application fait) de la **logique d’accès aux données** (comment tu les récupères).”

En pratique :

* Ta couche **métier** sait manipuler un objet `Compte`.
* Ta couche **DAO (Data Access Object)** sait aller chercher les `Comptes` dans une base, une API, ou même un fichier texte.

👉 **Image vulgarisée** :

* Ton code métier, c’est un cuisinier.
* Le DAO/Repository, c’est le marché.
  Le cuisinier dit “donne-moi des tomates” → il ne se soucie pas si elles viennent d’une serre locale, d’un import ou d’une boîte de conserve.

Avantages :

* Si tu changes de base (SQL Server → PostgreSQL → MongoDB), ton code métier ne change pas, seul le DAO change.
* Tu peux tester plus facilement (par exemple en remplaçant un vrai DAO par un DAO en mémoire).



## 15) **Migrations de base**

Quand ton application évolue, tes **classes changent** :

* tu ajoutes une nouvelle propriété (ex: `TypeCompte`),
* tu modifies une longueur (`MaxLength`),
* ou tu ajoutes une relation entre tables.

Mais attention : ta **base SQL** doit suivre ces changements. Sinon, EF ne saura plus comment s’y retrouver.
C’est là qu’interviennent les **migrations** : EF peut générer un script SQL pour **adapter la base à ton modèle C#**.

👉 **Image vulgarisée** :
Ta classe C# est comme une **carte d’identité**. Si tu changes de nom, ta carte doit être refaite.
La migration, c’est la **procédure administrative** qui fait que la nouvelle carte (la base) correspond à ta vraie identité (le code).

En pratique, avec EF Code First, tu utilises des commandes :

```bash
Add-Migration AjoutTypeCompte
Update-Database
```

Cela va générer un fichier de migration (du SQL automatique) et mettre à jour la base.

Sans migration, tu devrais aller à la main modifier la base (créer colonnes, tables, etc.). Avec migration, c’est EF qui s’en occupe.













## 16) **Navigation Properties (relations entre tables)**

En base de données, les données sont rarement isolées.

* Un **compte** appartient à un **client**.
* Un **client** peut avoir plusieurs **comptes**.

Ces liens, on les appelle des **relations** (1–1, 1–n, n–n).
Dans EF, on les représente par des **Navigation Properties** : ce sont des propriétés spéciales qui créent des **ponts** entre les classes.

Exemple :

```csharp
public class Client
{
    public int ClientId { get; set; }
    public string Nom { get; set; }

    // Relation : un client a plusieurs comptes
    public virtual ICollection<Compte> Comptes { get; set; }
}
```

👉 **Image vulgarisée** :

* Pense à une maison avec plusieurs pièces.
* La maison est ton `Client`, et chaque porte est une **Navigation Property** qui mène vers une pièce (`Comptes`).

Ces propriétés permettent de dire à EF : “Si je charge un client, je peux aussi naviguer vers ses comptes associés.”



## 17) **Lazy Loading vs Eager Loading**

Quand tu as des relations entre tables, une question se pose : **doit-on charger tout tout de suite, ou seulement quand on en a besoin ?**

* **Lazy Loading** (chargement paresseux) : EF attend que tu demandes explicitement la donnée pour aller la chercher.
  Exemple : tu charges un client, mais ses comptes ne seront chargés que quand tu écriras `client.Comptes`.
  Avantage → plus léger si tu n’utilises pas toutes les données.
  Inconvénient → plusieurs allers-retours à la base (peut ralentir).

* **Eager Loading** (chargement pressé) : tu dis à EF de tout charger en une fois avec `.Include()`.
  Exemple :

```csharp
var client = context.Clients.Include(c => c.Comptes).FirstOrDefault();
```

Avantage → une seule requête SQL.
Inconvénient → ça charge parfois trop de données inutiles.

👉 **Image vulgarisée** :

* Lazy Loading = tu vas au frigo chercher les aliments **uniquement quand tu en as besoin**.
* Eager Loading = tu vides tout le frigo sur la table dès le début.



## 18) **Transactions distribuées**

On a déjà parlé des transactions “simples” (tout passe ou rien ne passe).
Mais parfois, tes opérations concernent **plusieurs bases de données** ou même plusieurs serveurs différents.
Exemple :

* Tu mets à jour un compte bancaire dans une base locale,
* et tu enregistres une trace de la transaction dans une base distante.

Une **transaction distribuée** garantit que **toutes les bases** valident ensemble ou aucune ne le fait.

En pratique, c’est un peu plus rare dans les petits projets, mais ça existe dans les systèmes bancaires ou les applications qui gèrent des données dans plusieurs lieux.

👉 **Image vulgarisée** :

* C’est comme un mariage : ce n’est pas valide si une seule personne dit “oui”.
* Une transaction distribuée, c’est quand **tout le monde doit dire “oui” en même temps**, sinon on annule.



✅ Tu as maintenant :

* Les **Navigation Properties** (les liens entre objets/tables),
* Le **Lazy vs Eager Loading** (charger à la demande ou tout d’un coup),
* Les **Transactions distribuées** (plusieurs bases qui valident ensemble).













## 19) **Migrations automatiques vs manuelles**

Quand ton modèle C# évolue (tu ajoutes une propriété, tu changes un type, etc.), ta base doit suivre. C’est là que les **migrations** entrent en jeu.

* **Migration automatique** : EF essaie tout seul d’adapter la base dès que ton modèle change. Tu n’as rien à écrire, EF compare ton code et ta base, puis génère les changements.
  👉 Avantage : rapide, tu ne t’en occupes pas.
  👉 Inconvénient : dangereux en production, car EF peut faire des choix risqués (exemple : supprimer une colonne).

* **Migration manuelle** : tu génères un fichier de migration avec `Add-Migration` et tu le relis. Tu gardes donc le contrôle.
  👉 Avantage : sécurité, tu sais exactement ce qui change.
  👉 Inconvénient : un peu plus long, tu dois lancer des commandes.

**Image vulgarisée** :

* Migration automatique = comme une application qui met ton ordinateur à jour toute seule la nuit.
* Migration manuelle = tu reçois un message “voulez-vous installer la mise à jour ?” et tu lis la liste des changements avant d’accepter.



## 20) **Code First vs Database First**

Entity Framework peut travailler de deux manières différentes :

* **Code First** : tu écris tes classes en C# (`Compte`, `Client`, etc.) → EF crée la base SQL à partir de ton code.
  👉 Utile quand tu pars de zéro, tu laisses le code décider de la structure.

* **Database First** : tu pars d’une base déjà existante → EF génère automatiquement les classes C# correspondantes.
  👉 Utile quand une entreprise a déjà sa base en place et que tu dois l’utiliser.

**Image vulgarisée** :

* Code First = tu construis une maison à partir de ton plan papier (tes classes).
* Database First = tu visites une maison déjà construite et tu redessines le plan à partir des murs existants.


## 21) **LINQ avancé (GroupBy, Join, Projection)**

LINQ (Language Integrated Query) est l’outil magique qui permet d’écrire des requêtes SQL directement en C#.
Avec LINQ de base, tu fais des sélections simples (`Where`, `Select`). Mais avec LINQ avancé, tu peux faire :

* **GroupBy** : regrouper les données.
  Exemple : “donne-moi la moyenne des soldes par type de compte.”

```csharp
var result = context.CompteEntities
    .GroupBy(c => c.TypeCompte)
    .Select(g => new { Type = g.Key, Moyenne = g.Average(c => c.Solde) });
```

* **Join** : relier deux tables.
  Exemple : lier les `Clients` avec leurs `Comptes`.

```csharp
var result = from client in context.Clients
             join compte in context.CompteEntities
             on client.ClientId equals compte.ClientId
             select new { client.Nom, compte.Numero };
```

* **Projection** : transformer tes objets en une nouvelle forme.
  Exemple : ne garder que certaines infos (pas tout le modèle).

```csharp
var result = context.CompteEntities
    .Select(c => new { c.Numero, c.Solde });
```

👉 **Image vulgarisée** :

* `GroupBy` = tu ranges les élèves d’une classe par couleur de chemise.
* `Join` = tu fais se rencontrer deux listes (élèves et casiers) pour savoir qui a quel casier.
* `Projection` = tu prends juste le prénom des élèves au lieu de leur fiche complète.


✅ Voilà : tu comprends maintenant

* **Migrations automatiques vs manuelles**,
* **Code First vs Database First**,
* **LINQ avancé** avec `GroupBy`, `Join`, `Projection`.







## 22) **Les Annotations (Data Annotations)**

Les annotations sont des **petites étiquettes (attributs)** que tu écris directement au-dessus de tes propriétés dans la classe.
Elles viennent du namespace `System.ComponentModel.DataAnnotations`.

Exemple :

```csharp
[Table("T_Comptes")]
public class Compte
{
    [Key]
    [Column("IdCompte")]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public long CompteId { get; set; }

    [Column("C_Numero")]
    [Required]
    [MaxLength(20)]
    public string Numero { get; set; }

    [Column("C_Solde")]
    public float Solde { get; set; }
}
```

👉 Avantages :

* Très simple, **lisible directement dans la classe**.
* Idéal pour les petits projets ou quand tu veux que la configuration soit visible d’un coup d’œil.
* Pas besoin d’aller dans un autre fichier, tout est dans ton modèle.

👉 Inconvénients :

* Tu mélanges la **logique métier** (ta classe Compte) avec des règles de **persistance** (SQL).
* Tu es limité à ce que les annotations offrent (parfois trop simplistes).


## 23) **La Fluent API**

La Fluent API, elle, se configure dans le **Contexte EF** (méthode `OnModelCreating`).
Tu utilises des méthodes chaînées (`.HasKey()`, `.ToTable()`, `.HasMaxLength()`, etc.) pour définir tes règles.

Exemple :

```csharp
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    modelBuilder.Entity<Compte>()
        .ToTable("T_Comptes", "dbo")
        .HasKey(c => c.CompteId);

    modelBuilder.Entity<Compte>()
        .Property(c => c.Numero)
        .HasColumnName("C_Numero")
        .HasMaxLength(20)
        .IsRequired();
}
```

👉 Avantages :

* **Très puissant et flexible** (tu peux configurer tout ce que les annotations ne couvrent pas).
* Tu gardes ta classe métier **propre** (aucune dépendance aux détails SQL).
* Idéal pour des projets **complexes**, avec beaucoup de relations et de configurations fines.

👉 Inconvénients :

* Moins lisible d’un coup (tu dois ouvrir `ContextEf` pour voir la config).
* Peut sembler plus verbeux et compliqué pour un débutant.



## 24) **Quand utiliser quoi ?**

* **Annotations** :
  → Si ton projet est **petit** ou **moyen**, que tu veux aller vite et que tu veux garder les règles directement dans les classes.
  → Exemple : un projet scolaire, une démo rapide, un prototype.

* **Fluent API** :
  → Si ton projet est **grand**, que tu travailles en équipe, ou que tu veux une séparation claire entre le **code métier** et la **configuration de la base**.
  → Exemple : application bancaire, ERP, projet professionnel complexe.



## 25) **Image vulgarisée**

* **Annotations** = comme écrire les règles directement **sur la boîte** (exemple : “fragile”, “ne pas plier” collés dessus).
* **Fluent API** = comme avoir un **manuel de règles séparé** qui explique comment manipuler chaque boîte.

Les deux mènent au même résultat : la base est correctement configurée.
La différence, c’est où tu veux écrire les règles (sur l’objet lui-même ou dans un fichier séparé).







# 26 - Annotations vs Fluent API (Entity Framework)

| Critère                | **Annotations (Data Annotations)**                                                          | **Fluent API**                                                                        |
| ---------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Emplacement**        | Directement dans la classe métier (`Compte.cs`) avec des attributs `[Key]`, `[Table]`, etc. | Dans `ContextEf.cs`, méthode `OnModelCreating`, via configuration séparée.            |
| **Lisibilité**         | Très lisible d’un coup, tout est visible dans la classe.                                    | Moins visible directement, il faut aller voir le contexte.                            |
| **Simplicité**         | Simple et rapide à mettre en place (idéal pour débutants/projets petits).                   | Plus verbeux, mais très précis (idéal pour gros projets).                             |
| **Puissance**          | Limité aux attributs disponibles (pas toujours suffisant).                                  | Très puissant, permet de configurer des cas complexes (relations, contraintes, etc.). |
| **Couplage**           | Mélange un peu le métier (classe) et la persistance (SQL).                                  | Sépare bien le métier et la configuration (meilleure organisation).                   |
| **Quand l’utiliser ?** | Pour des projets scolaires, démos rapides, petites applis.                                  | Pour des applications professionnelles complexes avec beaucoup de relations.          |
| **Image vulgarisée**   | Comme **coller une étiquette sur l’objet** (ex: “fragile”).                                 | Comme écrire un **manuel externe** qui explique comment gérer chaque objet.           |



✅ **Résumé simple pour tes étudiants** :

* **Annotations** = rapides, visibles, mais limitées.
* **Fluent API** = plus longues, mais beaucoup plus puissantes et flexibles.



