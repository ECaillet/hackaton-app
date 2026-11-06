# Hackathon app

Application de démonstration composée d'un front web, d'une API Node.js,
d'une base PostgreSQL et d'un appel à un modèle de langage via l'API Mistral.

## Prérequis

- Git
- Docker Desktop (Windows/macOS) ou Docker Engine avec le plugin Compose (Linux)
- Visual Studio Code et l'extension **Dev Containers**
- Une clé API Mistral pour tester les réponses IA

Le projet installe Node.js 24 et le client PostgreSQL dans le dev container.
La base PostgreSQL 17 est lancée par Compose.

## Démarrage (nouvel arrivant)

1. Cloner le dépôt et l'ouvrir dans VS Code :

   ```sh
   git clone <URL_DU_DEPOT>
   cd <DOSSIER_DU_DEPOT>
   code .
   ```

2. Dans VS Code, exécuter **Dev Containers: Reopen in Container**. La première
   création du conteneur installe les dépendances npm et crée `api/.env` depuis
   `api/.env.example`. Attendre la fin de cette étape.

3. Renseigner une clé Mistral valide dans `api/.env` en remplaçant
   `AI_API_KEY=votre-clé-mistral`. Ne pas partager ni commiter ce fichier :
   les fichiers `.env` sont ignorés par Git.

4. Dans un terminal VS Code ouvert **dans le conteneur**, lancer le front et
   l'API :

   ```sh
   npm run dev
   ```

5. Ouvrir <http://localhost:5173>. L'indicateur de santé doit afficher
   `API OK` et la version de PostgreSQL. Soumettre une question pour tester
   l'appel IA et l'enregistrement de la réponse.

La base est accessible depuis l'hôte sur `localhost:5433` (utilisateur/base
`app`). Depuis les services du Compose, elle est accessible sous le nom `db`
sur le port `5432`. Le schéma et les données de démonstration sont initialisés
au premier démarrage dans le répertoire de données local `db/data/`.

## Vérifications manuelles

Depuis le terminal du conteneur, avec `npm run dev` actif :

```sh
curl -fsS http://localhost:3000/api/health
curl -fsS http://localhost:3000/api/questions
curl -fsS -X POST http://localhost:3000/api/questions \
  -H 'Content-Type: application/json' \
  -d '{"question":"Réponds en un mot : quelle est la capitale de la France ?"}'
```

Le POST doit renvoyer une question enregistrée avec une réponse non vide. Cet
appel utilise la clé Mistral configurée et peut entraîner une facturation du
fournisseur.

## Qualité et build

À la racine du dépôt, dans le conteneur :

```sh
npm run lint
npm run format:check
npm run build
```

Le hook `pre-commit` lance `lint-staged` sur les fichiers préparés au commit.
La CI GitHub exécute ces trois contrôles sur chaque pull request.

## Choix techniques et périmètre

- **Node.js 24 et TypeScript** : même environnement pour l'API et les outils du
  front; le typage et la compilation détectent certaines erreurs avant
  l'exécution.
- **Express** : API HTTP légère, séparée du front.
- **Vite** : serveur de développement et build du client web; les requêtes
  `/api` sont transmises à l'API locale.
- **PostgreSQL 17** : stockage relationnel des questions et réponses, avec
  initialisation reproductible du schéma par Compose.
- **Mistral API (`mistral-small-latest`)** : fournisseur externe configuré par
  variables d'environnement, sans clé intégrée au code. L'API est appelée à
  chaque soumission. Les coûts dépendent du modèle, des tarifs en vigueur et
  du volume de tokens; consulter les [tarifs API Mistral](https://mistral.ai/pricing/).

**Budget :** aucun plafond de dépense n'est configuré dans le dépôt. Avant une
démonstration avec une clé réelle, le responsable doit fixer un budget et des
limites d'utilisation dans le compte fournisseur, puis surveiller la
consommation. Les tests de lint, format et build ne font pas d'appel IA.

**Hors périmètre actuel :** authentification des utilisateurs, limitation de
débit, RAG, déploiement de production et automatisation de tests fonctionnels.
Le service Python FastAPI/Ollama dans `ai/` est une expérimentation distincte :
il n'est pas lancé par Compose et n'est pas appelé par l'application web/API
principale, qui utilise directement Mistral.

## GitHub et exécution multi-machine

Les équipes, leurs permissions, la protection de `main`/`develop` et les checks
requis de fusion se configurent dans les paramètres GitHub de l'organisation et
du dépôt. Vérifier que les deux branches exigent une pull request et que le
check `qualite` est requis. Ces permissions ne sont pas définies par le code du
dépôt.

Le dev container est prévu pour les systèmes pris en charge par Docker et VS
Code Dev Containers. Pour valider le critère multi-machine, refaire le
démarrage complet sur deux machines ou architectures distinctes, puis vérifier
le front, `/api/health` et une soumission IA. Le dépôt ne contient pas de test
automatisé multi-architecture.
