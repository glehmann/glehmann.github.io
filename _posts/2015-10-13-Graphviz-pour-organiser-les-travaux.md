---
title: Graphviz pour organiser les travaux
---

Changement de maison ! Et plein de travaux à faire, et à planifier.

La planification s'est avérée moins évidente que je ne le pensais, entre notre
désir de se ménager un espace habitable le plus agréable possible le plus
rapidement possible, et les nécessités techniques.

On était un peu perdu avec notre papier et notre crayon. Heureusement, graphviz
nous a permis de simplement représenter les dépendances et de se rendre compte
de ce qui devait impérativement être fait d'abord !

~~~~
digraph travaux_sibiril {

tableau_electrique
electricite_salon
electricite_bureau
electricite_cuisine
electricite_chambre_parents
electricite_chambre_enfants
electricite_chambre_amis
casser_cloison
monter_cuisine
chauffe_eau_solaire
demonter_chauffe_eau
demonter_isolation
velux
refaire_isolation
finition_salon
monter_cloison
casser_cheminee
monter_poele
finition_chambre_parents
finition_chambre_enfants
finition_chambre_amis
finition_bureau
parquet_cuisine

tableau_electrique->electricite_salon
tableau_electrique->electricite_bureau
tableau_electrique->electricite_cuisine
tableau_electrique->electricite_chambre_parents
tableau_electrique->electricite_chambre_enfants
tableau_electrique->electricite_chambre_amis
demonter_isolation->velux
demonter_isolation->refaire_isolation
demonter_isolation->electricite_chambre_parents
refaire_isolation->electricite_chambre_enfants
refaire_isolation->electricite_chambre_amis
electricite_chambre_parents->finition_chambre_parents
/*casser_cloison->monter_cloison*/
casser_cloison->parquet_cuisine
monter_cloison->parquet_cuisine
chauffe_eau_solaire->demonter_chauffe_eau
demonter_chauffe_eau->monter_cuisine
finition_salon->monter_cloison
monter_cloison->monter_poele
casser_cheminee->monter_poele
casser_cheminee->finition_chambre_parents
electricite_cuisine->monter_cuisine
electricite_bureau->finition_bureau
electricite_chambre_enfants->finition_chambre_enfants
electricite_chambre_amis->finition_chambre_amis
monter_cloison->electricite_chambre_parents
parquet_cuisine->monter_poele
}
~~~~

![Travaux]({{ site.url }}/images/travaux.png)
