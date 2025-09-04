## 1) Architecture en couches (Layered Architecture)

1. Principe : séparer l’application en **couches de responsabilités** (métier, accès aux données, présentation, utilitaires) pour réduire le couplage.
2. Chaque couche expose des **contrats clairs** (interfaces) et **cache ses détails d’implémentation** aux autres couches.
3. Le **flux des dépendances** descend typiquement de haut en bas (UI → Service/Métier → DAO/Infra), pas l’inverse.
4. Le pattern favorise les **tests unitaires** (on “mocke” les couches d’en dessous, ex. DAO) et la **substituabilité** (mémoire ↔ base).
5. Il améliore la **maintenabilité** : on peut changer de SGBD, d’ORM, ou d’interface sans toucher aux règles métier.
6. Il encourage la **clarté des frontières** (contraintes métier d’un côté, détails techniques de l’autre).
7. Les **dépendances “vers l’intérieur”** (Clean/Hexagonal) protègent le domaine des effets techniques et du temps.
8. Les couches peuvent être **cohésives** : un module n’expose que ce qui est nécessaire (principe de moindre connaissance).
9. Le revers : trop de couches ou frontières peut **alourdir** le projet si mal calibré (over-engineering).
10. L’objectif final est un **design évolutif** : faire grandir l’app sans dette technique explosive.



## 2) DAO / Repository Pattern

1. **DAO/Repository** encapsule l’accès aux données et offre une **API orientée domaine** (ex. `GetCompte` plutôt que “SELECT \*”).
2. Il **masque la source** (mémoire, SQL, API distante) et rend l’appelant **indépendant** de l’implémentation.
3. Il stabilise les **contrats de données** et les **invariants métier** (un seul endroit central pour requêter).
4. Couplé à une **Unité de Travail (DbContext)**, il agrège changements et **gère la persistance atomique**.
5. Il facilite le **test** (mock/fake repository) et donc le TDD.
6. Il impose une **discipline de requêtage** (centralisée) utile pour la performance et la sécurité.
7. Danger : multiplier des méthodes ultra-spécifiques peut créer des **repositories anémiques** ou trop bavards.
8. Alternative : **spécifications** (objets décrivant des critères) pour garder l’API concise et réutilisable.
9. Limite : ne pas **cacher l’ORM** au point de perdre ses forces (projections, includes, etc.).
10. Le bon repo expose des **cas d’usage** compréhensibles, pas juste des “pipes SQL”.



## 3) Entité métier (POCO)

1. **POCO** = objet “simple”, sans dépendance au framework ; il porte **état** et **comportements** métier.
2. Il doit capturer des **invariants** (ex. `Debiter` interdit si solde insuffisant).
3. Éviter le **“modèle anémique”** (seulement des propriétés) : laisser vivre le métier **dans** l’entité.
4. Une entité a une **identité** (clé) stable dans le temps, indépendamment de ses attributs.
5. Elle ne “sait pas” comment elle est stockée ; la **persistance est un détail d’infrastructure**.
6. Les constructeurs **garantissent** un état valide dès la création (dates, valeurs par défaut).
7. Les méthodes de **mutation** doivent rester **atomiques** et respectueuses du domaine.
8. Les entités doivent être **sérialisables** sans surprise (pour logs, tests, transports).
9. La **traçabilité** (ToString, Deconstruct) aide au debug et à la lisibilité.
10. Leur **stabilité** assure la cohérence du système quand l’infrastructure évolue.


## 4) LINQ (Language Integrated Query)

1. **LINQ** unifie la façon de requêter des collections (objets, SQL via EF, XML…) avec des opérateurs **déclaratifs**.
2. Deux syntaxes : **“method”** (`Where`, `Select`, …) et **“query”** (`from … in … select …`), équivalentes.
3. **Exécution différée** : la requête est un plan ; elle s’exécute au **moment du ToList/Count/First**.
4. Avec EF, LINQ est **traduit en SQL** par le provider ; tout ce qui n’est pas traduisible doit basculer **en mémoire** (`AsEnumerable`).
5. **Projections** ciblées évitent de charger des colonnes inutiles → meilleure **perf**.
6. **Agrégations** (`Sum`, `Average`, `Min`, `Max`) doivent être protégées en cas de séquence vide.
7. **`First` vs `FirstOrDefault`** : l’un lève si vide, l’autre renvoie valeur par défaut ; choisir selon le contrat.
8. **`AsNoTracking`** (EF) en lecture pure : évite le change-tracker → moins de **surcharge**.
9. Attentions aux **pièges de client-évaluation** (calcul côté client involontaire) et aux **`Contains`** sur grandes listes.
10. LINQ encourage un **style déclaratif** lisible et testable, mais requiert une **culture de la perf** (profiling SQL).



## 5) Tuples : `ValueTuple` vs `Tuple<T1,T2>`

1. **`ValueTuple`** (C# 7+) est **struct** (valeur), nommable (`(numero, solde)`), léger et lisible.
2. **`Tuple<T1,T2>`** (classe .NET) expose `Item1/Item2`, plus verbeux et **référence** (allocation différente).
3. Les **ValueTuples** se **déconstruisent** naturellement (`var (n,s)=…`) et se prêtent aux retours multiples rapides.
4. Readability : préférer ValueTuple **nommé** pour le sens ; pas de “magie” derrière.
5. Avec EF6, la **construction de Tuple** n’est pas traduisible en SQL → basculer **en mémoire** avant de construire.
6. Les Tuples sont **transitoires** : pas de contrat métier, juste des “paquets de données”.
7. Pour exposer un API public stable, préférer **DTO nommés** plutôt que des tuples crus.
8. Attention aux **changements d’ordre** des éléments (fragilité sémantique).
9. En cas de besoin d’**égalité sémantique**, ValueTuple implémente les comparaisons structurelles.
10. Ne pas en abuser pour **remplacer des types** métier clairs ; ils servent au **piping** rapide.



## 6) DTO (Data Transfer Object)

1. **DTO** : shape de données destinée à un **consommateur** (UI, API), souvent **différente** de l’entité persistée.
2. Objectifs : **limiter** les champs exposés (sécurité), **stabiliser** le contrat, **décorréler** présentation et persistance.
3. Ils réduisent la **charge réseau** et la **sérialisation** inutile.
4. Ils évitent l’**over-posting** (l’utilisateur ne peut pas modifier des champs non exposés).
5. La **projection LINQ** crée efficacement des DTO (Server-side avec EF).
6. Versionner des DTO est **moins risqué** que de faire évoluer des entités publiques.
7. La **validation** peut être spécifique au DTO (ex. formats d’entrée) sans polluer le domaine.
8. Les mappers (automatiques ou manuels) doivent rester **prévisibles** et **testés**.
9. Un DTO n’a **pas de logique métier** lourde : il transporte, il ne “décide” pas.
10. Attention à ne pas multiplier des **DTO redondants** : penser mutualisation et clarté.



## 7) Entity Framework 6 (EF6) – ORM

1. EF6 fait l’**impédance** entre objets et tables : **mappage** entités ↔ schéma, **tracking**, **SaveChanges**.
2. **DbContext** est l’unité de travail : il **suit** les entités, cumule les changements, **flushe** en base.
3. **DbSet<T>** représente une table logique et expose les opérations LINQ.
4. **Change tracker** maintient l’état (Added/Modified/Deleted/Unchanged) et gère les **relations**.
5. **Lazy/Eager/Explicit loading** : stratégies pour navigations ; à choisir selon perf et clarté.
6. **Conventions** par défaut (PK, pluralisation, types) et **surcharge** via Fluent/Annotations.
7. Les **providers** (SQL Server, etc.) pilotent les capacités de traduction et les types SQL.
8. EF6 privilégie **Code First** (classes → schéma) mais sait aussi **Database First**.
9. **SaveChanges** n’est pas transactionnel à l’échelle multi-contexte : penser **transaction** si agrégation.
10. EF6 offre une **API mature** mais nécessite **discipline** (profiling SQL, index, projections).



## 8) Data Annotations (Mapping par attributs)

1. Annotations = **métadonnées** directement sur la classe/propriétés : **lisible** et proche du modèle.
2. Elles couvrent mapping (**\[Table] \[Column] \[Key] \[DatabaseGenerated]**), contraintes (**\[Required] \[MaxLength]**), parfois **index** (EF 6.1+).
3. Elles mélangent parfois validation d’entrée et mapping → garder les **rôles distincts**.
4. Elles suffisent pour beaucoup de cas simples et **documentent** le modèle.
5. Pour les **types SQL** précis, utiliser `Column(TypeName=…)` (ex. `decimal`, `real`).
6. **Limites** : clés composites, conventions globales, précisions fines → Fluent plus adapté.
7. **Lisibilité** : l’essentiel du contrat est **dans** l’entité (pratique pour onboarder).
8. Elles restent **déclaratives** : pas de logique ; elles guident l’ORM.
9. Attention au **mélange** avec Fluent : définir la **source de vérité** pour éviter collisions.
10. Les annotations **n’imposent pas** l’ordre des colonnes physiques (sauf EF Core avec HasColumnOrder).



## 9) Fluent API (Mapping par code)

1. **Fluent** centralise la configuration dans le **DbContext** (ou classes de configuration).
2. Il gère **tous** les cas avancés : clés composites, index nommés, relations complexes, conventions custom.
3. Il **désencombre** l’entité des attributs, gardant les POCO “purs”.
4. Il permet des **conventions globales** (ex. toutes les `string` max 200 par défaut).
5. Il facilite la **lecture d’ensemble** du mapping (un seul endroit).
6. Il joue mieux avec la **génération de migrations** (EF6 : package add-migration/update-database).
7. Il est **verbos** mais **précis** ; idéal pour les schémas ambitieux.
8. Il n’empêche pas d’utiliser **quelques annotations** sémantiques (mais clarifier la hiérarchie).
9. Les **tests** de mapping sont plus simples à isoler (configurations dédiées).
10. Le coût : un **effort initial** plus important, amorti sur la durée d’un projet sérieux.



## 10) Initializers & Seed

1. Les **initializers EF6** pilotent la **stratégie de création** : créer si absent, drop+create si modèle change, toujours drop+create.
2. Le **Seed** s’exécute **après** la création et charge des **données initiales** (ex. comptes 101–107).
3. Le Seed est **idempotent** si on teste l’existence avant insertion (utile en dev/CI).
4. En **prod**, on évite les initializers destructifs ; on préfère **migrations** + scripts maîtrisés.
5. Le Seed doit **refléter** le mapping actif (Fluent/Annotations) sous peine d’échec.
6. Les initializers aident à **synchroniser** environnement de dev et schéma courant.
7. Ils simplifient les **démos/tests** (base prête en un run).
8. Répéter un Seed sans drop peut créer des **doublons** (d’où idempotence).
9. Ils n’adressent pas l’**évolution** incrémentale du schéma (c’est le rôle des migrations).
10. Bien les **isoler** (utilitaires) pour déclenchement explicite et lisible.



## 11) Conventions EF

1. Les **conventions** définissent un comportement par défaut : noms pluriels, PK déduites, types implicites.
2. Elles accélèrent le **démarrage** (zero-config), mais ne collent pas toujours aux **normes** d’équipe/BDD.
3. On peut **les retirer** ou **les surcharger** (ex. supprimer la pluralisation).
4. Elles influencent les **schémas** (ex. `nvarchar(max)` pour `string` sans longueur).
5. Bien connaître les conventions évite des **surprises** lors de la création automatique.
6. Elles diffèrent entre **EF6** et **EF Core** (ne pas transposer aveuglément).
7. Elles sont un **filet** ; pour les projets sérieux, préférer un mapping **explicite**.
8. La combinaison **Conventions + Annotations** couvre la majorité des cas simples.
9. Les conventions impactent aussi les **relations** (clés étrangères, cascade par défaut).
10. Les logs EF aident à **auditer** ce que les conventions ont produit.



## 12) Tracking vs `AsNoTracking()`

1. **Tracking** : le contexte suit les entités pour **détecter** les modifications et gérer les relations.
2. Utile pour les **updates** et scénarios où l’objet revient pour être modifié.
3. **AsNoTracking** désactive ce suivi : idéal pour **lectures massives** et “read-only”.
4. Moins de **mémoire** et de **CPU** (pas d’instantané ni de comparaisons).
5. Choix par défaut : **NoTracking** pour lecture, **Tracking** pour édition.
6. On peut **rattacher** une entité lue en NoTracking via `Attach`/`Entry`.
7. Le tracking facilite la **concurrence optimiste** (conflits détectés via RowVersion).
8. Trop de tracking sur de grandes listes → **ralentissements** significatifs.
9. Mélanger tracking et DTO peut **simplifier** la séparation lecture/écriture.
10. Mesurer : activer les logs et profiler les **allocations**.

---

## 13) CRUD avec EF

1. **Create** : ajouter au DbSet puis `SaveChanges()` agrège en une **transaction locale**.
2. **Read** : LINQ to Entities ; penser **projections** et **NoTracking** pour perf.
3. **Update** : récupérer entité suivie ou **Attach** et marquer champs **modifiés** explicitement.
4. **Delete** : charger puis supprimer ; attention aux **FK** et à la cascade.
5. **Validation** : côté domaine (invariants) + côté présentation (DTO) + côté BDD (contraintes).
6. **Exceptions** : gérer unicité, longueurs, null, collisions de concurrence.
7. **Unit of Work** (DbContext) : regrouper modifications cohérentes avant `SaveChanges()`.
8. **Transactions** : explicites pour composer plusieurs actions critiques.
9. **Idempotence** et **retries** (transient faults) si environnement distribué.
10. Observer la **télémétrie** (temps, requêtes, locks) pour prévenir les régressions.



## 14) SQL natif

1. Utile quand LINQ ne **couvre pas** un besoin (CTE, window functions, hints).
2. Toujours **paramétrer** pour éviter l’injection (`@p0`, `@p1`) et tirer parti des plans.
3. **Mapper** les résultats (alias colonnes → propriétés) pour objets forts ou DTO.
4. Avantage : contrôle fin, accès aux **spécificités** du SGBD.
5. Inconvénient : perd une partie des **garanties** typées et de la portabilité.
6. Bien **documenter** ces requêtes (dette de maintenance plus élevée).
7. Ne pas mélanger sans discernement LINQ + SQL natif dans une même méthode (lisibilité).
8. Mesurer le **coût réseau** et le **profiling** du plan d’exécution.
9. Stocker des requêtes critiques dans des **procédures** si gouvernance DBA.
10. Rester **cohérent** : SQL natif pour exceptions, LINQ pour le quotidien.

---

## 15) Transactions

1. **Transaction** = exécution **atomique** : tout réussit ou tout est annulé.
2. Dans EF : `Database.BeginTransaction()` ou transactions **ambientes** (TransactionScope).
3. Choisir l’**isolation** (ReadCommitted, Serializable…) selon les risques de lecture sale/phantom.
4. Systématiser **try/commit/rollback** ; ne jamais ignorer les erreurs.
5. Les transactions longues augmentent la **contention** (locks) : rester **court**.
6. Virements, stocks, commandes : cas classiques où la transaction est **indispensable**.
7. En microservices, prévoir **sagas**/transactions **distribuées** (out of EF scope).
8. Sur erreurs transitoires, envisager des **retries** (policies).
9. Logger les bornes (début/commit/rollback) pour **audit**.
10. Ne pas confondre **cohérence métier** et **isolation technique** : clarifier les invariants.



## 16) Index & unicité

1. Un **index** accélère les recherches (seek) au prix d’un **coût d’écriture** supplémentaire.
2. **Index unique** impose une contrainte d’**unicité** métier (ex. `Numero` de compte).
3. On peut les définir via **Annotations** (EF 6.1+) ou **Fluent** (IndexAnnotation).
4. Nommer les index (ex. `IX_T_Comptes_Numero`) facilite le **diagnostic**.
5. Les index doivent **correspondre** aux chemins d’accès réels (WHERE/ORDER BY).
6. Trop d’index tue les inserts/updates ; choisir **parcimonie**.
7. Mesurer via **plans d’exécution** et **DMV** (SQL Server).
8. Unicité au niveau SGBD est plus **fiable** qu’un check applicatif isolé.
9. Sur colonnes `nvarchar`, limiter la **longueur** pour indexabilité.
10. Revoir périodiquement les index (évolution des requêtes).



## 17) Types numériques : `float` vs `decimal`

1. `float` (IEEE 754) représente des **approximations** binaires → erreurs d’arrondi possibles.
2. `decimal` représente des **décimales exactes** (base 10) → idéal pour **montants** financiers.
3. En SQL Server, `real/float` vs `decimal(p,s)` ; choisir selon **tolérance aux erreurs**.
4. Les agrégations sur `float` peuvent **amplifier** l’erreur (somme, moyenne).
5. `decimal(18,2)` est un **classique** pour la monnaie ; adapter `p,s` au besoin.
6. La **cohérence** type C# ↔ SQL est essentielle (mapping explicite).
7. Penser aux **performances** : `decimal` peut être un peu plus lourd, mais la **précision** prime souvent.
8. Les **arrondis** doivent être explicites (Banker’s rounding si nécessaire).
9. Documenter le **choix de type** dans les décisions d’architecture.
10. Tester des **cas limite** (grandes valeurs, cumul, conversions).



## 18) Pagination & tri

1. **Pagination** évite de charger trop de lignes : UX et perf côté serveur/ réseau.
2. Stratégie simple : **offset-limit** (`Skip/Take`) ; facile mais coûteux sur de grands offsets.
3. **Keyset pagination** (par curseur/clé) est plus **scalable** (pas de scan complet).
4. Toujours un **ORDER BY déterministe** pour éviter le “page drift”.
5. Les compteurs (`COUNT(*)`) sur grandes tables sont **chers** ; prévoir des **estimations** si besoin.
6. Les filtres doivent être **indexés** pour que la pagination soit réellement efficace.
7. Exposer page **1-based** côté UI et **documenter** la convention.
8. Prévoir des **limites** de `pageSize` pour éviter les abus (DoS).
9. Cache côté client/API peut réduire la pression (pages populaires).
10. Mesurer et **réajuster** (télémétrie, A/B test de tailles de page).



## 19) Gestion des erreurs & `null`

1. **Taxonomie** EF : `DbUpdateException`, `DbEntityValidationException`, `SqlException`, etc.
2. **Contrats** de méthode : documenter les retours (ex. `null` si introuvable).
3. Éviter les **NullReferenceException** : vérifier systématiquement les résultats `FirstOrDefault`.
4. **Agrégations** : protéger `Average/Min/Max` quand la séquence est vide.
5. **Contrainte unique** : capturer et **traduire** en erreur métier claire.
6. **Validation** multi-niveaux : DTO (syntaxe), domaine (invariants), BDD (contraintes).
7. **Logs** riches (requêtes, paramètres, horodatage) pour diagnostiquer en prod.
8. Exposer des **messages d’erreur** utiles mais **non verbeux** (sécurité).
9. Penser **résilience** (retries) vs erreurs **déterministes** (corriger données/contrats).
10. Les tests doivent inclure des **cas d’échec** et des **cas limites** (données manquantes, invalides, concurrentes).

