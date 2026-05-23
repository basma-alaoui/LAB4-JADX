         LAB 4 – Analyse statique d’un APK avec JADX GUI + dex2jar + JD-GUI
Cours : Sécurité des applications mobiles
Application cible : UnCrackable-Level1.apk (OWASP MSTG)

Vue d’ensemble
Ce laboratoire vous initie aux techniques d’analyse statique d’applications Android. Vous apprendrez à :

Extraire et inspecter la structure d’un APK

Décompiler le bytecode DEX en Java lisible

Rechercher des chaînes sensibles, des mécanismes de protection (root, debug)

Comparer deux outils de décompilation : JADX GUI et JD‑GUI

Aucune exécution de l’application n’est requise – tout se fait sur les fichiers binaires.

Objectifs pédagogiques
À la fin de ce lab, vous serez capable de :

Vérifier l’intégrité et le type d’un fichier APK.

Extraire classes.dex et le convertir en JAR.

Utiliser JADX GUI pour naviguer dans le code source reconstitué.

Identifier des anti‑tampering (root, débogage) et des secrets codés en dur.

Comparer les résultats entre JADX et JD‑GUI.

Prérequis
Machine virtuelle ou système Linux avec :

unzip, hexdump, sha256sum

JADX GUI (installé)

dex2jar (d2j-dex2jar)

JD‑GUI (Java Decompiler)

APK cible : UnCrackable-Level1.apk

Règles de sécurité et périmètre
L’APK utilisé est volontairement vulnérable (OWASP Uncrackable).

Aucune connexion réseau externe n’est nécessaire.

Ne distribuez pas le rapport avec l’APK ; gardez‑le dans votre environnement de formation.

Glossaire
Terme	Explication
APK	Android Package – archive ZIP contenant le code compilé (DEX) et les ressources.
DEX	Dalvik Executable – bytecode Android.
JAR	Java Archive – conteneur pour fichiers .class (Java).
JADX	Outil de décompilation DEX → Java avec interface graphique.
JD‑GUI	Décompilateur Java pour fichiers .class ou JAR.
Anti‑tampering	Mécanismes détectant root, débogage, modifications.
Task 1 — Préparer le workspace et vérifier l’APK (10 min)
Créer un répertoire de travail et déplacer l’APK.

bash
mkdir -p ~/Documents/Pentest_Lab/Uncrackable
mv ~/Downloads/UnCrackable-Level1.apk ~/Documents/Pentest_Lab/Uncrackable/
cd ~/Documents/Pentest_Lab/Uncrackable
Vérification du type de fichier :

bash
hexdump -n 4 UnCrackable-Level1.apk
https://Capture%2520d'%25C3%25A9cran%25202026-05-23%2520104845.png

Les bytes 50 4B 03 04 (PK\x03\x04) confirment qu’il s’agit d’une archive ZIP valide (APK).

bash
sha256sum UnCrackable-Level1.apk
Sortie : 1da8bf57d266109f9a07c01bf7111a1975ce01f190b9d914bcd3ae3dbef96f21

            Task 2 — Extraire/obtenir l’APK (5‑10 min)
L’APK a déjà été téléchargé. On peut lister son contenu sans le décompresser entièrement :

bash
unzip -l UnCrackable-Level1.apk | head -20
https://Capture%2520d'%25C3%25A9cran%25202026-05-23%2520104834.png

Observations :

classes.dex – code compilé

AndroidManifest.xml (binaire)

res/ – ressources (layouts, chaînes, icônes)

META-INF/ – signatures (CERT.RSA, CERT.SF, MANIFEST.MF)

              Task 3 — Analyse avec JADX GUI (20‑30 min)
Lancer JADX GUI et ouvrir l’APK.

https://Capture%2520d'%25C3%25A9cran%25202026-05-23%2520104655.png

L’arborescence affiche :

AndroidManifest.xml décompilé (plus lisible)

sg.vantagepoint.uncrackable1 – package principal

res/values/strings.xml – chaînes textuelles

Points d’intérêt :

android:allowBackup="true" : possible exfiltration de données

android:debuggable non présent (valeur par défaut false), mais une vérification est présente dans le code (classe b)

          Task 4 — Recherche de chaînes sensibles (15‑20 min)
Dans JADX, utiliser la fonction de recherche globale (Ctrl+Shift+F).

Recherchez "Secret" :

https://Capture%2520d'%25C3%25A9cran%25202026-05-23%2520104859.png

Résultat : dans sg.vantagepoint.a.a on trouve :

java
str = "This is the correct Secret.";
Il s’agit de la clé utilisée pour AES, pas encore du mot de passe utilisateur.

Recherchez également "verify" ou "EditText" pour localiser la méthode de vérification.

          Task 5 — Convertir DEX → JAR avec dex2jar (15‑20 min)
L’APK contient classes.dex. Convertissez‑le en JAR pour l’ouvrir avec JD‑GUI.

bash
d2j-dex2jar UnCrackable-Level1.apk -o uncrackable.jar
Parfois, il faut d’abord extraire classes.dex : unzip UnCrackable-Level1.apk classes.dex

Le fichier uncrackable.jar contient des fichiers .class (bytecode Java).

          Task 6 — Comparaison JADX vs JD‑GUI (15‑20 min)
Ouvrez uncrackable.jar avec JD‑GUI.

Naviguez vers sg.vantagepoint.uncrackable1.a.class :

https://Capture%2520d'%25C3%25A9cran%25202026-05-23%2520104818.png

On voit clairement :

Chaîne Base64 : "5UJiFctbmgbDoLXmpl12mkno8hHT4Lv8dlat8FxR2G0="

Appel à sg.vantagepoint.a.a.b() avec la clé hex "8d127684cbc37c17616d806cf50473cc"

Différences JADX / JD‑GUI :

JADX présente souvent un code plus propre (noms de variables restaurés, commentaires automatiques).

JD‑GUI reste fidèle au bytecode mais peut produire des goto ou des variables temporaires.

Les deux permettent d’identifier le mécanisme de chiffrement AES‑ECB.

           Task 7 — Rédiger le mini‑rapport (20‑30 min)
Votre rapport doit inclure :

Méthodologie : commandes utilisées, captures d’écran.

Résultats :

Chaîne secrète trouvée (après déchiffrement) : "I want to believe"

Protections : détection root (c.a(), c.b(), c.c()) et debug (b.a())

Extrait de code vulnérable : clé AES en dur dans le code.

Comparaison outils : avantages/inconvénients de JADX et JD‑GUI.

Suggestion d’amélioration : stocker la clé dans un système sécurisé (Keystore), obfusquer le code.

Task 8 — Nettoyage (5 min)
Supprimer les fichiers générés :

bash
rm uncrackable.jar
rm -rf jadx_output  # si vous avez exporté depuis JADX
Conservez l’APK original pour d’éventuels ré‑analyses.
