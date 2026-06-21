# Boost — Site vitrine SMM

Site vitrine en un seul fichier HTML pour vendre followers/likes/vues, avec réception automatique des commandes dans un Google Sheet.

## Fichiers

- `index.html` — le site complet (catalogue, formulaire, design)
- `AppsScript.gs` — le script à coller dans Google Sheets pour recevoir les commandes (⚠️ ne va jamais sur GitHub)
- `DEPLOIEMENT_GITHUB.md` — guide pas à pas pour mettre le site en ligne via GitHub Pages, sans Git
- `README.md` — ce fichier

## Mise en route (10 minutes)

### 1. Créer le récepteur de commandes (Google Sheet)

1. Va sur [sheets.google.com](https://sheets.google.com) et crée un Sheet vide.
2. Renomme l'onglet en bas en **Commandes** (exactement ce nom, sans accent ni espace).
3. Menu **Extensions > Apps Script**.
4. Efface le code par défaut, colle tout le contenu de `AppsScript.gs`.
5. En haut du script, remplace :
   ```
   const EMAIL_NOTIFICATION = 'ton-email@exemple.com';
   ```
   par ta vraie adresse email (tu recevras une notif à chaque commande). Laisse vide `''` si tu ne veux pas de notif email.
6. Clique **Déployer > Nouveau déploiement**.
   - Type de déploiement : **Application Web**
   - Exécuter en tant que : **Moi**
   - Qui a accès : **Tout le monde**
7. Autorise les permissions Google demandées (normal, c'est ton propre script).
8. Copie l'URL qui s'affiche, du type :
   `https://script.google.com/macros/s/AKfycb.../exec`

### 2. Connecter le site à ce Sheet

Dans `index.html`, cherche cette ligne (vers la fin du fichier, dans le `<script>`) :
```js
const SCRIPT_URL = 'COLLE_ICI_TON_URL_APPS_SCRIPT';
```
Remplace par l'URL copiée à l'étape précédente.

### 3. Mettre le site en ligne

Voir `DEPLOIEMENT_GITHUB.md` pour la marche à suivre détaillée (tout se fait via l'interface web de GitHub, aucune commande à taper).

### 4. Avant de lancer publiquement

- Remplace les numéros Wave / Orange Money dans `index.html` (section paiement) par les tiens.
- Remplace `contact@boost-demo.sn` en bas de page par ton vrai contact.
- Teste une commande toi-même de bout en bout pour vérifier qu'elle arrive bien dans le Sheet.
- Si tu modifies `AppsScript.gs` plus tard, il faut redéployer : **Déployer > Gérer les déploiements > icône crayon > Nouvelle version**.

## Automatiser la livraison (une fois que tu as un panel SMM)

`AppsScript.gs` contient maintenant une partie 2 : un menu **Boost** qui apparaît directement dans ton Google Sheet, avec un bouton pour lancer la livraison automatiquement après que tu as vérifié le paiement.

### Comment ça marche

1. Une commande arrive sur le site → ligne ajoutée dans le Sheet, statut **Nouvelle**.
2. Le client paie (Wave/OM) → tu vérifies ça toi-même, comme avant. Cette étape reste manuelle, aucun service ne te permet de l'automatiser simplement sans compte marchand.
3. Tu cliques sur la ligne de la commande dans le Sheet, puis **Menu Boost > Accepter la commande sélectionnée**.
4. Le script appelle automatiquement l'API de ton panel SMM, qui démarre la livraison. Le statut passe à **Envoyée au fournisseur**, avec l'ID de commande du fournisseur dans la dernière colonne.
5. Plus tard, tu peux sélectionner la même ligne et cliquer **Menu Boost > Vérifier le statut** pour voir si le fournisseur a terminé (passe alors à **Livrée** automatiquement).

### Configuration nécessaire une fois que tu as choisi un panel

Dans `AppsScript.gs`, en haut du fichier :

```js
const PANEL_API_URL = 'https://nomdupanel.com/api/v2';  // l'URL d'API donnée par ton panel
const PANEL_API_KEY  = 'ta-vraie-cle-api';
```

Et dans `SERVICE_ID_MAP`, remplace les IDs fictifs (1001, 1002...) par les vrais IDs de service que ton panel te donnera dans son catalogue de services. Chaque ligne du panel (ex: "Instagram Followers — High Quality") a un ID numérique propre à utiliser ici.

Ce format d'API (paramètres `key`, `action`, `service`, `link`, `quantity`) est celui utilisé par la grande majorité des panels SMM du marché (souvent décrit comme API "compatible JAP"). Vérifie dans la documentation API de ton panel — si elle utilise ce format, aucune autre modification de code n'est nécessaire à part coller l'URL et la clé.

Après chaque modification du script : **Déployer > Gérer les déploiements > icône crayon > Nouvelle version**, sinon les changements ne sont pas pris en compte par le site.

### Choisir un panel SMM fiable

Avant de coller une clé API, vérifie pour chaque panel candidat :
- **Rétention annoncée** (quel % de followers/likes reste après 30 jours)
- **Avis récents** sur des forums spécialisés (pas seulement les témoignages du site du panel)
- **Support réactif** — teste en posant une question avant d'acheter en gros
- **Petit test d'abord** : commande la plus petite quantité possible avant d'engager du volume

## Prix dynamiques (marge sur le coût du panel + ajustement concurrentiel)

Les prix affichés sur le site ne sont plus figés en dur. Voici comment ça fonctionne maintenant :

### 1. Marge automatique sur le prix de gros du panel

Le script récupère le catalogue de prix de ton panel (`action=services`, standard sur la plupart des panels) et calcule :

```
prix de vente = prix de gros du panel × (1 + marge)
```

La marge se règle dans `AppsScript.gs` :

```js
const MARGE_PAR_DEFAUT = 0.8; // 80% au-dessus du prix de gros

const MARGE_PAR_PLATEFORME = {
  'Instagram': 0.8,
  'TikTok': 0.8,
  'YouTube': 0.7,
  'Facebook': 0.8
};
```

Si le panel augmente ou baisse ses prix de gros, ton site suit automatiquement — il n'y a rien à faire de ton côté, à condition d'avoir activé le rafraîchissement automatique (voir plus bas).

Tu peux aussi voir ces tarifs en clair dans un nouvel onglet **Tarifs** de ton Sheet (prix de gros, marge appliquée, prix de vente, date de dernière mise à jour) — pratique pour vérifier d'un coup d'œil sans avoir à raisonner en tête.

### 2. Activer le rafraîchissement automatique

Une fois ton panel configuré (PANEL_API_URL, PANEL_API_KEY, SERVICE_ID_MAP) :

1. Ouvre Apps Script, sélectionne la fonction `activerRafraichissementAutomatique` dans le menu déroulant en haut, clique **Exécuter** une fois.
2. À partir de là, les tarifs se rafraîchissent automatiquement toutes les 6 heures, sans action de ta part.
3. Tu peux aussi forcer un rafraîchissement à tout moment via **Menu Boost > Rafraîchir les tarifs**.

### 3. Ajustement concurrentiel (reste manuel, et c'est normal)

Il n'existe pas d'API qui donne "le prix moyen du marché" en temps réel — les sites concurrents n'exposent pas leurs tarifs pour ça. Donc cette partie reste à toi :

- De temps en temps, regarde 2-3 sites SMM concurrents pour une quantité de référence (ex: 1000 followers Instagram).
- Si tu es trop cher ou trop bon marché par rapport à eux, ajuste `MARGE_PAR_DEFAUT` ou `MARGE_PAR_PLATEFORME` dans le script.
- Redéploie (Déployer > Gérer les déploiements > crayon > Nouvelle version) — les nouveaux prix s'appliquent immédiatement sur le site.

### Sécurité : le site ne tombe jamais en panne de prix

Si l'appel au panel échoue (panel pas encore configuré, hors ligne, clé API invalide), le site utilise automatiquement des prix de secours codés dans `index.html` (variable `pricingSecours`) — le client voit toujours un prix, jamais une erreur. Pense à les ajuster une fois que tu as une idée plus précise de tes vrais coûts, même s'ils ne servent qu'en cas de souci de connexion au panel.

## Limites restantes

- La vérification du paiement reste manuelle (c'est ton clic "Accepter" qui fait foi).
- Pas de compte client / historique au-delà du Sheet.
- Pas de système anti-spam sur le formulaire public. Suffisant pour démarrer et tester la demande.

## Protection du endpoint (jeton secret)

Le site et le script partagent maintenant un jeton secret : toute requête vers Apps Script (nouvelle commande, demande de tarifs) doit le présenter, sinon le script répond une erreur sans rien traiter.

### Configuration

Le jeton doit être **exactement identique** dans les deux fichiers :

- `AppsScript.gs` → variable `JETON_SECRET`
- `index.html` → variable `JETON_SECRET` (dans le `<script>`, juste après `SCRIPT_URL`)

Remplace la valeur par défaut (`change-moi-en-une-longue-chaine-aleatoire-unique`) par une chaîne longue et imprévisible. Après modification du script, redéploie comme d'habitude (Déployer > Gérer les déploiements > crayon > Nouvelle version).

### ⚠️ Ce que ce jeton protège, et ce qu'il ne protège PAS

**Protège contre** : les bots qui scannent le web au hasard, les abus accidentels qui feraient exploser tes quotas Google, le spam de ton Sheet par des requêtes random.

**Ne protège PAS contre** : une personne qui consulte le code source de ton site (clic droit > "Afficher le code source de la page"). Le jeton est écrit en clair dans le JavaScript de `index.html`, donc visible par quiconque regarde. C'est une limite de tout site purement front-end (sans serveur intermédiaire) : un secret utilisé côté navigateur n'est jamais vraiment secret.

### Le vrai risque à surveiller : ta clé de panel SMM

`PANEL_API_KEY` (dans `AppsScript.gs`) est différente : elle ne part **jamais** vers le navigateur du client, elle reste uniquement côté serveur Google (Apps Script). C'est elle qui doit rester strictement confidentielle, parce que quiconque l'obtient peut commander à tes frais sur ton compte panel. Donc :

- **Ne mets jamais `AppsScript.gs` sur un repo GitHub public.**
- Ne partage ce fichier qu'avec des personnes de confiance.
- Si tu donnes accès à ton Google Sheet à quelqu'un en tant qu'éditeur, cette personne peut ouvrir Apps Script et voir `PANEL_API_KEY` en clair.
- Si tu suspectes une fuite de cette clé, régénère-la immédiatement sur ton panel SMM.

## Pour aller plus loin

- Ajouter un paiement en ligne (PayDunya, Wave Business API) avec webhook pour que le statut "Payée" se déclenche tout seul, en complément du clic "Accepter".
- Ajouter une vérification automatique périodique du statut fournisseur (au lieu de cliquer manuellement "Vérifier le statut").