# Hackathon

## Stack choisie

- API : Node.js 24, Express et TypeScript.
- Front : Vite et TypeScript.
- IA : Mistral, via son API compatible OpenAI.

## Démarrer (3 étapes)

1. Installez Docker avec Compose, VS Code et l'extension **Dev Containers**, puis clonez ce dépôt :
   ```sh
   git clone https://github.com/ECaillet/hackaton-app.git
   ```
2. Ouvrez le dossier cloné dans VS Code et choisissez **Dev Containers: Reopen in Container**. À la création, le conteneur installe les dépendances (`npm ci`) et copie `api/.env.example` vers `api/.env`. Dans le terminal intégré, vérifiez `node -v`, puis lancez `npm run dev`. Ouvrez <http://localhost:5173>.
3. Ajoutez votre clé Mistral dans `api/.env` (`AI_API_KEY`), puis redémarrez l'API. L'application interroge `/api/health` et `/api/questions`; soumettre une question appelle l'IA. Ne partagez pas et ne commitez pas ce fichier `.env`.

## Base de données

PostgreSQL démarre avec la stack. Le schéma et les données de test de `db/init/` sont appliqués à la première initialisation, et les données persistantes sont dans `db/data/`. Depuis l'hôte, `psql` ou TablePlus peut se connecter à `localhost:5432` avec la base `app`, l'utilisateur `app` et le mot de passe `app`. Depuis le conteneur `dev`, utilisez l'hôte `db`.

L'API IA est compatible avec les fournisseurs au format OpenAI : pour utiliser un autre fournisseur, modifiez `AI_BASE_URL`, `AI_MODEL` et `AI_API_KEY` dans `api/.env`.
