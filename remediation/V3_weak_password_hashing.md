# V3 — Hachage de mot de passe faible (MD5 sans sel)

## 1️⃣ Cause

Dans `models/user.ts`, les mots de passe étaient hashés avec `crypto.createHash('md5')`, un algorithme rapide et non salé, conçu pour l'intégrité de données et non pour la protection de mots de passe — cassable en quelques secondes via des rainbow tables ou du brute-force GPU :

```typescript
this.setDataValue('password', security.hash(clearTextPassword))
```

## 2️⃣ Remédiation

Création de deux fonctions dédiées dans `lib/insecurity.ts`, basées sur bcrypt :

```typescript
import bcrypt from 'bcrypt'

export const hashPassword = (password: string) => bcrypt.hashSync(password, 10)
export const comparePassword = (password: string, hashedPassword: string) => bcrypt.compareSync(password, hashedPassword)
```

La fonction `hash()` générique originale (MD5) a été **conservée intacte** pour ses autres usages non liés aux mots de passe (2FA, changement de mot de passe, dataExport, génération d'ID de commande), afin de ne pas casser le reste de l'application qui en dépend.

Modifications apportées :
- `models/user.ts` (ligne 76) : utilise désormais `security.hashPassword()` à la création du compte
- `routes/login.ts` : utilise désormais `security.comparePassword()` pour vérifier le mot de passe lors de la connexion, au lieu de comparer un hash directement dans la requête SQL

## 3️⃣ Justification

bcrypt intègre un sel aléatoire unique par mot de passe et un facteur de coût réglable (ici 10), rendant chaque hash unique et le brute-force nettement plus lent, contrairement à MD5 qui produit toujours le même hash pour une même entrée et ne dispose d'aucune protection contre les rainbow tables.

## 4️⃣ Vérification

- Création d'un nouveau compte de test (`test-bcrypt@test.com`) sur l'application corrigée
- Connexion réussie avec ce compte, confirmant que le cycle complet inscription (`hashPassword`) → connexion (`comparePassword`) fonctionne correctement
- Commit poussé sur GitHub (`Fix: Weak password hashing (CWE-916) - migrate to bcrypt for new passwords`)
- Nouveau build Jenkins déclenché automatiquement via webhook

## Limite connue

Cette correction s'applique uniquement aux **nouveaux mots de passe** créés après la migration. Les comptes préexistants (admin, jim, bender...) conservent leur hash MD5 d'origine et nécessiteraient une migration de données dédiée (ex : re-hash automatique au prochain login réussi) pour être pleinement sécurisés — un sujet identifié comme axe d'amélioration futur, hors du périmètre de ce correctif ponctuel.

## CWE associée

CWE-916 — Use of Password Hash With Insufficient Computational Effort
