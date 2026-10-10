# AIR — palet aérien

Jeu à deux joueurs pour iPhone. Le téléphone est **posé à plat sur une table, dans la
longueur, entre les deux joueurs**. Chacun fait glisser son index **au-dessus** de
l'écran, sans jamais le toucher : la caméra frontale, qui regarde le plafond, suit les
deux doigts et chacun pilote son palet de frappe. Premier à **7 buts**.

Une seule page autonome, aucune dépendance à installer, aucun échantillon : le son est
synthétisé en Web Audio.

> Le séquenceur développé précédemment reste dans le dépôt, sous **`drums.html`**.

---

## Installation sur la table

```
        JOUEUR 2
   ┌───────────────┐
   │   ▄▄▄▄▄▄▄     │  ← son but
   │               │
   │  - - - - - -  │  ← ligne médiane
   │               │
   │   ▄▄▄▄▄▄▄     │  ← son but
   └───────────────┘
        JOUEUR 1
```

Le téléphone à plat, écran vers le ciel, un joueur à chaque petit côté. La caméra
frontale pointe donc vers le plafond, et c'est au-dessus d'elle que tout se joue : les
mains évoluent en l'air, à peu près à 20–40 cm de la vitre. Le score du joueur 2 est
affiché à l'envers, pour qu'il le lise de sa place.

**Verrouille la rotation de l'écran en portrait** avant de commencer : à plat, iOS n'a
plus de repère de gravité et peut faire pivoter l'interface en pleine partie.

## Jouer

1. Ouvrir la page en **HTTPS** (voir « Mise en ligne »), appuyer sur **JOUER**,
   autoriser la caméra. Le modèle de suivi se charge une fois (~8 Mo, mis en cache).
2. Chacun tend l'index au-dessus de l'écran, de son côté. **Le squelette de ta main
   s'affiche sur le terrain**, avec le repère `J1` ou `J2` qui dit à quel joueur le jeu
   l'attribue, et un anneau lumineux : ton palet de frappe. Anneau simple pour le
   joueur 1, double pour le joueur 2.
3. Déplace ton doigt pour frapper. **La vitesse du geste se transmet au palet** —
   un coup sec envoie loin, un contact mou amortit.
4. Comme au vrai air hockey, **chaque palet reste dans son camp** : impossible de
   franchir la ligne médiane.
5. Si ta main franchit la médiane, ton palet reste bloqué sur la ligne et **une laisse
   en pointillés** relie ton doigt à ton palet : c'est le signe que tu débordes, pas
   que le suivi a lâché.
6. Après chaque but, le palet est engagé vers celui qui vient d'encaisser.

Si un anneau passe en pointillés et que **DOIGT PERDU** s'affiche, la caméra ne voit
plus ce doigt : remonte la main vers le centre du champ.

## Les réglages du menu

- **MIROIR ⇄** et **SENS ⇅** — l'orientation du flux caméra dépend du modèle et de la
  façon dont iOS présente les images quand l'appareil est à plat. Si ton palet part du
  mauvais côté, ou si vous vous retrouvez à piloter le palet de l'autre, bascule l'un
  ou l'autre. Miroir est actif par défaut.
- **VOIR CAMÉRA** — affiche l'image en fond, très atténuée. Utile pour vérifier que la
  caméra vous voit bien tous les deux ; inutile pendant la partie.

Le bouton **❚❚** sur le bord droit met en pause. Après une victoire, un appui n'importe
où relance une partie.

## Comment les doigts sont lus

- Suivi par **MediaPipe Hand Landmarker**, deux mains, dont on ne garde que le bout de
  l'index (point 8).
- Le centre du champ de la caméra est étiré sur tout le terrain, avec **12 % de marge
  ignorée sur les bords**, là où la détection décroche. Le geste se fait donc en l'air,
  dans un volume au-dessus du téléphone, et non en vis-à-vis exact de l'écran.
- **Affectation persistante des mains.** Chaque main reste accrochée au palet qu'elle
  pilotait : on l'apparie à l'endroit où ce doigt se trouvait à l'image précédente, et
  non à sa position absolue sur le terrain. C'est indispensable — une affectation
  refaite à chaque image (« le doigt le plus haut au joueur du haut ») intervertit les
  deux joueurs dès qu'un doigt approche la médiane ou qu'une main cligne.
  - L'ancre est la dernière position du **doigt**, jamais celle du palet : le palet
    étant bloqué à la médiane quand la main déborde, il s'éloignerait de sa propre
    main au point de sembler appartenir à l'adversaire.
  - Le **camp** ne départage que lorsque la continuité est rompue — main perdue puis
    réapparue ailleurs. Tant que les deux doigts sont près de là où ils étaient, la
    continuité seule décide, sans quoi deux joueurs chevauchant la médiane en même
    temps se feraient intervertir.
  - Chaque palet reste ensuite borné à son camp : personne ne peut prendre le contrôle
    du palet adverse ni franchir la ligne.
- Position lissée à 55 % par image, et la vitesse du palet de frappe est calculée sur
  cette position lissée : c'est elle qui est transmise au palet lors du choc.

## Physique

- Pas fixe de 1/120 s, découplé de l'affichage et de la caméra : le jeu reste régulier
  même si le suivi tourne à 30 images/s.
- Choc palet/frappeur traité comme une masse infinie contre une masse mobile,
  restitution 0,95, avec séparation immédiate et poussée minimale pour que le palet ne
  reste jamais collé.
- Rebonds sur les bandes à 0,93, vitesse plafonnée, légère friction.
- Le terrain s'arrête aux bandeaux de score, ce qui garde **les cages à l'écart de
  l'encoche et de la barre d'accueil** — sans quoi le but du haut serait partiellement
  masqué par la Dynamic Island.

## Mise en ligne

`getUserMedia` exige une origine sécurisée : `file://` ne marche pas, et l'iPhone a
besoin d'un vrai HTTPS. Avec GitHub Pages :

```
Settings → Pages → Source: Deploy from a branch → Branch: <branche> / (root)
```

Le jeu est alors servi à la racine, et le séquenceur sur `/drums.html`.

En local : `python3 -m http.server 8000`.

## Limites connues

- **L'orientation du flux caméra n'est pas devinable à l'avance** quand le téléphone
  est à plat. D'où les deux bascules MIROIR et SENS : c'est le premier réglage à
  essayer si les contrôles semblent inversés.
- **Le champ de la caméra frontale n'est pas infini.** L'objectif est sur un bord du
  téléphone : si un joueur descend trop la main ou s'écarte trop, il sort du cadre.
  Mains à plat au-dessus du centre de l'appareil, c'est là que ça marche le mieux.
- Si la caméra est refusée ou le suivi indisponible, le jeu bascule en **repli
  tactile** : chacun garde un doigt posé sur l'écran, de son côté.
- Le suivi tourne à la cadence de la caméra (~30 images/s), ce qui pose un plancher
  d'environ 30 ms entre le geste et le palet. C'est jouable, mais ce n'est pas la
  réactivité d'un écran tactile.
- Testé en navigateur avec des mains simulées : croisement des deux doigts au-dessus
  l'un de l'autre, perte puis retour d'une main, trois secondes de va-et-vient de part
  et d'autre de la médiane (zéro interversion), plus le bornage des camps, la frappe,
  les buts, les rebonds et la victoire. Pas encore avec deux vraies mains au-dessus
  d'un iPhone posé sur une table.
