\# Examen Final Pratique — Sécurité des Données

\*\*Licence 3 — Cybersécurité\*\*



Évaluation de sécurité de l'application OWASP Juice Shop : identification de vulnérabilités, analyse de risques (CIA), remédiation, et automatisation des contrôles de sécurité via un pipeline Jenkins.



\---



\## 📁 Structure du dépôt



```

project/

├── README.md                  # Ce fichier

├── Jenkinsfile                # Pipeline CI/CD de sécurité

├── reports/                   # Rapports générés par les outils d'analyse

├── screenshots/                # Captures d'écran (preuves)

├── security-config/           # Configuration des outils de sécurité

└── remediation/                # Fork corrigé + documentation des correctifs

```



\---



\## 🎯 Application cible



\*\*OWASP Juice Shop\*\* — application web volontairement vulnérable, utilisée à des fins pédagogiques.

\- Dépôt officiel : https://github.com/juice-shop/juice-shop

\- Fork utilisé pour ce projet (avec correctifs appliqués) : https://github.com/ABDOULAZIZNDIR/juice-shop



\---



\## 🚀 Comment exécuter le projet



\### Prérequis

\- Docker Desktop (ou Docker Engine) installé et démarré

\- Node.js 20+ (si exécution locale sans Docker)

\- Git



\### 1. Lancer l'application vulnérable (image officielle, non corrigée)



```bash

docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop

```



Accès : http://localhost:3000



\### 2. Lancer l'application corrigée (fork avec remédiations appliquées)



```bash

git clone https://github.com/ABDOULAZIZNDIR/juice-shop.git

cd juice-shop

npm install

npm start

```



Accès : http://localhost:3000 (arrêter d'abord le conteneur Docker ci-dessus pour libérer le port 3000)



\### 3. Lancer Jenkins (pour exécuter le pipeline de sécurité)



```bash

docker run -d --name jenkins \\

&#x20; -p 8080:8080 -p 50000:50000 \\

&#x20; -v jenkins\_home:/var/jenkins\_home \\

&#x20; -v /var/run/docker.sock:/var/run/docker.sock \\

&#x20; --privileged -u root \\

&#x20; jenkins/jenkins:lts-jdk17

```



Accès : http://localhost:8080



\---



\## 🔍 Comment lancer les analyses de sécurité



Le pipeline Jenkins (`Jenkinsfile`) automatise l'ensemble des contrôles. Il peut être déclenché de deux façons :



1\. \*\*Automatiquement\*\* : à chaque `git push` sur le dépôt fork, via un webhook GitHub configuré sur le job Jenkins `juice-shop-scan`.

2\. \*\*Manuellement\*\* : bouton "Build Now" depuis l'interface Jenkins.



\### Étapes du pipeline



| Étape | Description |

|---|---|

| 1. Checkout | Clonage du dépôt fork depuis GitHub |

| 2. Build / Preparation | `npm install` — installation des dépendances |

| 3. Security Analysis | `npm audit` — scan SCA rapide des dépendances npm |

| 4. Additional Security Check | OWASP Dependency-Check (SCA approfondi) + Gitleaks (Secret Detection) |

| 5. Report Generation | Archivage des rapports (`archiveArtifacts`) + publication HTML (`publishHTML`) |

| 6. Notification | Envoi automatique du rapport par e-mail (`emailext`) |



\### Lancer une analyse manuelle en local (hors Jenkins)



```bash

\# SCA — npm audit

npm audit --json > reports/npm-audit-report.json



\# SCA — OWASP Dependency-Check (nécessite l'outil installé localement ou via Docker)

dependency-check --scan . --format HTML --format XML --out reports/



\# Secret Detection — Gitleaks

docker run --rm -v $(pwd):/repo zricethezav/gitleaks:latest \\

&#x20; detect --source="/repo" --report-path="/repo/reports/gitleaks-report.json" --no-git

```



\---



\## 🛠️ Outils utilisés



| Outil | Catégorie | Rôle |

|---|---|---|

| \*\*npm audit\*\* | SCA | Analyse rapide des dépendances npm contre la base de vulnérabilités npm |

| \*\*OWASP Dependency-Check\*\* | SCA | Analyse approfondie croisée avec la base NVD, avec mapping CWE |

| \*\*Gitleaks\*\* | Secret Detection | Détection de secrets (clés API, mots de passe) codés en dur dans le code source |

| \*\*Jenkins\*\* | Orchestration CI/CD | Automatisation et déclenchement des scans à chaque modification du code |

| \*\*Docker / Docker Compose\*\* | Infrastructure | Isolation et déploiement de Juice Shop et Jenkins |



\---



\## 🔧 Vulnérabilités corrigées (voir `remediation/`)



| ID | Vulnérabilité | CWE | Statut |

|---|---|---|---|

| V1 | SQL Injection (login) | CWE-89 | ✅ Corrigée — requête paramétrée |

| V2 | DOM XSS (recherche) | CWE-79 | ✅ Corrigée — sanitization stricte |

| V3 | Hachage de mot de passe faible (MD5) | CWE-916 | ✅ Corrigée — migration vers bcrypt |



Détails complets (cause, correction, justification, vérification) disponibles dans `remediation/V1\_sqli\_login.md`, `remediation/V2\_xss\_search.md`, `remediation/V3\_weak\_password\_hashing.md`.



\---



\## ⚠️ Limites connues



\- Les correctifs de mot de passe (V3) s'appliquent uniquement aux \*\*nouveaux comptes\*\* créés après la migration. Les comptes préexistants (admin, jim, bender...) conservent leur hash MD5 d'origine et nécessiteraient une migration de données dédiée pour être pleinement sécurisés.

\- Le scan SCA révèle encore des vulnérabilités Critical/High dans les dépendances tierces (ex: `express-jwt`, `jsonwebtoken`, `lodash`), non corrigées dans le cadre de ce projet — voir le rapport de sécurité complet pour la décision de déploiement argumentée.



\---



\## 📄 Documentation complémentaire



Le rapport de sécurité complet (PDF), incluant l'analyse des vulnérabilités, l'analyse des risques (CIA), les remédiations détaillées, l'interprétation des résultats du pipeline et la décision de déploiement argumentée, est disponible séparément (livrable 1 de l'examen).



