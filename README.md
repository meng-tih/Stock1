# Kit CI/CD usages GCP

Ce zip contient deux squelettes de dépôts Git :

- `v7_cd_usage_entrypoint` : repo d'orchestration CD appelé par les repoS CI applicatifs CAAS.
- `v7_cd_usage` : repo de déploiement GCP, basé sur Terraform, Artifactory, GAR/GCR et Cloud Run Jobs.

Modèle retenu :

```text
repoS CI applicatifs CAAS
  -> v7_cd_usage_entrypoint
    -> v7_cd_usage
      -> GCP project cible
```

Règles retenues :

- environnements supportés : `dev`, `int`, `rec` ;
- pas de `pprd` / `prd` ;
- pas de promote dans ces dépôts ;
- pas de scan image ;
- `dev` prend l'image depuis le repository Artifactory `scratch` ;
- `int` et `rec` prennent l'image depuis le repository Artifactory `staging` ;
- `int` et `rec` reprennent par défaut le `dev.json`, puis appliquent les variables surchargées par la CI si elles sont fournies ;
- l'usage doit être référencé par GEA avant tout déploiement : présence de `usages/<NAME>/params.json` dans `v7_cd_usage_entrypoint`.

Les valeurs marquées `TODO` sont à adapter à vos conventions internes.


Points d'attentions :

- Le Token GIT_PUSH_TOKEN doit être mise à jour régulièrement avec expiration du GIT_PUSH_TOKEN_V7

- Ajouter le service principal manuellement au SA iac-wif-usage-sa
gcloud iam service-accounts add-iam-policy-binding \
  iac-wif-usage-sa@caa-cicd-usage-hprd-d6.iam.gserviceaccount.com \
  --project=caa-cicd-usage-hprd-d6 \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/793183347341/locations/global/workloadIdentityPools/gitlab-usage-wip/attribute.project_path/gea/sid/cd/centralisationinformation/collectedonnee/v7/v7_cd_usage"

- Operation Destroy, paramètres :
ACTION=destroy
NAME=ma-dsd-sinbdo-domsin
TARGET_ENVIRONMENT=dev
TARGET_PROJECT=caa-data-domsin-dev-e7

