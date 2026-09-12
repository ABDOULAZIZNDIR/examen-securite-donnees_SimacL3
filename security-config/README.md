# Configuration des outils de sécurité

## OWASP Dependency-Check
- Installation Jenkins : DP-Check (via plugin OWASP Dependency-Check)
- Clé API NVD configurée pour les scans
- Arguments : `--scan . --format HTML --format XML --nvdApiKey <clé>`

## Gitleaks
- Image Docker utilisée : zricethezav/gitleaks:latest
- Exécution via `docker run` dans le pipeline
- Commande : `detect --source=/repo --report-path=/repo/gitleaks-report.json --no-git`

## npm audit
- Intégré nativement via Node.js/npm, aucune configuration supplémentaire requise
- Commande : `npm audit --json > npm-audit-report.json`

## Jenkins
- Outil NodeJS configuré : NodeJS 20 (Manage Jenkins → Tools)
- Client Docker CLI installé manuellement dans le conteneur Jenkins pour permettre l'exécution de `docker run` depuis les stages du pipeline
