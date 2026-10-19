# ☄️ NASA Fireballs 3D — Historique des bolides sur un globe

Visualisation 3D interactive des **bolides** (boules de feu atmosphériques) détectés par les capteurs gouvernementaux américains et référencés par le programme **NASA CNEOS / JPL**. Chaque impact est rejoué sur un globe terrestre au fil du temps, de **1988 à 2026**.

> 🔗 **Démo en ligne** : `https://<ton-utilisateur>.github.io/nasa-fireballs-3d/`

---

## Aperçu

- Globe 3D de style « cartoon » (continents tracés à partir de données Natural Earth)
- Lecture chronologique des **888 événements** du jeu de données
- Chaque bolide apparaît sous la forme d'un **anneau rouge** qui s'étend et s'estompe, relié à une traînée verticale
- La **taille de l'anneau** dépend de l'énergie d'impact (en kilotonnes de TNT)
- Contrôles : lecture/pause, curseur temporel, vitesse de défilement, rotation/zoom à la souris

## Fonctionnalités

| Fonction | Détail |
|---|---|
| Timeline | Curseur pour naviguer librement dans la période |
| Vitesse | 1 mois/s, 6 mois/s, **1 an/s** (défaut), 3 ans/s |
| Caméra | Orbite, zoom (distance limitée entre 60 et 300 unités), amortissement |
| Données | Chargement du `fireball.json` local, avec repli sur l'API NASA/JPL |
| Carte | Chargement du GeoJSON local, avec repli sur un dessin vectoriel simplifié |
| Boucle | À la fin de la période, la lecture repart du début |

## Utilisation

### En ligne
Ouvre simplement l'URL GitHub Pages ci-dessus.

### En local
Le projet charge des fichiers via `fetch()` : l'ouverture directe de `index.html` (`file://`) est bloquée par le navigateur. Il faut un petit serveur HTTP :

```bash
git clone https://github.com/<ton-utilisateur>/nasa-fireballs-3d.git
cd nasa-fireballs-3d
python3 -m http.server 8000
```

Puis ouvre <http://localhost:8000>.

## Structure du projet

```
nasa-fireballs-3d/
├── index.html              # Application complète (HTML + CSS + JavaScript)
├── fireball.json           # Données NASA/JPL (888 bolides, 1988-04-15 → 2026-10-04)
├── ne_110m_land.geojson    # Contours des terres émergées (Natural Earth, 1:110m)
├── .nojekyll               # Désactive Jekyll sur GitHub Pages
└── README.md
```

## Fonctionnement technique

1. **Moteur 3D** : [Three.js](https://threejs.org/) v0.160.0 (chargé via `importmap` depuis unpkg) avec `OrbitControls`.
2. **Texture de la Terre** : un `<canvas>` 2048×1024 est dessiné à partir du GeoJSON (projection équirectangulaire) puis appliqué sur une sphère avec un `MeshToonMaterial`.
3. **Données** : chaque ligne de `fireball.json` est convertie en `{ time, lat, lon, energy }`. Les latitudes Sud et longitudes Ouest deviennent négatives.
4. **Placement** : latitude/longitude → coordonnées sphériques → `Vector3` à la surface du globe.
5. **Animation** : un temps virtuel avance à la vitesse choisie ; chaque événement franchi déclenche un impact animé pendant 1,5 s, puis ses objets sont libérés (`dispose`).

### Format des données (`fireball.json`)

Réponse de l'API [Fireball Data API](https://ssd-api.jpl.nasa.gov/doc/fireball.html) (version 1.2) avec les champs :

| Champ | Description | Unité |
|---|---|---|
| `date` | Date et heure du pic de luminosité | UTC |
| `energy` | Énergie rayonnée totale | 10¹⁰ J |
| `impact-e` | Énergie d'impact totale estimée | kt de TNT |
| `lat` / `lat-dir` | Latitude / hémisphère (N, S) | ° |
| `lon` / `lon-dir` | Longitude / hémisphère (E, W) | ° |
| `alt` | Altitude | km |
| `vel` | Vitesse (peut être `null`) | km/s |

Pour régénérer le fichier :

```bash
curl "https://ssd-api.jpl.nasa.gov/fireball.api?date-min=1988-01-01&req-loc=true" -o fireball.json
```

> Seuls `date`, `lat`, `lon` et `impact-e` (ou `energy` à défaut) sont utilisés par la visualisation. L'altitude et la vitesse ne sont pas exploitées pour l'instant.

## Limites connues

- Nécessite une connexion Internet pour charger Three.js depuis unpkg (pas de version embarquée).
- Seuls les bolides pour lesquels la position est connue sont affichés ; le jeu de données ne couvre que les événements détectés par les capteurs gouvernementaux, pas l'ensemble des bolides.
- La carte est à basse résolution (Natural Earth 1:110m).
- L'affichage de la date de la timeline est localisé en français (JJ/MM/AAAA).

## Pistes d'amélioration

- [ ] Infobulle au survol (date, énergie, altitude, vitesse)
- [ ] Filtre par énergie minimale
- [ ] Embarquer Three.js en local pour un usage hors ligne
- [ ] Mode sombre/clair pour la texture de la Terre
- [ ] Contrôle clavier (espace = pause, flèches = navigation)

## Sources et crédits

- **Données des bolides** : NASA/JPL — [Center for Near Earth Object Studies (CNEOS)](https://cneos.jpl.nasa.gov/fireballs/) et Fireball Data API.
- **Contours des continents** : [Natural Earth](https://www.naturalearthdata.com/) — domaine public.
- **Moteur 3D** : [Three.js](https://threejs.org/) — licence MIT.

## Licence

Aucune licence n'est définie pour le code à ce stade. Pense à en ajouter une (par exemple MIT) si tu souhaites autoriser la réutilisation.
