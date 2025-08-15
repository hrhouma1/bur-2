# Code 

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="10"/>  <!-- Colonne 0 -->
        <ColumnDefinition Width="Auto"/>  <!-- Colonne 1 -->
        <ColumnDefinition Width="Auto"/>  <!-- Colonne 2 -->
        <ColumnDefinition Width="Auto"/>  <!-- Colonne 3 -->
        <ColumnDefinition Width="50"/>  <!-- Colonne 4 -->
    </Grid.ColumnDefinitions>

    <Grid.RowDefinitions>
        <RowDefinition Height="*"/> <!-- Ligne 0 -->
        <RowDefinition Height="Auto"/>  <!-- Ligne 1 -->
        <RowDefinition Height="Auto"/>  <!-- Ligne 2 -->
        <RowDefinition Height="Auto"/>  <!-- Ligne 3 -->
        <RowDefinition Height="Auto"/>  <!-- Ligne 4 -->
        <RowDefinition Height="*"/> <!-- Ligne 5 -->
    </Grid.RowDefinitions>

    <Image Source="Resources/Clé.png"
           Grid.Column="1"
           Grid.Row="0"
           Grid.RowSpan="6"/>

    <Label Content="_Nom d’utilisateur :"
           Grid.Column="2"
           Grid.Row="1"
           HorizontalAlignment="Right"
           Margin="0,2"
           Target="{Binding ElementName=txtUtilisateur}"/>

    <TextBox x:Name="txtUtilisateur"
             Grid.Column="3"
             Grid.Row="1"
             Width="150"
             Margin="0,2"
             VerticalAlignment="Center"/>

    <Label Content="_Mot de passe :"
           Grid.Column="2"
           Grid.Row="2"
           HorizontalAlignment="Right"
           Margin="0,2"
           Target="{Binding ElementName=txtMotPasse}"/>

    <PasswordBox x:Name="txtMotPasse"
                 Grid.Column="3"
                 Grid.Row="2"
                 Width="150"
                 Margin="0,2"
                 VerticalAlignment="Center"/>

    <WrapPanel Grid.Column="3"
               Grid.Row="4">
        <Button x:Name="btnOk"
                Grid.Column="2"
                Grid.Row="4"
                Content="_OK"
                Width="50"
                Margin="0,2"
                Click="btnOk_Click"/>
        <Button x:Name="btnAnnuler"
                Grid.Column="3"
                Grid.Row="4"
                Content="_Annuler"
                Width="50"
                Margin="50,2,0,2"
                Click="btnAnnuler_Click"/>
    </WrapPanel>
</Grid>
```
