# Rapport d'audit de sécurité – Application Android

## Informations générales

| Champ | Valeur |
|-------|--------|
| **Application** | VulnerableApp |
| **Package** | com.example.vulnerableapp |
| **Version** | 1.0 |
| **Date d'audit** | [DATE] |
| **Auditeur** | [NOM] |
| **Outil utilisé** | Drozer 2.x |
| **Standard** | OWASP MASVS / MASTG |
| **Environnement** | Émulateur Android API 28–30 |

---

## Résumé exécutif

L'audit de sécurité de l'application `com.example.vulnerableapp` a révélé **5 vulnérabilités significatives** liées à une mauvaise configuration des composants Android. L'application expose sans protection adéquate des activités, services, broadcast receivers et content providers qui pourraient être exploités par des applications malveillantes installées sur le même appareil.

La vulnérabilité la plus critique concerne le `UserDataProvider` : ses URI sont accessibles en lecture et écriture sans aucune permission requise, exposant directement les données utilisateur. La `LoginActivity` est également exportée sans protection, permettant un contournement potentiel de l'authentification.

**Score de sécurité estimé : 30/100**

| Sévérité | Nombre | Composants concernés |
|----------|--------|----------------------|
| Critique | 1 | UserDataProvider |
| Élevée | 2 | LoginActivity, UserProfileActivity |
| Moyenne | 1 | DataSyncService |
| Faible | 1 | BootReceiver |

---

## Méthodologie

L'audit a suivi une approche défensive en 4 phases :

1. **Analyse statique du manifeste Android** – Lecture du fichier `AndroidManifest.xml` via Drozer pour identifier les composants déclarés et leurs attributs d'export.

2. **Cartographie des composants exposés** – Utilisation des modules Drozer (`app.activity.info`, `app.service.info`, `app.broadcast.info`, `app.provider.info`) pour lister tous les composants exportés.

3. **Vérification des protections en place** – Analyse des permissions déclarées, des intent-filters, et des URI accessibles via les content providers.

4. **Analyse des risques potentiels** – Évaluation de l'impact de chaque composant exposé selon les scénarios d'abus possibles, sans exploitation offensive.

---

## Découvertes principales

### V2 – UserDataProvider (CRITIQUE)

Le content provider `UserDataProvider` est exporté sans permission de lecture ou d'écriture. N'importe quelle application installée sur l'appareil peut accéder librement aux données utilisateur.

**Référence MASVS** : MSTG-STORAGE-2  
**Impact** : Fuite de données utilisateur, modification non autorisée de données

---

### V1 – LoginActivity (ÉLEVÉE)

L'activité `LoginActivity` est exportée sans aucune protection. Un attaquant peut lancer directement cette activité via un intent explicite, contournant potentiellement les mécanismes d'authentification.

**Référence MASVS** : MSTG-PLATFORM-1  
**Impact** : Contournement d'authentification

---

### V5 – UserProfileActivity (ÉLEVÉE)

L'activité `UserProfileActivity` est protégée par une permission de niveau trop faible. Des applications tierces peuvent obtenir cette permission et accéder au profil utilisateur.

**Référence MASVS** : MSTG-AUTH-1  
**Impact** : Accès non autorisé aux données de profil

---

### V3 – DataSyncService (MOYENNE)

Le service `DataSyncService` est exporté sans protection et sans validation des intents reçus. Un attaquant peut démarrer ce service depuis une application tierce pour déclencher des synchronisations non autorisées.

**Référence MASVS** : MSTG-PLATFORM-2  
**Impact** : Exécution de synchronisation non autorisée

---

### V4 – BootReceiver (FAIBLE)

Le broadcast receiver `BootReceiver` est exporté sans validation des intents. Un attaquant peut simuler un broadcast `BOOT_COMPLETED` pour déclencher ce receiver de façon non intentionnelle.

**Référence MASVS** : MSTG-PLATFORM-3  
**Impact** : Déclenchement non autorisé au démarrage

---

## Recommandations prioritaires

1. **[CRITIQUE] Protéger UserDataProvider** – Ajouter une permission `android:permission` de niveau `signature` sur le content provider. Implémenter une vérification de permission dans le code Java du provider.

2. **[ÉLEVÉE] Désactiver l'export de LoginActivity** – Définir `android:exported="false"` si l'activité n'a pas besoin d'être lancée depuis d'autres applications.

3. **[ÉLEVÉE] Renforcer la permission de UserProfileActivity** – Remplacer la permission actuelle par une permission de niveau `signature` déclarée dans le manifeste.

4. **[MOYENNE] Restreindre DataSyncService** – Définir `android:exported="false"` ou ajouter une permission `signature`. Implémenter une validation d'intent dans `onStartCommand()`.

5. **[FAIBLE] Valider les intents dans BootReceiver** – Ajouter une vérification de l'action reçue dans `onReceive()` et ajouter `android:permission="android.permission.RECEIVE_BOOT_COMPLETED"`.

---

## Annexes

- **Annexe A** : Tableau de triage complet → `triage.csv`
- **Annexe B** : Captures d'écran des preuves → `screenshots/`
- **Annexe C** : Mapping OWASP MASVS (voir tableau ci-dessous)

### Annexe C – Mapping OWASP MASVS

| ID | Vulnérabilité | Référence MASVS | Description |
|----|---------------|-----------------|-------------|
| V1 | Activities exportées sans protection | MSTG-PLATFORM-1 | L'application ne doit exposer que les composants nécessaires |
| V2 | Content Providers mal protégés | MSTG-STORAGE-2 | Aucune donnée sensible sans protection adéquate |
| V3 | Services exportés sans validation | MSTG-PLATFORM-2 | Les entrées externes doivent être validées |
| V4 | Broadcast Receivers sans validation | MSTG-PLATFORM-3 | L'application doit valider les intents reçus |
| V5 | Permissions insuffisantes | MSTG-AUTH-1 | Les mécanismes d'authentification doivent être robustes |
