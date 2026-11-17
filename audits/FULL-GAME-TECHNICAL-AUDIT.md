# Audit technique du jeu Ember Wing — constats, correctifs proposés et plan de validation

- **Dépôt inspecté :** `carinajacquot-sudo/best-game-01`, branche `main` telle qu’observée le 17 novembre 2026.
- **Dépôt de travail :** `carinajacquot-sudo/ember-wing-missions`.
- **Nature :** revue statique des extraits de sources récupérés via GitHub ; aucun navigateur, shell, playthrough ou test n’a été exécuté pour ce rapport.
- **But :** fournir à l’agent d’implémentation une revue technique exploitable, avec exemples de correctifs à re-dériver dans sa session enregistrée.
- **Important :** les propositions de code ci-dessous ne sont ni commits du jeu, ni patches testés. Les directives du projet imposent que l’agent d’implémentation écrive le code du jeu dans des sessions enregistrées.

## Résumé exécutif

La priorité est la fiabilité du jeu, pas l’ajout de décorations. La revue identifie des risques transversaux : la transition de victoire du boss dépend d’un objet supprimé au moment où les PV tombent à zéro ; les mouvements combinent incréments par frame et timers en secondes ; le magnet divise par la distance sans traiter une distance nulle ; le HUD annonce une pause dont le gestionnaire ne fait rien ; et le manifeste des captures ne reflète pas les scènes déjà prises en charge par le harness.

Les défauts directement visibles dans les extraits sont marqués **constat statique**. Les incidences en jeu sont à reproduire avant de les déclarer observées. Une revue de snippets GitHub n’est pas équivalente à une lecture complète du clone, une exécution ou une validation du jeu.

## P0 — risques fonctionnels à reproduire et corriger d’abord

### 1. Mort du boss et transition de victoire

**Constat statique.** Dans `game/src/main.js`, la branche de collision du boss traite son décès puis exécute `this.enemies.boss = null`. Dans `game/src/levels.js`, la progression du niveau 3 vérifie que `enemyManager.boss` existe et que `enemyManager.boss.hp <= 0`. Après la suppression, cette condition ne peut plus être satisfaite par cette voie. Il existe aussi un risque que le code de mise à jour retire ou fasse évoluer le boss avant que le gestionnaire du niveau n’observe l’état fatal.

**Patch proposé (concept : mémoriser le résultat, pas garder un boss mort actif).** Ajouter à `LevelManager` un drapeau de défaite explicite, réinitialisé lors d’un nouveau jeu, et le positionner à la collision fatale avant de retirer le boss :

```js
// LevelManager.constructor()
this.bossDefeated = false;

// LevelManager.reset()
this.bossDefeated = false;
```

```js
// main.js : branche du coup fatal contre le boss
if (this.enemies.boss.hp <= 0) {
  this.levels.bossDefeated = true;
  // jouer les effets/score du décès une seule fois
  this.enemies.boss = null;
}
```

```js
// levels.js : condition finale
} else if (this.currentLevel === 3 && this.bossDefeated) {
  this.lanternsLit = 9;
  this.levelCompleted = true;
}
```

Ce fragment est illustratif : vérifier les états réels, l’ordre des appels, le calcul du score, les transitions et le risque de double-déclenchement. Variante possible : événement de mort traité une fois. Éviter de garder un boss à 0 PV s’il continue à être dessiné, à tirer ou à entrer en collision.

**Test de caractérisation demandé avant correction :** mise en place niveau 3 → boss actif → tir fatal → niveau terminé une seule fois → état victory → aucun tir/collision du boss après la mort. Ajouter aussi un test reset/restart pour prouver que le drapeau ne fuit pas vers une nouvelle partie.

### 2. Unités de temps incohérentes (FPS dépendants)

**Constat statique.** Le loop de `main.js` calcule un `dt`, et certains timers l’utilisent. Plusieurs déplacements de modules sont pourtant des incréments par appel : notamment joueur, projectiles, ennemis, boss, étoiles, particules et powerups. Le résultat dépend du nombre d’images réellement rendues. Les fréquences aléatoires de tir observées dans les extraits d’`enemies.js` reposent également sur un seuil par mise à jour.

**Stratégie sûre.** D’abord inventorier chaque variable : pixels/frame, pixels/seconde, secondes, ou probabilité/frame. Choisir des unités par seconde ; pour préserver approximativement le tuning actuel à 60 Hz, convertir l’incrément historique `v` par `v * dt * 60` quand `v` était en pixels/frame. Ne pas appliquer ce facteur aveuglément aux timers déjà en secondes.

Exemple joueur :

```js
const frameScale = dt * 60;
this.x += dx * this.speed * frameScale;
this.y += dy * this.speed * frameScale;
```

Pour une probabilité historique `p` par image à 60 Hz, taux équivalent indépendant du framerate :

```js
const probability = 1 - Math.pow(1 - p, dt * 60);
if (rng.next() < probability) {
  // tirer
}
```

Employer un RNG contrôlé plutôt que `Math.random()` dans les systèmes devant être déterministes. Effectuer la même conversion sur tous les systèmes concernés dans un changement cohérent : corriger uniquement le joueur rendrait les entités plus rapides ou plus lentes les unes par rapport aux autres.

**Validation :** mêmes déplacements approximatifs sur 1 seconde simulée à 30, 60 et 120 Hz ; mêmes temps de cooldown ; fréquence de tir similaire sur une durée fixe ; mesurer les changements de difficulté et retuner si nécessaire.

### 3. Risque `NaN` dans l’attraction Magnet

**Constat statique.** `game/src/powerups.js` calcule `dx / dist` et `dy / dist` quand le Magnet est actif et la distance est sous 300. Si `dist` est exactement zéro, la normalisation n’est pas définie. Le déplacement fixe de 6 peut également dépasser la distance restante et faire osciller le pickup autour du joueur.

**Patch proposé :** tester la distance au carré avant la racine et plafonner le déplacement à la distance restante :

```js
const dx = player.x - p.x;
const dy = player.y - p.y;
const distSq = dx * dx + dy * dy;

if (distSq > 1 && distSq < 300 * 300) {
  const dist = Math.sqrt(distSq);
  const step = Math.min(6, dist);
  p.x += (dx / dist) * step;
  p.y += (dy / dist) * step;
}
```

À convertir ensuite en vitesse par seconde en même temps que le reste du jeu. Tester coïncidence exacte, distance très faible, seuil 300, capture du pickup et Magnet inactif.

### 4. Pause annoncée mais absente

**Constat statique.** Dans `main.js`, `KeyP` entre dans un bloc de commentaire `// Toggle pause or handle pause`; il n’existe pas de transition fonctionnelle dans ce code. Pourtant le HUD et l’aide annoncent la pause.

**Proposition :** ajouter un état de jeu `paused` ou un booléen cohérent avec la machine à états ; lors de pause, ne pas appeler la simulation du monde, mais continuer à dessiner un overlay pause et à accepter P pour reprendre. S’assurer que la pause n’accumule pas un énorme `dt` au retour (réinitialiser `lastTime` ou plafonner la reprise), que mute et focus restent cohérents, et que l’entrée ne déclenche pas accidentellement un tir.

**Tests :** déplacement et projectiles figés pendant la pause ; P reprend une seule fois ; reprise après onglet inactif sans téléportation ; pause impossible ou clairement définie sur titre, victoire et game over.

## P1 — anomalies visuelles et déterminisme de capture

### 5. Couleur de grille Canvas invalide

**Constat statique.** Dans `game/src/background.js`, la valeur assignée à `ctx.strokeStyle` contient un point-virgule dans la chaîne CSS :

```js
ctx.strokeStyle = 'rgba(0, 240, 255, 0.25);';
```

**Correction :**

```js
ctx.strokeStyle = 'rgba(0, 240, 255, 0.25)';
```

Le setter Canvas ignore les chaînes de couleur non valides ; vérifier le rendu réel après correction, notamment qu’aucun état de contexte antérieur ne masque le défaut.

### 6. Fenêtres de skyline sélectionnées par modulo sur flottants

**Constat statique.** La condition `(wx + wy) % 4 === 0` compare un modulo de coordonnées potentiellement décimales à une égalité exacte. Elle ne garantit pas une distribution régulière des fenêtres éclairées.

**Correction proposée :** s’appuyer sur des indices entiers de ligne/colonne et un index de bâtiment, avec un motif stable :

```js
for (let row = 0, wy = y + 15; wy < 540; row++, wy += 30) {
  for (let col = 0, wx = b.x + 10; wx < b.x + b.w - 10; col++, wx += 20) {
    if ((col + row * 3 + buildingIndex) % 4 === 0) {
      ctx.fillRect(wx, wy, 8, 12);
    }
  }
}
```

Adapter la boucle extérieure pour obtenir `buildingIndex` sans allocation par frame. Vérifier la lisibilité, le contraste, le coût draw et que les fenêtres ne dépassent pas les bâtiments.

### 7. RNG de harness incomplet

**Constat statique.** Le harness de `main.js` configure `this.rng` depuis `?seed=`, mais les extraits montrent aussi des `Math.random()` dans la génération du décor, des particules et des ennemis. Le seed du jeu ne suffit donc pas à garantir un rendu répété identique. Les positions de skyline sont créées dans le constructeur de `Background`, avant ou indépendamment de ce RNG.

**Proposition :** injecter un générateur contrôlé dans les systèmes génératifs (ou une source RNG dédiée au harness) et éviter les nombres aléatoires non seedés dans les scènes de capture. Ne pas altérer la variation du jeu normal plus que nécessaire. Attention aux consommations de RNG : ajouter un tirage dans une scène peut décaler les vagues suivantes. Le staging de capture devrait avoir un état explicite, indépendant du déroulement d’une partie réelle.

**Tests :** deux captures de même route/seed pixel-identiques ; deux seeds distincts donnent une variation voulue ; mode jeu normal ne plante pas ; `window.__shotReady` demeure signalé.

### 8. `shots.json` incomplet par rapport au harness et aux fichiers

**Constat statique.** `shots.json` énumère `start`, `play`, `end`. `main.js` contient également `boss` ; le dépôt contient quatre JPEG dont `3-boss.jpg`. Le manifeste et les artefacts sont donc incohérents.

**Proposition :** d’abord confirmer le contrat de l’outil de capture et l’ordre du contact sheet ; ensuite aligner le manifeste avec les quatre routes vraiment exécutables. N’ajouter `cover` qu’après implémentation et test d’un staging spécifique. Le plan M8, dans `deliverables/M8-screenshot-cover-strategy.md`, définit le storyboard cible ; il ne prouve pas que la scène existe déjà.

### 9. Limite réelle des screenshots

`main.ts` configure `buildDossier` avec `screenshots.max_bytes: 262144` (256 KiB) par image. La mission M8 évoque moins de 350 KB, mais le seuil utilisé par le code de soumission est plus strict. Le validateur réel et son unité doivent faire foi. Mesurer chaque image en octets et garder une marge sous 262144 ; ne pas se contenter d’une dimension correcte.

### 10. Background : gradients alloués à chaque rendu

**Constat statique.** `background.js` crée des gradients linéaire et radial à l’intérieur du dessin. Le brief M1 exige l’absence de création de gradients à chaque frame, avec cache à la construction. Cela peut coûter du temps CPU.

**Proposition :** mesurer d’abord. Les gradients dépendent du contexte Canvas ; mettre en cache après création dans un contexte valide, ou recréer lors d’un changement de contexte/canvas. Ne pas conserver un objet CanvasGradient lié à un contexte qui ne serait plus utilisé. Mesurer le rendu au budget d’entités/particules du brief avant/après, vérifier les captures et éviter d’introduire un cache fragile.

## P2 — expérience, audio et robustesse

### 11. Audio : état mute séparé de l’intention musicale

**Constat statique.** `toggleMute()` redémarre `startMusic()` lorsque le son revient et qu’aucun intervalle musical n’est actif. `stopMusic()` ne conserve que l’absence d’intervalle, sans état « musique voulue/arrêtée ». Un unmute après un écran victoire/game over peut donc relancer la boucle selon la séquence d’actions.

**Proposition :** distinguer `muted` de `musicRequested`/état de jeu. `startMusic()` établit explicitement l’intention ; `stopMusic()` l’efface ; `toggleMute()` ne fait que rendre audible/silencieux le son demandé. Garder une seule horloge musicale, et ne pas redémarrer la musique tant qu’un état de jeu ne la demande pas.

**Tests :** titre muet/unmute ; entrée en jeu ; mute/unmute pendant le jeu ; victoire/arrêt puis mute/unmute sans redémarrage involontaire ; aucune accumulation d’intervalles ou de nœuds. Les politiques d’autoplay du navigateur exigent également de tester après un geste utilisateur.

### 12. Input et changement de focus

**Constat statique.** `InputHandler` écoute les événements au niveau `window`. Le bouton de tir souris s’active sur n’importe quel `mousedown`, pas uniquement sur le canvas ; pas de remise à zéro visible des touches au `blur`. Une touche maintenue lors d’un changement d’onglet peut rester logiquement active jusqu’à un keyup.

**Proposition :** écouter `blur`/`visibilitychange` pour vider les touches et l’état souris ; limiter la souris au canvas ou au conteneur du jeu et convertir les coordonnées depuis le rectangle du canvas ; empêcher le menu contextuel si le clic droit est éventuellement pris en charge. Le code mentionne touch dans son commentaire, mais aucun gestionnaire tactile n’apparaît dans l’extrait : soit ajouter une prise en charge réelle et la tester, soit corriger le commentaire et ne pas prétendre que le tactile est supporté.

### 13. Responsive : étirement potentiel du canvas

**Constat statique.** Le canvas conserve son bitmap 1280×720, tandis que CSS donne au conteneur une largeur et une hauteur viewport avec seulement des maxima. Sur des ratios d’écran différents, width et height CSS peuvent étirer le jeu.

**Proposition :** préserver un ratio 16:9 en ajustant le conteneur à la dimension disponible et en acceptant des bandes libres ; si le DPR est pris en compte, séparer dimensions CSS et résolution de rendu sans casser la géométrie logique 1280×720. Tester 16:9, écran portrait, petite fenêtre et capture de référence.

### 14. Particules et performance

**Constat statique.** `ParticleManager` ajoute des objets et retire via `splice` ; il n’a pas de cap visible dans l’extrait. Les explosions peuvent accumuler des particules. Les draw calls avec `shadowBlur` ont un coût variable.

**Proposition mesurée :** instrumenter nombre maximal de particules et temps de frame ; fixer un cap compatible M2/M1 ; si nécessaire recycler les particules mortes avec un pool simple ; réutiliser les objets sans allouer dans les boucles chaudes. Respecter le critère de débit et l’absence d’allocation/frame spécifiés dans les missions, sans refactor de confort non justifié.

### 15. RNG utilitaire

`util.js` contient une expression `Math.imul;` sans effet dans `RNG.next()`. Elle semble superflue ; ne la supprimer que dans un changement mineur après un test qui fixe une séquence RNG de référence, afin de prouver que le générateur reste byte/comportementalement stable.

## Audit des surfaces du dépôt

| Fichier | Résultat de la revue statique | Priorité |
|---|---|---|
| `game/src/main.js` | Transition mort boss, pause incomplète, seed du harness à vérifier face aux sources aléatoires, collisions et transitions à rejouer | P0 |
| `game/src/levels.js` | Fin de niveau 3 dépend de l’objet boss encore présent ; drapeau d’état à ajouter/tester | P0 |
| `game/src/player.js` | Incréments mouvement par frame alors que le loop fournit `dt` | P0 |
| `game/src/bullets.js` | Déplacement par appel de mise à jour ; temps dépend du taux de rendu | P0 |
| `game/src/enemies.js` | Mouvements et tir aléatoire à normaliser au temps ; vérifier les transitions de phase | P0 |
| `game/src/powerups.js` | Division par distance nulle possible dans Magnet ; mouvement par frame | P0 |
| `game/src/particles.js` | Mouvement par frame, allocations/splice, pas de plafond observé dans l’extrait | P1 |
| `game/src/background.js` | `strokeStyle` invalide ; fenêtres fondées sur modulo flottant ; gradients recréés dans draw | P1 |
| `game/src/hud.js` | Affiche pause annoncée ; vérifier synchronisation avec états, boss HP et textes gameplay | P1 |
| `game/src/input.js` | Souris globale et absence de remise à zéro au blur visibles ; commentaire tactile à vérifier | P2 |
| `game/src/audio.js` | Mute redémarre la musique si timer absent ; pas d’état distinct de demande de musique | P2 |
| `game/src/config.js` | Configuration centrale, mais tuning/couleurs dispersés selon modules | P2, changements ciblés seulement |
| `game/src/util.js` | Helper RNG contient no-op apparent ; fixer la séquence avant toute modification | P2 |
| `game/index.html` | Dimensions CSS susceptibles d’étirer l’affichage hors 16:9 | P2 |
| `shots.json` | Manifeste omet `boss`, alors qu’un staging et un fichier image existent | P1 |
| `main.ts` | Seuil screenshot 262144 bytes ; conserve l’enveloppe de soumission | Ne pas modifier ses sémantiques selon DIRECTIVES |
| `vendor/makefun-game.ts` | Bibliothèque vendor déclarée byte-identique et intouchable | Ne pas modifier |
| `serve.ts` | Serveur local de développement ; aucune anomalie fonctionnelle confirmée par cet audit | Faible |

## Séquence de correction et critères de sortie

### Étape A — tests fonctionnels avant patch

1. Créer une session enregistrée de l’agent d’implémentation.
2. Ajouter un test/reproduction minimal de la mort du boss et de la victoire ; vérifier l’état de restart.
3. Ajouter un test Magnet coïncident ; s’assurer qu’aucun nombre de position ne devient non fini.
4. Ajouter des tests ou un harness de simulation temporelle pour mouvement à 30/60/120 Hz.
5. Garder ces cas comme caractérisation avant changement comportemental.

### Étape B — petits correctifs

1. Corriger et valider la transition boss → victory.
2. Protéger et tester l’attraction Magnet.
3. Corriger la couleur de grille et le motif des fenêtres ; comparer un screenshot.
4. Implémenter pause en conservant les contrôles publics annoncés.
5. Séparer état mute, demande musique et transition d’écran.
6. Normaliser les mouvements/tirs sur `dt` dans tous les modules et régler le tuning si le test montre une dérive.
7. Ajouter RNG injecté aux scènes harness et aligner `shots.json` une fois les scènes confirmées.
8. Mesurer gradients/particules avant toute optimisation.

### Étape C — vérification du projet avant soumission

- Lancer playthrough headless ≥60 s selon `STATUS.md` ; zéro erreur console ; vérifier que les frames changent.
- Vérifier déplacement, tirs, collision, bonus, Magnet, pause/reprise, mort du joueur, boss phases, boss death, victory et restart.
- Render chaque route de capture deux fois ; mêmes seed = sortie identique, sauf variation documentée et acceptable.
- Vérifier capture JPEG 1280×720, quatre captures maximum, taille par image sous le vrai plafond 262144 octets ; regarder aussi à hauteur 480 px.
- Vérifier `window.__shotReady` sur chaque scène.
- Exécuter `MAKEFUN_OUTPUT_DIR=out node main.ts`, puis `makefun check` et exiger 0 échec avant une nouvelle soumission.
- Ne pas modifier `main.ts` sémantiquement ni `vendor/makefun-game.ts`; ne pas toucher `.makefun/owner.json`; pas de dépendances, CDN, réseau ou assets externes.
- Publier les preuves réelles : commandes, sorties, dimensions/poids des images, commit et éventuels échecs. Ne pas annoncer un critère comme satisfait sans preuve.

## Limites et statut

- Les constats proviennent d’une inspection statique de fichiers GitHub et de leurs extraits disponibles le 17 novembre 2026. Ils ne remplacent pas l’accès à un clone complet ni l’exécution du navigateur.
- Aucune correction de jeu, test, capture ou mesure FPS n’a été réalisée par ce document.
- Les patchs sont des exemples à re-dériver par l’agent d’implémentation dans une session enregistrée, conformément au gate d’auteur de make.fun.
- Statut : **audit rédigé pour triage ; corrections et tests en attente de l’agent d’implémentation**.
