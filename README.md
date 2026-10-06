# Le marché du chargé de marketing digital

### 👉 **[Voir le site : aziziilias.github.io/metier](https://aziziilias.github.io/metier/)**

Le site est mis à jour chaque matin par une Action GitHub : elle interroge
l'API France Travail, enregistre les offres du jour et publie les chiffres.
Il suit un seul métier : **chargé / chargée de marketing digital**, code ROME
**M1718**. Les offres d'intérim (y compris CDI intérimaire) et les contrats
indépendants sont exclus des pages et des statistiques.

| | |
|---|---|
| [Accueil](https://aziziilias.github.io/metier/) | les filtres, les chiffres, la carte de France |
| [Ce que ça paie](https://aziziilias.github.io/metier/salaires.html) | fourchettes par niveau, contrat et territoire |
| [Ce qu'on vous demande](https://aziziilias.github.io/metier/exigences.html) | expérience, diplôme, outils, compétences |
| [Qui recrute](https://aziziilias.github.io/metier/recruteurs.html) | entreprises, secteurs, employeurs ouverts aux débutants |
| [Le marché bouge](https://aziziilias.github.io/metier/mouvement.html) | les extractions successives, la fraîcheur des annonces |

Dossier de travail pour la séance « Écouter le marché de votre métier »
(M2 MOD, IAE Clermont Auvergne). Dépôt de démonstration : il montre ce que
l'on attend d'un dossier `avenir`, étape par étape, et la chaîne complète
API → données → Action planifiée → page GitHub Pages.

## Le métier suivi

- **Intitulé** : chargé / chargée de marketing digital
- **Code ROME** : **M1718**
- Les intitulés des annonces peuvent varier ; la collecte s'appuie sur ce code
  ROME pour ne pas mélanger des métiers voisins.
- Les anciennes archives de collecte sont conservées, mais les pages et leur
  résumé ne retiennent que M1718.
- Les séries quotidiennes sont recalculées depuis les offres réellement actives
  chaque jour, en excluant ces types de contrat.

## Les questions que je pose à ce marché

1. Combien d'offres, et où : Clermont / Puy-de-Dôme, Auvergne-Rhône-Alpes,
   France, télétravail ?
2. Quels contrats et quels salaires affichés ?
3. Quels outils et compétences reviennent le plus — et lesquels la formation
   ne me donnera pas ?
4. Quelles entreprises publient le plus cet intitulé ?

## La chaîne

```
API France Travail  →  scripts/extraire.py  →  data/brut/<mois>/<ROME>.jsonl   chaque version d'annonce, une seule fois
                                            →  data/actives/<date>.csv         les offres actives du jour (rome, id)
                                            →  data/serie.csv                  par jour pour M1718 : total, nouvelles, modifiées
                       scripts/resumer.py   →  data/resume.json                ce que les pages affichent (+ data/geo/, cache des positions)
                       index.html + 4 pages →  https://aziziilias.github.io/metier/
                       .github/workflows/veille.yml : GitHub relance tout ça chaque jour
```

- `scripts/extraire.py` — une requête `codeROME` pour M1718 (token OAuth,
  pagination 150 / 1 150, total lu dans `Content-Range`). Le **brut est
  conservé intégralement** : une offre est écrite la première fois qu'on la
  voit, et de nouveau si son contenu change (empreinte SHA-1 du JSON, hors
  `dateActualisation`) — l'évolution d'une annonce est donc gardée, version
  par version. Relancer le même jour n'écrit rien deux fois.
- `scripts/resumer.py` — ne garde que les offres actives M1718, puis retravaille
  le brut : salaires
  (libellé texte → min/max annuels bruts), outils cités dans les descriptions
  (grille à adapter), position (lat/lon de l'API, sinon centre de la commune
  via geo.api.gouv.fr, sinon ville principale du département).
- Cinq pages HTML statiques, toutes servies telles quelles.
  Chacune charge `data/resume.json` et recalcule ses graphiques Chart.js dans le
  navigateur selon les filtres de contrat et de niveau ; net mensuel estimé = brut × 0,78 / 12.
  - `index.html` — les filtres de contrat et de niveau, les chiffres-clés, la carte Leaflet (survol =
    l'offre, clic = l'annonce sur France Travail), les départements, les
    contrats, et les liens vers les quatre autres pages.
  - `salaires.html` — ce que ça paie. `exigences.html` — ce qu'on vous demande.
    `recruteurs.html` — qui recrute. `mouvement.html` — le marché bouge, et les
    limites de ces chiffres (ancre `#limites`, liée depuis chaque pied de page).
- `assets/commun.js` et `assets/commun.css` — ce que les cinq pages partagent :
  chargement des données, panneau de filtres (mémorisé dans `localStorage`,
  replié ailleurs que sur l'accueil), barre de navigation, utilitaires et
  fabriques de graphiques. Une page ne contient que son HTML et son petit
  script `rendre(offres, D)`.

## Volume et limites GitHub

Jour 1 : 13 Mo de brut ; ensuite seulement le flux (nouvelles et modifiées),
de l'ordre de 2 à 3 Mo par jour, soit ~1 Go par an. GitHub gratuit : dépôt
1 Go recommandé, fichier ≤ 100 Mo, Pages 1 Go publié et 100 Go/mois de bande
passante, Actions illimitées sur un dépôt public. Quand le brut dépassera
quelques centaines de Mo, l'Action archivera chaque mois écoulé (compressé)
dans les Releases du dépôt ou sur Hugging Face Datasets, et le dépôt ne
gardera que les derniers mois.

## Faire tourner chez soi

```
py -3.12 -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
copy .env.example .env        (puis remplir avec ses identifiants francetravail.io)
.venv\Scripts\python.exe scripts\extraire.py --verifier
.venv\Scripts\python.exe scripts\extraire.py
.venv\Scripts\python.exe scripts\resumer.py
.venv\Scripts\python.exe -m http.server 8125      (puis http://localhost:8125)
```

## Faire tourner sans soi (GitHub)

1. Dépôt **public** (GitHub Pages gratuit ne fonctionne que sur un dépôt public).
2. Settings → Secrets and variables → Actions : `FT_CLIENT_ID` et `FT_CLIENT_SECRET`.
3. Settings → Pages → Source « Deploy from a branch », branche `main`, dossier `/ (root)`.
4. Actions → veille → Run workflow : le premier commit du bot arrive dans `data/`.

## Règles

- Les identifiants sont dans `.env` (local) ou dans les secrets du dépôt
  (GitHub) : jamais dans un fichier versionné.
- Un canal, une requête, une date : chaque chiffre du site les affiche.
- Pas de scraping de LinkedIn, APEC ou Indeed (interdit par leurs CGU).
