# IMPORTANT : comment ouvrir l'application
- **Ne double-cliquez pas sur index.html** : les navigateurs bloquent les modules et Firebase quand le fichier est ouvert depuis le disque (adresse en file://).
- Pour tester sur l'ordinateur : dans le dossier, lancez `npx serve` (ou l'extension « Live Server » de VS Code) puis ouvrez l'adresse affichée (http://localhost...).
- Pour la mettre en ligne : glissez le dossier sur Netlify (app.netlify.com/drop) ou Vercel / Firebase Hosting.
- Tant que `firebase-config.js` n'est pas rempli, l'application démarre en **mode démo** : sans connexion, données gardées sur l'appareil, toutes les fonctions Premium visibles. Une fois Firebase configuré, la connexion et la synchronisation s'activent.

# APEX MODE & STOCK – PWA avec Firebase

## 1. Créer le projet Firebase
1. console.firebase.google.com > Ajouter un projet.
2. Authentication > Commencer > Méthode de connexion > activer **Adresse e-mail/Mot de passe**.
3. Firestore Database > Créer une base de données (mode production, région europe-west ou la plus proche).
4. Firestore > onglet **Règles** : collez le contenu de `firestore.rules` puis Publier.
5. Paramètres du projet > Vos applications > ajoutez une application **Web** (</>) et copiez la configuration dans `firebase-config.js`.

## 2. Héberger
Envoyez le dossier sur un hébergement HTTPS (Firebase Hosting, Netlify, Vercel, GitHub Pages).
Authentication > Paramètres > **Domaines autorisés** : ajoutez le domaine de votre site.

## 3. Utiliser
- Créez le compte de la boutique (email + mot de passe) puis connectez-vous avec le même compte sur le téléphone et l'ordinateur : les données sont partagées en temps réel.
- Hors connexion, les ventes sont gardées sur l'appareil puis envoyées automatiquement au retour du réseau.
- Au premier lancement, si l'appareil contient des données de l'ancienne version locale, l'application propose de les importer.

## Plusieurs boutiques et factures
- Chaque propriétaire crée son compte puis sa boutique (nom, téléphone, adresse, mention de facture) : ses données sont totalement séparées de celles des autres boutiques.
- Chaque vente génère une facture numérotée (F-AAMM-001). On peut ajouter plusieurs tissus à la même facture, l'imprimer / l'enregistrer en PDF, ou l'envoyer au client par WhatsApp.

## Licences et abonnements
- À la première connexion, chaque boutique reçoit un **essai gratuit de 7 jours** (fonctions Standard). Ensuite l'accès est bloqué (les données restent, l'export CSV reste possible) jusqu'à activation.
- **Standard** 8 000 F/mois (96 000 F/an) et **Premium** 15 000 F/mois (180 000 F/an). Les fonctions Premium : factures PDF/impression, envoi WhatsApp, paiements et dettes clients, rapports et statistiques, fournisseurs.
- Configuration obligatoire dans `firebase-config.js` : `adminUid` (votre UID Firebase) et `contact` (votre numéro WhatsApp). Mettez le même UID dans `firestore.rules` à la place de `COLLER_ADMIN_UID`, puis republiez les règles.
- Pour activer un client : connectez-vous avec votre compte administrateur, onglet **Licences**, choisissez Standard/Premium, 1 mois ou 1 an. La durée s'ajoute à l'abonnement en cours. « Suspendre » coupe l'accès.
- Le blocage à l'expiration est appliqué par les règles Firestore avec l'heure du serveur (changer l'heure du téléphone ne le contourne pas).
- Les reçus et factures affichent le nom de la boutique, l'adresse, le téléphone, le RCCM et l'IFU (à renseigner dans l'onglet Boutique).

## Nouveautés
- **Import Excel/CSV** (Standard) : onglet Stock > Importer. Colonnes lues : Référence, Type, Couleur, Motif, Stock (m), Prix achat, Prix vente, Seuil alerte. Le bouton Modèle donne un fichier prêt à remplir ; l'export du stock peut être réimporté. Une référence déjà connue met à jour les infos et prix, sans toucher au stock. Les fichiers .xlsx demandent une connexion internet (lecteur chargé à la demande) ; le CSV marche hors ligne.
- **Historique détaillé du stock** (Premium) : Stock > Historique détaillé (ou « Historique » sur un tissu) : filtres tissu/entrée-sortie/dates, solde après chaque mouvement, export CSV.
- **Notifications avancées** (Premium) : onglet Boutique > Notifications. Alertes stock faible, dettes impayées (délai réglable), clients à relancer : panneau sur l'accueil + une notification par jour à l'ouverture de l'application (pas de push application fermée).
- **Personnalisation** (Premium) : onglet Boutique > Logo et couleur. La couleur colore l'en-tête et les factures, le logo apparaît sur les factures.

## Équipe : propriétaire et vendeurs
- **Propriétaire** : tout (abonnement, réglages, prix d'achat, marges, équipe).
- **Vendeur** : ventes, factures (y compris annuler), stock (entrées/sorties, tissus), clients, import/export, rapports. Il ne voit ni les prix d'achat ni les marges, et n'accède ni à l'abonnement ni aux réglages.
- **Inviter un vendeur** : onglet Boutique > Équipe > « Inviter un vendeur ». L'application donne un code à usage unique (envoi WhatsApp possible). Le vendeur ouvre l'application, clique sur « Créer le compte » et saisit son email, un mot de passe et le code. Limites : 2 vendeurs en Essai/Standard, 10 en Premium (modifiables dans `MAXSELL`).
- **Retirer l'accès** : bouton « Retirer » (réversible), effet immédiat.
- **Activité** : chaque vente, entrée/sortie de stock et suppression faites par un vendeur est inscrite dans l'onglet Activité du propriétaire (suppressions en rouge), avec un bandeau sur l'accueil et une notification si l'application est ouverte.
- **Prix d'achat protégés** : ils sont stockés à part (`shops/{id}/costs`), lisibles par le propriétaire seul grâce aux règles Firestore. Les anciennes données sont migrées automatiquement à la première sauvegarde du propriétaire.
- **Important : republiez `firestore.rules`** (nouvelles règles pour équipes, prix d'achat et activité), en remplaçant toujours `COLLER_ADMIN_UID`.
- **Essayer sans Firebase** : en mode démo, touchez le texte en haut à droite pour basculer entre la vue propriétaire et la vue vendeur.

## Sauvegardes automatiques (Premium)
- Une sauvegarde complète (tissus, mouvements, ventes, clients, paiements, fournisseurs, réglages, prix d'achat) est créée automatiquement **une fois par jour**, à la première ouverture de l'application par le propriétaire. Les 20 dernières sont conservées.
- Onglet Boutique > Sauvegardes : « Sauvegarder maintenant », « Télécharger (fichier) » (copie .json à garder sur votre téléphone ou ordinateur), « Restaurer depuis un fichier », et la liste des sauvegardes avec un bouton Restaurer.
- Avant toute restauration, l'état actuel est sauvegardé automatiquement (type « Avant restauration ») : une restauration peut donc être annulée.
- Les sauvegardes sont dans le même projet Firebase : elles protègent des erreurs (suppression, mauvaise manipulation), pas de la perte du compte Firebase. Téléchargez un fichier de temps en temps.
- Republiez `firestore.rules` (ajout des collections `backups` et `backupdata`).

## Si l'écran de connexion « reste bloqué »
- Regardez le texte rouge sous le mot de passe : depuis la version 9, il dit toujours ce qui ne va pas (firebase-config.js mal rempli, Firebase injoignable, Authentication non activé, domaine non autorisé, règles Firestore non publiées…).
- Le bas de l'écran de connexion affiche « Version 9 ». Si vous voyez une autre version (ou aucune), le navigateur utilise une ancienne copie : faites Ctrl+F5, ou F12 > Application > Service Workers > Unregister puis « Clear site data ».
- Décompressez le zip dans un **nouveau dossier** (pas par-dessus l'ancien) et lancez `npx serve` depuis le dossier qui contient `index.html`.
