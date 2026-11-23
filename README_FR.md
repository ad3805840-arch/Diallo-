# Déploiement AutoRig pour le studio 2D

Tous les fichiers nécessaires au dépôt GitHub sont à la racine de cette archive. Téléverse tous les fichiers et le fichier caché `.dockerignore` à la racine du dépôt. Le fichier `studio_2d_autorig-8.html` est la version du studio corrigée à conserver sur ton téléphone.

Ce dossier contient le service Node.js pour le studio. Il appelle le CLI officiel Rigby2D 0.6.0 pour traiter un PSD en couches, produit un fichier `.rigby`, ajoute deux cycles de base (`idle` et `walk`) avec des clés réelles sur les os reconnus, exporte le squelette JSON + atlas + texture, vérifie que les deux animations contiennent des pistes, puis renvoie une archive ZIP au studio.

## Ce que le service attend

- Node.js 20+ dans le conteneur (fourni par le Dockerfile).
- Un PSD Photoshop réellement en couches, avec les éléments du personnage sur des calques séparés et des noms de calques permettant de reconnaître les côtés gauche/droit et les cuisses.
- Un jeton `API_TOKEN` configuré par l'hébergeur.

## Déploiement

Le projet est prêt pour un hébergeur Node/Docker compatible avec `render.yaml` :

1. Créer un dépôt GitHub et téléverser les fichiers de cette archive directement à la racine du dépôt (ne pas les laisser dans un sous-dossier).
2. Créer un nouveau service Web depuis ce dépôt en choisissant le déploiement Docker / Blueprint.
3. Laisser l'hébergeur générer `API_TOKEN` automatiquement.
4. Attendre la fin du build : il clone Rigby2D v0.6.0 et installe ses dépendances.
5. Vérifier `https://ADRESSE-DU-SERVICE/health`. Le résultat attendu est `ok: true`, `autorig: true`, `rigbyVersion: "0.6.0"`.
6. Dans le studio, saisir l'adresse HTTPS du service et le jeton généré, puis appuyer sur « Enregistrer le service » et « Tester la connexion ».

L'adresse HTTPS n'est volontairement pas inventée ici : elle n'existe qu'après le déploiement sur ton compte d'hébergement.

## Contrat API attendu par le studio

- `GET /health` renvoie l'état du moteur et la version.
- `POST /api/autorig` reçoit `multipart/form-data` avec `artwork` (PSD) et `characterName`, et attend `Authorization: Bearer <API_TOKEN>`.
- Réponse : ZIP contenant `name.json`, `name.atlas` et `name.png` (ou autre page texture reconnue).

## Limites réelles

AutoRig ne découpe pas une image aplatie : il faut un PSD en couches. AutoRig construit le squelette, les meshes et l'IK. Le service ajoute ensuite seulement deux cycles de départ (`idle` et `walk`) ; la marche générée est une base automatique, pas une animation professionnelle garantie. Les courses, gestes, expressions, mouvements de bouche et animations complexes ne sont pas créés par ce service. Si les os gauche/droite ne sont pas reconnus, le service arrête le traitement et explique l'erreur au lieu de renvoyer un faux résultat. Un vrai PSD de test reste nécessaire pour valider le résultat visuel.

## Contrôles de sécurité

- Taille PSD limitée à 25 Mo.
- Vérification de la signature binaire PSD (`8BPS`).
- Jeton Bearer obligatoire pour l'envoi des fichiers.
- Une seule opération AutoRig à la fois.
- Répertoire temporaire supprimé après chaque tâche.
- Le service accepte les requêtes CORS sans cookies afin de fonctionner depuis les pages Android `content://` ; l'API reste protégée par le jeton.
