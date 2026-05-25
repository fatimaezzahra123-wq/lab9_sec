# LAB 9 – Analyse de surface d'attaque Android avec Drozer

> **Cours** : Sécurité des applications mobiles  
> **Type** : Audit défensif en environnement autorisé  
> **Outil principal** : Drozer  
> **Standard** : OWASP MASVS / MASTG

---

## Objectifs pédagogiques

- Maîtriser l'utilisation de Drozer pour l'analyse de sécurité Android
- Identifier les composants Android exposés et leurs vulnérabilités
- Évaluer les risques liés aux composants mal configurés
- Documenter méthodiquement les résultats d'un audit de sécurité
- Proposer des remédiations conformes aux standards OWASP MASVS

---

## Architecture du lab

```
PC Hôte (Drozer Console)
        │
        │  ADB (tcp:31415)
        ▼
Émulateur Android (API 28-30)
        │
        ▼
Application de test (VulnerableApp.apk) + Agent Drozer
```

---

## Application analysée

| Champ | Valeur |
|-------|--------|
| Nom | VulnerableApp |
| Package | `com.example.vulnerableapp` |
| Version | 1.0 |
| Plateforme | Android API 28–30 |

---

## Étapes du lab

### Étape 1 – Configuration de l'environnement

```bash
# Vérifier ADB
adb version
adb devices

# Installer l'agent Drozer et l'application vulnérable
adb install drozer-agent.apk
adb install VulnerableApp.apk

# Configurer le port forwarding
adb forward tcp:31415 tcp:31415
```

> **Screenshot** : Émulateur avec agent Drozer activé

![Étape 1 – Configuration](screenshots/etape1_config.png)

---

### Étape 2 – Connexion et validation

```bash
# Lancer la console Drozer
drozer console connect

# Vérifier la connexion
dz> device
dz> list
dz> run information.device
```

> **Screenshot** : Console Drozer connectée

![Étape 2 – Connexion Drozer](screenshots/etape2_connexion.png)

---

### Étape 3 – Cartographie des composants exposés

```bash
dz> run app.package.list
dz> run app.package.list -f vulnerable
dz> run app.package.info -a com.example.vulnerableapp
dz> run app.activity.info -a com.example.vulnerableapp
dz> run app.service.info -a com.example.vulnerableapp
dz> run app.broadcast.info -a com.example.vulnerableapp
dz> run app.provider.info -a com.example.vulnerableapp
```

> **Screenshot** : Résultats de la cartographie

![Étape 3 – Cartographie](screenshots/etape3_cartographie.png)

#### Composants identifiés

| Type | Composant | Exporté | Protection | Risque |
|------|-----------|---------|------------|--------|
| Activity | LoginActivity | ✅ Oui | ❌ Aucune | Élevé |
| Activity | UserProfileActivity | ✅ Oui | ⚠️ Permission faible | Moyen |
| Service | DataSyncService | ✅ Oui | ❌ Aucune | Moyen |
| Receiver | BootReceiver | ✅ Oui | ❌ Aucune | Faible |
| Provider | UserDataProvider | ✅ Oui | ❌ Lecture/Écriture libres | **Critique** |

---

### Étape 4 – Vérification des protections

```bash
dz> run app.package.manifest com.example.vulnerableapp
dz> run app.activity.info -a com.example.vulnerableapp -i
dz> run app.provider.info -a com.example.vulnerableapp -p
dz> run scanner.provider.finduris -a com.example.vulnerableapp
dz> run app.provider.finduri com.example.vulnerableapp
```

> **Screenshot** : Analyse des protections

![Étape 4 – Protections](screenshots/etape4_protections.png)

---

### Étape 5 – Analyse des risques

| Composant | Risque | Scénario d'abus |
|-----------|--------|-----------------|
| LoginActivity | Contournement d'authentification | Intent direct vers l'activité |
| UserDataProvider | Fuite de données | Lecture libre de l'URI du provider |
| DataSyncService | Exécution non autorisée | Démarrage du service depuis une app tierce |
| BootReceiver | Déclenchement non autorisé | Intent BOOT_COMPLETED simulé |
| UserProfileActivity | Accès aux données profil | Permission trop faible |

---

### Étape 6 – Collecte de preuves

> **Screenshot** : Preuves de découvertes

![Étape 6 – Preuves](screenshots/etape6_preuves.png)

---

## Résumé exécutif

L'application `com.example.vulnerableapp` présente une **surface d'attaque étendue** avec 5 composants Android exposés sans protection adéquate. La vulnérabilité la plus critique concerne le `UserDataProvider` qui permet un accès libre en lecture/écriture aux données utilisateur depuis n'importe quelle application installée sur l'appareil.

**Score de sécurité estimé : 30/100**

| Sévérité | Nombre |
|----------|--------|
| 🔴 Critique | 1 |
| 🟠 Élevée | 2 |
| 🟡 Moyenne | 1 |
| 🔵 Faible | 1 |

---

## Mapping OWASP MASVS

| ID | Vulnérabilité | Référence MASVS |
|----|---------------|-----------------|
| V1 | Activities exportées sans protection | MSTG-PLATFORM-1 |
| V2 | Content Providers mal protégés | MSTG-STORAGE-2 |
| V3 | Services exportés sans validation | MSTG-PLATFORM-2 |
| V4 | Broadcast Receivers sans validation | MSTG-PLATFORM-3 |
| V5 | Permissions insuffisantes | MSTG-AUTH-1 |

---

## Livrables

| Fichier | Description |
|---------|-------------|
| `rapport_final.md` | Rapport complet d'audit |
| `triage.csv` | Tableau de vulnérabilités priorisé |
| `checklist_fin.md` | Checklist de conformité de l'audit |
| `screenshots/` | Captures d'écran des étapes |

---

## Méthodologie

- Analyse statique du manifeste Android
- Cartographie des composants exposés avec Drozer
- Vérification des protections en place
- Analyse des risques potentiels
- Conformité OWASP MASVS/MASTG

---

> ⚠️ **Cadre légal** : Ce lab est réalisé dans un environnement contrôlé sur émulateur. Aucune donnée réelle n'est manipulée. Toutes les analyses sont strictement défensives.
