# NEON HAND — FINGER DRUMS

Instrument contrôlé à la main via la caméra de l'iPhone.
Les **boutons** tiennent les nappes (basses et synthés continus), les **doigts**
jouent les percussions : un doigt qui se plie déclenche un son, sur les deux mains.

Une seule page, aucune dépendance à installer, aucun échantillon à télécharger :
**tout le son est synthétisé en temps réel en Web Audio**.
Pensé pour **Safari iOS en mode portrait**.

---

## Utilisation

1. Ouvrir la page en **HTTPS** (obligatoire pour la caméra) — voir « Mise en ligne ».
2. Appuyer sur **DÉMARRER**. Le son part immédiatement avec une basse ; la caméra
   et le modèle de tracking se chargent ensuite (quelques secondes la première fois).
3. Présenter une main, ou les deux, devant la caméra. Le squelette apparaît, et le
   nom du son est écrit à côté de chaque bout de doigt.
4. **Plier un doigt d'un coup sec déclenche son son.** Tendre le doigt le réarme.
   Plus la flexion est vive, plus la frappe est forte.
5. Les **boutons du bas** allument et éteignent les nappes tenues. Elles montent et
   descendent en fondu, et plusieurs peuvent tourner ensemble.

## Le kit — un son par doigt

| Doigt | Main gauche | Main droite |
|-------|-------------|-------------|
| Pouce | KICK | 808 (sub long) |
| Index | HAT (charley fermé) | SHAKER |
| Majeur | CLAP | SNARE |
| Annulaire | RIM | TOM |
| Auriculaire | OPEN (charley ouvert) | COW (cloche) |

Les étiquettes sont dessinées en direct à côté de chaque doigt : pas besoin de
retenir la table. Si les deux kits vous semblent intervertis entre vos mains, voir
« Limites connues ».

## Les nappes

Six nappes tenues, allumées aux boutons, toutes accordées sur la même tonique :

| Bouton | Nappe |
|--------|-------|
| BASS 1 · SUB | sinus grave doublé, légère respiration d'amplitude |
| BASS 2 · ACID | dents de scie dans un passe-bas résonant balayé par deux LFO lents |
| BASS 3 · RUBBER | FM, indice de modulation qui dérive lentement |
| SYN 1 · PAD | accord de scies désaccordées, attaque de 2,6 s, filtre qui respire |
| SYN 2 · ORGUE | harmoniques empilées façon tirettes d'orgue, trémolo |
| SYN 3 · SHIMMER | octaves hautes en passe-bande balayé, mouvement stéréo |

**I WANNA DANCE** déclenche la phrase vocale. Ce n'est pas un échantillon : c'est une
**synthèse à formants**. Un carrier en dents de scie passe dans trois filtres
passe-bande dont les fréquences suivent une piste de phonèmes (`VOX_SEG` dans le
code), plus une voie de bruit pour le « D » et le « S » final. Le tout traverse un
quantificateur qui donne le grain digital.

## Les autres commandes

- **QUANT** — calé ou libre. Éteint (par défaut), chaque frappe part à l'instant où
  le doigt se plie. Allumé, elle est repoussée sur la prochaine double-croche : le jeu
  devient carré, au prix d'un retard pouvant aller jusqu'à une double-croche. La
  réglette de 16 cases en haut de l'écran montre la grille qui défile.
- **ALL OFF** — éteint toutes les nappes.
- **DELAY** — écho synchronisé au tempo (croche pointée) ; le potard dose le niveau
  envoyé et la réinjection.
- **DIST** — saturation douce sur l'ensemble, avec compensation de gain.
- **PITCH** — transposition de ±12 demi-tons. Les nappes actives glissent vers la
  nouvelle hauteur sans coupure. Double-tap pour revenir à zéro, cran magnétique au
  centre.
- **BPM** — curseur, ou bouton **TAP** (trois frappes suffisent). Sert à la grille de
  quantification et à la synchro du delay.
- **RND** — retire au sort le timbre des six nappes, la tonique, la gamme et la voix.
  Les nappes allumées sont reconstruites à la volée.

Les potards se règlent en **glissant verticalement**.

## Comment la frappe est détectée

Pour chaque doigt on mesure son extension : la distance entre le bout et la base,
divisée par la taille de la main. Cette grandeur ne dépend ni de la position de la
main dans l'image, ni de sa distance à la caméra — **seule l'articulation du doigt la
fait varier**. Une chute rapide de cette valeur est une flexion vive, donc une frappe.

Deux conséquences utiles :

- Agiter la main dans tous les sens ne déclenche rien tant que les doigts restent
  tendus. C'est vérifié par un test automatisé.
- La valeur est normalisée par le maximum observé pour ce doigt, ce qui **calibre tout
  seul** chaque doigt de chaque utilisateur, sans réglage.

Un doigt doit repasser par la position pliée **puis** tendue avant de pouvoir
redéclencher, ce qui évite qu'un même geste compte deux fois.

Les seuils sont en haut de la section « DÉTECTION DE FRAPPE » du code
(`FIRE_RATE`, `DOWN`, `REARM`, `MIN_GAP`) : baisser `FIRE_RATE` rend le déclenchement
plus facile, le monter demande des gestes plus francs.

## Mise en ligne

`getUserMedia` exige une origine sécurisée. `file://` ne fonctionne pas ; `localhost`
suffit pour développer, mais l'iPhone a besoin d'un vrai HTTPS.

Le plus simple, GitHub Pages :

```
Settings → Pages → Source: Deploy from a branch → Branch: main / (root)
```

La page est alors servie sur `https://<compte>.github.io/<repo>/`.

En local :

```bash
python3 -m http.server 8000
```

## Notes techniques

- Tracking par **MediaPipe Hand Landmarker** (`@mediapipe/tasks-vision`), deux mains,
  chargé depuis jsDelivr avec repli sur unpkg. Le modèle fait ~7,8 Mo et est mis en
  cache par le navigateur après la première visite.
- Le rendu tient compte du recadrage `object-fit: cover` de la vidéo : les points
  collent à l'image même quand le capteur n'a pas le format de l'écran.
- L'image est mise en miroir, la main à l'écran suit donc la main réelle.
- Si la caméra est refusée ou le modèle indisponible, un **kit tactile** de dix
  touches remplace les mains ; l'application reste jouable.
- Un objet `window.TR9` est exposé pour l'inspecteur web (état, bus audio, nappes,
  voix de percussion, injection de mains).

## Limites connues

- **Gauche et droite peuvent être inversées.** MediaPipe suppose que l'image lui
  arrive déjà en miroir ; le flux brut de la caméra frontale ne l'est pas, le code
  inverse donc le label. Selon l'appareil, les deux kits peuvent malgré tout se
  retrouver échangés entre vos mains. C'est sans conséquence sur le jeu — les
  étiquettes affichées restent celles de la main concernée — et la correction tient
  en une ligne dans `handKey()`.
- Les seuils de frappe ont été validés sur des mains simulées, pas sur une vraie main
  devant une caméra : le réglage de `FIRE_RATE` est l'endroit à toucher en premier si
  le déclenchement vous paraît trop sensible ou trop dur.
- L'inférence MediaPipe et le rendu partagent le GPU : sur un iPhone ancien, le
  tracking sera moins fluide, et le retard entre le geste et le son suit la cadence de
  la caméra (~30 images/s, soit ~30 ms au mieux).
- Le potard PITCH transpose les voix à la source ; ce n'est pas un pitch-shift du mix,
  donc pas d'artefacts, mais les percussions ne suivent qu'à moitié, volontairement.
