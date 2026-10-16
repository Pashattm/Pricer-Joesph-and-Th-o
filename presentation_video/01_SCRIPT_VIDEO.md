# Présentation vidéo — présentation du travail de groupe

**Théo : architecture, données, simulateur, Phoenix et intégration. Joseph : volatilité, Plain avec et sans rappel, valorisation et démonstration du Plain.**

Durée cible : **13 minutes, soit environ 6 min 30 chacun (Théo : 7 minutes ; Joseph : 6 minutes)**, avec les changements d’écran inclus. Ce sont des créneaux de répétition, pas des durées mesurées. Compléter la parole par le pointage des lignes et la démonstration ; ne pas accélérer pour lire tout le fichier.

## Introduire votre collaboration

Théo : « Dans le cadre de notre travail de groupe, nous présentons un pricer de produits structurés. Je vais expliquer son architecture, les données de marché et le Phoenix. Joseph présentera la volatilité, puis le Plain avec et sans rappel et sa valorisation. Nous montrerons ensuite le fonctionnement de l’application. »

Le projet est présenté collectivement. La répartition ci-dessous organise la parole ; elle n’attribue pas la propriété des fichiers. Les explications décrivent la version actuelle : sans rappel, les coupons sont conditionnés au niveau final.

## Déroulé et texte oral

### 00:00–00:30 — Présenter l’objectif

**Intervenant : Théo. À l’écran :** Application : menu et formulaire Phoenix.

« Nous présentons un pricer de produits structurés à un seul sous-jacent. Il couvre un Phoenix à coupons conditionnels et un Plain à coupons accumulés, avec une option pour désactiver le rappel. Notre objectif est de transformer des paramètres de marché et des règles contractuelles en une valeur théorique, puis d’expliquer les scénarios associés. Nous allons suivre le parcours des données jusqu’au résultat. »

### 00:30–01:15 — Expliquer l’architecture

**Intervenant : Théo. À l’écran :** Les quatre fichiers Python et templates/index.html.

**Repère :** [`main.py` — `api_autocall`, ligne 328](../main.py#L328).

« L’application comporte une interface et un serveur Python. Le HTML définit les champs, le CSS leur apparence et le JavaScript les interactions. Quand on clique sur Calculer, JavaScript envoie les paramètres au serveur Flask dans main.py. Celui-ci les vérifie, appelle le moteur et renvoie une réponse JSON. strutu_engine.py calcule les prix ; market_data_engine.py récupère le marché ; volatility_engine.py calcule la volatilité historique.»

### 01:15–02:00 — Présenter les données de marché

**Intervenant : Théo. À l’écran :** market_data_engine.py : recherche, historique et conversion.

**Repère :** [`market_data_engine.py` — `get_vol_historique`, ligne 77](../market_data_engine.py#L77).

« La recherche retrouve le symbole du titre. Le module récupère son dernier prix, sa devise et un historique de clôtures ajustées. La volatilité historique utilise par défaut un an de données. Le taux et le rendement de dividende restent saisis. Les données externes peuvent manquer : on garde un mode manuel, ce qui permet aussi une démonstration reproductible. Le VIX est une hypothèse alternative disponible, mais ce n’est pas la volatilité implicite propre au titre choisi. »

**Transition vers Joseph :** « Les cours sont chargés. Peux-tu expliquer comment on en déduit la volatilité ? »

### 02:00–03:00 — Expliquer la volatilité

**Intervenant : Joseph. À l’écran :** volatility_engine.py : les deux fonctions.

**Repère :** [`volatility_engine.py` — `log_returns`, ligne 4](../volatility_engine.py#L4).

« Pour chaque paire de cours successifs, nous calculons le logarithme du ratio des prix. prices[1:] contient les prix à partir du deuxième et prices[:-1] tous les prix sauf le dernier : les deux tableaux mettent donc chaque prix en face du précédent. Ensuite, nous prenons l’écart-type de ces rendements avec ddof égal à un, puis nous multiplions par racine de 252 pour annualiser des observations journalières. Le résultat est un décimal : 0,25 veut dire 25 %. La volatilité mesure la dispersion, pas le sens futur du marché. Dans cette copie, il n’y a pas de calcul EWMA. »

**Transition de Joseph vers Théo :** « Cette volatilité devient sigma dans le simulateur. Je te laisse montrer comment elle intervient dans les trajectoires. »

### 03:00–04:00 — Expliquer la simulation

**Intervenant : Théo. À l’écran :** strutu_engine.py : simulate_observation_dates.

**Repère :** [`strutu_engine.py` — `simulate_observation_dates`, ligne 15](../strutu_engine.py#L15).

« Nous simulons le sous-jacent uniquement aux dates d’observation avec un mouvement brownien géométrique. Le prochain cours est le cours précédent multiplié par une exponentielle. Dans l’exponentielle, une partie dépend du taux moins le dividende et de la correction de variance ; une autre dépend de la volatilité et d’un choc normal aléatoire. La simulation se fait sous une mesure risque-neutre : nous n’utilisons pas une prévision de hausse de l’action. Chaque ligne de la matrice est un scénario et chaque colonne un constat, avec S0 dans la première colonne. Une graine fixe permet de retrouver les mêmes tirages. »

### 04:00–05:00 — Expliquer le Phoenix

**Intervenant : Théo. À l’écran :** strutu_engine.py : coupons, mémoire et rappel.

**Repère :** [`strutu_engine.py` — `_simuler_phoenix`, ligne 51](../strutu_engine.py#L51).

« Le Phoenix vérifie à chaque constat les scénarios encore actifs. Si le seuil coupon est atteint, il verse le coupon. Sans mémoire, un coupon manqué est perdu ; avec mémoire, il est stocké puis rattrapé lorsqu’un constat devient favorable. Si le rappel est activé et sa condition atteinte avant maturité, le capital est rendu et le scénario s’arrête. À maturité, pour les scénarios restants, le principal est rendu intégralement au-dessus de la protection, sinon il suit le ratio entre le cours final et la référence. Avec une référence de 100, une protection à 60 et un cours final de 50, le principal remboursé vaut 50 % du nominal. »

**Transition vers Joseph :** « Le simulateur est commun. Je te laisse maintenant expliquer le Plain avec et sans rappel et la manière dont il utilise ces trajectoires. »

### 05:00–06:30 — Expliquer le Plain et l’option sans rappel

**Intervenant : Joseph. À l’écran :** strutu_engine.py : _flux_autocall et analyse_autocall_plain.

**Repère :** [`strutu_engine.py` — `_flux_autocall`, ligne 280](../strutu_engine.py#L280).

« Le Plain ne verse pas les coupons à chaque constat : il les accumule. Un rappel au deuxième constat avec un coupon de 4 % paie 100 % du nominal plus 8 %. Sans rappel, les coupons accumulés sont tous versés à maturité si le seuil final est atteint, sinon aucun. La case de rappel ne change que la possibilité de sortie anticipée : décochée, le produit va toujours à maturité. Un constat intermédiaire favorable ne suffit pas à acquérir un coupon. La protection conditionnelle du principal est une autre option, indépendante. Désactivée, le code rembourse toujours le nominal, sans modéliser le défaut de l’émetteur. »

### 06:30–07:30 — Expliquer précisément l’option sans rappel

**Intervenant : Joseph. À l’écran :** strutu_engine.py : analyse_autocall_plain puis _flux_autocall.

**Repère :** [`strutu_engine.py` — `analyse_autocall_plain`, ligne 376](../strutu_engine.py#L376).

« Dans la version intégrée, nous avons regroupé les deux configurations dans un seul produit. La fonction reçoit has_autocall, vrai par défaut. Lorsqu’il est faux, aucun scénario ne sort avant maturité. La condition des coupons reste finale : avec six constats à 4 %, on paie 24 % si le cours final atteint le seuil, sinon zéro coupon. Un cours au-dessus du seuil au premier constat puis en dessous à maturité ne crée donc aucun coupon acquis. Cette précision distingue la version actuelle de l’ancienne branche qui comptait les constats favorables. »

**Transition vers Théo :** « Le rappel est une option ; la protection du capital en est une autre. Je te laisse préciser cette distinction. »

### 07:30–08:15 — Distinguer rappel et protection du capital

**Intervenant : Théo. À l’écran :** Application : les deux cases du Plain ; strutu_engine.py : perte_capital.

**Repère :** [`strutu_engine.py` — `_flux_autocall`, ligne 280](../strutu_engine.py#L280).

« Le rappel anticipé et la protection conditionnelle sont deux mécanismes séparés. La première case décide si le produit peut se terminer avant la date finale. La seconde détermine le remboursement du principal à maturité. Avec une barrière à 60 %, un cours final à 50 % de S0 donne un principal de 50 % du nominal, pas de 90 %. Si la protection conditionnelle est désactivée dans ce code, le principal reste égal au nominal. Un remboursement nominal dans le modèle n’élimine pas le risque de défaut d’un émetteur réel. »

**Transition vers Joseph :** « À partir de ces remboursements, comment obtient-on la valeur aujourd’hui ? »

### 08:15–09:15 — Expliquer la valeur théorique

**Intervenant : Joseph. À l’écran :** strutu_engine.py : flux actualisés et statistiques.

**Repère :** [`strutu_engine.py` — `_analyse_autocall`, ligne 332](../strutu_engine.py#L332).

« Chaque scénario produit des paiements à une ou plusieurs dates. Nous multiplions chaque paiement par exponentielle de moins r fois t pour le ramener à aujourd’hui. La valeur théorique est ensuite la moyenne de ces flux actualisés. Le moteur travaille pour un nominal de un : une valeur de 0,95 signifie 95 % du nominal, soit 950 pour 1 000 investis. La composante de remboursement du capital est déjà incluse ; ajouter séparément un zéro-coupon compterait le capital une deuxième fois. L’intervalle de confiance mesure le bruit de Monte Carlo, pas l’incertitude sur le modèle financier. »

**Transition vers Théo :** « Je te laisse relier cette valeur aux indicateurs affichés. »

### 09:15–10:00 — Lire les indicateurs et le graphique

**Intervenant : Théo. À l’écran :** Résultats du Plain et _analyse_autocall.

**Repère :** [`strutu_engine.py` — `_analyse_autocall`, ligne 332](../strutu_engine.py#L332).

« À partir des mêmes trajectoires, nous calculons le prix, la durée de vie moyenne et les probabilités de sortie. La probabilité de perte concerne le principal avant les coupons. Le remboursement moyen si perte ne concerne que les trajectoires en perte. La marge affichée est simplement 100 % moins la valeur théorique, avant coûts et défaut. Enfin, le graphique montre des remboursements futurs non actualisés en fonction du cours final. Ce n’est ni le prix actuel ni une représentation de toutes les trajectoires. »

**Transition vers l’intégration :** « Ces indicateurs sont renvoyés par le serveur. Je vais maintenant montrer comment ils arrivent dans l’application. »

### 10:00–11:00 — Montrer l’intégration et une démonstration Phoenix

**Intervenant : Théo. À l’écran :** main.py : api_phoenix ; application : Phoenix.

**Repère :** [`main.py` — `api_phoenix`, ligne 218](../main.py#L218).

« Les paramètres sont saisis dans l’interface, puis transmis à Flask. Le serveur convertit par exemple 25 % en 0,25, vérifie les données et appelle le moteur. La réponse contient le prix, les statistiques et les points du graphique. Je montre ici le Phoenix et sa case mémoire : elle stocke les coupons manqués pour les rattraper à un constat favorable. Son graphique final ne montre pas les coupons déjà versés ni toute la mémoire. Pour comparer précisément deux prix, il faut aussi contrôler les tirages aléatoires. »

**Transition vers Joseph :** « À Théo de montrer maintenant les deux configurations du Plain dans cette même interface. »

### 11:00–12:00 — Démontrer le Plain avec puis sans rappel

**Intervenant : Joseph. À l’écran :** Application : Plain, mêmes paramètres et graine 42.

« Je prends trois ans, des constats semestriels, un coupon de 4 % par observation et un nominal de 1 000. Je calcule avec rappel, puis je décoche uniquement le rappel en conservant les autres paramètres et la graine. Sans rappel, la durée moyenne est trois ans, la probabilité de maturité est 100 % et les rappels sont nuls. À maturité, si le cours atteint le seuil final, les six coupons représentent 240, ajoutés au principal de 1 000. Le prix aujourd’hui est calculé par moyenne de flux actualisés ; il ne faut donc pas le confondre avec ces 1 240 futurs. »

### 12:00–12:30 — Présenter les vérifications

**Intervenant : Théo. À l’écran :** tests/test_api.py et tests/test_engine.py.

« Les tests contrôlent les conversions, les paramètres invalides, les égalités aux barrières et des trajectoires dont les paiements sont connus. Ils vérifient aussi le mode sans rappel : aucun rappel et aucune acquisition de coupon sur un simple constat intermédiaire favorable. Les quatorze tests passent dans la version préparée. Ces tests vérifient la logique et les routes ; ils ne garantissent pas la disponibilité des données externes. »

### 12:30–13:00 — Conclure sur les limites

**Intervenant : Joseph. À l’écran :** Application : résultats, puis écran final.

« Le pricer relie des données de marché, des règles de paiement et une valorisation par simulation. Ses limites sont un taux, un dividende et une volatilité constants, sans smile ni défaut de l’émetteur. Les probabilités sont risque-neutres et non prédictives. Les principales extensions seraient une courbe d’actualisation et une calibration aux options. »

