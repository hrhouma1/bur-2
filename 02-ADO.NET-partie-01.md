
# **Cours complet : ADO.NET – Accès aux données avec .NET**

## **1. Introduction à ADO.NET**

### 1.1 Définition

ADO.NET (*ActiveX Data Objects for .NET*) est un ensemble de classes du Framework .NET permettant d'accéder à des sources de données :

* Bases de données relationnelles (SQL Server, MySQL, Oracle…)
* Fichiers XML
* Services de données (API REST, OData…)

### 1.2 Objectifs principaux

* Connecter une application .NET à une source de données.
* Lire, insérer, mettre à jour et supprimer des données (*CRUD*).
* Manipuler les données en mémoire.
* Fournir un accès déconnecté aux données pour optimiser les performances.

### 1.3 Namespace principal

Toutes les classes ADO.NET sont regroupées dans :

```csharp
using System.Data;
using System.Data.SqlClient; // Pour SQL Server
```

---

## **2. Architecture générale d’ADO.NET**

ADO.NET se compose de **deux approches principales** :

### 2.1 Modèle connecté (*Connected Model*)

* Utilise un **DataReader**.
* Lecture en flux (*forward-only*, *read-only*).
* Idéal pour parcourir rapidement un grand volume de données.
* Nécessite une connexion ouverte durant tout le processus.

### 2.2 Modèle déconnecté (*Disconnected Model*)

* Utilise un **DataSet** ou **DataTable**.
* Les données sont stockées en mémoire.
* La connexion est ouverte uniquement pour charger ou sauvegarder les données.
* Permet des traitements hors ligne et des manipulations complexes.

---

## **3. Objets clés d’ADO.NET**

| Objet            | Rôle                                                                   | Namespace               |
| ---------------- | ---------------------------------------------------------------------- | ----------------------- |
| `SqlConnection`  | Gère la connexion à la base de données SQL Server                      | `System.Data.SqlClient` |
| `SqlCommand`     | Exécute une commande SQL ou un Stored Procedure                        | `System.Data.SqlClient` |
| `SqlDataReader`  | Lit les données en mode connecté                                       | `System.Data.SqlClient` |
| `SqlDataAdapter` | Sert de pont entre la base et un DataSet/DataTable                     | `System.Data.SqlClient` |
| `DataSet`        | Conteneur en mémoire de tables/données                                 | `System.Data`           |
| `DataTable`      | Représente une table de données en mémoire                             | `System.Data`           |
| `DataRow`        | Ligne d'une DataTable                                                  | `System.Data`           |
| `DataColumn`     | Colonne d'une DataTable                                                | `System.Data`           |
| `CommandType`    | Définit le type de commande (`Text`, `StoredProcedure`, `TableDirect`) | `System.Data`           |
| `SqlParameter`   | Représente un paramètre SQL (sécurité, requêtes paramétrées)           | `System.Data.SqlClient` |

---

## **4. Exemple basique – Connexion et lecture**

```csharp
using System;
using System.Data.SqlClient;

class Program
{
    static void Main()
    {
        string connectionString = "Server=localhost;Database=College;Trusted_Connection=True;";
        
        using (SqlConnection conn = new SqlConnection(connectionString))
        {
            conn.Open();
            Console.WriteLine("Connexion réussie.");

            string query = "SELECT Id, Nom, Prenom FROM Etudiants";
            SqlCommand cmd = new SqlCommand(query, conn);

            SqlDataReader reader = cmd.ExecuteReader();
            while (reader.Read())
            {
                Console.WriteLine($"{reader["Id"]} - {reader["Nom"]} {reader["Prenom"]}");
            }
        }
    }
}
```

**Points clés :**

* `using` : assure la fermeture automatique de la connexion.
* `SqlCommand` : exécute la requête.
* `SqlDataReader` : lit ligne par ligne.

---

## **5. Modèle connecté – SqlDataReader**

### 5.1 Avantages

* Performances optimales.
* Faible utilisation mémoire.

### 5.2 Inconvénients

* Connexion toujours ouverte.
* Lecture seulement en avant (*forward-only*).

---

## **6. Modèle déconnecté – DataSet et DataAdapter**

### 6.1 Exemple avec DataAdapter

```csharp
string connectionString = "Server=localhost;Database=College;Trusted_Connection=True;";
string query = "SELECT * FROM Etudiants";

using (SqlConnection conn = new SqlConnection(connectionString))
{
    SqlDataAdapter adapter = new SqlDataAdapter(query, conn);
    DataSet ds = new DataSet();
    adapter.Fill(ds, "Etudiants");

    foreach (DataRow row in ds.Tables["Etudiants"].Rows)
    {
        Console.WriteLine($"{row["Id"]} - {row["Nom"]} {row["Prenom"]}");
    }
}
```

---

## **7. Exécution de commandes SQL**

### 7.1 INSERT, UPDATE, DELETE

```csharp
using (SqlConnection conn = new SqlConnection(connectionString))
{
    conn.Open();
    string insertQuery = "INSERT INTO Etudiants (Nom, Prenom) VALUES (@Nom, @Prenom)";
    
    SqlCommand cmd = new SqlCommand(insertQuery, conn);
    cmd.Parameters.AddWithValue("@Nom", "Dupont");
    cmd.Parameters.AddWithValue("@Prenom", "Jean");
    
    int rows = cmd.ExecuteNonQuery();
    Console.WriteLine($"{rows} ligne(s) insérée(s).");
}
```

**Notes :**

* `ExecuteNonQuery()` : pour requêtes ne retournant pas de résultats.
* `AddWithValue` : ajoute un paramètre SQL pour éviter les injections SQL.

---

## **8. Transactions en ADO.NET**

```csharp
using (SqlConnection conn = new SqlConnection(connectionString))
{
    conn.Open();
    SqlTransaction transaction = conn.BeginTransaction();
    try
    {
        SqlCommand cmd = new SqlCommand("UPDATE Comptes SET Solde = Solde - 100 WHERE Id = 1", conn, transaction);
        cmd.ExecuteNonQuery();

        cmd = new SqlCommand("UPDATE Comptes SET Solde = Solde + 100 WHERE Id = 2", conn, transaction);
        cmd.ExecuteNonQuery();

        transaction.Commit();
        Console.WriteLine("Transaction réussie.");
    }
    catch
    {
        transaction.Rollback();
        Console.WriteLine("Transaction annulée.");
    }
}
```

---

## **9. Bonnes pratiques ADO.NET**

1. Toujours utiliser `using` pour fermer automatiquement les connexions.
2. Préférer les **requêtes paramétrées** pour éviter les injections SQL.
3. Gérer les exceptions avec `try-catch`.
4. Limiter le temps de connexion.
5. Utiliser le modèle **déconnecté** si la connexion permanente n’est pas nécessaire.

---

## **10. Schéma récapitulatif**

* **Connexion** (`SqlConnection`)
* **Commande** (`SqlCommand`)
* **Lecture directe** (`SqlDataReader`) → modèle connecté
* **Chargement mémoire** (`DataAdapter` + `DataSet`) → modèle déconnecté
* **Manipulation des données**
* **Mise à jour** (`ExecuteNonQuery`)
* **Transactions**

---

## **11. Cas avancés**

* Appel de procédures stockées (`CommandType.StoredProcedure`).
* Utilisation de `DataTable` sans DataSet.
* Mapping avec des objets (*Data Transfer Objects*).
* Gestion asynchrone avec `async` / `await` (`SqlCommand.ExecuteReaderAsync()`).
* Accès multi-bases (MySQL avec `MySql.Data.MySqlClient`).

