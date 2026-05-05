# RUNNA → GARMIN

Outil web mobile-first qui convertit des screenshots de séances de musculation
[Runna](https://www.runna.com) en fichiers `.FIT` importables dans
[Garmin Connect](https://connect.garmin.com) (Workouts → Importer).

Tout tient dans un seul fichier `index.html`. Aucun build, aucun serveur,
aucun backend. À héberger tel quel sur GitHub Pages, Netlify, ou un serveur
statique.

## Stack

- HTML / CSS / JS vanilla, single-file
- [`@garmin/fitsdk`](https://www.npmjs.com/package/@garmin/fitsdk) chargé via
  [esm.sh](https://esm.sh) (importmap)
- API Anthropic appelée directement depuis le navigateur avec
  `anthropic-dangerous-direct-browser-access: true`
- Modèle : `claude-sonnet-4-5` (Vision)

## Utilisation

1. Ouvrir `index.html` (en local ou via GitHub Pages).
2. Coller une clé API Anthropic (`sk-ant-...`). Stockée en `localStorage`,
   jamais transmise ailleurs que vers `api.anthropic.com`.
3. Drop les screenshots Runna de la séance (souvent 2-3 par séance car ça
   scrolle). PNG, JPG ou WEBP.
4. **Analyser les screenshots**. Claude Sonnet 4.5 extrait la structure en JSON.
5. Vérifier / éditer la séance : nom, parties, répétitions, repos, exercices,
   ordre, supersets.
6. **Générer .FIT**. Le fichier se télécharge avec le slug du nom de séance.
7. Dans Garmin Connect : *Entraînement → Entraînements → Importer un
   entraînement*.

## Mapping Runna → Garmin

| Runna                                  | Modèle interne                       | FIT                                          |
|----------------------------------------|--------------------------------------|----------------------------------------------|
| Séance                                 | `workout`                            | `WORKOUT` (`subSport: strength_training`)    |
| Partie répétée N fois                  | `part.repeat`                        | `WORKOUT_STEP` `repeatUntilStepsCmplt` × N   |
| Exercice "30s"                         | `type: time, duration_sec`           | `WORKOUT_STEP` `durationType: time` (en ms)  |
| Exercice "12 répétitions"              | `type: reps, reps`                   | `WORKOUT_STEP` `durationType: reps`          |
| "15 rép. / côté"                       | `per_side: true`                     | Note `Par côté` + " (par côté)" dans le nom  |
| Repos entre exercices (implicite, 15s) | injecté entre 2 exos                 | `WORKOUT_STEP` `intensity: rest` 15s         |
| Repos entre séries                     | `part.rest_between_sets_sec`         | step `rest` ajouté avant la step `repeat`    |
| Superset                               | `is_superset_with_next` sur le 1er   | pas de step `rest` injectée derrière         |

Le `wktStepName` est tronqué à 32 caractères (limite pratique de la spec FIT).

## Pièges Garmin Connect

- Garmin Connect importe le `.FIT` mais peut renommer la séance. Le nom utile
  côté montre est `wktStepName` de chaque step.
- Les durées `time` doivent être en **millisecondes** dans la spec FIT — c'est
  géré automatiquement.

## Sécurité

La clé Anthropic est stockée en `localStorage` côté client. C'est acceptable
pour un usage personnel (la clé sert uniquement depuis votre navigateur).
**Ne jamais utiliser ce schéma pour une app multi-utilisateurs** : la clé est
visible côté client.

## Développement

Aucune dépendance à installer. Servir avec n'importe quel serveur statique
(important : un module ES doit être servi en HTTP, pas ouvert via `file://`) :

```bash
python3 -m http.server 8000
# puis ouvrir http://localhost:8000
```

## Licence

MIT.
