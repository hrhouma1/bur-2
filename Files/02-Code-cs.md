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
            MessageBox.Show(
                "Vous avez cliqué sur le bouton OK.",
                "Attention !",
                MessageBoxButton.OK,
                MessageBoxImage.Exclamation
            );
        }

        private void btnAnnuler_Click(object sender, RoutedEventArgs e)
        {
            MessageBox.Show(
                "Vous avez cliqué sur le bouton Annuler.",
                "Attention !",
                MessageBoxButton.OK,
                MessageBoxImage.Exclamation
            );
        }
    }
}
```

