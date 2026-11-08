# Branche porteuse `wip-leximpact-plus-m`

Porte les correctifs de modèle de l'étude LexImpact [F10](https://git.leximpact.dev/leximpact/exploration/fiscalchemy).

**Cette branche est un artefact durable, pas une escale** (décision du 6 novembre 2026) : la revue de [PR openfisca-france#2829](https://github.com/openfisca/openfisca-france/pull/2829) n'est pas sous notre contrôle et peut ne jamais aboutir. Aucun travail du projet ne l'attend ; la branche est maintenue par nos soins. Correctifs portés :

1. Seuil de recouvrement : annulation **strictement inférieure** à 61 € (BOI-IR-LIQ-20-20-40-20180704, § 60/80) — l'amont annule à `<=`.
2. Décote calculée sur le barème plafonné **arrondi** (BOI §50 : tout correctif est arrondi ; demi-euro compté pour 1).
3. Double retrait de l'abattement des pensions réparé (8 novembre 2026, ticket fiscalchemy #141) — voir l'entrée datée ci-dessous.

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

## Double retrait de l'abattement des pensions (08/11/2026)

**Problème** : tout foyer avec pensions calculé par cette branche voyait son
revenu net imposable amputé d'un second abattement de 10 % des pensions
(témoin #138 : rni retraite 46200 au lieu de 48600 sur salaire 30 000 € +
pension imposable 24 000 €, revenus 2020).

**Cause** : commit `3a7b45e38` (« crée une variable d'abattement pour les
pensions », branche `refacto_super_brut_to_disponible`, 7 octobre 2026) —
importé aplati dans `wip-leximpact` (3bfe660e, 9 octobre 2026) puis hérité ici.
La formule `revenu_categoriel_tspr` (`ir.py`) retranchait
`abattement_pensions_retraites` **deux fois** (lectures dupliquées :675/:677,
soustractions :687-688). Le portage F10 n'y est pour rien.

**Correctif** (`7f914e276`) : un seul retrait sur une lecture unique ; la
sémantique du modèle épinglé `c19c0b96` est restaurée sur cette zone, rien
d'autre n'est touché.

**Preuve** (outils et témoins dans le dépôt fiscalchemy, branche
`comparaison-corpus`) : témoin pensions re-exécuté (46200 → 48600, égalité
avec l'épinglé, variables intermédiaires inchangées) ;
`tools/verifier_patch_porte.sh` ACCEPTE_AVEC_ECART sur le diff appliqué à
`11dfa4cbc` — les 2 écarts sont la réparation attendue (`tests/formulas/irpp.yaml`
1 failed → 119 passed ; cas joint témoin rouge → vert) et le cas sans pension
`seuil_de_recouvrement_61.yaml` reste identique (6 passed) ; témoin décote 056
re-exécuté, inchangé (−518 = M aux deux entrées) ; panel #139 re-exécuté sous
verrou sur le modèle corrigé après correctif : 18 `coherent` + 1 référence +
2 `excluded_cehr`, les deux cas retraite restent `coherent` (Δ RNI M/OF
identiques) et les 3 `divergent_arrondi_decote` de #139 deviennent `coherent`
(réparation de la famille #2829 portée ici, exécutée en propre).

**Signalement** : le même défaut vit dans `wip-leximpact` (branche de base de
la lignée, tips identiques côté fork `eraviart` et côté `openfisca` amont,
commit `3bfe660e8`) — non corrigé, décision laissée aux mainteneurs de cette
lignée.
