# Corrections des outils Agnès — octobre 2026

Ce dossier contient les **versions corrigées** des outils, rangées par dépôt GitHub, **sous leur nom de fichier d'origine** : il suffit de remplacer chaque fichier dans le dépôt correspondant.

- `<dépôt>/<fichier>.html` : le fichier corrigé, prêt à remplacer l'original.
- `<dépôt>/diffs/<fichier>.diff` : les différences exactes avec l'original (format diff unifié), pour relire chaque changement.

Seules les **versions les plus récentes** de chaque outil ont été relues et corrigées (26 fichiers). Les anciennes versions n'ont pas été modifiées.

**Comment c'est vérifié** : relecture complète du code de chaque outil, chaque bug reproduit puis corrigé avec des tests automatisés (faux navigateur, réponses Agnès simulées), contrôle de syntaxe, et démarrage réel de chaque outil dans Chrome. **Aucun test n'a été fait avec de vraies clés API** : un essai réel avec vos clés reste à faire.

Aucune clé API n'a été ajoutée dans le code (les seules chaînes `sk-…` sont les exemples des champs de saisie).

---

## ⚠ À savoir d'abord : le modèle vidéo `agnes-video-v2.0` n'existe plus

Depuis le **10 octobre 2026**, `agnes-video-v2.0` n'est plus proposé par Agnès (il n'apparaît plus dans `/v1/models` et les demandes sont refusées). Le modèle vidéo qui fonctionne est **`agnes-video-2.5-flash`** (c'est celui qu'utilise MAIA depuis, sans problème).

Ces 15 fichiers appellent encore `agnes-video-v2.0` : leur partie vidéo ne marchera pas tant que le nom du modèle n'est pas changé. Je ne l'ai pas changé moi-même (c'est un choix de ton côté : durée, prix, réglages propres à chaque modèle) :

- `Code-a-reparer/cine-genesis-studio (3).html`, `Code-a-reparer/deepseek_html_20261010_28d9b3.html`
- `Agnes-ai-film-maker-Universal-one-page-HTML-4-all/deepseek_html_20261010_d59eb0.html`
- `Voanh-Agnes-film-maker/index.html`
- `redesigned-potato/` : `deepseek_html_20260928_a49b89 (3).html`, `deepseek_html_20261007_d9219c (2) (2).html`, `deepseek_html_20260925_fdc9e9 (6) (5).html`, `deepseek_html_20261008_68b448 (1).html`, `deepseek_html_20260930_a237a0 (3).html`, `deepseek_html_20261006_367208 (1) (2).html`, `deepseek_html_20261009_0a2943 (1).html`, `deepseek_html_20261006_618dc3 (2).html`, `deepseek_html_20261008_f10acd.html`, `deepseek_html_20261009_1e97ca (1).html`, `deepseek_html_20261008_faf9ac.html`

---

## Points importants, communs à plusieurs outils

1. **Images de référence : `extra_body.image`.** D'après la documentation officielle Agnès (agnes-image-2.1-flash), les images de référence et `response_format` vont **dans** `extra_body`. Style Forge v5 et Forge Engine v4 envoyaient `image` au premier niveau : la référence était probablement ignorée (visages non conservés). Corrigé. Cinéma Studio News et ViMax n'envoyaient pas du tout leurs références : corrigé.
2. **Stockage partagé entre outils.** Ouverts depuis le disque (`file://`), tous les fichiers partagent le même `localStorage` et les mêmes bases IndexedDB dans Chrome. Deux collisions corrigées :
   - Forge Engine v1 et v4 utilisaient tous deux `fe_packs` avec des formats différents → la v4 plantait au démarrage après ouverture de la v1. La v4 utilise maintenant `fe4_packs` (reprise automatique et normalisation des anciens packs).
   - Studio Portrait 9:16 (`0a2943`) et Studio 40 images (`9918ac`) partageaient la base `studio_db_v10` avec des stores différents et se réécrivaient mutuellement leurs sessions. Portrait ajoute les stores manquants ; 40 images utilise maintenant sa propre base `studio_db_v10_40img` (reprise unique de ses sessions).
   - À éviter pour les prochains outils : réutiliser un nom de clé ou de base d'un autre outil.
3. **Sécurité des clés.** Studio Portrait, Studio Blocs infinis et Scene Studio écrivaient les clés API **en clair dans la sauvegarde exportée** (JSON téléchargé). Les exports ne contiennent plus aucune clé ; l'import accepte les anciennes et les nouvelles sauvegardes.
4. **Ne pas réessayer la création de vidéo sur 502/504 ou coupure réseau** : la tâche a pu être créée côté serveur (doublon facturé). Les autres requêtes (texte, image, suivi) sont réessayées ; la création de vidéo affiche une erreur avec un bouton de relance.
5. **Erreurs « terminales » avalées dans le suivi vidéo.** Dans plusieurs outils, un statut `failed`/`error` levé dans un `try` était rattrapé par son propre `catch` et le suivi continuait 35 à 80 min. Corrigé partout.
6. **Ne jamais stocker un objet non clonable** (Promise, fonction) dans un objet sauvegardé en IndexedDB : `put` échoue (DataCloneError) et toute la sauvegarde est perdue (cas de CINÉ-GENESIS Voanh).

---

## Corrections par dépôt et par fichier

### Voanh-Agnes-film-maker — `index.html` (CINÉ-GENESIS)
*(Agnes-ai-film-maker-Universal-one-page-HTML-4-all — `deepseek_html_20261010_d59eb0.html` : même fichier, mêmes corrections.)*
- Le projet n'était plus sauvegardé après la 1re image (Promise de conversion stockée sur l'image → DataCloneError IndexedDB) : film perdu au rechargement.
- Le modèle texte de secours n'était jamais essayé (erreur 4xx marquée fatale).
- File vidéo bloquée après rechargement (drapeau `busy` sauvegardé) ; plans non recréés après « Refaire les scènes » ; erreurs répétées au démarrage si le stockage est lent ; pastille « file saturée » déformée (classe `cd` en conflit).
- Étapes 4 et 5 : l'outil s'y arrête pour laisser choisir (IA ou personnages perso ; nombre de scènes), au lieu de lancer l'IA automatiquement ; textes d'aide mis en cohérence.

### Code-a-reparer
- `cine-genesis-studio (3).html` : mêmes corrections que ci-dessus (modèle de secours, file bloquée, plans après nouvelles scènes, démarrage, pastille).
- `deepseek_html_20261010_28d9b3.html` : file vidéo qui ne démarrait jamais (`!Q.busy` au lieu de `Q.busy`) ; deux fonctions `act()` (le journal écrasait les actions) ; bouton « Court-métrage d'auteur » cassé par l'apostrophe ; projet jamais mémorisé (clé `last` jamais écrite) ; page Production jamais rafraîchie ; double appel du modèle de secours ; code de sauvegarde des clés incohérent.

### redesigned-potato
- **`NewsReel_Studio_v2 (3).html`** : erreurs définitives (400/401/404) réessayées comme des coupures réseau (jusqu'à ~16 min de blocage par image) ; « Intensité du mouvement » désormais utilisée dans les prompts ; résolution affichée « 720P (seule disponible) » ; interrupteur « Garder l'écran allumé » ; bandeau visible quand le JT utilise les news de démonstration.
- **`deepseek_html_20261008_68b448 (1).html`** (Cinéma Studio JT) : échec vidéo suivi ~40 min ; clé refusée → essai d'une autre clé ; échec de la réécriture des titres → repli sur les titres bruts ; suivi avec la clé créatrice ; cartes « Arrêté » après arrêt.
- **`deepseek_html_20261007_d9219c (2) (2).html`** (News × Réalisateurs) : la scène i utilise la i-ème image cochée ; références personnage/style enfin envoyées ; `crossOrigin` retiré du lecteur ; plus de « Génération… » sans fin après arrêt.
- **`deepseek_html_20261008_faf9ac.html`** (VISUAL.JOURNAL Chœur) : échec vidéo suivi ~80 min ; une erreur API passagère n'arrête plus tout le journal (solutions de secours) ; message correct après arrêt. *(Le dépôt contient une copie identique `deepseek_html_20261008_faf9ac (1) (2).html` : la remplacer aussi ou la supprimer.)*
- **`deepseek_html_20260930_a237a0 (3).html`** (Kata Studio) : un 502/504 pendant le suivi faisait perdre la vidéo ; plantage si l'IA ne renvoyait pas de personnages ; 500/502/504 réessayés (sauf création vidéo) avec bouton « Relancer » ; durée réelle utilisée au lieu de « 6,5 s » en dur.
- **`deepseek_html_20261009_1e97ca (1).html`** (Ciné Studio) : rien n'était sauvegardé (pas de session créée) ; « Ajuster » bloquait le plan ; drapeaux « en cours » sauvegardés ; attente infinie sans clé ; clés supprimées qui revenaient ; `scrollTo` récursif ; quota texte séparé du quota vidéo ; « Durée par plan » appliquée ; relance d'une image en erreur ; anciennes vidéos effacées après « Autre scénario » ; double génération du scénario.
- **`deepseek_html_20261008_f10acd.html`** (Studio d'Auteur) : échec vidéo suivi ~35 min ; arrêt affiché comme « Création complète » ; faux « 24 vidéos téléchargées » ; **sauvegarde et reprise du projet** (IndexedDB `studio_auteur_v12`) ; « Relancer » ne refait que les vidéos manquantes ; suivi avec la clé créatrice ; bascule de clé sur 401/403 ; bouton « Nouveau film ».
- **`deepseek_html_20260925_fdc9e9 (6) (5).html`** (Dessin Animé IA) : attente qui grossissait de 10 % à chaque clip, sans limite (mesure faussée) ; image `b64_json` sans préfixe `data:` ; lecture tolérante du JSON du scénario ; clips « Arrêté » après arrêt.
- **`deepseek_html_20261008_623f87.html`** (Plan-Séquence **v30**) : plantage après chargement de la photo (« Reprendre » inutilisable) ; cartes en échec non relançables (`er`/`err`) ; faux « Film prêt ! » ; quota journalier jamais remis à zéro ; une erreur d'image arrêtait tout le film ; détection de quota qui confondait un 429 de débit ; statuts de suivi élargis ; 25e image inutile en mode montage.
- **`deepseek_html_20261009_0a2943 (1).html`** (Studio Portrait 9:16) : stores manquants si la base a été créée par 40 images ; sauvegarde auto qui écrivait une session sous l'id d'une autre ; session supprimée qui revenait ; renommage qui effaçait les images ; attentes infinies ; progression du film invisible ; bouton **« ■ Arrêter »** ; quota texte séparé ; confirmation avant d'effacer les images ; sessions bloquées pendant une génération ; **export sans clés API**.
- **`deepseek_html_20261009_9918ac.html`** (Studio 40 images) : base propre `studio_db_v10_40img` (n'abîme plus les sessions de Studio Portrait), reprise unique de ses données.
- **`deepseek_html_20261009_f81bff.html`** (Studio Blocs infinis) : session supprimée qui revenait ; favoris qui affichaient le mauvais bloc ; attente 429 jamais appliquée ; images « loading » à vie ; changement de session pendant une génération ; nouvelle photo non utilisée comme référence ; chat en mode JSON forcé ; import annulé par une clé en double ; bouton **« ■ Arrêter »** ; relance d'une image en erreur ; **export sans clés API**.
- **`deepseek_html_20261009_c1cb28.html`** (Scene Studio) : génération jamais verrouillée (2e clic = galerie effacée + 40 images en parallèle) ; mêmes corrections de sessions/429/import que Blocs infinis ; bouton « Régénérer » actif sur les images en erreur ; bouton **« ■ Arrêter »** ; **export sans clés API**.
- **`deepseek_html_20261009_23906d (1).html`** (Copy Studio) : attente infinie sans clé ; cartes « Génération… » à vie après 429 ou arrêt ; messages qui restaient affichés ; bouton « 5 variations » grisé ; double génération ; les 429 ne désactivent plus les clés ; ré-enregistrer les clés les réactive.
- **`deepseek_html_20261006_618dc3 (2).html`** (Style Studio) : progression vidéo invisible ; réponse de suivi illisible qui faisait échouer la vidéo ; anti-veille coupé à tort ; bouton « ↻ Relancer l'analyse » ; 500/502/504 réessayés (sauf création vidéo, y compris coupure réseau).
- **`appforge.html`** : champ requis nombre/case à cocher toujours bloquant ; étape vidéo sans `each` qui plantait ; ancienne génération qui continuait en changeant d'app ; clé Agnès exigée seulement si nécessaire ; statut vidéo insensible à la casse.
- **`deepseek_html_20260928_a49b89 (3).html`** (MoneyPrinterTurbo) : erreur passagère qui faisait échouer une vidéo lancée ; anti-veille jamais actif ; scène « Préparation… » à vie après arrêt ; arrêt réactif pendant les attentes ; images > 12 Mo signalées ; échec de la sauvegarde de secours signalé.
- **`deepseek_html_20261006_367208 (1) (2).html`** (ViMax) : « Personnage (undefined) » dans les prompts ; vidéo perdue sur deux erreurs passagères ; son qui continuait après arrêt ; références des personnages utilisées pour les frames ; `crossOrigin` retiré ; téléchargement sans un onglet par scène.

### Agn-s-g.od-motor — `deepseek_html_20261014_afa37c.html` (Genesis·Decoder v10)
- Les angles créés par l'évolution et les angles « profonds » n'étaient jamais choisis (auto-organisation inopérante).
- Stop puis Démarrer rapprochés lançaient deux coordinateurs ; Stop pendant le chargement sautait un chapitre ; bouton Pause mal réinitialisé.
- Sauvegarde et reprise de la position de lecture et des scores des angles ; erreurs passagères réessayées sans sauter de chapitre ; chapitres tronqués non mis en cache ; bootstrap tolérant (`v` nombre ou chaîne) et limité à 3 essais.

### Agn-s-bulk-photo-studio — `style-forge-v5-studio-réalisateur-&-maison-d-art (1).html` (Style Forge v5)
- Ids d'images qui repartaient de `it1` au rechargement (images écrasées en IndexedDB, récursion possible) ; enfant affiché en double ; images rangées dans la série consultée au lieu de la série lancée ; 2e série en parallèle possible ; barre d'onglets invisible ; référence dans `extra_body.image`.
- Texte imprimé sur vêtement : plus de texte par défaut (l'ancien défaut est remplacé par vide s'il n'avait pas été modifié) ; plans importés conservés au rechargement ; bascule de clé sur 401/403 ; état « déplié » enregistré ; nombre de workers appliqué à chaque série.

### Agnes-ai-modular-bulk-picture-factory — `forge-engine-v4-…-à-options.html` (Forge Engine v4)
- Plantage au démarrage après ouverture de la v1 (clé `fe_packs` partagée) → clé `fe4_packs` + reprise des anciens packs.
- Éditer un pack avec un JSON refusé effaçait l'application ; relance de l'assistant sans la demande initiale ; production bloquée « en cours » ; image chargée affichée vide ; référence dans `extra_body.image` ; format du générateur respecté ; pack démo non réinstallé après désinstallation ; compteur non remis à zéro par une action ; bascule de clé sur 401/403.

### Agnes-ai-news-maker-bulk-automatic — `deepseek_html_20261012_accee5 (4).html` (NewsReel · Actualités en images)
- Ids `it1, it2…` réutilisés à chaque génération : **les images des sessions précédentes étaient écrasées** en IndexedDB, et des images arrivaient dans la mauvaise carte. Ids désormais uniques par session.
- Sessions restaurées : galerie vide / mauvais titres, texte d'analyse perdu, compteur faux ; après « Stop », plus aucune génération sans recharger ; délai d'attente des clés < pause 429 ; 401 sans essai d'une autre clé ; diaporama lu deux fois (⏭/⏮) et Stop qui ne coupait pas la voix ; galerie qui s'empilait ; écran « Préparation » bloqué si IndexedDB échoue ; ancien Blob téléchargé après ↻ ; proxy CORS sans délai max.
- ⚠ La version `deepseek_html_20261012_accee5 (2).html` (n2) partage la même base et garde le bug des ids : elle peut encore écraser des images. À retirer du dépôt ou à corriger de la même façon.

---

## Restent à décider (non modifiés)
- NewsReel Studio v2 : chaque vidéo reçoit la même image en `first_frame` et `last_frame` (la vidéo revient à son image de départ).
- Les URL d'images/vidéos Agnès sont stockées telles quelles : si le serveur les fait expirer, les anciennes sessions ne s'afficheront plus.
- Kata Studio : les plans jamais lancés après un arrêt restent « En file » sans bouton de relance.
- Le « mode combinaison folle » de Cinéma Studio News n'est utilisé ni par le scénario ni par les vidéos.
