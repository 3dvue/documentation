# Documentation — `<viewer-3dvue>`

---

## Avant-propos - comprendre les urls

Toute scène 3D est identifiée par le paramètre `p` dans l'URL du viewer. Ce paramètre encode à la fois **quel(s) objet(s) afficher** et **quels matériaux leur appliquer**, sous forme d'une chaîne de codes à 2 caractères.

### Structure d'un modèle

Un modèle décrit un objet et ses matériaux :

```
[ID objet (2 char.)][matériau mesh 1 (2 char.)][matériau mesh 2 (2 char.)]...
```

Exemple pour un objet composé de 3 meshs :

```
01  01  1g  00
↑   ↑   ↑   ↑
│   │   │   └─ mesh 3 → hidden
│   │   └───── mesh 2 → plastique-gris
│   └───────── mesh 1 → cuir-rouge
└───────────── ID objet → chaise_beluga_2
```

Ce qui donne dans l'URL : `?p=01011g00`

### Plusieurs objets

Plusieurs objets sont séparés par un `-` :

```
?p=01011g00-020100
   ↑         ↑
   objet 1   objet 2
```

### Référence des objets et matériaux

La composition des objets (ID + liste ordonnée des meshes) est disponible sous forme de csv que 3Dvue fournira :

| Name | ID | Mesh 1 | Mesh 2 | Mesh 3 | ... |
|---|---|---|---|---|---|
| chaise_beluga_2 | `01` | mesh_001 | mesh_002 | mesh_003 | ... |
| poele-strato | `02` | mesh_001 | mesh_002 | mesh_003 | ... |
| augustin | `04` | mesh_001 | mesh_005 | mesh_009 | ... |

La liste des matériaux disponibles (ID + nom) est disponible sous forme de csv que 3Dvue fournira :
| Name | ID |
|---|---|
| hidden | `00` |
| cuir-rouge | `01` |
| plastique-gris | `1g` |

> [!NOTE]
> L'**index 0** de chaque modèle est réservé à l'ID de l'objet. Les matériaux des meshes commencent donc à l'**index 1**. C'est pourquoi `setMaterial({ "1": "01" })` cible le premier mesh, pas l'index 0.

---

## Installation

Inclure le script du composant dans la page :

```html
<script src="viewer-3dvue.js"></script>
```

---

## Déclaration du composant

```html
<viewer-3dvue
  name="monViewer"
  src="https://app.3dvue.fr/view?p=AB01"
  debug
></viewer-3dvue>
```

| Attribut | Requis | Description |
|---|---|---|
| `name` | ✅ | Identifiant unique du viewer (utilisé pour le ciblage HTML) |
| `src` | ✅ | URL du viewer contenant le paramètre `?p=` avec l'ID du modèle |
| `debug` | — | Active les logs de débogage dans la console |

---

## Partie 1 — Utilisation Basique

Le composant écoute automatiquement les événements `click` et `change` sur n'importe quel élément de la page portant les bons attributs. **Aucun JavaScript n'est nécessaire.**

Tout élément déclencheur doit cibler un viewer via :

```html
viewer-3dvue="nomDuViewer"
```

---

### `setMaterial` sur clic

Applique des matériaux au clic sur un élément (bouton, div, etc.).

```html
<button
  viewer-3dvue="monViewer"
  viewer-3dvue-clic="setMaterial"
  viewer-3dvue-meshs-materials='{"1":"0n","2":"1a"}'
>
  Appliquer couleur rouge
</button>
```

| Attribut | Description |
|---|---|
| `viewer-3dvue-clic` | Doit valoir `"setMaterial"` |
| `viewer-3dvue-meshs-materials` | JSON `{ "indexMesh": "idMaterial", ... }` |

> [!NOTE]
> L'index `0` est réservé et ignoré. Les index de mesh commencent à `1`.

**Exemple — plusieurs matériaux simultanément :**

```html
<button
  viewer-3dvue="monViewer"
  viewer-3dvue-clic="setMaterial"
  viewer-3dvue-meshs-materials='{"1":"0n","3":"2b","5":"1c"}'
>
  Thème bois naturel
</button>
```

---

### `setMaterial` sur `<select>`

Applique le matériau sélectionné à un ou plusieurs meshes lorsque la valeur du `<select>` change.

```html
<select
  viewer-3dvue="monViewer"
  viewer-3dvue-change="setMaterial"
  viewer-3dvue-mesh="2,4"
>
  <option value="0n">Rouge</option>
  <option value="1a">Bleu</option>
  <option value="2b">Vert</option>
</select>
```

| Attribut | Description |
|---|---|
| `viewer-3dvue-change` | Doit valoir `"setMaterial"` |
| `viewer-3dvue-mesh` | Index des meshes à affecter, séparés par des virgules |

> [!NOTE]
> La `value` sélectionnée est appliquée à **tous les meshes** listés dans `viewer-3dvue-mesh`.

---

## Partie 2 — API JavaScript

Récupérer la référence du composant :

```js
const viewer = document.querySelector('viewer-3dvue[name="monViewer"]');
```

> [!WARNING]
> Toutes les méthodes vérifient que le viewer est prêt (réception de `LOAD_COMPLETE`). Si ce n'est pas le cas, un warning est émis et l'appel est ignoré.

---

### `loadModel(id)`

Charge un nouveau modèle en remplaçant le modèle actuel.

```js
viewer.loadModel("AB01");
```

| Paramètre | Type | Description |
|---|---|---|
| `id` | `string` | ID complet du modèle à charger |

---

### `addModel(id)`

Ajoute un ou plusieurs objets à la scene actuelle.

```js
viewer.addModel("EF03");
viewer.addModel("EF03-GH04"); // plusieurs objets
```

| Paramètre | Type | Description |
|---|---|---|
| `id` | `string` | Models(s) à ajouter |

---

### `removeModel(objectIndex)`

Supprime un model par son index et recharge la scène.

```js
viewer.removeModel(1); // supprime le 2e modele
```

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `objectIndex` | `number` | `0` | Index du model à supprimer |

> [!CAUTION]
> Impossible de supprimer le dernier model restant.

---

### `setMaterial(materialList, objectIndex)`

Applique une liste de matériaux sur un objet.

```js
viewer.setMaterial({ "1": "0n", "3": "2b" });    // objet 0 (défaut)
viewer.setMaterial({ "1": "1a" }, 1);             // objet 1
```

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `materialList` | `Object` | — | `{ "indexMesh": "idMaterial", ... }` |
| `objectIndex` | `number` | `0` | Index du model cible |

---

### `resetCamera()`

Replace la caméra à sa position initiale.

```js
viewer.resetCamera();
```

---

### `pauseRenderer()` / `startRenderer()`

Mettent en pause ou relancent le rendu 3D.

```js
viewer.pauseRenderer();
viewer.startRenderer();
```

> [!NOTE]
> Ces méthodes sont appelées automatiquement selon la visibilité du composant dans la page via un `IntersectionObserver`.

---

### `captureToBase64()`

Capture le rendu courant en image base64. Le résultat est retourné via un événement.

```js
viewer.captureToBase64();

viewer.addEventListener("onCaptureBase64Complete", (e) => {
  const { base64, error } = e.detail;
  if (error) { console.error(error); return; }
  document.querySelector("img").src = base64;
});
```

**Événement :** `onCaptureBase64Complete`

| `e.detail` | Type | Description |
|---|---|---|
| `base64` | `string` | Image encodée en base64 |
| `error` | `string \| null` | Message d'erreur si échec |

---

## Partie 3 — API SVG

### `exportSVG()`

Déclenche l'export de la map SVG du produit. Le résultat est retourné via un événement.

```js
viewer.exportSVG();

viewer.addEventListener("onSVGExportComplete", (e) => {
  const { path, error } = e.detail;
  if (error) { console.error(error); return; }
  console.log("SVG disponible :", path);
});
```

**Événement :** `onSVGExportComplete`

| `e.detail` | Type | Description |
|---|---|---|
| `path` | `string \| null` | Chemin du fichier SVG généré |
| `error` | `string \| null` | Message d'erreur si échec |

---

### Notion de dropzone

Une **dropzone** est un rectangle défini dans le fichier SVG du produit, identifié par son attribut `id`. Il n'est pas rendu visuellement mais sert de zone de destination pour les éléments ajoutés dynamiquement (texte, image). L'élément ajouté est positionné et dimensionné pour s'inscrire dans ce rectangle.

```xml
<!-- Exemple dans le SVG source -->
<rect id="dropzone001" x="50" y="80" width="300" height="150" fill="none" />
```

> [!NOTE]
> La dropzone par défaut est `"dropzone001"`. Si plusieurs zones sont disponibles dans le SVG, il suffit de passer l'`id` correspondant via l'option `dropzone`. Si 'il n'y a pas de dropzone impossible d'ajouter un élément

---

### `addSVGText(text, opts)`

Ajoute un texte dans une dropzone du SVG.

```js
viewer.addSVGText("Mon texte", {
  fill: "#ff0000",
  fontFamily: "Arial",
  dropzone: "dropzone001"
});
```

| Option | Type | Défaut | Description |
|---|---|---|---|
| `fill` | `string` | `"#000000"` | Couleur du texte (hex) |
| `fontFamily` | `string` | `''` | Police de caractères |
| `dropzone` | `string` | `"dropzone001"` | `id` du rectangle cible dans le SVG |

---

### `addSVGImage(source, opts)` *(async)*

Ajoute une image dans une dropzone du SVG. Accepte un objet `File` ou un `ArrayBuffer`.

```js
// Depuis un input file
const file = document.querySelector('input[type="file"]').files[0];
await viewer.addSVGImage(file, {
  dropzone: "dropzone001"
});

// Depuis un ArrayBuffer
const response = await fetch("/logo.png");
const buffer = await response.arrayBuffer();
await viewer.addSVGImage(buffer, { dropzone: "dropzone002" });
```

| Paramètre | Type | Description |
|---|---|---|
| `source` | `File \| ArrayBuffer` | Source de l'image |

| Option | Type | Défaut | Description |
|---|---|---|---|
| `dropzone` | `string` | `"dropzone001"` | `id` du rectangle cible dans le SVG |

---

### `setSVGElementParam(parameter, value, id)`

Modifie un paramètre d'un élément SVG existant.

```js
// Cibler un élément par son id
viewer.setSVGElementParam("fill", "#ff0000", "img-1234567890");

// Cibler l'élément actuellement sélectionné dans le viewer
viewer.setSVGElementParam("opacity", 0.5, null);
```

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `parameter` | `string` | — | Nom du paramètre à modifier |
| `value` | `any` | — | Nouvelle valeur |
| `id` | `string \| null` | `null` | ID de l'élément ciblé, `null` pour cibler l'élément **actuellement sélectionné** |

Les paramètres disponibles incluent notamment :

| `parameter` | Description |
|---|---|
| `fill` | Couleur de remplissage (texte ou forme) |
| `fontFamily` | Police (texte uniquement) |
| `opacity` | Opacité (0 à 1) |
| `angle` | Rotation en degrés |
| `scaleX` / `scaleY` | Échelle sur chaque axe |

> [!NOTE]
> L'`id` d'un élément ajouté dynamiquement est retourné par `addSVGText()` et `addSVGImage()`.

---

### Événements SVG

| Événement | `e.detail` | Description |
|---|---|---|
| `onSVGObjectSelected` | `{ obj }` | Un objet SVG a été sélectionné dans le viewer |
| `onSVGObjectDeselected` | *(vide)* | La sélection SVG a été annulée |

```js
viewer.addEventListener("onSVGObjectSelected", (e) => {
  console.log("Objet sélectionné :", e.detail.obj);
});

viewer.addEventListener("onSVGObjectDeselected", () => {
  console.log("Désélectionné");
});
```
