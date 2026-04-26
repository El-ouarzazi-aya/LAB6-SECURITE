# LAB6-SECURITE
# 📱 Lab — Analyse Statique d'APK avec MobSF

> **Cadre pédagogique** | Cybersécurité Mobile | VM Mobexler + MobSF v4.5.0 | réalisé par: El Ouarzazi Aya

---

## 📋 Table des matières

1. [Vue d'ensemble](#vue-densemble)
2. [Prérequis](#prérequis)
3. [Installation et configuration](#installation-et-configuration)
4. [Structure du dossier de travail](#structure-du-dossier-de-travail)
5. [Déroulement du lab](#déroulement-du-lab)
6. [Résultats obtenus](#résultats-obtenus)
7. [Vulnérabilités identifiées](#vulnérabilités-identifiées)
8. [Corrélation OWASP MASVS](#corrélation-owasp-masvs)
9. [Commandes de référence](#commandes-de-référence)
10. [Ressources](#ressources)

---

## 🎯 Vue d'ensemble

Ce lab introduit l'**analyse statique d'applications Android** à l'aide de **MobSF** (Mobile Security Framework), un outil open-source d'analyse automatisée de sécurité mobile.

L'analyse s'effectue dans un environnement isolé (VM Mobexler) et permet d'identifier les vulnérabilités potentielles **sans exécuter l'application**.

### Objectifs pédagogiques
- Comprendre le processus d'analyse statique d'un APK
- Identifier les composants sensibles d'une application Android
- Interpréter un rapport d'analyse de sécurité mobile
- Reconnaître les vulnérabilités courantes (OWASP MASVS)
- Formuler des recommandations de sécurité
- Produire un mini-rapport d'audit professionnel

### APK analysé
| Champ | Valeur |
|---|---|
| **Nom** | InsecureBankv2 |
| **Package** | com.android.insecurebankv2 |
| **Version** | 1.0 |
| **SHA256** | b18af2a0e44d7634bbcdf93664d9c78a2695e050393fcfbb5e8b91f902d194a4 |
| **Taille** | 3.3 MB |
| **Score sécurité MobSF** | **28/100** 🔴 |

---

## ⚙️ Prérequis

- VM **Mobexler** installée dans VirtualBox (≥ 6 GB RAM recommandé)
- Connexion Internet dans la VM
- APK pédagogique (`app-debug.apk`)

---

## 🛠️ Installation et configuration

### Problèmes rencontrés et solutions

> La VM Mobexler tourne sur **Debian Buster** (dépôts obsolètes).
> MobSF v4.5.0 requiert Python 3.12 et SQLite 3.31+.
> Voici les étapes complètes pour une installation fonctionnelle.

### 1. Corriger les dépôts Debian

```bash
sudo nano /etc/apt/sources.list
# Remplacer le contenu par :
# deb http://archive.debian.org/debian buster main contrib non-free
# deb http://archive.debian.org/debian-security buster/updates main contrib non-free

sudo apt-get update -o Acquire::Check-Valid-Until=false
```

### 2. Compiler SQLite 3.45

```bash
cd /tmp
wget https://www.sqlite.org/2024/sqlite-autoconf-3450200.tar.gz
tar xzf sqlite-autoconf-3450200.tar.gz
cd sqlite-autoconf-3450200
./configure --prefix=/usr/local
make && sudo make install
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
echo 'export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH' >> ~/.zshrc
```

### 3. Installer Python 3.12 via pyenv

```bash
curl https://pyenv.run | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
source ~/.zshrc

LDFLAGS="-L/usr/local/lib" CPPFLAGS="-I/usr/local/include" pyenv install 3.12.0
pyenv global 3.12.0
python3 --version  # doit afficher Python 3.12.0
```

### 4. Installer MobSF

```bash
cd /opt
sudo git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF
~/.pyenv/versions/3.12.0/bin/python3 -m pip install poetry
~/.pyenv/versions/3.12.0/bin/python3 -m poetry env remove --all
~/.pyenv/versions/3.12.0/bin/python3 -m poetry env use ~/.pyenv/versions/3.12.0/bin/python3
~/.pyenv/versions/3.12.0/bin/python3 -m poetry install
```

### 5. Initialiser la base de données

```bash
cd /opt/Mobile-Security-Framework-MobSF
~/.pyenv/versions/3.12.0/bin/python3 -m poetry run python manage.py migrate --run-syncdb
~/.pyenv/versions/3.12.0/bin/python3 -m poetry run python manage.py createsuperuser
# Username : admin / Password : admin1234
```

### 6. Lancer MobSF

```bash
sudo kill -9 $(lsof -t -i:8000) 2>/dev/null
cd /opt/Mobile-Security-Framework-MobSF
~/.pyenv/versions/3.12.0/bin/python3 -m poetry run gunicorn \
  -b 127.0.0.1:8000 mobsf.MobSF.wsgi:application \
  --workers=1 --threads=2 --timeout=3600
```

Accès : **http://127.0.0.1:8000**

---

## 📁 Structure du dossier de travail

```
~/apk_analysis/2026-04-26/
├── app-debug.apk              # APK analysé (InsecureBankv2)
├── apk_hash.txt               # Hash SHA256 pour traçabilité
├── analyse_info.txt           # Métadonnées de l'analyse
├── permissions.txt            # Permissions dangereuses identifiées
├── vulnerabilites.txt         # Vulnérabilités classées par sévérité
├── correlation_masvs.txt      # Corrélation OWASP MASVS
├── rapport_final.md           # Mini-rapport d'audit
└── MobSF_Report_*.pdf         # Rapport complet exporté depuis MobSF
```

---

## 📋 Déroulement du lab

| Task | Description | Durée |
|---|---|---|
| **Task 1** | Préparation de l'environnement | 10 min |
| **Task 2** | Lancement de MobSF | 5-10 min |
| **Task 3** | Import et analyse de l'APK | 10-15 min |
| **Task 4** | Analyse du manifeste et permissions | 15-20 min |
| **Task 5** | Analyse de la configuration réseau | 15 min |
| **Task 6** | Analyse du code et des ressources | 20-25 min |
| **Task 7** | Corrélation OWASP MASVS | 15-20 min |
| **Task 8** | Export et analyse du rapport | 10-15 min |
| **Task 9** | Rédaction du mini-rapport | 20-30 min |

---

## 📊 Résultats obtenus

### Score de sécurité
```
Score MobSF : 28 / 100  🔴
```

### Résumé des findings

| Sévérité | Nombre |
|---|---|
| 🔴 Critique | 4 |
| 🟠 Élevée | 3 |
| 🟡 Moyenne | 2 |
| 🟢 Faible | 0 |

### Permissions
- **Dangereuses :** 7/9
- **Normales :** 2/9
- **Trackers détectés :** 3 (Google AdMob, Analytics, Tag Manager)

---

## 🔓 Vulnérabilités identifiées

### 🔴 Critiques

#### 1. Débogage activé en production
```xml
<!-- AndroidManifest.xml -->
android:debuggable="true"
```
**Impact :** Permet d'attacher un débogueur (JDWP/Frida), dumper la mémoire, intercepter les appels

#### 2. Secrets hardcodés
```
"This is the super secret key 123"
"superSecurePassword"
+ 23 clés cryptographiques en clair
```
**Impact :** Extractibles directement depuis l'APK sans exécution

#### 3. Vulnérabilité Janus (CVE-2017-13156)
```
v1 signature: True
v2 signature: False  ← manquant
v3 signature: False  ← manquant
```
**Impact :** Injection de code malveillant sur Android 5.0-8.0

#### 4. StrandHogg 2.0
```
Activities vulnérables :
- com.android.insecurebankv2.PostLogin
- com.android.insecurebankv2.DoTransfer
- com.android.insecurebankv2.ViewStatement
```
**Impact :** Phishing par hijacking de tâche Android

### 🟠 Élevées

#### 5. Composants exportés sans protection
```xml
android:exported="true"  <!-- sans permission requise -->
```
Composants exposés : `PostLogin`, `DoTransfer`, `ViewStatement`, `TrackUserContentProvider`, `MyBroadCastReceiver`

#### 6. Sauvegarde ADB non restreinte
```xml
android:allowBackup="true"
```
**Impact :** `adb backup` extrait toutes les données sans root

#### 7. SDK minimum trop bas
```xml
minSdkVersion="15"  <!-- Android 4.0 non patché -->
```

---

## 🛡️ Corrélation OWASP MASVS

| Vulnérabilité | Référence MASVS | Statut |
|---|---|---|
| Sauvegarde non restreinte | MASVS-STORAGE-1 | ❌ Non conforme |
| Secrets hardcodés | MASVS-STORAGE-2 | ❌ Non conforme |
| Composants exportés | MASVS-PLATFORM-1 | ❌ Non conforme |
| Permissions excessives | MASVS-PLATFORM-3 | ❌ Non conforme |
| SDK obsolète | MASVS-CODE-1 | ❌ Non conforme |
| Signature APK faible | MASVS-CODE-2 | ❌ Non conforme |
| Debug en production | MASVS-RESILIENCE-2 | ❌ Non conforme |

**Score de conformité : 0/7 contrôles** 🔴

---

## 💡 Recommandations prioritaires

1. **Désactiver** `android:debuggable="true"` en production
2. **Supprimer** tous les secrets hardcodés → utiliser **Android Keystore**
3. **Ajouter** la signature APK v2 et v3
4. **Restreindre** les composants exportés avec `android:exported="false"`
5. **Désactiver** `android:allowBackup="false"`
6. **Mettre à jour** `minSdkVersion` à 29 minimum (Android 10)
7. **Supprimer** les permissions non nécessaires (`SEND_SMS`, `READ_CONTACTS`)

---
