# Présentation vidéo — présentation du travail de groupe

**Théo : architecture, données, simulateur, Phoenix et intégration. Joseph : volatilité, Plain avec et sans rappel, valorisation et démonstration du Plain.**

Durée indicative de cette version détaillée : **18 à 22 minutes**, à mesurer en répétition. La répartition précédente sert de fil conducteur ; les explications de code ajoutées allongent les séquences. Les rôles restent proches de la moitié chacun. Pour une contrainte de 13 minutes, sélectionner les extraits essentiels plutôt que lire tous les commentaires.

## Introduire votre collaboration

Théo : « Dans le cadre de notre travail de groupe, nous présentons un pricer de produits structurés. Je vais expliquer son architecture, les données de marché et le Phoenix. Joseph présentera la volatilité, puis le Plain avec et sans rappel et sa valorisation. Nous montrerons ensuite le fonctionnement de l’application. »

Le projet est présenté collectivement. La répartition ci-dessous organise la parole ; elle n’attribue pas la propriété des fichiers. Les explications décrivent la version actuelle : sans rappel, les coupons sont conditionnés au niveau final.

## Déroulé et texte oral

### Présenter l’objectif

**Intervenant : Théo. À l’écran :** Application : menu et formulaire Phoenix.

« Nous présentons un pricer de produits structurés à un seul sous-jacent. Il couvre un Phoenix à coupons conditionnels et un Plain à coupons accumulés, avec une option pour désactiver le rappel. Notre objectif est de transformer des paramètres de marché et des règles contractuelles en une valeur théorique, puis d’expliquer les scénarios associés. Nous allons suivre le parcours des données jusqu’au résultat. »

### Expliquer l’architecture

**Intervenant : Théo. À l’écran :** Les quatre fichiers Python et templates/index.html.

**Repère :** [`main.py` — `api_autocall`, ligne 328](../main.py#L328).

« L’application comporte une interface et un serveur Python. Le HTML définit les champs, le CSS leur apparence et le JavaScript les interactions. Quand on clique sur Calculer, JavaScript envoie les paramètres au serveur Flask dans main.py. Celui-ci les vérifie, appelle le moteur et renvoie une réponse JSON. strutu_engine.py calcule les prix ; market_data_engine.py récupère le marché ; volatility_engine.py calcule la volatilité historique. Cette séparation permet de tester les calculs sans ouvrir le navigateur. »


**Code à pointer à l’écran** — extraits de `main.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
from market_data_engine import search_tickers, get_spot, get_devise, get_vix, get_vol_historique
```

**Explication orale du code et lien avec la suite :**

Théo : « Cet import nous donne accès aux fonctions de marché ; main.py ne recopie pas leur code. De la même façon, l’import suivant récupère les fonctions d’analyse du moteur structuré. Une route est un point d’entrée : GET récupère des données, POST transmet un formulaire de calcul. Nous allons ouvrir les modules appelés pour suivre ce que deviennent ces paramètres, puis revenir ici pour voir comment les résultats repartent vers l’écran. »

### Présenter les données de marché

**Intervenant : Théo. À l’écran :** market_data_engine.py : recherche, historique et conversion.

**Repère :** [`market_data_engine.py` — `get_vol_historique`, ligne 77](../market_data_engine.py#L77).

« La recherche retrouve le symbole du titre. Le module récupère son dernier prix, sa devise et un historique de clôtures ajustées. La volatilité historique utilise par défaut un an de données. Le taux et le rendement de dividende restent saisis. Les données externes peuvent manquer : on garde un mode manuel, ce qui permet aussi une démonstration reproductible. Le VIX est une hypothèse alternative disponible, mais ce n’est pas la volatilité implicite propre au titre choisi. »


**Code à pointer à l’écran** — extraits de `market_data_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
from volatility_engine import historical_volatility
    cours = get_historical_prices(ticker, periode)
    vol = historical_volatility(cours)
    return float(vol * 100)
```

**Explication orale du code et lien avec la suite :**

Théo : « Regardons d’abord l’import en haut du fichier : nous récupérons historical_volatility depuis volatility_engine. Nous n’écrivons donc pas une deuxième formule de volatilité dans market_data. Ici, get_vol_historique organise le travail : récupérer les cours, vérifier l’historique, appeler le calcul, puis convertir sa réponse en pourcentage. Joseph va maintenant ouvrir la fonction importée pour expliquer ce calcul. »

Théo : « Le seuil de vingt cours dans ce fichier est un minimum choisi pour l’application. Joseph montrera un autre contrôle, à trois cours, dans la fonction mathématique : ce n’est pas une contradiction. On distingue un historique minimal pour l’application et le minimum nécessaire au calcul. »

**Transition vers Joseph :** « Les cours sont chargés. Peux-tu expliquer comment on en déduit la volatilité ? »

### Expliquer la volatilité

**Intervenant : Joseph. À l’écran :** volatility_engine.py : les deux fonctions.

**Repère :** [`volatility_engine.py` — `log_returns`, ligne 4](../volatility_engine.py#L4).

« Pour chaque paire de cours successifs, nous calculons le logarithme du ratio des prix. prices[1:] contient les prix à partir du deuxième et prices[:-1] tous les prix sauf le dernier : les deux tableaux mettent donc chaque prix en face du précédent. Ensuite, nous prenons l’écart-type de ces rendements avec ddof égal à un, puis nous multiplions par racine de 252 pour annualiser des observations journalières. Le résultat est un décimal : 0,25 veut dire 25 %. La volatilité mesure la dispersion, pas le sens futur du marché. Dans cette copie, il n’y a pas de calcul EWMA. »


**Code à pointer à l’écran** — extraits de `volatility_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
    returns = np.log(prices[1:] / prices[:-1])
    returns = log_returns(prices)
    daily_volatility = np.std(returns, ddof=1)
    annual_volatility = daily_volatility * np.sqrt(annualization)
    return float(annual_volatility)
```

**Explication orale du code et lien avec la suite :**

Joseph : « C’est exactement la fonction appelée dans le fichier que Théo vient de montrer. Elle reçoit les cours, pas le ticker : elle peut donc fonctionner sur un tableau sans connexion à Yahoo. Elle appelle d’abord log_returns, définie juste au-dessus. Avec trois cours, nous obtenons deux rendements ; il faut au moins ces deux rendements pour calculer l’écart-type avec la correction ddof=1. »

Joseph : « np.std mesure la dispersion autour du rendement moyen. ddof=1 demande une division par le nombre de rendements moins un. Nous annualisons ensuite avec racine de 252. La fonction renvoie par exemple 0,25 ; market_data multiplie par 100 pour afficher 25 %. Lorsque cette valeur est sélectionnée pour le pricing, main.py la divise à nouveau par 100. Théo va montrer ce passage vers sigma dans le simulateur. Chaque conversion a donc un destinataire précis : décimal pour le calcul, pourcentage pour la saisie. »

**Transition de Joseph vers Théo :** « Cette volatilité devient sigma dans le simulateur. Je te laisse montrer comment elle intervient dans les trajectoires. »

### Expliquer la simulation

**Intervenant : Théo. À l’écran :** strutu_engine.py : simulate_observation_dates.

**Repère :** [`strutu_engine.py` — `simulate_observation_dates`, ligne 15](../strutu_engine.py#L15).

« Nous simulons le sous-jacent uniquement aux dates d’observation avec un mouvement brownien géométrique. Le prochain cours est le cours précédent multiplié par une exponentielle. Dans l’exponentielle, une partie dépend du taux moins le dividende et de la correction de variance ; une autre dépend de la volatilité et d’un choc normal aléatoire. La simulation se fait sous une mesure risque-neutre : nous n’utilisons pas une prévision de hausse de l’action. Chaque ligne de la matrice est un scénario et chaque colonne un constat, avec S0 dans la première colonne. Une graine fixe permet de retrouver les mêmes tirages. »


**Code à pointer à l’écran** — extraits de `strutu_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
    prix = np.zeros((n_simulations, n_dates + 1))
        dt = t - t_precedent  # temps écoulé depuis la dernière obs
        Z = rng.standard_normal(n_simulations)  # je tire un choc par scénario
        croissance = (r - q - sigma**2 / 2) * dt
        choc = sigma * np.sqrt(dt) * Z
        prix[:, i + 1] = prix[:, i] * np.exp(croissance + choc)
```

**Explication orale du code et lien avec la suite :**

Théo : « Le sigma expliqué par Joseph est utilisé ici. Nous créons un tableau avec une ligne par scénario et une colonne de plus que le nombre de constats pour conserver le cours initial. Dans prix[:, i], les deux-points signifient toutes les lignes : on met à jour tous les scénarios pour une même date. dt mesure le temps depuis le constat précédent ; Z contient un choc par scénario. »

Théo : « Cette fonction ne sait pas si nous valorisons un Phoenix ou un Plain. Elle renvoie seulement des cours. La partie Phoenix que je vais montrer et la partie Plain que Joseph présentera appliquent des règles différentes à ces trajectoires. C’est pour cela que nous réutilisons le même simulateur au lieu d’en écrire un par produit. »

### Expliquer le Phoenix

**Intervenant : Théo. À l’écran :** strutu_engine.py : coupons, mémoire et rappel.

**Repère :** [`strutu_engine.py` — `_simuler_phoenix`, ligne 51](../strutu_engine.py#L51).

« Le Phoenix vérifie à chaque constat les scénarios encore actifs. Si le seuil coupon est atteint, il verse le coupon. Sans mémoire, un coupon manqué est perdu ; avec mémoire, il est stocké puis rattrapé lorsqu’un constat devient favorable. Si le rappel est activé et sa condition atteinte avant maturité, le capital est rendu et le scénario s’arrête. À maturité, pour les scénarios restants, le principal est rendu intégralement au-dessus de la protection, sinon il suit le ratio entre le cours final et la référence. Avec une référence de 100, une protection à 60 et un cours final de 50, le principal remboursé vaut 50 % du nominal. »


**Code à pointer à l’écran** — extraits de `strutu_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
        actifs = ~rappele
        coupon_du = actifs & (S_k >= B_phoenix_level)
            coupons_en_attente += np.where(actifs, coupon_periode, 0.0)
            montant_coupon = np.where(coupon_du, coupons_en_attente, 0.0)
        coupons_actualises += montant_coupon * actualisation
        rappele = rappele | autocall_now
```

**Explication orale du code et lien avec la suite :**

Théo : « Ici, nous passons des cours aux flux. rappele mémorise les scénarios déjà terminés ; le signe tilde inverse ce tableau pour garder les actifs. Le symbole & exige que le scénario soit actif et que le seuil coupon soit atteint. np.where choisit un montant ou zéro pour chaque scénario. Avec mémoire, nous ajoutons d’abord le coupon au stock en attente, puis nous payons ce stock si la condition est remplie. La ligne suivante dans le fichier le remet à zéro après paiement. »

Théo : « Nous actualisons le coupon à sa propre date, car il peut être payé avant la sortie finale. Le symbole | conserve les rappels passés et ajoute ceux de cette date. Ces scénarios ne recevront plus de paiement aux constats suivants. Joseph va montrer une différence importante : dans le Plain, les coupons ne sont pas payés périodiquement, ils sont réglés ensemble à la sortie. »

**Transition vers Joseph :** « Le simulateur est commun. Je te laisse maintenant expliquer le Plain avec et sans rappel et la manière dont il utilise ces trajectoires. »

### Expliquer le Plain et l’option sans rappel

**Intervenant : Joseph. À l’écran :** strutu_engine.py : _flux_autocall et analyse_autocall_plain.

**Repère :** [`strutu_engine.py` — `_flux_autocall`, ligne 280](../strutu_engine.py#L280).

« Le Plain ne verse pas les coupons à chaque constat : il les accumule. Un rappel au deuxième constat avec un coupon de 4 % paie 100 % du nominal plus 8 %. Sans rappel, les coupons accumulés sont tous versés à maturité si le seuil final est atteint, sinon aucun. La case de rappel ne change que la possibilité de sortie anticipée : décochée, le produit va toujours à maturité. Un constat intermédiaire favorable ne suffit pas à acquérir un coupon. La protection conditionnelle du principal est une autre option, indépendante. Désactivée, le code rembourse toujours le nominal, sans modéliser le défaut de l’émetteur. »


**Code à pointer à l’écran** — extraits de `strutu_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
        au_dessus = prix[:, k + 1] >= K_auto * S0
                rappel_du = (~rappele) & au_dessus
                coupons_payes[rappel_du] = coupon * (k + 1)
                date_paiement[rappel_du] = dates[k]
                rappele[rappel_du] = True
```

**Explication orale du code et lien avec la suite :**

Joseph : « Nous récupérons les cours du simulateur présenté par Théo. La colonne k+1 correspond au constat courant, car la colonne zéro contient S0. Dans cette boucle, k commence à zéro : k+1 est donc aussi le nombre de coupons accumulés. À la deuxième observation, k vaut un, et deux coupons sont dus. »

Joseph : « rappel_du sélectionne les scénarios encore en vie au-dessus du seuil, à condition que la date autorise le rappel. L’écriture entre crochets modifie uniquement ces scénarios. Nous stockons leur coupon, leur date de paiement et le fait qu’ils sont terminés. La date ne sert pas seulement au calendrier : elle servira à actualiser le paiement dans _analyse_autocall, que je montrerai ensuite. »

### Expliquer précisément l’option sans rappel

**Intervenant : Joseph. À l’écran :** strutu_engine.py : analyse_autocall_plain puis _flux_autocall.

**Repère :** [`strutu_engine.py` — `analyse_autocall_plain`, ligne 376](../strutu_engine.py#L376).

« Dans la version intégrée, nous avons regroupé les deux configurations dans un seul produit. La fonction reçoit has_autocall, vrai par défaut. Lorsqu’il est faux, aucun scénario ne sort avant maturité. La condition des coupons reste finale : avec six constats à 4 %, on paie 24 % si le cours final atteint le seuil, sinon zéro coupon. Un cours au-dessus du seuil au premier constat puis en dessous à maturité ne crée donc aucun coupon acquis. Cette précision distingue la version actuelle de l’ancienne branche qui comptait les constats favorables. »


**Code à pointer à l’écran** — extraits de `strutu_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
        if has_autocall:
    coupon_final_du = non_rappeles & (S_T >= K_auto * S0)
    coupons_payes[coupon_final_du] = coupon * n_dates
```

**Explication orale du code et lien avec la suite :**

Joseph : « La case du formulaire devient has_autocall dans Python. Si elle est fausse, on ne rentre pas dans le bloc de rappel. Mais le bloc final reste exécuté : c’est lui qui teste le cours à maturité et attribue tous les coupons ou aucun. Nous utilisons donc un seul moteur avec une option, sans créer un produit séparé. Théo montrera dans main.py comment la case est lue puis transmise jusqu’à cette fonction. »

**Transition vers Théo :** « Le rappel est une option ; la protection du capital en est une autre. Je te laisse préciser cette distinction. »

### Distinguer rappel et protection du capital

**Intervenant : Théo. À l’écran :** Application : les deux cases du Plain ; strutu_engine.py : perte_capital.

**Repère :** [`strutu_engine.py` — `_flux_autocall`, ligne 280](../strutu_engine.py#L280).

« Le rappel anticipé et la protection conditionnelle sont deux mécanismes séparés. La première case décide si le produit peut se terminer avant la date finale. La seconde détermine le remboursement du principal à maturité. Avec une barrière à 60 %, un cours final à 50 % de S0 donne un principal de 50 % du nominal, pas de 90 %. Si la protection conditionnelle est désactivée dans ce code, le principal reste égal au nominal. Un remboursement nominal dans le modèle n’élimine pas le risque de défaut d’un émetteur réel. »


**Code à pointer à l’écran** — extraits de `strutu_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
    remboursement = np.ones(n_simulations)
        perte_capital = non_rappeles & (S_T < B_pdi * S0)
        remboursement[perte_capital] = S_T[perte_capital] / S0
```

**Explication orale du code et lien avec la suite :**

Théo : « Le tableau remboursement commence à un, soit 100 % du nominal. Si une barrière est définie, seules les trajectoires non rappelées et strictement sous cette barrière sont remplacées par le ratio final. À égalité, le nominal reste entier. Cette sélection est distincte de celle des coupons. Joseph va maintenant utiliser ce tableau et les dates de paiement pour obtenir la valeur actuelle. »

**Transition vers Joseph :** « À partir de ces remboursements, comment obtient-on la valeur aujourd’hui ? »

### Expliquer la valeur théorique

**Intervenant : Joseph. À l’écran :** strutu_engine.py : flux actualisés et statistiques.

**Repère :** [`strutu_engine.py` — `_analyse_autocall`, ligne 332](../strutu_engine.py#L332).

« Chaque scénario produit des paiements à une ou plusieurs dates. Nous multiplions chaque paiement par exponentielle de moins r fois t pour le ramener à aujourd’hui. La valeur théorique est ensuite la moyenne de ces flux actualisés. Le moteur travaille pour un nominal de un : une valeur de 0,95 signifie 95 % du nominal, soit 950 pour 1 000 investis. La composante de remboursement du capital est déjà incluse ; ajouter séparément un zéro-coupon compterait le capital une deuxième fois. L’intervalle de confiance mesure le bruit de Monte Carlo, pas l’incertitude sur le modèle financier. »


**Code à pointer à l’écran** — extraits de `strutu_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
    actualisation = np.exp(-r * res["date_paiement"])
    flux = (res["remboursement"] + res["coupons"]) * actualisation
    valeur = float(np.mean(flux))
    erreur_type = float(np.std(flux, ddof=1) / np.sqrt(n_simulations))
```

**Explication orale du code et lien avec la suite :**

Joseph : « Le dictionnaire res rassemble les tableaux de paiements calculés juste avant. Nous additionnons le principal et les coupons, puis actualisons chaque ligne à sa date de sortie. np.mean fait ensuite la moyenne entre les scénarios : c’est ici que les trajectoires deviennent un prix. Le moteur renvoie un dictionnaire de résultats ; Théo montrera comment main.py le transforme en valeur monétaire et en indicateurs pour le navigateur. »

Joseph : « La dernière ligne mesure la précision statistique de cette moyenne. Ce n’est pas l’écart-type des cours ni une probabilité de perte : c’est une erreur type sur l’estimation du prix, utilisée pour construire son intervalle de confiance. »

**Transition vers Théo :** « Je te laisse relier cette valeur aux indicateurs affichés. »

### Lire les indicateurs et le graphique

**Intervenant : Théo. À l’écran :** Résultats du Plain et _analyse_autocall.

**Repère :** [`strutu_engine.py` — `_analyse_autocall`, ligne 332](../strutu_engine.py#L332).

« À partir des mêmes trajectoires, nous calculons le prix, la durée de vie moyenne et les probabilités de sortie. La probabilité de perte concerne le principal avant les coupons. Le remboursement moyen si perte ne concerne que les trajectoires en perte. La marge affichée est simplement 100 % moins la valeur théorique, avant coûts et défaut. Enfin, le graphique montre des remboursements futurs non actualisés en fonction du cours final. Ce n’est ni le prix actuel ni une représentation de toutes les trajectoires. »


**Code à pointer à l’écran** — extraits de `strutu_engine.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
        "proba_maturite": float(np.mean(res["date_rappel"] == -1)),
        "duree_de_vie_moyenne": float(np.mean(res["date_paiement"])),
```

**Explication orale du code et lien avec la suite :**

Théo : « Le même dictionnaire que Joseph vient d’utiliser contient aussi les informations de sortie. Le code -1 signifie jamais rappelé. La comparaison donne un tableau vrai ou faux et sa moyenne donne une proportion, car vrai compte pour un. La durée moyenne utilise les dates de paiement déjà déterminées par le moteur. Nous ne lançons donc pas de nouveaux scénarios pour chaque indicateur. »

**Transition vers l’intégration :** « Ces indicateurs sont renvoyés par le serveur. Je vais maintenant montrer comment ils arrivent dans l’application. »

### Montrer l’intégration et une démonstration Phoenix

**Intervenant : Théo. À l’écran :** main.py : api_phoenix ; application : Phoenix.

**Repère :** [`main.py` — `api_phoenix`, ligne 218](../main.py#L218).

« Les paramètres sont saisis dans l’interface, puis transmis à Flask. Le serveur convertit par exemple 25 % en 0,25, vérifie les données et appelle le moteur. La réponse contient le prix, les statistiques et les points du graphique. Je montre ici le Phoenix et sa case mémoire : elle stocke les coupons manqués pour les rattraper à un constat favorable. Son graphique final ne montre pas les coupons déjà versés ni toute la mémoire. Pour comparer précisément deux prix, il faut aussi contrôler les tirages aléatoires. »


**Code à pointer à l’écran** — extraits de `main.py` ; les lignes peuvent appartenir à plusieurs passages du fichier.

```python
        sigma = float(data["sigma"]) / 100
        has_autocall = _to_bool(data.get("has_autocall", True), "has_autocall")
        res = analyse_autocall_plain(**arguments, first_call_date=premiere, has_autocall=has_autocall)
```

**Explication orale du code et lien avec la suite :**

Théo : « La première ligne se trouve dans la route Phoenix ; la conversion existe aussi pour le Plain. Nous passons du pourcentage du formulaire au décimal du moteur. Pour suivre l’option expliquée par Joseph, ouvrons ensuite api_autocall : _to_bool lit la case et le dernier appel transmet has_autocall au moteur. Les deux étoiles devant arguments déplient le dictionnaire en paramètres nommés, comme S0, r, sigma et coupon. »

Théo : « Le moteur renvoie une fraction du nominal ; main.py multiplie par le nominal pour le montant, ou par cent pour le pourcentage. jsonify prépare la réponse JSON. Dans index.html, fetch envoie le formulaire et lit cette réponse ; renderStructuredMetrics affiche les indicateurs, renderChart construit le graphique SVG. Les règles financières restent dans Python, tandis que JavaScript organise les échanges et l’affichage. Joseph va maintenant utiliser ce parcours complet pour comparer les deux configurations du Plain. »

**Transition vers Joseph :** « Joseph, à toi de montrer maintenant les deux configurations du Plain dans cette même interface. »

### Démontrer le Plain avec puis sans rappel

**Intervenant : Joseph. À l’écran :** Application : Plain, mêmes paramètres et graine 42.

« Je prends trois ans, des constats semestriels, un coupon de 4 % par observation et un nominal de 1 000. Je calcule avec rappel, puis je décoche uniquement le rappel en conservant les autres paramètres et la graine. Sans rappel, la durée moyenne est trois ans, la probabilité de maturité est 100 % et les rappels sont nuls. À maturité, si le cours atteint le seuil final, les six coupons représentent 240, ajoutés au principal de 1 000. Le prix aujourd’hui est calculé par moyenne de flux actualisés ; il ne faut donc pas le confondre avec ces 1 240 futurs. »

### Présenter les vérifications

**Intervenant : Théo. À l’écran :** tests/test_api.py et tests/test_engine.py.

« Les tests contrôlent les conversions, les paramètres invalides, les égalités aux barrières et des trajectoires dont les paiements sont connus. Ils vérifient aussi le mode sans rappel : aucun rappel et aucune acquisition de coupon sur un simple constat intermédiaire favorable. Les quatorze tests passent dans la version préparée. Ces tests vérifient la logique et les routes ; ils ne garantissent pas la disponibilité des données externes. »

### Conclure sur les limites

**Intervenant : Joseph. À l’écran :** Application : résultats, puis écran final.

« Le pricer relie des données de marché, des règles de paiement et une valorisation par simulation. Ses limites sont un taux, un dividende et une volatilité constants, sans smile ni défaut de l’émetteur. Les probabilités sont risque-neutres et non prédictives. Les principales extensions seraient une courbe d’actualisation et une calibration aux options. »

