# Analyse de la carte « Atlas des compétences — Sud & Corse »

Site analysé : https://banatic-sud-corse.marcdeturcan.chatgpt.site/

L’interface et ses ressources publiques ont été inspectées le 19 octobre 2026. La carte est réalisée en SVG avec du JavaScript natif. Les ressources observées ne chargent ni Leaflet, ni Mapbox, ni OpenLayers, ni React. L’affichage et les filtres fonctionnent côté navigateur à partir de deux fichiers JSON statiques.

## Ressources et responsabilités

| Ressource publique | Rôle |
|---|---|
| [Page HTML](https://banatic-sud-corse.marcdeturcan.chatgpt.site/) | Structure de l’interface et élément SVG de la carte |
| [app.js](https://banatic-sud-corse.marcdeturcan.chatgpt.site/app.js) | Chargement, filtres, rendu SVG, fiches, zoom, déplacement, favoris et export des groupements |
| [style.css](https://banatic-sud-corse.marcdeturcan.chatgpt.site/style.css) | Mise en page, couleurs, contours, survol, états sélectionnés et adaptation mobile |
| [data.json](https://banatic-sud-corse.marcdeturcan.chatgpt.site/data.json) | Données BANATIC, rattachements et tracés de la carte : 2 075 030 octets |
| [ugap.js](https://banatic-sud-corse.marcdeturcan.chatgpt.site/ugap.js) | Correspondances compétences/offres, filtres UGAP, catalogue et exports associés |
| [ugap.json](https://banatic-sud-corse.marcdeturcan.chatgpt.site/ugap.json) | Référentiel éditorial des offres et des correspondances : 57 792 octets |

L’initialisation télécharge `data.json`, puis `ugap.json`. Il n’y a pas d’appel à une API métier à chaque filtre. Les requêtes de cette interface ne nécessitent pas de serveur applicatif pour reproduire son fonctionnement ; un serveur de fichiers statiques suffit. Cela ne permet pas de conclure sur toute l’infrastructure interne de l’hébergeur.

## Construction de la carte

Le HTML contient :

```html
<svg id="map" viewBox="0 0 1000 730">
  <g id="mapshapes"></g>
  <g id="maplabels"></g>
</svg>
```

Chaque commune possède déjà un tracé `path`, contenant des commandes SVG `M`, `L` et `Z`. Les EPCI disposent également de leur tracé. Le navigateur insère ces chaînes dans l’attribut `d` d’éléments `<path>`. Il ne calcule pas une projection géographique à l’exécution.

Les champs `xy` contiennent des coordonnées dans le même espace SVG ; ce ne sont pas des couples longitude/latitude. Ils servent notamment à positionner les libellés et les points représentant les sièges des syndicats.

En vue « Intercommunalités », 71 éléments `<path>` représentent les EPCI. En vue « Communes », 1 306 éléments `<path>` représentent les communes. Les groupements sans fiscalité propre sont représentés par des cercles de rayon SVG 4, lorsque les filtres les retiennent : 277 points avec le filtre « Tous les groupements ».

Les libellés de Marseille, Nice, Toulon, Avignon, Gap, Ajaccio et Bastia sont placés à partir des coordonnées de leurs communes.

Le fond bleu-gris, les points décoratifs, la mention « Mer Méditerranée », la légende et les boutons sont des éléments CSS/HTML. Aucun fond routier ou satellite n’est chargé. La grille utilise un `radial-gradient`, avec un espacement de 12 px.

## Données réellement présentes

| Contenu | Nombre |
|---|---:|
| Groupements | 348 |
| EPCI à fiscalité propre | 71 |
| Autres groupements | 277 |
| Communes | 1 306 |
| Rubriques de compétences | 125 |
| Offres et solutions UGAP | 32 |
| Liens compétences/offres | 160 |
| Compétences avec correspondance UGAP | 87 |
| Compétences sans correspondance documentée | 38 |

L’état initial n’affiche que les EPCI : 71 résultats, 1 306 communes distinctes et 98 compétences distinctes parmi ces résultats. Les 125 rubriques correspondent au référentiel complet.

Le fichier principal contient quatre clés : `date`, `groups`, `communes`, `competences`. La date embarquée est `2026-10-16`.

Un groupement porte notamment : `id` (SIREN), `name`, `type`, `epci` (booléen), `dep`, `pop`, `memberCount`, `members`, `communes` (codes INSEE), `comps` (codes de compétences), `values` (modalités BANATIC), `seat`, `path`, `xy`.

Une commune porte : `id` (code INSEE), `name`, `epci` (SIREN de rattachement), `dep`, `pop`, `path`, `xy`.

Les contrôles de cohérence effectués ne trouvent aucun identifiant dupliqué, aucune commune référencée mais absente, aucune compétence référencée sans définition et aucune commune sans EPCI présent dans cet extrait.

La fenêtre « Sources & méthode » indique que les données viennent de l’export BANATIC/DGCL du 16 octobre 2026, des contours et rattachements de l’API Découpage administratif, et d’une table SIREN/INSEE de mars 2025 pour certains rattachements syndicaux. Elle décrit les périmètres EPCI comme l’union des communes. Le script de préparation des données n’est pas exposé par les ressources inspectées : l’outil de fusion, la projection et les paramètres de simplification exacts ne sont pas identifiables dans le code client. La provenance annoncée par le site n’a pas été recoupée avec les exports nationaux.

## Couleurs et interactions

Le rendu des zones correspondant aux filtres utilise quatre verts, selon le nombre de compétences du groupement :

| Nombre de compétences | Couleur |
|---|---|
| 0–19 | `#b4d6c7` |
| 20–39 | `#87bfa9` |
| 40–59 | `#5da68f` |
| 60 et plus | `#328a76` |

En vue communale, la nuance dépend du nombre de compétences de l’EPCI de rattachement. Lorsqu’au moins une compétence est cochée, les zones correspondantes prennent un vert uniforme `#238b77`. Les autres zones sont grisées en `#d2dede`, et restent dessinées. Les points syndicaux sont bleu foncé `#203d59`.

Survol et sélection utilisent une couleur ocre `#efba69` et un contour foncé. Les bordures ordinaires sont blanches, d’épaisseur 0,7 ; `vector-effect: non-scaling-stroke` empêche leur épaisseur de grossir avec le zoom.

Le survol affiche le nom dans une infobulle HTML. Le clic ouvre une fiche latérale : compétences, offres UGAP et membres pour un groupement ; rattachements pour une commune. Le clic sur une ligne du tableau ouvre la même fiche.

Le zoom modifie le `viewBox` du SVG : le bouton « + » multiplie sa largeur par 0,75, et le bouton « − » par 1,33. La molette utilise d’autres coefficients, 0,88 et 1,13. La largeur est bornée entre 70 et 1 700 unités SVG. Le déplacement ajuste l’origine du `viewBox` suivant le mouvement du pointeur. Le recentrage calcule une emprise à partir des coordonnées des communes des résultats ; ce n’est pas une emprise géométrique exacte des contours.

Les filtres combinent recherche textuelle, territoire, département, type de structure, population, compétences en logique OU/ET, domaines et correspondances UGAP. Le filtrage géographique des groupements repose sur leur département de siège. Ce détail compte pour les syndicats intervenant sur plusieurs départements. La recherche normalise casse et accents et inclut les noms, SIREN, sièges, membres et communes rattachées.

Les favoris sont enregistrés dans le `localStorage`, sous la clé `atlas-favorites`. Les CSV sont générés dans le navigateur avec `Blob` et une URL temporaire. Les correspondances UGAP sont des règles éditoriales explicites, avec niveau direct/conditionnel et justification ; elles ne sont pas calculées par une IA lors de la navigation.

## Méthode conseillée pour la reproduire

Pour reproduire cette version précise, la solution la plus courte consiste à conserver le principe SVG et les géométries précalculées, puis à reconstruire la structure HTML, les styles, les filtres et les fiches autour du même schéma de données. Le SVG convient à ce volume et à cette carte thématique régionale. Aucun abonnement cartographique ni clé d’API de fond de carte n’est nécessaire pour ce mécanisme.

Pour disposer de données actualisables et étendre ensuite la couverture :

1. Importer un export BANATIC, dédupliquer les groupements par SIREN et conserver les modalités des compétences.
2. Importer les contours communaux et les rattachements EPCI de l’API géographique, puis relier les codes INSEE et les SIREN.
3. Préparer hors navigateur les géométries : validation, simplification préservant autant que possible les limites communes, construction des périmètres EPCI et projection vers un espace SVG cohérent.
4. Produire `path` et `xy`, et exporter le fichier JSON sans modifier les identifiants textuels — notamment les zéros initiaux des codes INSEE.
5. Construire un état de filtres partagé par carte, statistiques et tableau ; rendre la vue EPCI ou communale et les fiches.
6. Ajouter les correspondances UGAP documentées, les favoris et les CSV.
7. Tester zoom, déplacement, sélection, recherche avec accents, filtres OU/ET, absence de résultats et mobile.

Une chaîne Python avec pandas pour les tableaux, Shapely/GeoPandas pour les géométries et une projection documentée convient à la préparation ; ce sont des recommandations de reconstruction, pas des bibliothèques identifiées sur le site. D3 peut aussi servir à projeter des GeoJSON et produire des tracés SVG. Leaflet ou MapLibre deviennent pertinents si le projet doit ajouter rues, adresses, nombreux marqueurs ou fonds de carte, mais changeraient l’architecture observée.

Le point à reconstruire avec soin est la préparation et la jointure des données. Réutiliser les `path` existants suffit pour cet instantané ; mettre à jour les territoires exige de documenter cette chaîne de préparation. Il faut conserver la distinction entre une compétence déclarée par un groupement et son exercice effectif sur chaque commune. Un siège syndical reste une localisation indicative.

## Validation et limites

Les ressources publiques ont été téléchargées en HTTPS avec vérification TLS. Chromium n’acceptant pas le certificat du proxy de l’environnement, les essais interactifs ont été effectués sur une copie locale de ces mêmes fichiers, sans désactiver la vérification TLS. Les polices Google externes ont été bloquées pour ce rendu local : les captures utilisent les polices de remplacement et ne constituent pas une comparaison typographique exacte avec le site en ligne.

Contrôles exécutés : état initial à 71 groupements/1 306 communes/98 compétences ; ouverture d’une fiche par clic sur une zone ; bascule à 1 306 tracés communaux ; recherche « Ajaccio » donnant la CA du Pays Ajaccien ; zoom faisant passer le `viewBox` de `0 0 1000 730` à `125 91.25 750 547.5` ; affichage des 348 groupements et 277 points syndicaux ; ouverture des sources ; rendu à 1 440 px et 390 px. Aucune erreur JavaScript applicative n’a été observée pendant ce parcours. Les téléchargements CSV et tous les croisements possibles de filtres n’ont pas fait l’objet d’un essai exhaustif.

Captures : [ordinateur](carte-desktop.png) et [mobile](carte-mobile.png).

Aucun code applicatif n’a été ajouté ou modifié dans `/workspace/lab`. Ce rapport prépare la reconstruction ; il ne constitue pas encore son implémentation.
