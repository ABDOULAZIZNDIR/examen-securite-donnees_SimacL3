# V1 — SQL Injection (Login)

## 1️⃣ Cause

Dans `routes/login.ts`, la requête SQL était construite par concaténation directe des entrées utilisateur (`req.body.email`) dans la chaîne de requête, sans séparation entre code SQL et donnée :

```typescript
models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email}' AND password = '${security.hash(req.body.password)}' ...`)
```

Un attaquant pouvait injecter des méta-caractères SQL (`'`, `OR`, `--`) interprétés comme du code exécutable, permettant de contourner l'authentification.

## 2️⃣ Remédiation

Remplacement par une requête paramétrée utilisant les `replacements` de Sequelize :

```typescript
models.sequelize.query(
  'SELECT * FROM Users WHERE email = :email AND deletedAt IS NULL',
  { replacements: { email: req.body.email || '' }, model: UserModel, plain: true }
)
```

## 3️⃣ Justification

Avec des paramètres liés, la base de données traite toujours l'entrée comme une valeur de comparaison, jamais comme du code SQL exécutable — l'injection devient structurellement impossible, quel que soit le contenu envoyé.

## 4️⃣ Vérification

- **Avant correction** : test avec email = `' OR 1=1--` → connexion réussie (bypass de l'authentification)
- **Après correction** : même test → `401 Invalid email or password.`
- Commit poussé sur GitHub (`Fix: SQL Injection in login (CWE-89) - use parameterized query`)
- Build Jenkins #23 déclenché automatiquement via webhook et exécuté avec succès

## CWE associée

CWE-89 — Improper Neutralization of Special Elements used in an SQL Command
