# Branche porteuse `wip-leximpact-plus-m`

Porte les correctifs de modèle de l'étude LexImpact [F10](https://git.leximpact.dev/leximpact/exploration/fiscalchemy) en attendant la fusion amont de [PR openfisca-france#2829](https://github.com/openfisca/openfisca-france/pull/2829) :

1. Seuil de recouvrement : annulation **strictement inférieure** à 61 € (BOI-IR-LIQ-20-20-40-20180704, § 60/80) — l'amont annule à `<=`.
2. Décote calculée sur le barème plafonné **arrondi** (BOI §50 : tout correctif est arrondi ; demi-euro compté pour 1).

Base : `wip-leximpact` (3bfe660e). **À supprimer** dès que la PR amont est fusionnée et que les consommateurs sont re-pinnés sur l'amont.
