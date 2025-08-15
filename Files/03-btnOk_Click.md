
```csharp
private void btnOk_Click(object sender, EventArgs e)
{
    // Récupération des valeurs saisies par l’utilisateur  
    string utilisateur = txtUtilisateur.Text;
    string motPasse = txtMotPasse.Password;

    // Affichage du message de bienvenue
    MessageBox.Show(
        $"Bonjour !{Environment.NewLine}" +
        $"Utilisateur : {utilisateur}{Environment.NewLine}" +
        $"Mot de passe : {motPasse}{Environment.NewLine}",
        "Bienvenue !",
        MessageBoxButton.OK
    );
}
```

