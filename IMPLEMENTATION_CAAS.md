# Ce que CAAS doit implémenter dans les repoS CI applicatifs

## Principe

Les repoS CI applicatifs CAAS doivent déclencher le pipeline GitLab du repo `v7_cd_usage_entrypoint` avec les variables nécessaires.

Ils ne doivent pas modifier les fichiers `params.json`. Ces fichiers sont maintenus manuellement par GEA.

## Pré-requis côté CAAS

Avant d'appeler la CD :

```text
dev     : l'image doit exister dans Artifactory scratch
int/rec : l'image doit exister dans Artifactory staging
```

Le dépôt CD ne fait pas de promote d'image.

## Exemple de job GitLab côté CAAS

```yaml
trigger_cd_usage:
  stage: deploy
  image: curlimages/curl:latest
  script:
    - |
      curl --fail --request POST \
        --form token="${V7_CD_USAGE_ENTRYPOINT_TRIGGER_TOKEN}" \
        --form ref="${V7_CD_USAGE_ENTRYPOINT_REF:-main}" \
        --form "variables[TARGET_ENVIRONMENT]=${TARGET_ENVIRONMENT}" \
        --form "variables[NAME]=${NAME}" \
        --form "variables[GROUP_ID]=${GROUP_ID}" \
        --form "variables[VERSION]=${VERSION}" \
        "${CI_API_V4_URL}/projects/${V7_CD_USAGE_ENTRYPOINT_PROJECT_ID}/trigger/pipeline"
```

## Appel pour dev

```text
TARGET_ENVIRONMENT=dev
NAME=ma-dsd-sinbdo-domsin
GROUP_ID=fr.caa.systemeentreprise.pilotage.ma
VERSION=0.0.15
```

## Appel pour int

Minimum :

```text
TARGET_ENVIRONMENT=int
NAME=ma-dsd-sinbdo-domsin
```

Avec surcharge possible :

```text
TARGET_ENVIRONMENT=int
NAME=ma-dsd-sinbdo-domsin
VERSION=0.0.16
```

## Appel pour rec

Minimum :

```text
TARGET_ENVIRONMENT=rec
NAME=ma-dsd-sinbdo-domsin
```

Avec surcharge possible :

```text
TARGET_ENVIRONMENT=rec
NAME=ma-dsd-sinbdo-domsin
GROUP_ID=fr.caa.systemeentreprise.pilotage.ma
VERSION=0.0.16
```

## Erreur attendue si l'usage n'est pas référencé par GEA

Si `usages/<NAME>/params.json` n'existe pas dans `v7_cd_usage_entrypoint`, le pipeline échoue.

Message attendu :

```text
Usage non référencé par GEA
```

## Erreur attendue si le project_id est absent

Si `params.json` ne contient pas l'environnement demandé, le pipeline échoue.

Exemple :

```text
Aucun project_id trouvé pour env=rec dans usages/<NAME>/params.json
```
