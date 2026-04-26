# 🔐 Lab Sécurité Android — Rooting & Analyse d'intégrité

- **Module** : Sécurité des Applications Mobiles
- **Filière** : GCDSTE 
- **Auteur** : DOSSAH Yao Landry 
- **Date** : 26 Avril 2026  

---

## 📋 Fiche Périmètre

| Critère | Détail |
|---|---|
| Application / Environnement | Android Emulator 36.4.10.0 |
| Support | AVD Pixel 6 Pro — Google APIs x86_64 (API 36) |
| Objectif | Comprendre le rooting Android et ses impacts sur la sécurité |
| Type de données | Fictives uniquement — aucune donnée réelle |
| Réseau | Environnement de test isolé |

> ⚠️ **Règles du lab** : App de test + labo uniquement. Données fictives uniquement. Rien ne sort du périmètre de test. Aucun device personnel manipulé.

---

## 🎯 Objectif du Lab

Ce lab a pour but de comprendre ce que le **rooting** change sur un système Android, comment le vérifier, comment tracer les actions effectuées, et comment remettre l'environnement à zéro.

Le rooting est l'équivalent Android du *jailbreak* sur iOS ou de l'accès administrateur sur Windows. Il s'agit d'obtenir les privilèges les plus élevés sur le système (uid=0).

---

## 🧱 Concepts clés

### Sécurité Android (résumé)

1. **Sandboxing** — chaque app tourne dans son propre bac à sable isolé (UID unique), séparée des autres applications.
2. **Modèle de permissions** — l'app doit explicitement demander l'accès aux ressources sensibles (caméra, contacts, stockage...).
3. **Intégrité système** — Verified Boot + dm-verity empêchent toute modification non autorisée des partitions système.

> Le root casse les 3 à la fois : il sort du sandbox, bypasse les permissions, et désactive verity.

---

### Verified Boot

**Objectif principal** : Garantir que le système qui démarre est exactement celui prévu par le fabricant, sans modifications malveillantes.

**Chain of trust (chaîne de confiance)** : Série de vérifications où chaque composant vérifie l'authenticité du suivant avant de lui faire confiance — Bootloader → Kernel → System. Si un maillon est compromis, toute la chaîne tombe.

**Pourquoi c'est critique** : Si le démarrage est compromis, toutes les protections qui suivent peuvent être contournées. C'est comme une forteresse dont la porte principale serait ouverte.

| Couleur | Signification |
|---|---|
| 🟢 Green | Système vérifié et intègre |
| 🟡 Yellow/Orange | Bootloader déverrouillé, système modifié mais fonctionnel |
| 🔴 Red | Intégrité compromise, danger |

---

### AVB — Android Verified Boot 2.0

AVB est la version moderne de Verified Boot. Il apporte :

- **Vérification d'intégrité** : chaque partition (system, vendor, boot) est signée cryptographiquement — toute modification est détectée au démarrage.
- **Protection anti-rollback** : empêche de revenir à une ancienne version du système qui contiendrait des failles connues.
- **Sur notre AVD** : `vbmeta.digest = 7466b5c2...` visible dans les logs — hash présent mais bootloader orange = AVB non strict.

---

### Définition du Rooting

1. Le root donne les privilèges super-utilisateur (uid=0), permettant d'accéder à toutes les ressources du système sans restriction.
2. Il modifie les protections Android : sandbox cassé, permissions bypassées, verity désactivé.
3. En laboratoire, il permet d'observer des comportements normalement cachés (fichiers système, logs, données apps).
4. Risqué en production : nécessite un environnement isolé, une traçabilité complète et un reset après usage.

---

### Intérêt du Lab (environnement non opérationnel)

En labo, un environnement privilégié peut aider à :
- Observer des artefacts système normalement inaccessibles (fichiers, bases de données, logs internes)
- Analyser les comportements runtime de l'application à bas niveau
- Tester la robustesse du stockage face à un attaquant privilégié

> ⚠️ **Labo autorisé uniquement** — ces manipulations sont réservées à un environnement de test isolé, sur données fictives, dans le cadre du Module M45 ENSA Marrakech.

---

## ⚙️ Environnement de test

### Prérequis

- Android Studio installé
- SDK Platform Tools configuré dans le PATH
- AVD Pixel 6 Pro (Google APIs — sans Google Play) démarré

### Configuration PATH (si nécessaire)

```
%LOCALAPPDATA%\Android\Sdk\platform-tools
%LOCALAPPDATA%\Android\Sdk\emulator
```

---

## 🚀 Étapes réalisées

### Étape 1 — Vérification ADB

```cmd
adb devices
```

> **Résultat attendu** : `emulator-5554   device`

📸 <img width="304" height="55" alt="1" src="https://github.com/user-attachments/assets/93484abc-7be9-4d43-8f7c-7d83146c7c39" />


---

### Étape 2 — Liste des AVDs disponibles

```cmd
emulator -list-avds
```

> **Résultat** : `Medium_Phone_API_36.1` et `Pixel_6_Pro`

📸 <img width="324" height="56" alt="2" src="https://github.com/user-attachments/assets/4dad2bce-e1a3-4c10-aff8-d90263550cd1" />

---

### Étape 3 — Lancement de l'émulateur en mode writable

```cmd
emulator -avd Pixel_6_Pro -writable-system
```

> ℹ️ Le flag `-writable-system` permet de modifier les partitions système — normalement en lecture seule.

📸 <img width="859" height="177" alt="6" src="https://github.com/user-attachments/assets/23611848-1683-474f-8620-d761ed40cd8f" />


---

### Étape 4 — Activation du mode root ADB

```cmd
adb root
adb shell id
```

> **Résultat** : `uid=0(root) gid=0(root)` ✅

📸 <img width="851" height="106" alt="3" src="https://github.com/user-attachments/assets/1ae4bc69-330f-46b4-8ccb-46e459beb0c8" />

---

### Étape 5 — Vérification de l'état du système

```cmd
adb shell getprop ro.boot.verifiedbootstate
adb shell getprop ro.boot.veritymode
adb shell getprop ro.boot.vbmeta.device_state
adb disable-verity
```

| Propriété | Valeur observée | Interprétation |
|---|---|---|
| `verifiedbootstate` | `orange` | Bootloader déverrouillé |
| `veritymode` | `enforcing` → désactivé | Verity actif puis désactivé |
| `vbmeta.device_state` | *(vide)* | Normal sur AVD |

📸 <img width="874" height="324" alt="5" src="https://github.com/user-attachments/assets/7800861d-916f-4677-964e-73959e8e6448" />


---

### Étape 6 — Remontage des partitions en RW

```cmd
adb root
adb remount
```

> **Résultat** :
> ```
> Remounted /system as RW
> Remounted /vendor as RW
> Remounted /product as RW
> Remounted /system_dlkm as RW
> Remounted /system_ext as RW
> Remount succeeded
> ```

📸 <img width="308" height="161" alt="4" src="https://github.com/user-attachments/assets/744ed366-e0be-42c2-95d5-37e308c00fe5" />

---

### Étape 8 — Journalisation

```cmd
adb logcat -d > logcat_root_check.txt
```

> Le fichier `logcat_root_check.txt` est sauvegardé localement comme preuve de l'état du système.
<img width="422" height="38" alt="image" src="https://github.com/user-attachments/assets/f03662ec-b3f6-4685-b36e-a9dfaad34314" />
<img width="1429" height="756" alt="image" src="https://github.com/user-attachments/assets/c1969614-b503-43a7-b8c4-e3c6749c694b" />

---

## 📊 Matrice de risques

| # | Risque | Impact |
|---|---|---|
| 1 | Intégrité non garantie | Conclusions biaisées sur la sécurité réelle |
| 2 | Surface d'attaque accrue | Exposition si l'appareil sort du labo |
| 3 | Données sensibles exposées | Violation potentielle de confidentialité |
| 4 | Instabilité système | Tests non reproductibles |
| 5 | Mélange comptes perso/test | Fuite d'informations personnelles |
| 6 | Mauvais nettoyage fin de séance | Persistance de données sensibles |
| 7 | Réseau non isolé | Effets involontaires sur systèmes externes |
| 8 | Traçabilité insuffisante | Impossible de reproduire ou auditer |

---

## 🛡️ Mesures défensives

| # | Mesure |
|---|---|
| 1 | Réseau isolé pour éviter toute communication non contrôlée |
| 2 | Données fictives uniquement pour éliminer tout risque de fuite réelle |
| 3 | Device/AVD dédié exclusivement aux tests de sécurité |
| 4 | Snapshots ou wipe en fin de séance pour ne laisser aucune trace |
| 5 | Journal de configuration détaillé pour assurer la reproductibilité |
| 6 | Aucun compte personnel pour éviter tout mélange de données |
| 7 | Contrôle strict des APK installées pour limiter les risques |
| 8 | Horodatage + captures des étapes pour une traçabilité complète |

---

## 📚 Références OWASP

### MASVS — 2 exigences clés

| Exigence | Description |
|---|---|
| **STORAGE-1** | Les données sensibles (API keys, mots de passe, tokens) doivent être stockées de manière chiffrée — jamais en clair dans les SharedPreferences ou SQLite. |
| **NETWORK-1** | Les communications réseau doivent utiliser TLS avec une configuration correcte et vérification des certificats — pas de HTTP en clair. |

### MASTG — 2 idées de tests

1. **Test stockage** : Vérifier si les fichiers SharedPreferences contiennent des données sensibles en clair dans `/data/data/[package]/shared_prefs/` — accessible avec les privilèges root.
2. **Test logs** : Analyser les logs de l'application avec `adb logcat` pour détecter des fuites d'informations sensibles pendant l'exécution.

---

## 🔄 Remise à zéro (obligatoire fin de séance)

```cmd
adb emu avd stop
```

> 📸 <img width="640" height="84" alt="image" src="https://github.com/user-attachments/assets/46920291-ad94-46e4-a11a-958a5be4c128" />

## 📁 Structure du dépôt

```
lab-securite-android/
├── README.md
├── screenshots/
│   ├── 1.png        ← adb devices — émulateur détecté
│   ├── 2.png        ← emulator -list-avds
│   ├── 3.png        ← adb root + adb shell id (uid=0)
│   ├── 4.png        ← adb remount — partitions RW
│   ├── 5.png        ← getprop verifiedbootstate + disable-verity
│   ├── 6.png        ← lancement émulateur writable-system
│   └── reset_proof.png      ← preuve wipe AVD (à ajouter)
└── logcat_root_check.txt
```

---

*Lab réalisé dans un cadre pédagogique strictement contrôlé — Module M45, ENSA Marrakech, GCDSTE S4.*
