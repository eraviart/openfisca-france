# Branche porteuse `wip-leximpact-plus-m`

Porte les correctifs de modèle de l'étude LexImpact [F10](https://git.leximpact.dev/leximpact/exploration/fiscalchemy).

**Cette branche est un artefact durable, pas une escale** (décision du 6 novembre 2026) : la revue de [PR openfisca-france#2829](https://github.com/openfisca/openfisca-france/pull/2829) n'est pas sous notre contrôle et peut ne jamais aboutir. Aucun travail du projet ne l'attend ; la branche est maintenue par nos soins. Correctifs portés :

1. Seuil de recouvrement : annulation **strictement inférieure** à 61 € (BOI-IR-LIQ-20-20-40-20180704, § 60/80) — l'amont annule à `<=`.
2. Décote calculée sur le barème plafonné **arrondi** (BOI §50 : tout correctif est arrondi ; demi-euro compté pour 1).

Base : `wip-leximpact` (3bfe660e).

## Maintenance

Quand `wip-leximpact` avance en amont, reporter les correctifs ci-dessus sur la nouvelle base et **vérifier par exécution**, jamais par `git apply --check` : un patch qui s'applique peut casser le code (constaté le 29/10/2026 — voir ci-dessous). L'outil du projet qui fait ce contrôle (copie jetable, cas de référence avant/après, verdict en code de sortie) est `tools/verifier_patch_porte.sh` dans le dépôt fiscalchemy.

Une fusion amont de #2829 reste un **gain possible**, jamais un préalable : elle supprimerait seulement cette branche.

## Adaptation à la lignée `wip-leximpact` (29/10/2026)

Le patch amont (#2829) a été écrit contre `c19c0b96`. Porté textuellement sur
`wip-leximpact` (3bfe660e), il corrompait deux formules (ligne de chargement
d'`ir_plaf_qf` remplacée par l'expression d'arrondi) et retirait l'import
`round_` encore utilisé par les abattements. Adaptation : rechargement puis
arrondi demi-euro supérieur à l'entrée de la décote et du gain, `round_`
conservé. Vérifié par exécution : `tests/impot_revenu/seuil_de_recouvrement_61.yaml`
6/6 (17 043 € → 61 € dus ; 17 037 € → 0 €).
