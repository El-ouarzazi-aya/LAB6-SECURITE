# Rapport d'analyse statique — InsecureBankv2

## Informations générales
- **Date :** 2026-04-26
- **Analyste :** El Ouarzazi Aya
- **APK :** app-debug.apk (InsecureBankv2 v1.0)
- **Package :** com.android.insecurebankv2
- **Outils :** MobSF v4.5.0 — VM Mobexler

## Résumé exécutif
Score de sécurité : 28/100. Niveau de risque ÉLEVÉ.
Vulnérabilités critiques : débogage activé en production,
secrets hardcodés, composants exportés sans protection,
signature APK vulnérable à Janus.

## Vulnérabilités critiques

### 1. Débogage activé en production
- **Sévérité :** Critique
- **MASVS :** MASVS-RESILIENCE-2
- **Preuve :** android:debuggable=true dans AndroidManifest.xml
- **Impact :** Permet le hooking via débogueur, dump mémoire
- **Remédiation :** Désactiver en production

### 2. Secrets hardcodés
- **Sévérité :** Critique
- **MASVS :** MASVS-STORAGE-2
- **Preuve :** "This is the super secret key 123", "superSecurePassword"
- **Impact :** Compromission totale des données
- **Remédiation :** Utiliser Android Keystore

### 3. Composants exportés non protégés
- **Sévérité :** Élevée
- **MASVS :** MASVS-PLATFORM-1
- **Preuve :** PostLogin, DoTransfer, ViewStatement exportés
- **Impact :** Accès non autorisé aux activités sensibles
- **Remédiation :** Ajouter android:exported=false ou permissions

### 4. Vulnérabilité Janus
- **Sévérité :** Critique
- **MASVS :** MASVS-CODE-2
- **Preuve :** Signature v1 uniquement, v2/v3 absentes
- **Impact :** Injection de code malveillant dans l'APK
- **Remédiation :** Signer avec v2 et v3

### 5. Sauvegarde non protégée
- **Sévérité :** Élevée
- **MASVS :** MASVS-STORAGE-1
- **Preuve :** android:allowBackup=true
- **Impact :** Extraction des données via ADB
- **Remédiation :** Désactiver allowBackup

## Recommandations prioritaires
1. Désactiver android:debuggable=true en production
2. Supprimer tous les secrets hardcodés → Android Keystore
3. Ajouter signature v2/v3 à l'APK
4. Restreindre les composants exportés
5. Désactiver android:allowBackup

## Annexes
### Permissions dangereuses
SEND_SMS, READ_CONTACTS, ACCESS_COARSE_LOCATION,
GET_ACCOUNTS, USE_CREDENTIALS, READ_PROFILE,
WRITE_EXTERNAL_STORAGE

### Composants exportés
- PostLogin, DoTransfer, ViewStatement (Activities)
- TrackUserContentProvider (Content Provider)
- MyBroadCastReceiver (Receiver)
