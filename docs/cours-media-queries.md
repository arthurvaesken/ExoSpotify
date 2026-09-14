# Media queries et responsive — fiche de référence

## 1. Ce qu'est une media query

Un bloc CSS **conditionnel**. Les règles qu'il contient ne s'appliquent que si la condition
est vraie. Le navigateur la réévalue en continu : redimensionner la fenêtre suffit à faire
basculer les styles, sans rechargement.

```css
/* toujours appliqué */
.panneau {
    width: 100%;
}

/* appliqué uniquement si la fenêtre fait 1024px ou plus */
@media (min-width: 1024px) {
    .panneau {
        width: 280px;
    }
}
```

## 2. Anatomie

```css
@media (min-width: 1024px) { ... }
/*  ^       ^         ^
    |       |         valeur du seuil
    |       caractéristique testée
    règle-at (at-rule)                */
```

Les trois formes utiles :

| Écriture | Sens |
|---|---|
| `@media (min-width: 768px)` | largeur **≥** 768px |
| `@media (max-width: 767px)` | largeur **≤** 767px |
| `@media (orientation: landscape)` | fenêtre plus large que haute |

Combinaisons :

```css
@media (min-width: 768px) and (max-width: 1023px) { }   /* ET : entre les deux */
@media (max-width: 600px), (orientation: portrait) { }  /* la virgule = OU     */
```

> **Important : `min-width` teste la largeur du *viewport*, pas celle de l'élément.**
> Pour réagir à la largeur d'un conteneur, il faut les *container queries* (`@container`) —
> autre sujet, pas nécessaire ici.

### px ou em ?

`@media (min-width: 64em)` respecte la taille de police choisie par l'utilisateur : quelqu'un
qui agrandit la police par défaut de son navigateur bascule de palier plus tôt, ce qui est le
comportement souhaitable en accessibilité. `px` ignore ce réglage.

Pour ce projet, `px` reste acceptable et plus lisible. Retiens simplement que `em` existe et
pourquoi, c'est une question classique en entretien.

## 3. Le mécanisme à comprendre absolument

**Une media query n'ajoute AUCUNE spécificité.**

`@media (min-width: 1024px) { .panneau { width: 280px } }` a exactement le même poids que
`.panneau { width: 100% }`. À spécificité égale, c'est **la dernière règle écrite qui gagne**.

Conséquence directe :

```css
/* ✅ CORRECT — la media query est APRÈS */
.panneau { width: 100%; }
@media (min-width: 1024px) { .panneau { width: 280px; } }

/* ❌ FAUX — la media query est avant, la règle de base l'écrase */
@media (min-width: 1024px) { .panneau { width: 280px; } }
.panneau { width: 100%; }
```

Dans le second cas, à 1440px les deux règles s'appliquent, mais celle du bas est écrite en
dernier : la largeur vaut 100%. La media query ne sert à rien.

**Règle de survie : les media queries se placent toujours après les règles qu'elles surchargent,
et par valeur de seuil croissante.**

```css
/* base : mobile */
@media (min-width: 768px)  { }   /* tablette */
@media (min-width: 1024px) { }   /* desktop  */
```

À 1440px, les deux blocs s'appliquent (1440 ≥ 768 ET ≥ 1024). Celui de 1024, écrit en dernier,
a le dernier mot sur les propriétés qu'ils partagent. C'est exactement l'empilement voulu.

## 4. Mobile-first : la méthode

1. **Écris d'abord la version mobile, sans aucune media query.** C'est la base, elle
   s'applique partout.
2. **Élargis la fenêtre** jusqu'à ce que la mise en page commence à mal vieillir — trop
   d'espace perdu, lignes de texte trop longues, éléments trop étirés.
3. **Pose un palier à cet endroit** et n'y écris **que ce qui change**.
4. Recommence.

Le point de rupture est dicté par **le contenu**, jamais par une liste de tailles d'appareils
recopiée sur internet. Savoir dire « j'ai posé mon palier à 1024px parce qu'en dessous, mon
panneau de 280px plus mes cartes de 196px ne tiennent plus » vaut infiniment mieux que
« c'est la taille d'un iPad ».

## 5. Où les placer dans le fichier

Deux écoles :

- **Toutes regroupées en fin de fichier**, sous un commentaire `/*RESPONSIVE*/`. Simple à
  relire, on voit tous les paliers d'un coup. **Recommandé pour commencer.**
- **Chacune collée sous le composant qu'elle modifie.** Plus proche du code concerné, mais
  éparpille les paliers dans tout le fichier.

Dans les deux cas, la media query vient **après** la règle de base (cf. point 3).

## 6. Les pièges

| Piège | Conséquence |
|---|---|
| `<meta name="viewport">` absent ou sans `initial-scale=1` | le mobile rend la page comme un écran de 980px : **aucune media query ne se déclenche** |
| Media query écrite avant la règle de base | elle est écrasée, aucun effet |
| Chevauchement `max-width: 768` et `min-width: 768` | à 768px exactement, les deux s'appliquent |
| Redéclarer dans la media query ce qui ne change pas | duplication inutile, maintenance doublée |
| Tester en réduisant la fenêtre du navigateur | ne reproduit ni le tactile, ni la densité d'écran, ni le user-agent mobile |

## 7. Comment tester

`Cmd + Option + I` → `Cmd + Shift + M` (barre d'appareil). Saisir des largeurs précises :
**375** (mobile), **768** (tablette), **1024**, **1440**.

Tester aussi le **paysage** : bascule via l'icône de rotation. Le critère du référentiel
mentionne explicitement portrait **et** paysage.

## 8. `order` en flexbox — le complément

`order` change l'ordre **d'affichage** des éléments d'un conteneur flex, sans toucher au HTML.
Valeur par défaut : `0`. Les valeurs négatives remontent, les positives descendent.

```css
.conteneur { display: flex; flex-direction: column; }
.panneau   { order: 2; }   /* affiché après */
.principal { order: 1; }   /* affiché avant */
```

⚠️ **`order` ne modifie que le rendu visuel.** L'ordre de tabulation clavier et la lecture par
un lecteur d'écran suivent l'ordre du **document HTML**, pas l'ordre visuel. Un écart trop
grand entre les deux devient un défaut d'accessibilité (critère WCAG 1.3.2 « ordre séquentiel
logique »).

Ici l'écart reste minime — deux blocs intervertis sur mobile — et c'est un usage admis.
Mais ne t'en sers jamais pour réorganiser toute une page.

## 9. Application au projet Spotify

### Exemple complet, à recopier tel quel

```css
/*RESPONSIVE*/

/* --- Barre de recherche --- */

.barre_recherche {
    width: 100%;
    height: 48px;
}

@media (min-width: 1024px) {
    .barre_recherche {
        width: 474px;
    }
}
```

La règle de base perd `width: 474px` au profit de `100%` ; les 474px partent dans le palier
1024. `height: 48px` ne change jamais : il reste dans la base et n'est **pas** répété.

### Ce qui reste à écrire, sur le même modèle

| Classe | Base (mobile) | Palier 1024px |
|---|---|---|
| `.panneau_bibliotheque` | `width: 100%` · `order: 2` · *aucun `position`, aucun `height`* | `position: fixed` · `top: 64px` · `width: 280px` · `height: calc(100vh - 72px)` · `order: 0` |
| `.zone_principale` | `margin-left: 0` · `margin-top: 64px` · `order: 1` · `padding: 16px` | `margin-left: 288px` · `flex: 1` · `min-width: 0` · `padding: 22px 40px` |
| `.pied_de_page` | `margin-left: 0` | `margin-left: 288px` |

### Le conteneur intermédiaire

`<div class="display_flex p_left_right_8">` doit passer en colonne sur mobile, sinon `order`
n'a rien à ordonner et l'aside restera collé à côté du main. Donne-lui une classe composant
(`.zone_contenu` par exemple) plutôt qu'un utilitaire, puisque sa valeur change selon le palier :

```css
.zone_contenu {
    display: flex;
    flex-direction: column;
    padding: 0 8px;
}

@media (min-width: 1024px) {
    .zone_contenu {
        flex-direction: row;
    }
}
```
