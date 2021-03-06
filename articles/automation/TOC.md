# Áttekintés
## [Mi az Azure Automation?](automation-intro.md)
# Bevezetés
## [Ismerkedés az Azure Automation szolgáltatással](automation-offering-get-started.md)
## Runbook-oktatóanyag
### [Grafikus runbook létrehozása](automation-first-runbook-graphical.md)
### [PowerShell-alapú runbook létrehozása](automation-first-runbook-textual-powershell.md)
### [PowerShell-alapú munkafolyamati runbook létrehozása](automation-first-runbook-textual.md)
# Útmutató
## Hitelesítés és biztonság
### [Önálló Automation-fiók létrehozása](automation-create-standalone-account.md)
### [Azure AD felhasználói fiók létrehozása](automation-create-aduser-account.md)
### [Hitelesítés beállítása az AWS használatával](automation-config-aws-account.md)
### [Automation futtató fiók létrehozása](automation-create-runas-account.md)
### [Automation-fiók konfigurációjának érvényesítése](automation-verify-runas-authentication.md)
### [Szerepköralapú hozzáférés-vezérlés kezelése](automation-role-based-access-control.md)
### [Automation-fiók kezelése](automation-manage-account.md)
## Runbookok létrehozása
### [Runbook-típusok](automation-runbook-types.md)
### [Runbookok létrehozása és importálása](automation-creating-importing-runbook.md)
### [Szöveges runbookok szerkesztése](automation-edit-textual-runbook.md)
### [Grafikus runbookok szerkesztése](automation-graphical-authoring-intro.md)
### [Runbook tesztelése](automation-testing-runbook.md)
### [A PowerShell-munkafolyamat megismerése](automation-powershell-workflow.md)
### [Gyermek runbookok](automation-child-runbooks.md)
### [Runbook kimenete](automation-runbook-output-and-messages.md)
### [Forráskezelés integrálása](automation-source-control-integration.md)
## Automatizálás
### [Runbook indítása](automation-starting-a-runbook.md)
### [Runbook indítása webhookból](automation-webhooks.md)
### [Runbook bemeneti paramétereinek konfigurálása](automation-runbook-input-parameters.md)
### [Hibakezelés grafikus runbookokban](automation-runbook-graphical-error-handling.md)
### [Runbook-feladat nyomon követése](automation-runbook-execution.md)
### [Runbook beállításainak módosítása](automation-runbook-settings.md)
### [Azure Automation-adatok kezelése](automation-managing-data.md)
### [Azure Automation-runbook hívása Log Analytics-riasztásból](automation-invoke-runbook-from-omsla-alert.md)
### [JSON-objektum továbbítása Azure Automation-runbookba](automation-pass-json-string.md)
## hibrid runbook-feldolgozó
### [Hibrid runbook-feldolgozó üzembe helyezése](automation-hybrid-runbook-worker.md)
### [Runbookok futtatása feldolgozón](automation-hrw-run-runbooks.md)
## Konfigurációkezelés (DSC) üzembe helyezése
### [A célállapot-konfiguráló (DSC) áttekintése](automation-dsc-overview.md)
### [Első lépések](automation-dsc-getting-started.md)
### [Gépek előkészítése kezelésre](automation-dsc-onboarding.md)
### [Célállapot-konfigurálók fordítása](automation-dsc-compile.md)
### [Folyamatos üzembe helyezés a Chocolatey használatával](automation-dsc-cd-chocolatey.md)
### [Az Azure Automation DSC jelentési adatainak továbbítása az OMS Log Analyticsbe](automation-dsc-diagnostics.md)
## Eszközök kezelése
### [Tanúsítványok](automation-certificates.md)
### [Kapcsolatok](automation-connections.md)
### [Hitelesítő adatok](automation-credentials.md)
### [Integrációs modulok](automation-integration-modules.md)
### [Ütemezések](automation-schedules.md)
### [Változók](automation-variables.md)
### [Azure PowerShell-modulok frissítése](automation-update-azure-modules.md)
## Forgatókönyvek
### [Runbook-katalógus](automation-runbook-gallery.md)
### [Amazon-webszolgáltatásbeli virtuális gép létrehozása](automation-scenario-aws-deployment.md)
### [Azure VM-riasztás szervizelése](automation-azure-vm-alert-integration.md)
### [Virtuális gép indítása és leállítása JSON-címkékkel](automation-scenario-start-stop-vm-wjson-tags.md)
### [Erőforráscsoport eltávolítása](automation-scenario-remove-resourcegroup.md)
### [Forráskezelés integrálása a GitHub Enterprise-zal](automation-scenario-source-control-integration-with-github-ent.md)
### [Forráskezelés integrálása a VSTS-sel](automation-scenario-source-control-integration-with-VSTS.md)
### [Azure Automation-runbook hívása Log Analytics-riasztásból](automation-invoke-runbook-from-omsla-alert.md)
### [Azure Resource Manager-sablon üzembe helyezése Azure Automation PowerShell-runbookban](automation-deploy-template-runbook.md)
## Megoldások
### [Változáskövetés](../log-analytics/log-analytics-change-tracking.md)
### [Frissítéskezelés](../operations-management-suite/oms-solution-update-management.md)
### [Virtuális gépek indítása és leállítása munkaidőn kívül](automation-solution-vm-management.md)
## Figyelés
### [Azure Automation-feladat adatainak továbbítása a Log Analyticsbe](automation-manage-send-joblogs-log-analytics.md)
### [Azure Automation-fiók leválasztása a Log Analyticsről](automation-unlink-from-log-analytics.md)
## Migrate (Áttelepítés)
### [Áttelepítés az Orchestratorból](automation-orchestrator-migration.md)
### [Automation-fiók áthelyezése](automation-migrate-account-subscription.md)
## Hibaelhárítás
### [Gyakori hibák elhárítása](automation-troubleshooting-automation-errors.md)
### [A hibrid runbook-feldolgozó hibaelhárítása](automation-troubleshooting-hybrid-runbook-worker.md)
# Referencia
## [PowerShell](/powershell/module/azurerm.automation)
## [PowerShell (klasszikus)](/powershell/module/azure/?view=azuresmps-3.7.0)
## [.NET](/dotnet/api/microsoft.azure.management.automation)
## [REST](/rest/api/automation)
## [REST (klasszikus)](https://msdn.microsoft.com/library/azure/mt163781)
# Erőforrások
## [Automation bevezető videója](https://azure.microsoft.com/documentation/videos/azure-automation-101-with-powershell-and-eamon-o-reilly/)
## [Azure Automation-képzés](https://mva.microsoft.com/en-US/training-courses/automating-the-cloud-with-azure-automation-8323?l=C6mIpCay_4804984382)
## [Azure-ütemterv](https://azure.microsoft.com/roadmap/?category=monitoring-management)
## [Képzési terv](https://azure.microsoft.com/documentation/learning-paths/automation/)
## [MSDN-fórum](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=azureautomation)  
## [Díjszabás](https://azure.microsoft.com/pricing/details/automation/)  
## [Díjkalkulátor](https://azure.microsoft.com/pricing/calculator/)
## [Kibocsátási megjegyzések](https://azure.microsoft.com/updates/?product=automation)
## [Szolgáltatási hírek](https://azure.microsoft.com/updates/?product=automation)
## [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-automation)
## [Videók](https://azure.microsoft.com/documentation/videos/index/?services=automation)
