# Tics d'écriture propres au français

Ces règles complètent `style_guide.md`, écrit pour l'anglais, dont les tropes restent
valables en français sous leur forme traduite (« plongeons dans », « il convient de
noter », « ce n'est pas X, c'est Y »). Elles ciblent les tics que la traduction ne fait
pas apparaître, et signalent où une règle anglaise ne s'applique pas telle quelle au
français.

Corriger un tic demande de reconstruire la phrase. Remplacer un mot par un autre
laisse le motif en place.

## La chute en deux clauses

Ne pas terminer un paragraphe sur le motif « blablabla : truc truc. ». Il passe
inaperçu une fois et devient un tic à la troisième. Proche de « The X? A Y. » dans
`style_guide.md`, il se passe du point d'interrogation.

- À éviter : « Un constat trouvé chez un client cesse d'appartenir au dossier qui l'a
  découvert : il doit profiter au suivant. »
- À éviter : « On a passé six mois à migrer vers Kubernetes. La leçon tient en une
  phrase : la complexité ne disparaît jamais, elle se déplace. »
- Avant : « Le cache n'était pas en cause. Le vrai coupable était ailleurs : le pool de
  connexions. »
- Après : « Le pool de connexions plafonnait à 10, et chaque requête lente en bloquait
  une. »

Les deux-points restent bons pour introduire une définition ou une énumération, en
milieu de phrase.

## L'image à la place du fait

Une expression toute faite (« changer de côté », « taillé sur mesure », « trancher »,
« changer la donne », « remettre l'église au milieu du village ») remplace la
conséquence concrète. Le lecteur doit la traduire, et il ne retombe pas toujours sur le
fait. « The Dead Metaphor » vise une métaphore filée sur tout un texte, et « Think of
It As » une analogie annoncée. Ici, une seule expression suffit à masquer le fait.

- Avant : « Plus de la moitié du travail de validation changerait de côté. »
- Après : « Avec un export de sa configuration, il ne resterait que 9 ou 10 points à
  recetter chez le client. »
- Avant : « La prochaine migration ira vers une cible différente. Elle tranchera. »
- Après : « Je le saurai à la prochaine migration qui vise une cible différente. »
- Avant : « Cette refacto remet l'église au milieu du village côté gestion des
  erreurs. »
- Après : « Cette refacto regroupe la gestion des erreurs dans un seul intercepteur, au
  lieu de trois try/catch par contrôleur. »
- Avant : « L'IA générative change la donne pour les équipes QA. »
- Après : « Les tests générés couvrent les cas nominaux. Les cas limites restent à
  écrire à la main. »

## La conclusion sous-entendue

Une phrase pose l'observation et laisse le lecteur en tirer la conclusion. Écrire la
conclusion, même quand elle paraît évidente à l'auteur. Elle s'écrit une seule fois, là
où elle découle du fait, et ne justifie pas les récapitulatifs que dénonce « Fractal
Summaries ».

- Avant : « Une passe de calibration faisait apparaître le même symptôme là où rien
  n'avait changé. »
- Après : « Les sessions s'y perdaient aussi. Le format des cookies n'y était donc pour
  rien. »
- Avant : « L'alerte s'est déclenchée à 3 h 12. Le premier ticket client date de
  2 h 40. »
- Après : « L'alerte s'est déclenchée à 3 h 12 et le premier ticket client date de
  2 h 40. Nos clients ont donc vu la panne une demi-heure avant notre supervision. »

## L'incise entre le verbe et son complément

Déplacer la précision en fin de phrase ou en tête, plutôt que de couper le verbe de ce
qu'il régit.

- Avant : « produisent, octet pour octet, un fichier identique »
- Après : « produisent un fichier identique, octet pour octet »
- Avant : « La commande supprime, sans demander de confirmation, toutes les branches
  fusionnées. »
- Après : « La commande supprime toutes les branches fusionnées sans demander de
  confirmation. »

Même chose pour une incise entre le sujet et le verbe : « Le même constat, restitué
dans deux rapports différents, y arrive-t-il… » devient « Restitué dans deux rapports
différents, le même constat y arrive-t-il… ».

Les appositions qui expliquent un terme restent bonnes, par exemple « Keycloak, le
gestionnaire d'identités open source, ». Une apposition qui décrit un mécanisme ou pose
une consigne mérite en revanche sa propre phrase.

- Avant : « Il a aussi tenu un garde-fou posé le premier jour, ne pas rejouer la faille
  de sécurité elle-même. »
- Après : « Le premier jour, je lui avais interdit de reproduire la faille de
  sécurité. »

## Les phrases longues

Découper au-delà de 30 mots quand la longueur vient d'enchâssements. Une énumération
longue se lit d'une traite, une cascade de relatives non. Viser une médiane autour de
15 mots par phrase, avec des longueurs variées. Découper ne veut pas dire hacher,
car une suite de phrases de quatre mots retombe dans les « Short Punchy Fragments ».

- Avant : « Le service, qui reçoit les événements que le broker publie dès qu'un contrat
  que l'équipe métier a validé change d'état, met à jour la vue qui alimente le tableau
  de bord des conseillers. »
- Après : « Quand l'équipe métier valide un changement d'état sur un contrat, le broker
  publie un événement. Le service le reçoit et met à jour le tableau de bord des
  conseillers. »

## La voix active

Un outil n'exige rien, un contexte ne décide pas, une règle ne naît pas toute seule.
Nommer la personne ou le programme qui agit. Le passif impersonnel (« il a été décidé
de », « une décision a été prise ») cache la même absence.

- Avant : « Le banc de test exige que chaque explication porte sa provenance. »
- Après : « Depuis, chaque explication porte sa provenance. »
- Avant : « Il a été décidé de déprécier l'endpoint v1. »
- Après : « L'équipe API déprécie l'endpoint v1 le 1er mars. »

## Les chaînes de noms

Raconter un processus avec un sujet et des verbes, pas avec une suite de noms
(« une gêne transformée en question puis en mesure », « la mise en place de »).

- Avant : « L'agent ne signale pas de lui-même l'incertitude qu'il faudrait aller
  vérifier. »
- Après : « L'agent n'a jamais signalé de lui-même qu'il n'était pas sûr. »
- Avant : « Mise en place d'une optimisation de la gestion de la récupération des
  données du tableau de bord. »
- Après : « Le tableau de bord charge ses données en une requête au lieu de douze. »

## La ponctuation

L'em dash est déjà traité par « Em-Dash Addiction » et par les préférences du skill.

Le point-virgule colle deux propositions sans dire le lien entre elles. Le remplacer par
un point, ou par le connecteur qui nomme ce lien (mais, donc, parce que).

- Avant : « Le build passe en local ; il échoue en CI. »
- Après : « Le build passe en local mais échoue en CI. »

Les guillemets « » sont la norme typographique en français, et la règle « Unicode
Decoration » ne les vise pas. Elle s'applique toujours aux flèches et aux guillemets
anglais courbes.

## Le gras

Une ou deux emphases par section, sur les thèses. Pas une par paragraphe sur sa
morale, car le procédé répété trop de fois n'appuie plus rien. « Bold-First Bullets »
couvre les listes, cette règle couvre le gras au fil de la prose.

## Les répétitions

Trois motifs à traquer :

- un mot de contenu répété dans la même phrase
- un groupe de deux mots répété dans le même paragraphe
- deux phrases consécutives ouvertes par le même mot (deux suffisent, là où « Anaphora
  Abuse » vise une série)

Garder la répétition quand elle porte le sens, par exemple « il jugeait l'outil avec
l'outil », « 14 sur 14 chez un client, 13 sur 13 chez l'autre ».

Corriger en reformulant, pas en cherchant un synonyme. La variation élégante (« la
solution », « l'outil », « ce dernier ») est le pendant français de « The Serves As
Dodge » et oblige le lecteur à vérifier qu'il s'agit bien de la même chose.

- Avant : « Kafka reçoit les événements. La plateforme de streaming les conserve sept
  jours, et ce dernier les réplique sur trois nœuds. »
- Après : « Kafka reçoit les événements, les conserve sept jours et les réplique sur
  trois nœuds. »

## Les chiffres

« 22-23 » se lit comme une fourchette alors qu'il s'agit de deux mesures. Dire à quoi
correspond chaque chiffre, et à qui renvoie un possessif quand la phrase compte deux
propriétaires possibles.

- Avant : « 22-23 restent aujourd'hui à sa charge. »
- Après : « Selon la migration, 22 ou 23 reposent entièrement sur la recette du
  client. »
- Avant : « L'équipe infra a migré le service de l'équipe data vers son cluster. »
- Après : « L'équipe infra a migré le service de l'équipe data vers le cluster de
  l'infra. »

## Le vocabulaire

Définir un terme à sa première occurrence, et vérifier qu'il correspond au vocabulaire
du projet source plutôt qu'à une invention de session. Pour les étiquettes inventées
de toutes pièces, voir « Invented Concept Labels ».

Garder en anglais les termes que les développeur·se·s disent en anglais (tag, email,
rollback, commit, merge, build, pull request), et traduire la prose autour. À
l'inverse, ne pas franciser un verbe anglais quand le français en a un.

- Avant : « Nous avons effectué une restauration à l'état antérieur après l'échec du
  déploiement. »
- Après : « Nous avons fait un rollback après l'échec du déploiement. »
- Avant : « On a leveragé le cache existant. »
- Après : « On a réutilisé le cache existant. »
