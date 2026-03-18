# RAPPORT D'ANALYSE STATIQUE - PROJET LAB8
**Date d'analyse :** 18 Mars 2026  
**Analyste :** Zaynab Bellafrikh (4IIR-G2)  
**Application :** Lab8 (com.example.lab8)  

---

## 1. Informations générales
- **APK analysé :** `app-debug.apk`
- **SHA-256 :** `fdcb4f8b24435020a850c58bb91ff78085f6494feb9bfeda924b7a9ea3b18e49`
- **Version :** 1.0 (Target SDK 36)
- **Outils utilisés :** MobSF v4.0, VM Mobexler, JADX-GUI.

## 2. Résumé exécutif
L'analyse statique de l'application **Lab8** révèle un niveau de risque **ÉLEVÉ** (Security Score MobSF : 33/100). Les vulnérabilités majeures concernent l'absence de chiffrement des communications réseau et l'exposition potentielle de données personnelles via les journaux système. Bien que le code ne contienne pas de secrets hardcodés, l'application est configurée pour un environnement de développement non sécurisé, rendant les données des étudiants vulnérables à l'interception.

## 3. Vulnérabilités critiques

### 3.1 Communication réseau non chiffrée (HTTP)
- **Sévérité :** Élevée 🔴
- **Référence MASVS :** MASVS-NETWORK-1
- **Description :** L'application autorise explicitement le trafic HTTP non chiffré via l'attribut `android:usesCleartextTraffic="true"`.
- **Preuve :** Fichier `AndroidManifest.xml` et endpoints détectés : `http://10.0.2.2/lab8/ws/createetudiant.php`.
- **Impact :** Un attaquant sur le même réseau peut intercepter ou modifier les données des étudiants (Nom, CNE) via une attaque Man-in-the-Middle (MITM).
- **Remédiation :** Passer `usesCleartextTraffic` à `false` et migrer l'infrastructure vers le protocole **HTTPS**.

### 3.2 Exposition de données sensibles dans les Logs (CWE-532)
- **Sévérité :** Moyenne 🟡
- **Référence MASVS :** MASVS-STORAGE-3
- **Description :** L'application utilise des fonctions de journalisation (`Log.d`, `System.out.println`) pour traiter des informations métiers.
- **Preuve :** Alerte identifiée dans `com.example.lab8.AddEtudiant` et `com.example.lab8.MainActivity`.
- **Impact :** Les données privées saisies par l'étudiant sont lisibles via la console système (`adb logcat`) par toute personne ayant un accès physique ou via debug au terminal.
- **Remédiation :** Supprimer les logs de debug ou configurer **ProGuard/R8** pour les retirer automatiquement lors de la compilation "Release".

## 4. Autres observations
- **Composants exportés :** Le composant `androidx.profileinstaller.ProfileInstallReceiver` est exporté sans protection, augmentant inutilement la surface d'attaque.
- **Permissions :** Les permissions `INTERNET` et `ACCESS_NETWORK_STATE` sont déclarées, ce qui est cohérent avec les besoins de l'application mais nécessite un transport sécurisé (TLS).

## 5. Recommandations prioritaires
1. **Migration TLS :** Remplacer toutes les URLs `http://` par `https://` sur le serveur et dans le code Java.
2. **Durcissement du Manifeste :** Désactiver le mode `debuggable` et interdire le trafic en clair.
3. **Protection du code :** Implémenter l'obfuscation pour protéger la logique métier et masquer les endpoints d'API.

## 6. Annexes
- **Endpoints identifiés :**
  - `http://10.0.2.2/lab8/ws/createetudiant.php` (Source: AddEtudiant.java)
  - `http://10.0.2.2/lab8/ws/loadetudiant.php` (Source: MainActivity.java)
- **Composants exportés :** 1 Receiver identifié.
