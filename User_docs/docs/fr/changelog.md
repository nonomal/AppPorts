---
outline: deep
---

# Journal des modifications

## v1.8.0

### Nouveautés

- Répertoires de scan locaux personnalisés : l'en-tête « Mac Apps Locales » dispose maintenant d'un bouton « + » pour ajouter des répertoires de scan d'applications supplémentaires. Utile pour les outils comme JetBrains Toolbox et Steam qui installent des applications en dehors de `/Applications`. Les répertoires ajoutés sont sauvegardés et automatiquement surveillés (#48).
- Synchronisation de version Stub Portal : lorsqu'une application externe est mise à jour via l'App Store, les informations de version du Stub Portal local sont automatiquement synchronisées et le cache macOS Launch Services est actualisé. Le menu « Ouvrir avec » n'affiche plus de numéros de version obsolètes (#50).
- Détection de répertoires d'outils pour Gradle (`~/.gradle`), les données de développement Android (`~/.android`) et le cache Flutter/Dart Pub (`~/.pub-cache`) (#49).
- Migration de répertoires : ajoutez des dossiers utilisateur arbitraires dans l'onglet « Migration de répertoires », migrez de grands projets, modèles, bibliothèques de ressources ou caches d'outils vers le stockage externe, puis reliez-les ou restaurez-les (#54).
- Avertissement pour les applications protégées : avant de migrer des apps App Store ou appartenant à root, AppPorts avertit que la suppression ou le remplacement automatique peut échouer faute de permissions et suggère de déplacer d'abord l'app manuellement dans Finder avant de créer un lien (#55).

### Améliorations

- Scan d'applications plus rapide : lectures Info.plist par application réduites de 7 à 1 (via cache en mémoire).
- Protection de timeout de scan : le sous-processus `codesign` a maintenant un timeout de 10 secondes.
- Limite de sécurité pour le calcul de taille : une limite de 500 000 fichiers a été ajoutée aux calculs de taille récursifs.
- Journalisation de trace de scan : journalisation TRACE par application ajoutée à la boucle de scan.
- Correspondance plus précise des répertoires de données : les mots TLD génériques comme `app`, `com`, `org` sont maintenant filtrés.
- Détection de re-liaison des répertoires d'outils plus complète : si le répertoire local est absent mais qu'un répertoire géré existe encore à l'emplacement canonique du stockage externe, AppPorts l'affiche comme « Nécessite une re-liaison » ; changer de stockage externe actualise automatiquement cet état.
- Localisation et accessibilité améliorées : les statuts des applications, répertoires de données et répertoires personnalisés, les libellés de tri/filtre, les interrupteurs de réglages et les badges de statut suivent plus systématiquement la langue choisie et exposent des libellés d'accessibilité plus clairs.
- Les tailles d'apps utilisent maintenant un cache au niveau de la session, ce qui réduit les retours à « Calcul en cours » ou les pertes de taille après actualisation (#55).
- Rollback plus sûr pour les migrations de données : avant de créer le lien, AppPorts renomme la source locale en sauvegarde cachée. Si la création du lien ou le nettoyage de la sauvegarde échoue, il conserve autant que possible la sauvegarde locale et la copie externe (#54).

### Corrections

- Correction : Trae et applications similaires scannées extrêmement lentement.
- Correction : les informations de version du Stub Portal n'étaient pas mises à jour après les mises à jour App Store.
- Correction : le bouton d'actualisation ne déclenchait pas la synchronisation de version.
- Correction : la re-liaison ou la normalisation des répertoires de données pouvait traiter un fichier ordinaire externe comme un répertoire ; les fichiers ordinaires sont maintenant rejetés et conservés.
- Correction : les corps de boîtes de dialogue multilignes pouvaient revenir au chinois dans certaines langues ; les traductions russes de l'interface ont été complétées et la boîte de dialogue système Stub Portal « stockage externe non connecté » suit maintenant la langue du système (#55).

## v1.7.0

### Nouveautés

- Ajout du statut « Migration sortante en attente » : lorsque la vraie application locale est plus récente que l'application du même nom sur le stockage externe, AppPorts la marque comme prête à être migrée vers l'extérieur afin de remplacer l'ancienne copie externe.
- Ajout d'une confirmation de re-signature pour la migration des données : avant de migrer des données à l'intérieur d'un conteneur d'application, AppPorts peut demander s'il faut appliquer automatiquement une re-signature Ad-hoc à l'application associée après la migration, afin de réduire le risque de données non reconnues, d'avertissements ou d'échecs de lancement (#44).

### Améliorations de l'interface

- Réorganisation de la barre d'outils supérieure : les boutons de bascule entre la page des applications et celle des répertoires de données utilisent désormais un style plus compact avec icône + texte.
- Optimisation de la barre d'actions des répertoires de données : le sélecteur « Répertoires d'outils / Données d'application », l'option de re-signature après migration, le bouton de restauration de la signature d'origine et le bouton d'actualisation sont regroupés dans la barre d'outils supérieure.
- Ajout du badge d'état « Migration sortante en attente » pour identifier les applications dont la version locale est plus récente que l'ancienne copie externe.
- Localisation de la boîte de dialogue de confirmation de re-signature lors de la migration des données, y compris le titre, le texte et les boutons.

### Améliorations

- Sécurité de migration d'application renforcée : lorsque la destination externe existe déjà, AppPorts ne la nettoie automatiquement que si elle est reconnue comme ancien portail géré par AppPorts, reste d'une ancienne migration, ou si l'application est en état « Migration sortante en attente ».
- Vérification de récupération des répertoires de données renforcée : la récupération automatique ne repose plus sur une taille de dossier proche, mais exige une correspondance complète des AppPorts metadata.
- Analyse des données d'application plus stable : lors d'un changement rapide d'application, les résultats d'anciennes tâches d'analyse n'écrasent plus la liste des répertoires de données de l'application actuellement sélectionnée.
- Échappement renforcé pour les commandes administrateur et AppleScript : les chemins contenant guillemets, barres obliques inverses, espaces ou caractères chinois sont traités plus sûrement.
- Localisation améliorée : correction des contenus d'aide, invites et confirmations de migration de données qui pouvaient rester en chinois ou être incomplètement traduits après un changement de langue ; traductions complétées pour toutes les langues prises en charge (#43).

### Corrections

- Correction d'un cas où la migration de répertoire de données pouvait traiter à tort un vrai répertoire externe comme cible récupérable.
- Correction d'un cas où la migration d'application pouvait supprimer par erreur une vraie application externe portant le même nom.
- Correction de la détection et du nettoyage instables des anciens portails AppPorts externes ou des restes d'anciennes migrations.
- Correction de la construction incorrecte d'AppleScript ou de commandes administrateur lorsque le chemin contient des caractères spéciaux.
- Correction d'un cas où la migration en arrière-plan ou la re-signature après migration pouvait lire une application déjà changée.
- Correction du badge « Migration sortante en attente » qui n'apparaissait pas dans la liste des applications.

## v1.6.2

- Nouveau : Re-signature automatique à la connexion. Re-signe automatiquement les applications migrées avec des signatures expirées à chaque connexion de l'utilisateur, sans action manuelle. Activé par défaut, peut être désactivé dans les Paramètres
- Amélioration : Stub Portal utilise désormais un lanceur binaire Mach-O natif au lieu du script bash hérité, corrigeant le problème où un double-clic sur les documents associés dans le Finder ne parvenait pas à ouvrir l'application externe (#42)
- Amélioration : Mise en page de la page À propos optimisée avec une zone de contenu défilable, corrigeant le contenu tronqué lorsque la fenêtre est trop petite
- Corrigé : Le Stub Portal natif était incorrectement identifié comme une application locale normale
- Corrigé : Impossible de nettoyer correctement le Stub Portal natif lors du déplacement des applications vers le stockage local
- Corrigé : Le shell de l'application était traité comme une application complète lors des opérations de liaison inverse
- Corrigé : AutoResignInstaller signalait un succès silencieusement lorsque l'installation échouait

## v1.6.1

- Corrigé : La re-signature automatique après la migration du répertoire de données signe maintenant correctement la vraie application externe au lieu du shell stub local
- Corrigé : Les opérations de re-signature et de restauration de signature résolvent maintenant correctement le chemin réel pour les applications liées
- Corrigé : La détection du statut « Re-signé » pour les applications liées identifie maintenant correctement le statut de signature de la vraie application externe
- Amélioré : La sortie des logs inclut des codes d'erreur structurés et des informations de chemin associées

## v1.6.0

- Les applications migrées n'affichent plus de badges fléchés
- Les applications à mise à jour automatique ne sont plus corrompues par les mises à jour après migration
- Ajout de la fonctionnalité de gestion de signature d'application pour corriger les messages « Endommagé » après migration
- La déconnexion du stockage externe affiche maintenant des avertissements rouges « Lien orphelin »
- Les utilisateurs de macOS 15.1+ peuvent installer des applications App Store directement sur des disques externes
- Migration des répertoires de données plus sûre : prévention de la migration accidentelle du répertoire système, récupération automatique après interruption
- Scan et calcul de taille plus rapides ; la liste ne saute plus
- Copie de fichiers vers le stockage externe plus stable ; plus d'erreurs d'interruption
- Badges de statut d'application redessinés avec des informations plus riches et des détails cliquables
- La liste d'applications conserve la sélection après actualisation ; les répertoires de données supportent la vue arborescente
- Améliorations UI : recherche, tri, cartes de groupe, chargement d'icônes, etc.
- Ajout de l'option de langue Martien
- Mises à jour des tests automatisés

## v1.5.5

- Ajout du support d'installation externe d'applications App Store macOS 15.1+
- Ajout de la fonctionnalité de re-signature automatique (exécutée automatiquement après la migration du répertoire de données)
- Ajout des tests d'audit de localisation `LocalizationAuditTests`
- Amélioration de la logique de génération du Info.plist du Stub Portal
- Correction du problème de perte d'icône Launchpad pour certaines applications après migration

## v1.4.0

- Ajout de la vue en arborescence des répertoires de données
- Ajout de la détection des répertoires d'outils (30+ outils de développement)
- Ajout de la fonctionnalité d'exportation de package de diagnostic
- Amélioration de la détection des mises à jour automatiques (Chrome, Edge et autres mises à jour personnalisées)
- Correction du mécanisme de récupération automatique après interruption de migration

## v1.3.0

- Ajout de la fonctionnalité de migration des répertoires de données
- Ajout de la gestion des signatures de code (sauvegarde/restauration des signatures originales)
- Ajout de la détection automatique des applications Sparkle et Electron
- Amélioration de la protection de migration verrouillée (`chflags uchg`)
- Correction des problèmes d'affichage des badges dans le Finder

## v1.2.0

- Ajout de la stratégie de migration Stub Portal (remplaçant Deep Contents Wrapper)
- Ajout du support de migration des applications iOS (applications iOS version Mac)
- Amélioration des performances de migration par lots
- Correction du problème où certaines applications ne pouvaient pas se lancer après restauration

## v1.1.0

- Ajout du support multilingue (20+ langues)
- Ajout de la migration des répertoires de suites d'applications (par ex., Microsoft Office)
- Amélioration de la détection de stockage externe hors ligne
- Correction du problème de pénétration de lien symbolique avec la stratégie Deep Contents Wrapper

## v1.0.0

- Première version officielle
- Support de la migration d'applications vers le stockage externe (Deep Contents Wrapper / Whole App Symlink)
- Support de la restauration d'applications et de la gestion des liens
- Support de la surveillance de système de fichiers en temps réel FolderMonitor
