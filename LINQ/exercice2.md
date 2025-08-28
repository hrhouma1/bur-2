

### 1) LINQ : quel est l’avantage principal de l’« exécution différée » (deferred execution) ?

A. Elle exécute plus vite toutes les requêtes
B. Elle reporte l’exécution jusqu’à l’itération réelle de la séquence
C. Elle empêche l’utilisation de `ToList()`
D. Elle convertit automatiquement en tableaux
**Réponse : B**
**Explication :** En différé, l’expression n’est évaluée que lors de l’énumération (ex. `foreach`, `Count()`), ce qui permet de refléter les modifications de la source.

---

### 2) Quelle méthode force l’exécution immédiate d’une requête LINQ ?

A. `Where`
B. `Select`
C. `ToList`
D. `Where(...).Select(...)`
**Réponse : C**
**Explication :** `ToList()` (comme `ToArray()` ou `ToDictionary()`) **matérialise** la requête immédiatement.

---

### 3) Avec `List<int> l = new() {1,2,3,4,5,3};`, quel code retourne seulement les pairs en **syntaxe requête** ?

A. `l.Where(x => x % 2 == 0)`
B. `from x in l where x % 2 == 0 select x`
C. `l.Select(x => x % 2 == 0)`
D. `from x in l select x % 2 == 0`
**Réponse : B**
**Explication :** En syntaxe requête : `from … where … select …`.

---

### 4) Même question que Q3 mais en **syntaxe méthode** matérialisée :

A. `l.Where(x => x % 2 == 0).ToList()`
B. `l.Select(x => x % 2 == 0).ToList()`
C. `l.ToList().Where(x => x % 2 == 0)`
D. `l.Where(x => x % 2 == 0).Select(x => x).First()`
**Réponse : A**
**Explication :** `Where` filtre, `ToList()` force l’exécution.

---

### 5) Que retourne `var q = l.Where(x => x % 2 == 0);` tant que l’on ne l’itère pas ?

A. Un `List<int>` déjà rempli
B. Un `IQueryable<int>`
C. Un `IEnumerable<int>` différé
D. Un `int[]`
**Réponse : C**
**Explication :** Sur les collections en mémoire, LINQ to Objects retourne typiquement un `IEnumerable<T>` évalué en différé.

---

### 6) Quelle différence entre `First()` et `FirstOrDefault()` ?

A. Aucune
B. `FirstOrDefault()` retourne la valeur par défaut si séquence vide
C. `First()` retourne `null` si vide
D. `FirstOrDefault()` lève toujours une exception si vide
**Réponse : B**
**Explication :** `First()` lève une exception si vide, `FirstOrDefault()` renvoie `default(T)`.

---

### 7) Quelle méthode vérifie si **au moins un** élément satisfait un prédicat ?

A. `All`
B. `Any`
C. `Contains`
D. `Count`
**Réponse : B**
**Explication :** `Any(predicate)` teste l’existence d’au moins un élément correspondant.

---

### 8) Quelle méthode vérifie si **tous** les éléments satisfont un prédicat ?

A. `All`
B. `Any`
C. `Contains`
D. `Sum`
**Réponse : A**
**Explication :** `All(predicate)` retourne vrai seulement si le prédicat est vrai pour chaque élément.

---

### 9) Pour projeter/changer la forme des données, on utilise principalement…

A. `Where`
B. `Select`
C. `GroupBy`
D. `Join`
**Réponse : B**
**Explication :** `Select` réalise la **projection** (transformation) des éléments.

---

### 10) Quelle méthode « aplatie » une séquence de séquences (ex. liste de listes) ?

A. `Select`
B. `SelectMany`
C. `GroupBy`
D. `Distinct`
**Réponse : B**
**Explication :** `SelectMany` effectue une projection et **aplanit** les sous-séquences en une seule.

---

### 11) Dans LINQ, `OrderBy(...).ThenBy(...)` sert à…

A. Regrouper puis filtrer
B. Ordonner par un critère puis sous-ordonner par un autre
C. Filtrer deux fois
D. Empêcher l’exécution différée
**Réponse : B**
**Explication :** `OrderBy` définit le tri principal, `ThenBy` ajoute des critères secondaires.

---

### 12) `GroupBy` retourne une séquence…

A. D’éléments simples
B. De clés uniquement
C. De groupes `IGrouping<TKey, TElement>`
D. D’index entiers
**Réponse : C**
**Explication :** Chaque groupe expose une `Key` et une séquence d’éléments.

---

### 13) Quelle méthode supprime les doublons d’une séquence selon l’égalité par défaut ?

A. `Union`
B. `Distinct`
C. `Except`
D. `Intersect`
**Réponse : B**
**Explication :** `Distinct()` enlève les doublons.

---

### 14) Quelle différence principale entre `IEnumerable<T>` et `IQueryable<T>` ?

A. Aucune
B. `IQueryable<T>` permet la traduction d’expressions vers d’autres sources (ex. SQL)
C. `IEnumerable<T>` ne peut pas être itéré
D. `IQueryable<T>` est uniquement pour les tableaux
**Réponse : B**
**Explication :** `IQueryable` exprime l’arborescence d’expression, souvent traduite côté provider (LINQ to SQL/EF).

---

### 15) Quel opérateur de quantification est le **plus performant** pour savoir s’il existe au moins un élément ?

A. `Count() > 0`
B. `First() != null`
C. `Any()`
D. `Sum() > 0`
**Réponse : C**
**Explication :** `Any()` court-circuite dès le premier match, contrairement à `Count()` qui parcourt potentiellement toute la séquence.

---

### 16) Quel appel provoque l’**exécution immédiate** ?

A. `Where(...)`
B. `Select(...)`
C. `ToArray()`
D. `Select(...).Where(...)`
**Réponse : C**
**Explication :** `ToArray()` matérialise comme `ToList()`.

---

### 17) Sur une `List<int>`, quel appel **ne** change pas le comportement différé/immédiat mais change seulement le type retourné en mémoire ?

A. `AsQueryable()`
B. `AsEnumerable()`
C. `ToList()`
D. `ToArray()`
**Réponse : B**
**Explication :** `AsEnumerable()` bascule vers l’interface `IEnumerable<T>` sans matérialiser.

---

### 18) Pour joindre deux séquences par clé (équijointure), on utilise…

A. `GroupBy`
B. `Join`
C. `SelectMany`
D. `Aggregate`
**Réponse : B**
**Explication :** `Join` effectue l’équijointure (clé externe/interne).

---

### 19) Quelle méthode réalise une **jointure groupée** (un parent avec sa collection d’enfants) ?

A. `GroupJoin`
B. `Join`
C. `SelectMany`
D. `Concat`
**Réponse : A**
**Explication :** `GroupJoin` associe un élément externe à un **groupe** d’éléments internes correspondants.

---

### 20) Différence `Union`, `Intersect`, `Except` :

A. `Union` = intersection, `Intersect` = union, `Except` = différence
B. `Union` = union, `Intersect` = intersection, `Except` = différence
C. Tous font la même chose
D. `Except` = union
**Réponse : B**
**Explication :** Ce sont les ensembles classiques : union, intersection et différence.

---

### 21) Quelle méthode limite le nombre de résultats retournés ?

A. `Skip`
B. `Take`
C. `Where`
D. `Select`
**Réponse : B**
**Explication :** `Take(n)` retourne les `n` premiers éléments.

---

### 22) Quelle méthode saute les `n` premiers éléments puis retourne le reste ?

A. `Skip`
B. `Take`
C. `Select`
D. `OrderBy`
**Réponse : A**
**Explication :** `Skip(n)` ignore les `n` premiers éléments.

---

### 23) Quel est l’effet de `Single()` sur une séquence vide ou avec plus d’un élément ?

A. Retourne `default(T)`
B. Lève une exception si 0 ou >1 élément
C. Retourne le premier
D. Retourne le dernier
**Réponse : B**
**Explication :** `Single()` exige exactement un seul élément. Sinon, exception.

---

### 24) `SingleOrDefault()` diffère de `FirstOrDefault()` car…

A. Ils sont identiques
B. `SingleOrDefault()` exige au plus un élément
C. `FirstOrDefault()` exige exactement un élément
D. `FirstOrDefault()` lève une exception si plus d’un élément
**Réponse : B**
**Explication :** `SingleOrDefault()` accepte 0 ou 1; `FirstOrDefault()` accepte 0 ou plusieurs (mais renvoie le premier).

---

### 25) Avec `l.Where(x => x%2==0)` sur une `List<int>` mutée après la déclaration mais avant l’itération, que se passe-t-il en **différé** ?

A. Les modifications de `l` sont reflétées
B. Rien ne change
C. Exception garantie
D. `Where` devient `ToList()`
**Réponse : A**
**Explication :** En différé, l’énumération lit l’état **courant** de la collection au moment de l’itération.

---

### 26) En syntaxe requête, quel mot-clé introduit une variable intermédiaire calculée ?

A. `into`
B. `let`
C. `var`
D. `from`
**Réponse : B**
**Explication :** `let` permet de nommer une expression pour la réutiliser dans la requête.

---

### 27) `Aggregate` sert à…

A. Filtrer une séquence
B. Trier une séquence
C. Réduire/accumuler tous les éléments en une seule valeur
D. Joindre deux séquences
**Réponse : C**
**Explication :** `Aggregate` applique un accumulateur (ex. concaténer, calculer un total avec logique spécifique).

---

### 28) `Select` vs `Where` :

A. `Select` filtre, `Where` projette
B. `Select` projette, `Where` filtre
C. Les deux filtrent
D. Les deux projettent
**Réponse : B**
**Explication :** `Where` garde/élimine; `Select` transforme la forme.

---

### 29) Pourquoi `Count()` peut-il être moins efficace que `Any()` pour tester l’existence ?

A. Parce qu’il ne compile pas
B. Parce qu’il parcourt potentiellement toute la séquence
C. Parce qu’il force la conversion en tableau
D. Parce qu’il trie d’abord
**Réponse : B**
**Explication :** `Count()` doit compter, alors qu’`Any()` s’arrête dès qu’il trouve un élément.

---

### 30) Pour convertir une requête différee en résultat fixe à un instant T, on utilise…

A. `AsEnumerable()`
B. `Where()`
C. `Select()`
D. `ToList()`
**Réponse : D**
**Explication :** `ToList()` **matérialise** la séquence immédiatement en une liste.
