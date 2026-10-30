# Branche porteuse `wip-leximpact-plus-m`

Porte les correctifs de modèle de l'étude LexImpact [F10](https://git.leximpact.dev/leximpact/exploration/fiscalchemy) en attendant la fusion amont de [PR openfisca-france#2829](https://github.com/openfisca/openfisca-france/pull/2829) :

1. Seuil de recouvrement : annulation **strictement inférieure** à 61 € (BOI-IR-LIQ-20-20-40-20180704, § 60/80) — l'amont annule à `<=`.
2. Décote calculée sur le barème plafonné **arrondi** (BOI §50 : tout correctif est arrondi ; demi-euro compté pour 1).

Base : `wip-leximpact` (3bfe660e). **À supprimer** dès que la PR amont est fusionnée et que les consommateurs sont re-pinnés sur l'amont.

## Adaptation à la lignée `wip-leximpact` (29/10/2026)

Le patch amont (#2829) a été écrit contre `c19c0b96`. Porté textuellement sur
`wip-leximpact` (3bfe660e), il corrompait deux formules (ligne de chargement
d'`ir_plaf_qf` remplacée par l'expression d'arrondi) et retirait l'import
`round_` encore utilisé par les abattements. Adaptation : rechargement puis
arrondi demi-euro supérieur à l'entrée de la décote et du gain, `round_`
conservé. Vérifié par exécution : `tests/impot_revenu/seuil_de_recouvrement_61.yaml`
6/6 (17 043 € → 61 € dus ; 17 037 € → 0 €).
