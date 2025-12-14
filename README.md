Lab 1 : Du notebook au mini-système production-ready

Étape 1 : Initialiser la structure du projet
Crée un dossier de travail :
 
Crée les dossiers suivants :
   
Crée le fichier identifiant le modèle actif :
 
Étape 2 : Préparer l'environnement Python
Instructions :
Créer un environnement virtuel (Windows PowerShell) :
Mettre à jour pip :
Installer les dépendances :
 
 


Étape 3 : Générer le dataset
Instructions :
Créer le fichier :
 
L’objectif de cette étape est de créer un dataset synthétique qui servira de base à l’ensemble du pipeline MLOps qui contrend entraînement, évaluation, déploiement et monitoring du modèle. Le script generate_data.py produit un fichier data/raw.csv contenant des données clients liées à un service d’abonnement.
Étape 4 : Préparer les données + quality checks
Instructions :
Créer le fichier :
 
Cette étape on a  transformer les données brutes en données exploitables par un modèle de machine learning, c’est la phase de prétraitement et validation des données dans un pipeline MLOps. L’objectif principal est d’éviter que des données incorrectes, incohérentes ou non conformes n’entrent dans la phase d’entraînement, ce qui pourrait dégrader les performances du modèle ou rendre le pipeline instable.
Les etapes que le script suit sont :
1.	Chargement et nettoyage des données
2.	Correction des valeurs numériques incohérentes
3.	Normalisation des variables catégorielles
4.	Contrôles de qualité des données 
5.	Vérification du schéma des données
6.	Contrôle du taux de valeurs manquantes
7.	Validation des types de données
8.	Sauvegarde des données prétraitées
9.	Calcul et traçabilité des statistiques d’entraînement

Étape 5 : Entraîner, versionner et valider le modèle
Instructions :
Créer le fichier :
 
Dans cette etapes on fait l’entraînement du modèle de machine learning, sa comparaison à une baseline, on lui donne un statut de modèle courant. L’objectif c’est de garantir que seuls les modèles présentant une qualité suffisante puissent être utilisés en production, grâce à un mécanisme de validation automatique qui est le deployment gate. Le script utilise la régression logistique binaire et une séparation train / test , et une evaluation des performances du modèle ou plusieurs métriques sont calculées afin d’obtenir une évaluation complète :
•	Accuracy : performance globale
•	Precision : qualité des prédictions positives
•	Recall : capacité à détecter les churns
•	F1-score : compromis entre précision et rappel
Un mécanisme de deployment gate est implémenté afin de contrôler la promotion du modèle. Un modèle est accepté uniquement si son F1-score est supérieur ou égal au seuil défini et à celui de la baseline. 
Étape 6 : Inspecter la registry et le modèle courant
Instructions :
Créer le fichier :
 
L’objectif de cette étape est de mettre en place et d’inspecter une registry de modèles, c’est-à-dire un mécanisme permettant de conserver l’historique des modèles entraînés, stocker leurs performances et métadonnées et identifier clairement quel modèle est considéré comme le modèle courant en production. Donc nous avons mis en place une registry de modèles minimale avec les fichiers :
•	metadata.json qui contient l’historique de tous les modèles entraînés, avec :
1.	la version du modèle ;
2.	la date d’entraînement ;
3.	les métriques (accuracy, precision, recall, F1) ;
4.	la valeur de la baseline ;
5.	le seuil du gate (gate_f1) ;
6.	le statut du modèle (validé ou refusé).
•	current_model.txt qui contient le nom du modèle officiellement actif, c’est-à-dire celui autorisé à être utilisé en production.
Même si un modèle est entraîné et sauvegardé, il n’est pas automatiquement déployé car le déploiement dépend d’un quality gate basé sur la métrique F1, grâce au fichier current_model.txt, il est possible de savoir quel modèle est actif à un instant donné, éviter toute ambiguïté sur la version réellement utilisée et faciliter les audits, le monitoring et les rollbacks. Si aucun modèle ne passe le gate, aucun modèle n’est activé, ce qui est un comportement souhaitable en MLOps, donc un modèle ML ne doit pas être déployé uniquement parce qu’il fonctionne et la notion de registry est essentielle pour la traçabilité des modèles, la reproductibilité des expériences, la gouvernance des déploiements, les quality gates sont un élément clé de la mise en production industrielle des modèles ML.

Étape 7 : Créer une API /predict qui utilise le modèle courant
Instructions : Créer le fichier :
 
Lancer l’API et Tester :
•	GET http://127.0.0.1:8000/health
•	POST http://127.0.0.1:8000/predict (avec un JSON d’exemple)
 
 
 
 
 
 
Étape 8 : Détecter une dérive des données via les logs
Instructions :
Créer le fichier :
 
L’objectif de cette étape est de surveiller le comportement du modèle en production, en détectant une éventuelle dérive des données d’entrée (data drift) car même un modèle performant peut devenir inefficace dans le temps si les données reçues en production ne suivent plus la même distribution que celles utilisées à l’entraînement. Cette étape correspond à la phase Monitor d’un pipeline MLOps.
Conclusion du monitoring
Résultat : 3 alerte(s) de drift.
Analyse recommandée + retraining possible.
Cela signifie que :
•	les données en production ne ressemblent plus aux données d’entraînement ;
•	les prédictions du modèle peuvent devenir peu fiables ;
•	une analyse approfondie est nécessaire.



Étape 9 : Gérer les versions du modèle et revenir en arrière
Instructions :
Entraîner une nouvelle version (v2) :
Créer le script de rollback
Effectuer un rollback automatique vers la version précédente :
Ou rollback vers une version précise (remplacer le nom par un vrai fichier existant) :
 
Dans cette etape notre objectif est de mettre en place une gestion de versions des modèles (model versioning) et à permettre un retour rapide vers un modèle précédent en cas de problème après un déploiement. Nous avons entraîné une nouvelle version du modèle (v2) et le script rollback.py agit comme un outil de gestion du model registry, il permet de, lire la liste des modèles enregistrés dans registry/metadata.json, définir quel modèle est actuellement actif via current_model.txt et revenir automatiquement à la version précédente si nécessaire.

Conclusion :
Ce lab a permis de mettre en pratique l’ensemble du cycle de vie d’un modèle machine learning dans un contexte MLOps minimal mais réaliste, depuis la préparation des données jusqu’au déploiement et au monitoring en production.
Nous avons appris à :
•	Structurer un projet ML avec des dossiers clairs et une séparation des responsabilités (données, modèles, logs, scripts).
•	Préparer et valider les données, en appliquant des contrôles qualité pour garantir la fiabilité du pipeline.
•	Entraîner, évaluer et versionner les modèles, en utilisant un mécanisme de quality gate pour ne déployer que des modèles performants.
•	Mettre en place une registry de modèles, permettant de tracer l’historique des modèles, de connaître le modèle actif et de faciliter les audits et le suivi des performances.
•	Créer une API de prédiction pour servir le modèle en production, illustrant la phase Serve d’un pipeline MLOps.
•	Surveiller les données et détecter un drift, afin de garantir la performance du modèle dans le temps et anticiper les besoins de ré-entraînement.
•	Gérer les versions et effectuer des rollbacks, assurant un déploiement sécurisé et réversible en cas de problème avec une nouvelle version.
En résumé, ce lab a démontré comment transformer un notebook expérimental en un mini-système production-ready, intégrant toutes les étapes clés d’un pipeline MLOps : Train, Serve, Monitor et Manage. Il illustre l’importance de la traçabilité, de la reproductibilité et de la robustesse dans la mise en production des modèles ML.










Lab2 : Code Source management
Étape 1 : Initialiser Git dans mlops-lab-01
Instructions :
Aller dans le dossier du projet :
Initialiser le dépôt Git :
 
Afficher les dossiers cachés:
 
Créer le fichier .gitignore :
 
Vérifier l’état du dépôt :
 
Ici on doit initier un dépôt Git local pour versionner l’ensemble du projet MLOps et préparer un historique traçable des changements.
Étape 2 : Premier commit du projet MLOps
Instructions :
Ajouter les dossiers et fichiers principaux (code + configuration) :
Créer le premier commit :
Afficher l’historique :
 
On remarque que le dossier data n’est pas ajouter car on la mit dans le fichier .gitignore pour etre ignoré
Étape 3 : Observer une modification avec git diff
Instructions :
Modifier un script existant, par exemple dans src/monitor_drift.py :
 
Afficher les différences par rapport au dernier commit :
Ajouter le fichier modifié à l’index :
Afficher les différences en staging :
Créer un commit :
 
Ici on doit inspecter les changements locaux avant de les valider dans l’historique.
Étape 4 : Créer une branche de fonctionnalité liée au lab
Instructions :
Créer une branche :
 
Modifier src/api.py (par exemple ajouter une gestion de request_id automatique) :
 
Ajouter et committer :
Lister les branches :
Revenir sur la branche principale :
 
Ici on doit isoler la nouvelle fonctionnalité de gestion automatique de request_id dans une branche dédiée pour développer sans perturber master.
Étape 5 : Fusionner la branche feature dans la branche principale
Instructions :
Depuis la branche principale :
Fusionner la branche :
Vérifier l’historique :
 
Ici on doit intégrer la fonctionnalité testée dans la branche principale après validation.
Étape 6 : Créer un conflit de merge sur src/train.py
Instructions :
Créer une nouvelle branche :
 
Modifier src/train.py (par exemple gate_f1 à 0.60) :
 
Ajouter et committer :
Revenir sur la branche principale :
 
Modifier la même ligne dans src/train.py (par exemple gate_f1 à 0.75) :
 
Ajouter et committer :
Une erreur de conflix est apparu que j’ai resulut en acceptant les modification uncomming
Tenter la fusion :
 
Résoudre le conflit dans src/train.py (choisir une valeur, par exemple 0.70), puis :
 
Ici on a tout simplement rencontrer un conflit Git et on a appris a le resoudre correctement
Étape 7 : Utiliser git stash dans le contexte du lab
Instructions :
Modifier un fichier sans vouloir committer (par exemple src/rollback.py) :
 
Afficher l’état :
Mettre de côté les modifications :
Lister les stash :
Récupérer les modifications :
(Option alternative pour appliquer + supprimer) :
 
Ici on doit apprendre à mettre temporairement de côté des modifications non terminées pour pouvoir switcher facilement de branche sans probleme.
Étape 8 : Tester git reset sur un fichier d’expérimentation
Instructions :

Créer un dossier d’expérimentation :
Créer un fichier de test :
Modifier puis committer deux fois :
Effectuer un reset soft :
Effectuer un reset mixed :
Effectuer un reset hard :
 
 
Ici on a Comprendris les différents types de git reset : soft, mixed et hard et leur effet sur l’index et le working tree.
Étape 9 : Annuler un commit avec git revert
Instructions :
Ajouter un changement non souhaité dans src/api.py :
Lister les commits :
Revert du dernier commit :
Vérifier le contenu du fichier :
 
Étape 10 : Rebase d’une branche feature sur la branche principale
Instructions :
Créer une branche :
 
Modifier src/monitor_drift.py (changer last_n, par exemple 500) :
 
Ajouter et committer :
Revenir sur la branche principale et créer un nouveau commit sur un autre fichier (par exemple src/generate_data.py) :
Revenir sur la branche feature :
Rebaser la branche feature sur la branche principale :
Vérifier l’historique :
 
Ici on doit apprendre à annuler un commit 

Conclusion : 
Ce deuxième lab a permis de maîtriser les fondamentaux de la gestion de code source avec Git, un outil indispensable dans les projets MLOps et plus généralement en ingénierie logicielle et data. À travers des manipulations progressives et réalistes, nous avons appris à structurer un dépôt, versionner correctement le code, et gérer l’évolution d’un projet de manière contrôlée et traçable. L’utilisation de .gitignore nous a sensibilisés à l’importance de ne pas versionner des artefacts non reproductibles ou volumineux tels que les données, les modèles entraînés et les environnements virtuels, ce qui est crucial dans un contexte MLOps. Les commandes git diff, git add et git commit ont renforcé la discipline de validation incrémentale des changements, favorisant des commits clairs, atomiques et compréhensibles. Le travail avec les branches de fonctionnalité, leur fusion dans la branche principale, ainsi que la résolution de conflits, a mis en évidence les problématiques réelles du travail collaboratif. La création volontaire d’un conflit de merge a permis de comprendre en profondeur les mécanismes de Git et d’adopter une démarche réfléchie lors de la résolution, plutôt qu’une acceptation aveugle des changements. Les commandes avancées telles que git stash, git reset, git revert et git rebase ont permis de distinguer clairement les opérations sûres de celles potentiellement destructrices, et d’identifier les bonnes pratiques à adopter selon que l’on travaille localement ou sur une branche partagée. En particulier, la différence entre reset et revert est essentielle pour préserver l’intégrité de l’historique dans un environnement collaboratif.
En conclusion, ce laboratoire ne se limite pas à l’apprentissage de commandes Git, mais constitue une mise en situation réaliste des workflows utilisés en MLOps, où la reproductibilité, la collaboration et la traçabilité sont critiques. Les compétences acquises dans ce lab forment une base solide pour la gestion de pipelines de données et de modèles à grande échelle, et sont directement applicables dans des projets professionnels et industriels.


