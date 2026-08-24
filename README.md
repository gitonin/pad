# NEON HAND — TR-9

Séquenceur 9 pads contrôlé à la main via la caméra de l'iPhone.
Une seule page, aucune dépendance à installer, aucun échantillon audio à télécharger :
**tout le son est synthétisé en temps réel en Web Audio**.

Pensé pour **Safari iOS en mode portrait**.

---

## Utilisation

1. Ouvrir la page en **HTTPS** (obligatoire pour la caméra) — voir « Mise en ligne ».
2. Appuyer sur **DÉMARRER** : le son part immédiatement, la caméra et le modèle de
   tracking se chargent en tâche de fond (quelques secondes la première fois).
3. Présenter la main devant la caméra : les 21 articulations sont suivies et reliées
   par des filets néon, avec des traînées sur les bouts de doigts.
4. **Tendre l'index vers un pad et approcher la main de l'écran.** Un anneau se
   remplit autour du pad ; une fois plein, la piste démarre.
   - Plus la main est proche, plus le remplissage est rapide (~0,4 s au plus près).
   - Une poussée franche vers la caméra déclenche instantanément.
   - La jauge verticale à droite indique la proximité détectée.
5. **Retaper le même pad coupe sa piste.** Il faut sortir du pad puis y revenir : tant
   que le doigt reste dessus, rien ne se répète.

Les pads répondent aussi au **toucher direct**, ce qui sert de repli si la caméra est
refusée ou indisponible.

## Les 9 pistes

| Pad | Piste | Son |
|-----|-------|-----|
| 01 | BASS 1 | basse acid façon 303 : filtre résonant, accents, glissandos |
| 02 | BASS 2 | sub sinusoïdal avec attaque pitchée |
| 03 | BASS 3 | basse FM « rubber », rapport et indice de modulation variables |
| 04 | SYN 1 | stab d'accord détuné, la marque de fabrique new beat |
| 05 | SYN 2 | pluck court en bande passante |
| 06 | SYN 3 | lead à trois oscillateurs désaccordés, filtre balayé |
| 07 | RYTM 1 | grosse caisse + clap, avec ghost notes |
| 08 | RYTM 2 | charleys fermés/ouverts, rimshot, toms |
| 09 | VOX | « I WANNA DANCE » |

La voix n'est pas un échantillon : c'est une **synthèse à formants**. Un carrier en
dents de scie passe dans trois filtres passe-bande dont les fréquences suivent une
piste de phonèmes (`VOX_SEG` dans le code), plus une voie de bruit pour le « D » et
le « S » final. Le tout traverse un quantificateur qui donne le grain digital.

## Séquenceur

- **8 pas** par boucle, une croche par pas, soit une mesure de 4/4.
- Toutes les pistes partagent le même compteur de pas et le même horloge
  d'ordonnancement (fenêtre de 140 ms, révision toutes les 22 ms) : **elles sont donc
  toujours calées entre elles**, quel que soit le moment où on les active.
- **BPM** de 88 à 152 au curseur, ou au bouton **TAP** (trois frappes suffisent).

## Commandes du bas

- **DELAY** — départ d'écho synchronisé au tempo (croche pointée), le potard dose à la
  fois le niveau envoyé et la réinjection.
- **DIST** — saturation douce sur l'ensemble du mix, avec compensation de gain et
  fermeture progressive du filtre de sortie.
- **PITCH** — transposition globale de ±12 demi-tons. Elle s'applique aux notes, aux
  fûts (de façon atténuée) et à la voix. Double-tap pour revenir à zéro ; un cran
  magnétique retient le centre.
- **RND SOUNDS** — retire au sort les timbres des 9 pistes, la tonique et la gamme.
- **RND PATTERN** — retire au sort les 9 séquences.
- **ALL OFF** — coupe toutes les pistes.
- **▶ / ■** — arrête ou relance l'horloge.

Les potards se règlent en **glissant verticalement**.

## Mise en ligne

`getUserMedia` exige une origine sécurisée. `file://` ne fonctionne pas ; `localhost`
fonctionne pour le développement, mais l'iPhone a besoin d'un vrai HTTPS.

Le plus simple, GitHub Pages :

```
Settings → Pages → Source: Deploy from a branch → Branch: main / (root)
```

La page est alors servie sur `https://<compte>.github.io/<repo>/`.

En local :

```bash
python3 -m http.server 8000
# puis http://localhost:8000 sur la machine de développement
```

## Notes techniques

- Tracking par **MediaPipe Hand Landmarker** (`@mediapipe/tasks-vision`), chargé
  depuis jsDelivr avec repli sur unpkg. Le modèle fait ~7,8 Mo et est mis en cache
  par le navigateur après la première visite.
- La détection de proximité utilise l'envergure de la paume mesurée en pixels
  d'affichage et rapportée à la hauteur du cadre : elle reste donc stable quelle que
  soit l'orientation de la main, contrairement à une mesure faite directement en
  coordonnées normalisées.
- Le rendu tient compte du recadrage `object-fit: cover` de la vidéo, si bien que les
  points dessinés collent à l'image même quand le capteur n'a pas le format de l'écran.
- L'image de la caméra est mise en miroir : la main à l'écran suit la main réelle.
- Un objet `window.TR9` est exposé pour l'inspecteur web (état, bus audio, injection
  de points de main).

## Limites connues

- Une seule main suivie à la fois (`numHands: 1`), pour garder le framerate.
- L'inférence MediaPipe et le rendu partagent le GPU : sur un iPhone ancien, attendez-
  vous à un tracking moins fluide. L'audio, lui, n'est jamais affecté — il est
  ordonnancé par l'horloge audio, pas par la boucle d'affichage.
- Le potard PITCH transpose les voix à la source ; ce n'est pas un pitch-shift du mix,
  donc pas d'artefacts, mais les percussions restent volontairement peu affectées.
