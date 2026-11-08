# Branche porteuse `wip-leximpact-plus-m`

Porte les correctifs de modèle de l'étude LexImpact [F10](https://git.leximpact.dev/leximpact/exploration/fiscalchemy).

**Topologie depuis le 8 novembre 2026 (ticket fiscalchemy #148)** : les correctifs
F10 vivent dans la branche de base elle-même — `wip-leximpact` (tip `ac5f0e93a`,
fork `eraviart`) = lignée amont `0ede737f7` (correctif pensions, PR
openfisca-france#2832) + seuil 61 € strict + décote arrondie. Cette branche
`wip-leximpact-plus-m` ne rajoute plus que le journal `CARRIER.md` (aucune
divergence de modèle) ; un exemplaire unique de chaque correctif subsiste dans
la lignée.

**Historique — correctifs portés par cette branche avant #148** (transférés
dans `wip-leximpact` le 8 novembre 2026, voir le journal en fin de fichier) :

1. Seuil de recouvrement : annulation **strictement inférieure** à 61 € (BOI-IR-LIQ-20-20-40-20180704, § 60/80) — l'amont annule à `<=`.
2. Décote calculée sur le barème plafonné **arrondi** (BOI §50 : tout correctif est arrondi ; demi-euro compté pour 1).
3. Double retrait de l'abattement des pensions réparé (8 novembre 2026, ticket fiscalchemy #141) — voir l'entrée datée ci-dessous.

Base : `wip-leximpact` (`ac5f0e93a`, porte désormais les trois correctifs ci-dessus).

**Divergence fork/org (signalée, non traitée)** : depuis #148, `wip-leximpact`
côté fork `eraviart` (`ac5f0e93a`) porte les correctifs F10, tandis que la
branche du même nom côté org `openfisca` (`0ede737f7`) ne les porte pas. La
coordination de cette divergence relève des mainteneurs de la lignée (sujet
humain) ; aucune action d'agent sur l'org amont.

## Maintenance

Quand `wip-leximpact` avance (amont ou fork), reporter les correctifs ci-dessus
sur la nouvelle base et **vérifier par exécution**, jamais par `git apply --check` : un patch qui s'applique peut casser le code (constaté le 29/10/2026 — voir ci-dessous). L'outil du projet qui fait ce contrôle (copie jetable, cas de référence avant/après, verdict en code de sortie) est `tools/verifier_patch_porte.sh` dans le dépôt fiscalchemy.

Une fusion amont de #2829 reste un **gain possible**, jamais un préalable.
(Historique : quand cette branche portait les correctifs, une telle fusion
aurait simplement supprimé la branche ; depuis #148, elle ne subsiste que pour
ce journal et l'amont n'est plus en jeu.)

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

**Erratum du 8 novembre 2026 (#148)** : ce défaut est réparé dans la lignée
amont elle-même — la branche `wip-leximpact` côté org (`openfisca/openfisca-france`)
a intégré la réparation par [PR #2832](https://github.com/openfisca/openfisca-france/pull/2832)
(`bc3c26679` → `0ede737f7`, « Réparer le double retrait de l'abattement des
pensions dans revenu_categoriel_tspr »), patch identique à `7f914e276` près
l'ordre de deux lectures pures adjacentes. `wip-leximpact` côté fork l'hérite
depuis le portage #148.

## Synchronisation sur la lignée amont (08/11/2026, ticket fiscalchemy #148)

Décision utilisateur : la lignée LexImpact doit porter elle-même les correctifs
F10. Exécuté :

- `wip-leximpact` (fork) : `3bfe660e8` → `ac5f0e93a` — avance rapide sur la
  lignée org (`0ede737f7`, correctif pensions #2832) puis portage des correctifs
  F10 (seuil 61 € strict + décote arrondie, forme de `75fcd764b` : recharge
  `ir_plaf_qf` puis arrondi, import `round_` conservé) avec son test
  `tests/impot_revenu/seuil_de_recouvrement_61.yaml`.
- Cette branche : rebase résolu sur le nouveau tip — les trois correctifs ont
  été retirés du compte propre (`8299709fa`, `75fcd764b`, `7f914e276` : déjà
  dans la base, un seul exemplaire dans la lignée) ; ne restent que la
  documentation (`8084948ec`, `94437f428`, ce journal) puis poussée fork avec
  `--force-with-lease` (branche à nous, révision changée).
- **Équivalence d'arbre** : `ir.py` du tip `94437f428` est identique au blob
  près à celui de `ac5f0e93a` ; par rapport à l'ancien tip `fd50f38ae`, la seule
  différence est l'ordre de deux lectures pures adjacentes dans
  `revenu_categoriel_tspr` (`revenu_assimile_pension` avant ou après la lecture
  de l'abattement — ordre du PR amont #2832, aucune dépendance mutuelle,
  sémantique identique). Les zones F10 et pensions sont identiques octet pour
  octet.
- **Vérification par exécution** (worktree dédié `openfisca-france-plus-m-f10`,
  venv propre, sur le tip `ac5f0e93a` ; témoins et scripts du dépôt fiscalchemy,
  branche `comparaison-corpus`) : `seuil_de_recouvrement_61.yaml` 6 passed ;
  `tests/formulas/irpp.yaml` 119 passed ; témoin décote 056 IRN −518 = M aux
  deux entrées ; témoin pensions rni 48600. Témoins re-exécutés à l'identique
  sur cette branche après synchronisation (révision du témoin mise à jour,
  erratum cité dans le dépôt fiscalchemy).
- Suites de mesures du chantier non refaites : aucun verdict n'a changé, et
  l'équivalence d'arbre ci-dessus couvre le code exécuté.
