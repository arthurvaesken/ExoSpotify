# Hover et transitions — fiche de référence

## 1. Les pseudo-classes d'état

Une pseudo-classe cible un élément **dans un certain état**, sans rien changer au HTML.

| Pseudo-classe | État ciblé |
|---|---|
| `:hover` | le curseur est au-dessus |
| `:focus-visible` | l'élément a le focus **et** le navigateur juge utile de le montrer (clavier) |
| `:focus` | l'élément a le focus, quelle qu'en soit la cause — souris comprise |
| `:active` | l'élément est en cours de clic |

**`:focus-visible` plutôt que `:focus` :** avec `:focus`, un simple clic souris laisse un contour
disgracieux après le clic. C'est ce qui a poussé des générations de développeurs à écrire
`outline: none` et à casser l'accessibilité. `:focus-visible` laisse le navigateur décider :
l'indicateur n'apparaît qu'en navigation clavier.

> **Règle d'accessibilité :** tout effet de survol doit avoir son équivalent au focus clavier.
> Sinon l'information visuelle n'existe que pour les utilisateurs de souris.

## 2. Anatomie de `transition`

```css
transition: color 0.2s ease 0s;
/*          ^     ^    ^    ^
            |     |    |    delai avant demarrage (optionnel)
            |     |    courbe d'acceleration
            |     duree
            propriete a animer                        */
```

- `ease` (défaut) : démarre vite, ralentit à la fin. Naturel dans 90 % des cas.
- `linear` : vitesse constante. Pour les rotations, pas pour les déplacements.
- `ease-out` : le choix des interfaces réactives, l'effet paraît immédiat.

Plusieurs propriétés se séparent par des virgules :

```css
transition: color 0.2s ease, transform 0.2s ease;
```

⚠️ Éviter `transition: all` : le navigateur surveille alors toutes les propriétés, y compris
celles qu'on ne touche pas. Gaspillage, et animations parasites.

## 3. Le piège du placement

```css
/* CORRECT — la transition vit sur l'etat NORMAL */
nav a {
    color: #B3B3B3;
    transition: color 0.2s ease;
}
nav a:hover { color: #FFFFFF; }
```

```css
/* FAUX */
nav a { color: #B3B3B3; }
nav a:hover {
    color: #FFFFFF;
    transition: color 0.2s ease;   /* n'existe que pendant le survol */
}
```

**Pourquoi :** la transition appartient à l'état *depuis lequel* on part. Dans la version fausse,
la déclaration disparaît avec l'état au moment où le curseur sort — l'aller s'anime, le retour
est sec. Effet asymétrique très reconnaissable.

## 4. Tout ne s'anime pas

Une propriété n'est animable que si elle a des valeurs intermédiaires calculables.

| Animable | Non animable |
|---|---|
| `color`, `background-color`, `opacity` | `display` |
| `transform`, `box-shadow`, `border-radius` | `position` |
| `width`, `height`, `margin`, `padding` | `font-family` |

Un `display: none` → `display: block` ne se fond donc jamais. Contournement classique :
`opacity` + `visibility`.

## 5. Performance — argument éco-conception

Les propriétés animables ne coûtent pas la même chose :

- `width`, `height`, `margin`, `top` → **reflow** : le navigateur recalcule la position de tous
  les éléments voisins, à chaque image, 60 fois par seconde.
- `transform`, `opacity` → traitées par le **compositeur**, souvent sur le GPU, sans toucher
  à la mise en page.

> **Pour agrandir un élément au survol : `transform: scale(1.04)`, jamais `width`/`height`.**
> Rendu identique, coût sans commune mesure. C'est l'argument à sortir sur le critère
> « éco-conception » — et c'est ce que fait Spotify.

## 6. `prefers-reduced-motion`

Certains utilisateurs souffrent de troubles vestibulaires : les animations leur provoquent
nausées et vertiges. Les systèmes exposent un réglage « Réduire les animations », lisible en CSS.

```css
@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        transition-duration: 0.01ms !important;
        animation-duration: 0.01ms !important;
    }
}
```

Critère **WCAG 2.3.3**. Test sur macOS : Réglages Système → Accessibilité → Écran →
Réduire les animations.

## 7. Application au projet Spotify

```css
/*ETATS INTERACTIFS*/

nav a {
    transition: color 0.2s ease;
}

nav a:hover,
nav a:focus-visible {
    color: #FFFFFF;
}
```

La couleur au repos vient déjà de la classe `.grey_B3B3B3` : inutile de la redéclarer,
seule la transition manque.

Reste à écrire : l'effet du bouton « Se connecter » (`.bg_white`), qui grossit légèrement
au survol sur le vrai Spotify. Viser `scale(1.04)` — au-delà, ça fait pataud.
