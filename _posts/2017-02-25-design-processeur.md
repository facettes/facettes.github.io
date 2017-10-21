---
layout: post
title:  "Design de Processeur : clône d'un atmega 8 en VHDL"
categories: [informatique]
comments: true
image :
  feature: proz_design/diagramme_preview.png
---

J'ai réalisé ce projet dans le cadre de mon semestre en Allemagne à Jena _(ou bien Iéna, commen on dit en France)_ en Automne 2016. Au programme : du VHDL à tout va !

<!--more-->

# Informations générales sur le projet

Le code est disponible sur [Github][pd_gh].

Technologies utilisées :

- FPGA Xilinx Artix V7 - prêt gracieux de la Fachhochschule de Jena.
- Vivado - comme plateforme de développement.
- Emacs - ou plutôt devrais-je dire Evil Emacs, hein.
- Git - _As my lord & savior_

# Mon processeur

Sans transition, voici un beau petit schéma de "la bête" :
![schéma du processeur](/img/proz_design/diagramme.png)

Un certain nombre d'instructions ont été implémentées :

- Opérations de calcul
    - addition, soustraction, comparaison, entre deux registres (̀`ADD`) ou entre un registre et une constante (`ADDI` pour _add immediate_)
    - Incrémentation, décrémentation...
    - opérations logiques
    - Logical Shifts
- Opérations de mémoire
    - Lecture / Ecriture dans une case de la mémoire
    - Push / Pop dans la pile.
- Opérations de saut
    - Sauts et branchements conditionnels
    - appel de "fonction"

## Exemple de fonctionnement d'une instruction

Alors concrètement, comment ça fonctionne ? Bien, je dirais. Mais encore ? Regardons celà de plus près avec le schéma suivant.

![schéma du processeur](/img/proz_design/diagramme_add_addi_ldi.png)

Commençons par suivre les aventures d'une instructions `ADD` (en jaunâtre).

- Avec la valeur du `Program Counter`, le `Program Memory` récupère l'instruction (disons `ADD R1, R2`) et l'envoie au décoder.
- Le décoder se charge de correctement régler les différents multiplexeur. Il lit l'instruction, envoie ̀`addrA = 1` et `addrB = 2` au `register file` et `OPCODE = ADD ` à l'ALU.
- l'ALU reçoit le contenus des registres 1 et 2 à travers les signaux `dataA` et ̀`dataB`. Il renvoit la valeur de `dataA + dataB`
- Le register file reçoit cette valeur et l'enregistre dans `R1`.
- l'ALU envoie également le masque du SREG, qui sera mis à jour si nécessaire


## Ecriture dans la mémoire et pile

La mémoire est gérée par le composant ̀`decoder_memory`. Ce dernier prend en entrée ̀`index_z`, concaténation des registres 31 et 30, pointant vers l'espace mémoire qui sera affecté.

Conformément à la documentation, la mémoire RAM commence à l'addresse 0x0060. L'addresse des pins / ports sont détaillés dans la documentation du _repository_.

Les pins et ports sont déclarés comme des instances du composant générique `ports` et sont reliés directement aux leds et aux switches au niveau hardware.

La mémoire RAM est un composant classique de RAM de type _distributed_ (voir section sur le pipeline à 3 niveaux pour plus d'informations).

`decoder_memory` gère également le pointeur de pile et les instructions push et pop sont illustrées par le schéma ci-dessous :

![schéma du processeur](/img/proz_design/diagramme_push_pop.png)

## Les sauts

### Branchement classique et conditionnels

Les branchements fonctionnent très simplement de la façon suivante :

- le decoder lit la valeur du saut qu'il doit apppliquer (saut relatif) et l'envoie à travers le signal ̀`offset_pc` au Program Counter.
- Le program counteur prend la valeur de `program_counter + offset_pc`.

En cas de branchement conditionnel, le décodeur vérifiera d'abord la valeur du drapeau voulu dans le SREG afin de choisir si oui ou non le branchement doit être effecté.

![schéma du processeur](/img/proz_design/diagramme_rjump_branches.png)


### Appels de fonction

Les instructions d'appel de fonction ont un fonctionnement légèrement plus compliqué que les autres. En effet, elles doivent s'exécuter sur 2 cycles : l'addresse de retour de la fonction doit être stockée sur la pile et donc sur 2 octets (le programme counter a une taille de 11 bits).

Le décodeur envoie un signal `two_cycle` au Program memory afin que ce dernier reste figé un cycle de plus, pendant que le Program counter change de valeur.

Le fonctionnement général de ces instructions est montré par le schéma ci-dessous :

![schéma du processeur](/img/proz_design/diagramme_ret_rcall.png)

## Pipelines


Un pipeline deux niveau a été implémenté (Fetch - Decode), comme montré par le schéma ci- dessous :
![schéma du processeur](/img/proz_design/diagramme_first_stage_pipeline.png)

### Pipelines à trois niveaux ?

Le pipeline à trois niveaux était au programme et aurait dû être implémenté, mais, faute de temps, je n'ai pas pu le faire.

Le plus grand avantage de ce pipeline aurait été de pouvoir implémenter le `decoder_memory`avec une RAM de type BLOCK et non DISTRIBUTED, ce qui aurait grandement amélioré les performances.

![schéma du processeur](/img/proz_design/diagramme_three_steps_pipeline.png)

# Conclusion

C'était vraiment un chouette projet ! Il m'a permit de redécouvrir le VHDL que je n'avais qu'effleuré à l'UTC. J'ai adoré la logique un peu bizarre de la création de signaux qui a connu son apogée lors de l'implémentation des pipelines.

J'ai pu également réellement comprendre la nécessité d'être rigoureux dans le versionnement de son code, notamment à un moment où j'ai dû réalisé un refactoring total de mon code à grand coup de gitmerge... tout ça à cause de commits mal testés. "C'est comme ça que l'on apprend", diraient certains.




[pd_gh]: https://github.com/tamicasireim/prozdesign

