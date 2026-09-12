# V2 — Cross-Site Scripting / DOM XSS (Recherche)

## 1️⃣ Cause

Dans `frontend/src/app/search-result/search-result.component.ts`, le terme de recherche saisi par l'utilisateur était traité avec `sanitizer.bypassSecurityTrustHtml(queryParam)`, une fonction qui **désactive volontairement** la protection XSS native d'Angular pour la valeur concernée, alors que `queryParam` provient directement de l'URL (donc de l'utilisateur, non fiable) :

```typescript
this.searchValue = this.sanitizer.bypassSecurityTrustHtml(queryParam)
```

Le résultat était ensuite injecté dans le DOM via `[innerHTML]` dans le template HTML, permettant l'exécution de payloads comme `<iframe src="javascript:alert('xss')">`.

## 2️⃣ Remédiation

Remplacement par une sanitization réelle via l'API Angular :

```typescript
import { SecurityContext } from '@angular/core'
...
this.searchValue = this.sanitizer.sanitize(SecurityContext.HTML, queryParam) ?? ''
```

## 3️⃣ Justification

`sanitizer.sanitize()` nettoie réellement le HTML en retirant les balises et attributs dangereux (comme `<iframe>` avec un `src` en `javascript:`), contrairement à `bypassSecurityTrustHtml()` qui ne fait aucune vérification et fait confiance à l'entrée telle quelle.

## 4️⃣ Vérification

- **Avant correction** : URL `http://localhost:3000/#/search?q=<iframe src="javascript:alert('xss')">` → une alerte JavaScript `xss` se déclenche
- **Après correction** : même URL → aucune alerte, la page affiche "No results found", le payload est neutralisé
- Commit poussé sur GitHub (`Fix: DOM XSS in search (CWE-79) - use sanitize() instead of bypassSecurityTrustHtml()`)
- Build Jenkins #24 déclenché automatiquement via webhook et exécuté avec succès

## CWE associée

CWE-79 — Improper Neutralization of Input During Web Page Generation (Cross-site Scripting)
