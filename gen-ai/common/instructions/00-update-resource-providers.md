---
lab:
  title: Aktivieren von Ressourcenanbietern
  module: Setup
---

# Aktivieren von Azure-Ressourcenanbietern

Es gibt einige Ressourcenanbieter, die in Ihrem Azure-Abonnement registriert sein müssen. Befolgen Sie diese Schritte, um sicherzustellen, dass sie registriert sind.

1. Öffnen Sie in einem neuen Webbrowserfenster oder einer neuen Registerkarte das Azure-Portal (``portal.azure.com``).

1. Melden Sie sich mit den Microsoft-Anmeldeinformationen, die Ihrem Abonnement zugeordnet sind, beim Portal an.

1. Wählen Sie auf der **Startseite** **Abonnements** aus.

    > &#128161; Erweitern Sie alternativ das **&#8801;**-Menü, wählen Sie **Alle Dienste** aus. Wählen Sie dann in der Kategorie **Alle** die Option **Abonnements** aus.

1. Wählen Sie Ihr Azure-Abonnement.

1. Wählen Sie auf dem Blatt für Ihr Abonnement im Abschnitt **Einstellungen** die Option **Ressourcenanbieter** aus.

1. Überprüfen Sie in der Liste der Ressourcenanbieter, ob folgende Anbieter registriert sind:
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.Insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; Wenn ein Anbieter nicht registriert ist, wählen Sie diesen aus, und klicken Sie dann auf **Registrieren**.

1. Schließen Sie Ihr Webbrowserfenster oder die Registerkarte.

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites
