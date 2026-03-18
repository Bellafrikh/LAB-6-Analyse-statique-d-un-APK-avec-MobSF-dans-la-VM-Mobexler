# LAB-6-Analyse-statique-d-un-APK-avec-MobSF-dans-la-VM-Mobexler
**Auteur :** Zaynab Bellafrikh (4IIR-G2)  
**Institution :** EMSI - École Marocaine des Sciences de l'Ingénieur  
**Date :** 18 Mars 2026  
**Outils :** MobSF, VM Mobexler

---

##  Sommaire
1. [Objectifs du Lab](#-objectifs-du-lab)
2. [Étape 0 : Préparation et Empreinte Numérique](#-étape-0--préparation-et-empreinte-numérique)
3. [Étape 1 :  Initialisation de l'Environnement Mobexler](#-étape-1--initialisation-de-l'environnement-Mobexler)
4. [Étape 2 : Analyse statique du Manifeste](#-étape-2--analyse-statique-du-manifeste)
5. [Étape 3 : Analyse des Permissions](#-étape-3--analyse-des-permissions)
6. [Étape 4 : Analyse Réseau et Endpoints](#-étape-4--analyse-réseau-et-endpoints)
7. [Étape 5 : Analyse du Code Source et Secrets](#-étape-5--analyse-du-code-source-et-secrets)
8. [Étape 6 : Corrélation OWASP MASVS](#-étape-6--corrélation-owasp-masvs)
9. [Étape 7 : Synthèse](#-étape-7--synthese)
10. [Étape 8 : Exportation et Rapport Final](#-étape-8--exportation-et-rapport-final)


---

## 🎯 Objectifs du Lab
L'objectif est de réaliser un audit de sécurité statique complet sur l'application **Lab8.apk** afin de détecter des failles de configuration, des fuites de données et des non-conformités aux standards internationaux de sécurité mobile.

---

## 🛠 Étape 0 : Préparation et Empreinte Numérique
Avant toute analyse, nous devons garantir l'intégrité de l'APK.
- **Action :** Générer le hash SHA-256 via le terminal.
- **Commande :** `sha256sum app-debug.apk`
- **Résultat :** `fdcb4f8b24435020a850c58bb91ff78085f6494feb9bfeda924b7a9ea3b18e49`

<img width="762" height="566" alt="image" src="https://github.com/user-attachments/assets/bc53678d-200d-4629-8c73-ee3e9619f59a" />

<img width="666" height="297" alt="image" src="https://github.com/user-attachments/assets/4b92db5e-5a6c-4dc2-b722-30b2797c9ef2" />

<img width="728" height="202" alt="image" src="https://github.com/user-attachments/assets/b60acc1c-2e49-4a39-8896-1acc7e118f20" />


> **Note :** Ce hash permet de s'assurer que l'APK n'a pas été modifié (injecté) entre le développement et l'audit.


---
## Étape 1 : Initialisation de l'Environnement Mobexler
Avant de commencer l'analyse, nous devons préparer la plateforme d'audit.
1. **Lancement de la VM :** Démarrer Mobexler (basée sur Ubuntu).
2. **Accès à MobSF :** Ouvrir le navigateur et accéder à `http://localhost:8000`.
3. **Upload :** Glisser-déposer le fichier `app-debug.apk` dans l'interface de MobSF pour lancer le scan statique automatique.

<img width="728" height="490" alt="image" src="https://github.com/user-attachments/assets/b9fc248c-c36d-4803-a01d-5d7bc92bbea9" />

<img width="765" height="706" alt="image" src="https://github.com/user-attachments/assets/078083ee-6edd-413a-a80a-dd0a475b52df" />

<img width="777" height="702" alt="image" src="https://github.com/user-attachments/assets/58683643-a1d7-4ead-b263-cc1f693312e0" />

<img width="995" height="652" alt="image" src="https://github.com/user-attachments/assets/23cd9fe2-58b0-4552-b898-353997b0b168" />

<img width="1600" height="589" alt="image" src="https://github.com/user-attachments/assets/bfc2e1e7-1cf5-44f3-842f-e46bad12a1d3" />

<img width="730" height="267" alt="image" src="https://github.com/user-attachments/assets/30a273ae-6e8d-42f5-8fbe-f155b01bb40c" />


---

##  Étape 2 : Analyse statique du Manifeste
Le fichier `AndroidManifest.xml` est la porte d'entrée de l'application. Nous vérifions les mauvaises configurations.
- **Points critiques identifiés :**
  - `android:usesCleartextTraffic="true"` : Autorise le trafic HTTP non chiffré.
  - `android:debuggable="true"` : Permet d'attacher un debugger au code.
- **Composants exposés :** `ProfileInstallReceiver` est marqué comme `exported="true"`.

![WhatsApp Image 2026-03-17 at 23 03 37](https://github.com/user-attachments/assets/923fba79-678d-4835-92db-3bd2f93e2107)

<img width="848" height="412" alt="image" src="https://github.com/user-attachments/assets/327a4e44-9f0d-478a-bac3-ddb4817ede3e" />

<img width="838" height="398" alt="image" src="https://github.com/user-attachments/assets/ac0e84a3-9fab-4408-a558-76c0cf680f76" />


---

## 🔐 Étape 3 : Analyse des Permissions
Nous listons les permissions demandées pour vérifier le principe du "Moindre Privilège".
- **Permissions détectées :**
  - `INTERNET` : Nécessaire pour les Web Services.
  - `ACCESS_NETWORK_STATE` : Vérifie la connexion.
- **Analyse :** Pas de permissions intrusives (caméra, contacts), ce qui est une bonne pratique.

<img width="777" height="355" alt="image" src="https://github.com/user-attachments/assets/f62f7fde-62b2-485d-adcc-4ca81f0ce849" />

---

##  Étape 4 : Analyse Réseau et Endpoints
Extraction des adresses IP et domaines codés dans l'application.
- **Endpoints critiques :**
  - `http://10.0.2.2/lab8/ws/createetudiant.php`
  - `http://10.0.2.2/lab8/ws/loadetudiant.php`
- **Risque :** L'utilisation de **HTTP** (au lieu de HTTPS) expose les données des étudiants à une interception réseau.

<img width="857" height="397" alt="image" src="https://github.com/user-attachments/assets/98a856d6-174a-4ce2-b02c-db7c11a4f7d1" />

*Analyse des Composants*

<img width="767" height="481" alt="image" src="https://github.com/user-attachments/assets/5497f724-a762-4022-b8e6-31a6ccec64d9" />

---

##  Étape 5 : Analyse du Code Source et Secrets
Utilisation de l'outil **Code Analysis** de MobSF pour scanner la logique métier.
- **Vulnérabilité de Logs (CWE-532) :** Utilisation de `Log.d()` dans `AddEtudiant.java` pour afficher des données sensibles.
- **Hardcoded Secrets :** Bonne nouvelle, aucune clé d'API ou mot de passe n'a été trouvé en clair.

<img width="851" height="462" alt="image" src="https://github.com/user-attachments/assets/43d3f268-3b62-48f4-96ee-59f4bc1667e8" />

<img width="852" height="412" alt="image" src="https://github.com/user-attachments/assets/c738eb33-f9a5-4f93-940f-d305f6b3ad13" />

---

## Étape 6 : Corrélation OWASP MASVS
Nous associons les failles trouvées aux standards de l'OWASP.
- **MASVS-NETWORK-1 :** Échec dû à l'autorisation du trafic en clair (HTTP).
- **MASVS-STORAGE-3 :** Échec dû à l'écriture de données privées dans les logs système.

<img width="717" height="370" alt="image" src="https://github.com/user-attachments/assets/5f15fd4d-c029-4853-a6d0-35a53fa6db54" />

---

## 📄 Étape 7 : Synthèse
C'est le résumé final avant la conclusion.

<img width="845" height="657" alt="image" src="https://github.com/user-attachments/assets/e39f05ab-9704-4533-8669-51d5790c7c10" />

---

## 📄 Étape 8 : Exportation et Rapport Final
Génération du rapport de synthèse destiné aux développeurs.
- **Score de sécurité final :** 33/100 (Risque Élevé).
- **Livrables générés :** `rapport_final.md`, `endpoints.txt`, `vulnerabilites.txt`.

![Génération du rapport PDF final](screens/08_final_report.png)

---

## 🏁 Conclusion et Recommandations
1. **Migration HTTPS :** Sécuriser les échanges avec TLS.
2. **Configuration Réseau :** Désactiver `usesCleartextTraffic` dans le Manifeste.
3. **Nettoyage du code :** Supprimer les logs de debug et activer l'obfuscation (ProGuard).
