
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


Si tu veux, je peux aussi te fournir **la version intégrée directement dans ton code `frmLoggin`** avec à la fois le bouton **OK** qui affiche ce message et le bouton **Annuler** qui ferme la fenêtre.
Veux-tu que je te fasse cette version complète ?
