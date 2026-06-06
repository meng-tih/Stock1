# Questions à poser à CAAS

## 1. Nommage de l'usage

Confirmer la règle officielle du `NAME` envoyé à la CD.

Exemples possibles :

```text
ma-dsd-sinbdo-domsin
ma_dsd_sinbdo_domsin
```

Règle actuellement prévue côté CD :

```text
domain      = dernier bloc du NAME
usage_short = avant-dernier bloc du NAME
```

Exemple :

```text
NAME        = ma-dsd-sinbdo-domsin
domain      = domsin
usage_short = sinbdo
```

La CD supporte deux séparateurs : `-` et `_`, avec auto-détection.

## 2. Image Docker disponible avant l'appel CD

Confirmer que l'image existe déjà avant l'appel à `v7_cd_usage_entrypoint` :

```text
dev     -> Artifactory scratch
int/rec -> Artifactory staging
```

Si l'image n'existe pas, le déploiement CD échoue et l'erreur doit remonter à la CI applicative CAAS.

## 3. Variables envoyées par les repoS CI applicatifs CAAS

Pour `dev`, variables obligatoires :

```text
TARGET_ENVIRONMENT=dev
NAME=<usage>
GROUP_ID=<groupId Maven / chemin Artifactory logique>
VERSION=<tag image Docker>
```

Pour `int` et `rec`, variables obligatoires :

```text
TARGET_ENVIRONMENT=int|rec
NAME=<usage>
```

Variables optionnelles pour `int` et `rec` :

```text
GROUP_ID=<surcharge éventuelle>
VERSION=<surcharge éventuelle>
```

Si `GROUP_ID` ou `VERSION` ne sont pas fournis en `int` / `rec`, la CD reprend les valeurs du manifest `dev.json`.

## 4. Repository Artifactory source

Confirmer que :

```text
dev     -> caas-shared-docker-scratch-intranet
int/rec -> caas-shared-docker-staging-intranet
```

## 5. Tag image

Confirmer que la variable `VERSION` correspond exactement au tag Docker à déployer.

Exemple :

```text
VERSION=0.0.15
image=<registry>/<group_path>/<NAME>:0.0.15
```

## 6. Pas de chaînage automatique

Confirmer que la CD ne doit pas enchaîner automatiquement :

```text
dev -> int -> rec
```

Chaque environnement doit être déclenché explicitement par la CI applicative CAAS ou par un lancement manuel GEA.

## 7. Chemin Artifactory

Confirmer que le chemin image est construit par transformation du `GROUP_ID` :

```text
GROUP_ID=fr.caa.systemeentreprise.pilotage.ma
GROUP_PATH=fr/caa/systemeentreprise/pilotage/ma
```

Image source finale :

```text
<registry>/<GROUP_PATH>/<NAME>:<VERSION>
```

## 8. Contrat de retour d'erreur

Confirmer que si `v7_cd_usage_entrypoint` ou `v7_cd_usage` échoue, la CI applicative CAAS doit être considérée en échec.
