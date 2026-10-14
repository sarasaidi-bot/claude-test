# Reproduire « Dashboard Recrutement » dans Looker Studio

Source : onglet `Dashboard Recrutement` du classeur *Pilotage Recrutement Maroc*.
Cet onglet est une page de synthèse : chaque chiffre est une formule sur les autres onglets. Looker Studio ne sait pas lire ces blocs de mise en page, il lui faut des **tables plates**. La méthode est donc : (1) créer 5 onglets `LS_*` plats dans le Google Sheet, (2) les brancher dans Looker Studio, (3) reconstruire la mise en page.

> Aucun onglet existant n'est modifié : on n'ajoute que des onglets `LS_*`.

---

## 1. Inventaire de la page (ce qu'il faut reproduire)

| # | Bloc | Cellules source | Contenu |
|---|------|-----------------|---------|
| A | En-tête | B1:B2 | Titre + « Mis à jour le … » |
| B | 6 KPI | B4:L6 | Atteinte objectif 2026, Reste à recruter, Candidats en process, En attente de 1re action, Acceptation des offres, Départs 2026 |
| C | Objectifs par chapter | B9:I14 | Objectif / Réalisé / Reste / Atteinte / Pipe / Préqualif / Couverture |
| D | Objectifs par grade | B17:I23 | Même colonnes, par grade + **graphique** colonnes groupées (Objectif, Réalisé, Pipe) |
| E | Entonnoir | B27:K31 | Suivis, E1, E2, Offres, Recrutés, Refusées, KO, Backlog, conversion E1→recruté + **graphique** (E1, E2, Recrutés par chapter) |
| F | Offres & acceptation | B36:J47 | Taux par chapter, par grade, par origine + **graphique** barres horizontales (taux par origine) ; 2 compteurs (refus pour salaire, ruptures) |
| G | Arrivées par mois | B51:C66 | **Graphique** colonnes |
| H | Plan d'intégration 2027 | B69:J73 | **Graphique** colonnes empilées par chapter |
| I | Rythme de recrutement | B76:F80 | H1 / H2 / plan 2027 / reste 2026 |
| J | Départs & rotation | B84:F90 | Par chapter |
| K | Sourcing | B94:L104 | Top 8 statuts × 4 onglets Sourcing |
| L | Contrôles de fiabilité | B109:I118 | Écarts + complétude |
| M | Lecture stratégique | B121:B128 | 8 phrases calculées |

Style d'origine : police Arial ; titres de tableau fond `#15132B` texte blanc ; cartes KPI fond `#F4F5F9` (libellé gris `#5B5773`) ; mise en forme conditionnelle verte `#E2F4EB` / ambre `#FFEFD5` / rose `#FAE6EB`.

---

## 2. Onglets plats à créer dans le Google Sheet

Créer 5 onglets vides, coller la formule **uniquement en A1** (ou A2 si indiqué). Plages étendues à la ligne 1000/5000 (le dashboard actuel s'arrête à la ligne 131).

### 2.1 `LS_Objectifs` — objectifs par chapter × grade
Ligne 1 (en-têtes, à saisir) : `Chapter | Grade | Objectif_avant | Departs | Objectif | Realise | Pipe | Prequalif`
En A2 :
```
={
 ARRAYFORMULA(REGEXREPLACE('Objectifs 2026 vs Pipe'!C4:C8&"","^.*$","Data Consulting")),   'Objectifs 2026 vs Pipe'!C4:C8,   'Objectifs 2026 vs Pipe'!D4:G8,   'Objectifs 2026 vs Pipe'!J4:K8;
 ARRAYFORMULA(REGEXREPLACE('Objectifs 2026 vs Pipe'!C13:C17&"","^.*$","Data Engineering")), 'Objectifs 2026 vs Pipe'!C13:C17, 'Objectifs 2026 vs Pipe'!D13:G17, 'Objectifs 2026 vs Pipe'!J13:K17;
 ARRAYFORMULA(REGEXREPLACE('Objectifs 2026 vs Pipe'!C22:C26&"","^.*$","Data Science")),     'Objectifs 2026 vs Pipe'!C22:C26, 'Objectifs 2026 vs Pipe'!D22:G26, 'Objectifs 2026 vs Pipe'!J22:K26;
 ARRAYFORMULA(REGEXREPLACE('Objectifs 2026 vs Pipe'!C31:C35&"","^.*$","Data Analytics")),    'Objectifs 2026 vs Pipe'!C31:C35, 'Objectifs 2026 vs Pipe'!D31:G35, 'Objectifs 2026 vs Pipe'!J31:K35
}
```
Colonnes source : C = grade, D = objectif avant ajustement, E = départs, F = objectif après ajustement, G = réalisé, J = pipe en process, K = préqualif. Le dashboard utilise F (« après ajustements »).

### 2.2 `LS_Pipeline` — entonnoir (DC + DS + DE empilés)
En-têtes : `Chapter | Candidat | Statut | Date_E1 | Date_E2 | Suivi | E1 | E2 | Offre | Recrute | Offre_refusee | KO | Backlog`
En A2 :
```
={
 FILTER({ARRAYFORMULA(REGEXREPLACE('Pipeline DC'!A2:A1000&"","^.*$","Data Consulting")),'Pipeline DC'!A2:A1000&" "&'Pipeline DC'!B2:B1000,'Pipeline DC'!C2:C1000,'Pipeline DC'!M2:M1000,'Pipeline DC'!O2:O1000},'Pipeline DC'!A2:A1000<>"");
 FILTER({ARRAYFORMULA(REGEXREPLACE('Pipeline DS'!A2:A1000&"","^.*$","Data Science")),'Pipeline DS'!A2:A1000&" "&'Pipeline DS'!B2:B1000,'Pipeline DS'!C2:C1000,'Pipeline DS'!M2:M1000,'Pipeline DS'!O2:O1000},'Pipeline DS'!A2:A1000<>"");
 FILTER({ARRAYFORMULA(REGEXREPLACE('Pipeline DE'!A2:A1000&"","^.*$","Data Engineering")),'Pipeline DE'!A2:A1000&" "&'Pipeline DE'!B2:B1000,'Pipeline DE'!C2:C1000,'Pipeline DE'!M2:M1000,'Pipeline DE'!O2:O1000},'Pipeline DE'!A2:A1000<>"")
}
```
Les 3 onglets Pipeline ont le même emplacement pour Statut (C), Date E1 (M), Date E2 (O).
Colonnes de drapeaux (une formule par colonne, en ligne 2) :

| Col | Formule |
|-----|---------|
| F Suivi | `=ARRAYFORMULA(IF(A2:A="",,1))` |
| G E1 | `=ARRAYFORMULA(IF(A2:A="",,--(D2:D<>"")))` |
| H E2 | `=ARRAYFORMULA(IF(A2:A="",,--(E2:E<>"")))` |
| I Offre | `=ARRAYFORMULA(IF(A2:A="",,--REGEXMATCH(C2:C&"","(?i)^(Offre\|Recruté)")))` |
| J Recrute | `=ARRAYFORMULA(IF(A2:A="",,--REGEXMATCH(C2:C&"","(?i)^Recruté")))` |
| K Offre_refusee | `=ARRAYFORMULA(IF(A2:A="",,--REGEXMATCH(C2:C&"","(?i)^Offre refusée")))` |
| L KO | `=ARRAYFORMULA(IF(A2:A="",,--REGEXMATCH(C2:C&"","(?i)^KO")))` |
| M Backlog | `=ARRAYFORMULA(IF(A2:A="",,--REGEXMATCH(C2:C&"","(?i)^(Préqualif à faire\|CV à Valider)")))` |

(Les `\|` sont à saisir `|` dans Sheets.) Le dashboard compte « CV à Valider » uniquement pour DS ; ici le motif s'applique aux 3 chapters, sans impact tant que DC/DE n'ont pas ce statut.

### 2.3 `LS_Offres` — offres acceptées/refusées (onglet Nouveaux arrivants)
En-têtes : `Candidat | Statut | Date_arrivee | Chapter | Grade | Origine | Motif | Acceptee | Refus_abandon | Rupture | Refus_salaire | Origine_groupe`
En A2 :
```
=FILTER({'Nouveaux arrivants'!D2:D5000,'Nouveaux arrivants'!G2:G5000,'Nouveaux arrivants'!H2:H5000,'Nouveaux arrivants'!I2:I5000,'Nouveaux arrivants'!K2:K5000,'Nouveaux arrivants'!L2:L5000,'Nouveaux arrivants'!N2:N5000},'Nouveaux arrivants'!G2:G5000<>"")
```
Drapeaux, ligne 2 :

| Col | Formule |
|-----|---------|
| H Acceptee | `=ARRAYFORMULA(IF(B2:B="",,--REGEXMATCH(B2:B,"(?i)^Offre accept")))` |
| I Refus_abandon | `=ARRAYFORMULA(IF(B2:B="",,--REGEXMATCH(B2:B,"(?i)^(Refus\|Abandon)")))` |
| J Rupture | `=ARRAYFORMULA(IF(B2:B="",,H2:H*REGEXMATCH(G2:G&"","(?i)^Rupture")))` |
| K Refus_salaire | `=ARRAYFORMULA(IF(B2:B="",,REGEXMATCH(B2:B,"(?i)^Refus")*REGEXMATCH(G2:G&"","(?i)^Salaire")))` |
| L Origine_groupe | `=ARRAYFORMULA(IF(B2:B="",,IFERROR(REGEXEXTRACT(F2:F&"","(?i)^(GH\|Chasse\|Cooptation\|Forum\|Hackaton\|Ancienne candidature\|Reco\|Linkedin)"),"Non renseignée")))` |

La date d'arrivée est parfois du texte (`dd/mm/yyyy`). Ajouter en M `Date_norm` :
`=ARRAYFORMULA(IF(B2:B="",,IF(ISNUMBER(C2:C),C2:C,IFERROR(DATEVALUE(C2:C),))))` puis formater la colonne en date. C'est M qui sert pour les arrivées par mois.

### 2.4 `LS_Plan2027` — plan d'intégration (format long)
En-têtes : `Chapter | Ordre | Mois | Nb`
En A2 :
```
={
 ARRAYFORMULA(SPLIT(TRANSPOSE("Data Consulting|"&(COLUMN('Objectifs de recrutement 2027'!C1:I1)-2)&"|"&'Objectifs de recrutement 2027'!C1:I1&"|"&'Objectifs de recrutement 2027'!C6:I6),"|"));
 ARRAYFORMULA(SPLIT(TRANSPOSE("Data Science|"&(COLUMN('Objectifs de recrutement 2027'!C1:I1)-2)&"|"&'Objectifs de recrutement 2027'!C1:I1&"|"&'Objectifs de recrutement 2027'!C11:I11),"|"));
 ARRAYFORMULA(SPLIT(TRANSPOSE("Data Engineering|"&(COLUMN('Objectifs de recrutement 2027'!C1:I1)-2)&"|"&'Objectifs de recrutement 2027'!C1:I1&"|"&'Objectifs de recrutement 2027'!C16:I16),"|"))
}
```
Lignes totaux utilisées : 6 (DC), 11 (DS), 16 (DE). `Ordre` sert à trier les mois.

### 2.5 `LS_Sourcing` — statuts des prospects
En-têtes : `Source | Id | Statut`
En A2 :
```
={
 FILTER({ARRAYFORMULA(REGEXREPLACE('Sourcing DC'!B2:B1000&"","^.*$","DC")),'Sourcing DC'!B2:B1000,'Sourcing DC'!H2:H1000},'Sourcing DC'!B2:B1000<>"");
 FILTER({ARRAYFORMULA(REGEXREPLACE('Sourcing DS'!A2:A1000&"","^.*$","DS")),'Sourcing DS'!A2:A1000,'Sourcing DS'!J2:J1000},'Sourcing DS'!A2:A1000<>"");
 FILTER({ARRAYFORMULA(REGEXREPLACE('Sourcing DE'!D2:D1000&"","^.*$","DE")),'Sourcing DE'!D2:D1000,'Sourcing DE'!F2:F1000},'Sourcing DE'!D2:D1000<>"");
 FILTER({ARRAYFORMULA(REGEXREPLACE('Sourcing Septembre 26'!D2:D1000&"","^.*$","Sept. 26")),'Sourcing Septembre 26'!D2:D1000,'Sourcing Septembre 26'!F2:F1000},'Sourcing Septembre 26'!D2:D1000<>"")
}
```

### 2.6 Petits tableaux repris tels quels (pas de transformation)
Pour Rythme, Départs, Contrôles et Lecture stratégique, les calculs sont déjà dans le dashboard : dans Looker Studio, créer la source avec l'option **« Plage »** (Google Sheets → onglet `Dashboard Recrutement` → cocher *Utiliser une plage*) :

| Source Looker | Plage | En-têtes |
|---------------|-------|----------|
| `Rythme` | `B76:F80` | ligne 1 de la plage |
| `Departs` | `B84:F88` | ligne 1 de la plage |
| `Controles` | `B109:E118` | ligne 1 de la plage |
| `Completude` | `G109:I116` | ligne 1 de la plage |
| `Lecture` | `B121:B128` | décocher « utiliser la 1re ligne comme en-têtes » |

Tant que le dashboard Excel/Sheets reste en place, ces 5 blocs restent justes sans recoder les formules (`TODAY()`, hypothèse de clôture fin novembre, etc.).

---

## 3. Sources de données Looker Studio

Ressources → *Gérer les sources de données* → Ajouter → **Google Sheets** → choisir le classeur → un onglet par source, « Utiliser la 1re ligne comme en-têtes » coché :
`LS_Objectifs`, `LS_Pipeline`, `LS_Offres`, `LS_Plan2027`, `LS_Sourcing` + les 5 plages du §2.6.

Types à vérifier à l'import : `Date_E1`, `Date_E2`, `Date_norm` → **Date** ; drapeaux et `Nb` → **Nombre** (agrégation *Somme*) ; le reste → Texte.
Ajouter un **filtre de rapport** `Chapter ≠ (vide)` sur `LS_Pipeline` et `LS_Offres`.

### Champs calculés (au niveau de la source)

`LS_Objectifs`
```
Grade_norm = CASE WHEN REGEXP_MATCH(Grade, "(?i).*(partner|directeur).*") THEN "Partners / Directeurs" ELSE Grade END
Reste      = SUM(Objectif) - SUM(Realise)
Atteinte   = SUM(Realise) / SUM(Objectif)
Couverture = SUM(Pipe) / (SUM(Objectif) - SUM(Realise))     -- afficher vide si Reste <= 0 : CASE WHEN SUM(Objectif)-SUM(Realise) > 0 THEN SUM(Pipe)/(SUM(Objectif)-SUM(Realise)) END
```
`LS_Pipeline`
```
Conv_E1_recrute = SUM(Recrute) / SUM(E1)
```
`LS_Offres`
```
Taux_acceptation = SUM(Acceptee) / (SUM(Acceptee) + SUM(Refus_abandon))
Mois_arrivee     = DATETIME_TRUNC(Date_norm, MONTH)      -- ou la dimension Date_norm en granularité « Mois »
```
Les libellés des KPI qui contiennent du texte dynamique se font avec des métriques texte, par exemple :
```
Sous_titre_atteinte = CONCAT(SUM(Realise), " recrutés sur ", SUM(Objectif), " postes")
Sous_titre_reste    = CONCAT("dont ", <Reste Senior>, " Seniors et ", <Reste Manager>, " Managers")
```
(pour ces deux-là : `SUM(CASE WHEN Grade_norm="Senior" THEN Objectif-Realise END)` etc.). Les afficher dans un **tableau sans en-tête**, ou en simple scorecard texte.

---

## 4. Reconstruction de la page

Rapport vierge, **page unique** personnalisée (largeur 1440 px, hauteur ≈ 4 300 px) qui reprend l'ordre du classeur. Thème : police Arial, fond blanc, titres de tableaux `#15132B`/blanc. Option plus lisible : 4 pages (Synthèse = A à D, Entonnoir & offres = E à G, Plan & rythme = H à J, Sourcing & fiabilité = K à M) ; le contenu reste identique.

**Contrôles (en haut)** : un seul filtre de période n'existe pas dans l'original ; n'en ajouter que si demandé.

| Bloc | Composant | Source | Dimension | Métriques |
|------|-----------|--------|-----------|-----------|
| A | 2 zones de texte | — | — | « Dashboard recrutement - Maroc » (Arial 22 gras) + « Mis à jour automatiquement » |
| B1 Atteinte | Scorecard, format `%` 1 décimale | LS_Objectifs | — | `Atteinte` ; sous-titre : `Sous_titre_atteinte` |
| B2 Reste | Scorecard | LS_Objectifs | — | `Reste` ; sous-titre `Sous_titre_reste` |
| B3 En process | Scorecard | LS_Objectifs | — | `SUM(Pipe)` ; sous-titre `SUM(Prequalif)` « candidats en préqualification » |
| B4 En attente | Scorecard | LS_Pipeline | — | `SUM(Backlog)` ; sous-titre « contre X en process » |
| B5 Acceptation | Scorecard `%` | LS_Offres | — | `Taux_acceptation` ; sous-titre `SUM(Acceptee)` « acceptées sur » `SUM(Acceptee)+SUM(Refus_abandon)` |
| B6 Départs | Scorecard | LS_Objectifs | — | `SUM(Departs)` ; sous-titre : part du volume = `SUM(Departs)/SUM(Realise)` |
| C | Tableau + ligne de total | LS_Objectifs | `Chapter` | `SUM(Objectif)`, `SUM(Realise)`, `Reste`, `Atteinte`, `SUM(Pipe)`, `SUM(Prequalif)`, `Couverture` |
| D | Tableau + total | LS_Objectifs | `Grade_norm` (tri : Partners/Directeurs, Manager, Senior, Junior, Stagiaire) | idem C |
| D′ | Colonnes groupées | LS_Objectifs | `Grade_norm` | `Objectif`, `Realise`, `Pipe` |
| E | Tableau + total | LS_Pipeline | `Chapter` | `SUM(Suivi)`, `SUM(E1)`, `SUM(E2)`, `SUM(Offre)`, `SUM(Recrute)`, `SUM(Offre_refusee)`, `SUM(KO)`, `SUM(Backlog)`, `Conv_E1_recrute` |
| E′ | Colonnes groupées | LS_Pipeline | `Chapter` | `SUM(E1)`, `SUM(E2)`, `SUM(Recrute)` |
| F1 | Tableau | LS_Offres | `Chapter` | `SUM(Acceptee)`, `SUM(Refus_abandon)`, `Taux_acceptation` |
| F2 | Tableau (tri : Stagiaire, Junior, Senior, Manager, Directeur) | LS_Offres | `Grade` | idem |
| F3 | Tableau | LS_Offres | `Origine_groupe` | idem |
| F3′ | Barres horizontales | LS_Offres | `Origine_groupe` | `Taux_acceptation` |
| F4 | 2 scorecards | LS_Offres | — | `SUM(Refus_salaire)` ; `SUM(Rupture)` |
| G | Colonnes | LS_Offres | `Date_norm` (mois), filtre `Acceptee = 1` | `SUM(Acceptee)` ; tri chronologique |
| H | Colonnes **empilées** | LS_Plan2027 | `Mois` (tri `Ordre` ↑) ; dimension de répartition `Chapter` | `SUM(Nb)` |
| H′ | Tableau pivoté | LS_Plan2027 | ligne `Chapter`, colonne `Mois` | `SUM(Nb)` |
| I | Tableau | Rythme | toutes colonnes | — |
| J | Tableau | Departs | toutes colonnes | — |
| K | 4 tableaux « Top 8 » côte à côte | LS_Sourcing | `Statut` (filtre `Source` = DC / DS / DE / Sept. 26) | `COUNT(Id)` ; tri décroissant ; 8 lignes |
| K′ | 4 scorecards « Total prospects » | LS_Sourcing | — | `COUNT(Id)` par Source |
| L | 2 tableaux | Controles, Completude | — | — |
| M | Tableau 1 colonne, retour à la ligne activé | Lecture | — | — |

### Mises en forme conditionnelles (Style → Règles)
- `Atteinte` et `Taux_acceptation` : ≥ 75 % vert `#E2F4EB` ; 60–74,99 % ambre `#FFEFD5` ; < 60 % rose `#FAE6EB`.
- `Couverture` : ≥ 3 vert ; 1,5–2,99 ambre ; < 1,5 rose.
- Sur les tableaux : en-tête fond `#15132B` texte blanc ; ligne de total en gras fond `#F4F5F9`.

---

## 5. Points d'attention relevés dans le classeur

1. **Plages tronquées** : les formules du dashboard s'arrêtent à la ligne 131 alors que les valeurs enregistrées (ex. 179 candidats DC) dépassent 130 lignes. Les onglets `LS_*` lisent jusqu'à 1000/5000 : les chiffres Looker peuvent donc légèrement différer du classeur tant que ces plages ne sont pas alignées. À trancher avant de valider les écarts.
2. **Écarts déjà signalés par le dashboard** (bloc Contrôles) : Recrutés Objectifs (78) vs H1+H2 (74) ; Départs Objectifs (2) vs onglet Turnover (0).
3. **Ruptures de période d'essai** : 10 offres acceptées marquées « Rupture » sont comptées à la fois en recrutés et en départs (voir B90).
4. **Dates en texte** dans `Nouveaux arrivants` (colonne H) : normalisées par `Date_norm`. Les en-têtes de mois du plan 2027 et d'arrivées sont parfois convertis en dates par Sheets ; c'est la raison de la colonne `Ordre`.
5. Les onglets *Kanban 2027* et *Ramp up 2030* ne sont pas utilisés par le dashboard, ils ne sont donc pas repris.

## 6. Ordre de réalisation recommandé

1. Créer les 5 onglets `LS_*` (§2.1 à 2.5) et vérifier que les totaux correspondent : Σ Objectif = 110, Σ Realise = 78, Σ Pipe = 74, Σ Acceptee = 78 (+ plan 2027 = 214).
2. Brancher les sources (§3), ajouter les champs calculés.
3. Construire d'abord les 6 KPI et les 2 tableaux Objectifs, comparer aux valeurs du classeur (`71 %`, `32`, `74`, `124`, `67 %`, `2`).
4. Ajouter graphiques, puis mises en forme conditionnelles.
