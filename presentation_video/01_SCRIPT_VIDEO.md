# Script oral — Théo et Joseph

Version plus naturelle : les phrases servent de repères. Vous pouvez changer les mots et vous arrêter pour montrer le code. Les indications « À l’écran » ne se lisent pas. Prévoir environ **15 à 18 minutes**, selon les manipulations et le rythme, à vérifier en répétition.

## 1. Présenter le projet — Théo

**À l’écran :** Application.

On va vous présenter notre pricer de produits structurés. On a deux produits : le Phoenix et le Plain, avec ou sans rappel. On va vous montrer comment on passe des données de marché au prix, puis faire un calcul dans l’application.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [templates/index.html, ligne 156](../templates/index.html#L156)** — lignes 156 à 156.

```html
          <option value="autocall_plain">Autocall Plain — coupons accumulés</option>
```

**Pendant la vidéo :** privilégier ici la page de l’application ; le HTML est un repère complémentaire si vous voulez montrer la commande correspondante.

## 2. Montrer les fichiers — Théo

**À l’écran :** main.py : imports en haut du fichier.

Pour se repérer, main.py fait le lien entre la page et les calculs. Market data récupère les données, volatility calcule la volatilité, et strutu_engine contient la simulation et les règles des produits. Ici, les imports permettent de réutiliser les fonctions de ces fichiers.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [main.py, ligne 4](../main.py#L4)** — lignes 4 à 6.

```python
from market_data_engine import search_tickers, get_spot, get_devise, get_vix, get_vol_historique
from strutu_engine import (analyse_phoenix, payoff_phoenix_maturite,
                           analyse_autocall_plain)
```

## 3. Récupérer les cours — Théo

**À l’écran :** market_data_engine.py : get_historical_prices et get_vol_historique.

Ici, on récupère les cours sur Yahoo. On garde les clôtures ajustées et on retire les valeurs manquantes. Ensuite, dans get_vol_historique, on passe ces cours à historical_volatility. Cette fonction est importée depuis volatility_engine : on ne refait donc pas le calcul ici. On multiplie juste le résultat par cent pour l’afficher en pourcentage. Joseph va vous montrer ce qu’il y a dans cette fonction.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [market_data_engine.py, ligne 3](../market_data_engine.py#L3)** — lignes 3 à 3.

```python
from volatility_engine import historical_volatility
```

**Ouvrir [market_data_engine.py, ligne 89](../market_data_engine.py#L89)** — lignes 89 à 101.

```python
    cours = get_historical_prices(ticker, periode)

    # 2. Vérification du nombre de cours
    if len(cours) < 20:
        raise ValueError(
            "Pas assez d'historique pour calculer une volatilité."
        )

    # 3. Calcul de la volatilité historique
    vol = historical_volatility(cours)

    # 4. Conversion en pourcentage
    return float(vol * 100)
```

**Ouvrir [market_data_engine.py, ligne 54](../market_data_engine.py#L54)** — lignes 54 à 66.

```python
    data = asset.history(
        period=periode,
        auto_adjust=True
    )

    # 3. Vérification des données
    if data.empty:
        raise ValueError(
            f"Aucune donnée historique disponible pour {ticker}"
        )

    # 4. Extraction des cours de clôture
    prices = data["Close"].dropna()
```

## 4. Calculer la volatilité — Joseph

**À l’écran :** volatility_engine.py : log_returns et historical_volatility.

Donc ici, on reçoit les cours que Théo vient de montrer. On commence par calculer les rendements. Avec prices[1:] et prices[:-1], on met chaque cours en face du précédent, puis on prend le logarithme du ratio. Avec trois cours, on obtient deux rendements.

Ensuite, np.std calcule leur dispersion. Le ddof=1, c’est pour utiliser le nombre de rendements moins un dans le calcul de l’écart-type. Et on multiplie par racine de 252 pour passer d’une volatilité journalière à une volatilité annuelle.

La fonction renvoie un décimal : 0,25, c’est 25 %. Il faut au moins trois cours pour ce calcul. Dans market data, on en demande vingt : c’est un minimum supplémentaire pour l’application.

Cette volatilité va devenir sigma dans le simulateur. Théo, tu peux montrer où elle intervient ?


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [volatility_engine.py, ligne 32](../volatility_engine.py#L32)** — lignes 32 à 32.

```python
    returns = np.log(prices[1:] / prices[:-1])
```

**Ouvrir [volatility_engine.py, ligne 58](../volatility_engine.py#L58)** — lignes 58 à 79.

```python
    returns = log_returns(prices)

    # 2. Vérification du nombre de rendements
    if len(returns) < 2:
        raise ValueError(
        "Il faut au moins trois prix pour calculer la volatilité."
    )

    # 3. Vérification du facteur d'annualisation
    if annualization <= 0:
        raise ValueError(
        "Le facteur d'annualisation doit être positif."
    )

    # 4. Calcul de l'écart-type des rendements
    daily_volatility = np.std(returns, ddof=1)

    # 5. Annualisation
    annual_volatility = daily_volatility * np.sqrt(annualization)

    # 6. Retour en décimal
    return float(annual_volatility)
```

## 5. Simuler les cours — Théo

**À l’écran :** strutu_engine.py : simulate_observation_dates.

On retrouve sigma ici. Le tableau prix contient une ligne par scénario, et une colonne par date, plus le cours de départ. Les deux-points dans prix[:, i], ça veut dire qu’on prend tous les scénarios à cette date.

À chaque étape, on calcule le temps écoulé, dt, puis on tire un choc aléatoire par scénario avec standard_normal. Le prochain cours dépend du taux, du dividende, de la volatilité et de ce choc. On simule sous une mesure risque-neutre, avec une dérive r moins q : ce n’est pas une prévision du cours de l’action.

Cette fonction renvoie seulement les cours. Ensuite, on applique les règles du Phoenix ou du Plain dessus. Le simulateur est commun aux deux.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [strutu_engine.py, ligne 29](../strutu_engine.py#L29)** — lignes 29 à 44.

```python
    n_dates = len(obs_dates)
    prix = np.zeros((n_simulations, n_dates + 1))
    prix[:, 0] = S0  # je fais partir tous les scénarios du même spot

    t_precedent = 0.0
    for i in range(n_dates):
        t = obs_dates[i]
        dt = t - t_precedent  # temps écoulé depuis la dernière obs
        Z = rng.standard_normal(n_simulations)  # je tire un choc par scénario
        # mouvement brownien géométrique (j'ai repris la formule de mon Monte Carlo options)
        croissance = (r - q - sigma**2 / 2) * dt
        choc = sigma * np.sqrt(dt) * Z
        prix[:, i + 1] = prix[:, i] * np.exp(croissance + choc)
        t_precedent = t

    return prix
```

## 6. Appliquer les règles du Phoenix — Théo

**À l’écran :** strutu_engine.py : _simuler_phoenix.

Pour le Phoenix, on regarde d’abord quels scénarios sont encore en vie. C’est actifs = ~rappele : on écarte ceux qui sont déjà terminés.

Ensuite, on teste le seuil du coupon. Sans mémoire, si le seuil n’est pas atteint, le coupon est perdu. Avec mémoire, on le garde dans coupons_en_attente. Dès qu’un constat est favorable, on paie le stock, puis on le remet à zéro.

Par exemple, avec un coupon de 4 %, si on manque le premier mais qu’on touche le deuxième, on reçoit 8 % avec mémoire, contre 4 % sans mémoire.

Si le produit est rappelé avant maturité, on rend aussi le capital et on arrête ce scénario. Sinon, le capital est traité à la fin selon la barrière de protection. On peut aussi décocher le rappel : le Phoenix va alors jusqu’à maturité, mais les coupons continuent à être versés aux constats favorables, avec ou sans mémoire. Ce n’est donc pas le même fonctionnement que le Plain sans rappel, que Joseph va montrer.

Il y a aussi le bonus final. On choisit son seuil et son montant. Si le produit n’a pas été rappelé et que le cours final atteint ce seuil, on ajoute le bonus au remboursement. Par exemple, avec une référence de 100, un seuil bonus à 110 et un bonus de 5 %, un cours final de 115 donne un bonus de 50 pour 1 000 investis. Le dernier coupon s’ajoute aussi si sa propre condition est remplie.

Dans le code, k < n_dates réserve le rappel aux dates avant maturité. La dernière date passe par le remboursement final, ce qui permet justement de traiter le bonus. Joseph va maintenant montrer le Plain, où les coupons sont payés d’un coup.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [strutu_engine.py, ligne 86](../strutu_engine.py#L86)** — lignes 86 à 105.

```python
        actifs = ~rappele

        # je bloque le rappel à la dernière date, à maturité je passe par le remboursement final
        # (sinon mon bonus ne serait jamais versé)
        if has_autocall and k < n_dates:
            autocall_now = actifs & (S_k >= K_auto_level)
        else:
            autocall_now = np.zeros(n_simulations, dtype=bool)

        coupon_du = actifs & (S_k >= B_phoenix_level)

        coupon_periode = coupon_schedule[k - 1]
        if memory:
            # j'accumule le coupon de la période, et je paie tout le stock dès que c'est au-dessus
            coupons_en_attente += np.where(actifs, coupon_periode, 0.0)
            montant_coupon = np.where(coupon_du, coupons_en_attente, 0.0)
            coupons_en_attente = np.where(coupon_du, 0.0, coupons_en_attente)  # je remets à 0 si c'est payé
        else:
            # sans mémoire un coupon raté est perdu
            montant_coupon = np.where(coupon_du, coupon_periode, 0.0)
```

**Ouvrir [strutu_engine.py, ligne 122](../strutu_engine.py#L122)** — lignes 122 à 127.

```python
    remboursement_T = np.where(S_T >= B_pdi_level, 1.0, S_T / strike_reference)
    if bonus_threshold is not None:
        bonus_du = non_rappelees & (S_T >= bonus_threshold * strike_reference)
        remboursement_T = remboursement_T + np.where(bonus_du, bonus_amount, 0.0)

    flux_actualises += np.where(non_rappelees, remboursement_T * actualisation_T, 0.0)
```

**Ordre à montrer :** activité → rappel activé ou non → coupons et mémoire → capital final et bonus. Le bonus est ajouté au remboursement final ; les coupons ont déjà été traités dans la boucle.

## 7. Expliquer le Plain — Joseph

**À l’écran :** strutu_engine.py : _flux_autocall.

Dans le Plain, on ne verse pas les coupons à chaque date. On attend la sortie du produit.

Ici, rappel_du sélectionne les scénarios qui peuvent être rappelés. Pour ceux-là, on met de côté le montant des coupons et la date du paiement. Le coupon vaut coupon fois k+1, parce que k commence à zéro. Au deuxième constat, ça fait donc deux coupons : avec 4 %, on rend 100 % du capital plus 8 %.

Les crochets permettent de modifier seulement les scénarios concernés. Et on les marque comme rappelés pour ne plus les payer ensuite.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [strutu_engine.py, ligne 296](../strutu_engine.py#L296)** — lignes 296 à 307.

```python
    for k in range(n_dates):
        au_dessus = prix[:, k + 1] >= K_auto * S0
        if has_autocall:
            rappel_autorise = first_call_date is None or dates[k] >= first_call_date
            # La dernière observation relève du remboursement final, pas d'un rappel.
            if k < n_dates - 1 and rappel_autorise:
                rappel_du = (~rappele) & au_dessus
                coupons_payes[rappel_du] = coupon * (k + 1)
                nombre_coupons[rappel_du] = k + 1
                date_paiement[rappel_du] = dates[k]
                date_rappel[rappel_du] = k
                rappele[rappel_du] = True
```

## 8. Désactiver le rappel — Joseph

**À l’écran :** strutu_engine.py : has_autocall et coupon_final_du.

La case de rappel devient has_autocall dans le moteur. Si elle est décochée, on ne passe plus dans le bloc de rappel. Le produit reste donc jusqu’à maturité.

Par contre, on garde la condition finale des coupons. Avec six constats à 4 %, on reçoit 24 % si le seuil final est atteint, sinon aucun coupon. Un bon constat au début ne suffit pas à gagner un coupon. C’est bien le niveau final qui compte dans ce mode.

Théo, tu peux préciser l’autre case, celle de la protection ?


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [strutu_engine.py, ligne 309](../strutu_engine.py#L309)** — lignes 309 à 314.

```python
    non_rappeles = ~rappele
    S_T = prix[:, -1]
    # Le rappel est optionnel ; la condition finale des coupons reste identique.
    coupon_final_du = non_rappeles & (S_T >= K_auto * S0)
    coupons_payes[coupon_final_du] = coupon * n_dates
    nombre_coupons[coupon_final_du] = n_dates
```

**Ouvrir [strutu_engine.py, ligne 376](../strutu_engine.py#L376)** — lignes 376 à 377.

```python
def analyse_autocall_plain(S0, r, q, sigma, obs_dates, K_auto, B_pdi, coupon,
                            n_simulations=50000, first_call_date=None, seed=None, has_autocall=True):
```

## 9. Distinguer les deux cases — Théo

**À l’écran :** strutu_engine.py : remboursement et perte_capital.

Oui, les deux cases sont indépendantes. Le rappel décide si on peut sortir avant la fin. La protection conditionnelle décide ce qu’on récupère à maturité.

Le tableau remboursement commence à un : ça représente tout le nominal. Si le cours finit sous la barrière de protection, on remplace ce montant par S_T divisé par S0. Donc avec un départ à 100 et une arrivée à 50, on récupère la moitié du capital. À la barrière exactement, le nominal est remboursé.

Si on désactive cette perte conditionnelle, le code rend toujours le nominal. Joseph, à partir de ces paiements, comment on obtient le prix aujourd’hui ?


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [strutu_engine.py, ligne 316](../strutu_engine.py#L316)** — lignes 316 à 319.

```python
    perte_capital = np.zeros(n_simulations, dtype=bool)
    if B_pdi is not None:
        perte_capital = non_rappeles & (S_T < B_pdi * S0)
        remboursement[perte_capital] = S_T[perte_capital] / S0
```

## 10. Calculer la valeur actuelle — Joseph

**À l’écran :** strutu_engine.py : _analyse_autocall.

On récupère les montants et les dates de paiement calculés juste avant. On additionne le capital et les coupons, puis on les multiplie par exp de moins r fois la date de paiement. C’est l’actualisation.

Ensuite, np.mean fait la moyenne de tous les scénarios. C’est cette moyenne qui donne le prix. Le moteur travaille pour un nominal de un : 0,95 signifie 95 %, donc 950 pour 1 000 investis.

Le capital est déjà dans ce calcul. On n’a donc pas à ajouter un zéro-coupon à côté.

Juste après, on calcule l’erreur type et l’intervalle de confiance. Ça mesure la précision de la simulation, pas si toutes les hypothèses du modèle sont bonnes. Théo va maintenant relier ces résultats à ce qu’on voit à l’écran.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [strutu_engine.py, ligne 346](../strutu_engine.py#L346)** — lignes 346 à 351.

```python
    actualisation = np.exp(-r * res["date_paiement"])
    flux = (res["remboursement"] + res["coupons"]) * actualisation
    if not np.all(np.isfinite(flux)):
        raise ValueError("Flux actualisés non finis : vérifiez les paramètres.")
    valeur = float(np.mean(flux))
    erreur_type = float(np.std(flux, ddof=1) / np.sqrt(n_simulations))
```

## 11. Lire les résultats — Théo

**À l’écran :** Application et _analyse_autocall.

Le moteur renvoie aussi les statistiques des mêmes scénarios. Par exemple, date_rappel égal à moins un, ça veut dire jamais rappelé. La moyenne de cette condition donne la proportion qui arrive à maturité.

La probabilité de perte concerne le capital, avant les coupons. Et la marge affichée, c’est simplement cent moins le prix : ce n’est pas un bénéfice net.

Le graphique, lui, montre ce qu’on reçoit à maturité si le produit n’a pas été rappelé. Ce sont des montants futurs, pas le prix aujourd’hui. Pour le Phoenix, il ne montre que le dernier coupon, pas les coupons déjà reçus ni tout l’effet mémoire.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [strutu_engine.py, ligne 363](../strutu_engine.py#L363)** — lignes 363 à 368.

```python
        "proba_rappel_par_date": [float(np.mean(res["date_rappel"] == k)) for k in range(len(dates))],
        "proba_maturite": float(np.mean(res["date_rappel"] == -1)),
        "proba_perte_capital": float(np.mean(perte)),
        "remboursement_moyen_si_perte": remboursement_si_perte,
        "duree_de_vie_moyenne": float(np.mean(res["date_paiement"])),
        "coupons_moyens_actualises": float(np.mean(res["coupons"] * actualisation)),
```

## 12. Relier le calcul à la page — Théo

**À l’écran :** main.py : api_phoenix et api_autocall ; index.html : fetch.

On revient dans main.py. La page envoie les paramètres ici. On vérifie les données et on convertit les unités : par exemple, 25 dans le formulaire devient 0,25 pour sigma.

Pour le Plain, on lit aussi has_autocall et on le transmet à la fonction que Joseph a montrée. Les deux étoiles devant arguments servent à passer les valeurs du dictionnaire comme paramètres nommés.

Après le calcul, on renvoie les résultats en JSON. Dans le HTML, fetch récupère la réponse, puis les fonctions d’affichage mettent à jour les indicateurs et le graphique. Le JavaScript gère donc les échanges et l’affichage ; les calculs financiers restent en Python.

Je montre rapidement la case mémoire du Phoenix, puis je décoche le rappel : les coupons restent périodiques, mais le produit va jusqu’à maturité. Je montre aussi les deux champs du bonus, son seuil et son montant. Le bonus n’est payé qu’à la fin, si sa condition est remplie et si le produit est encore en vie. Ensuite Joseph va comparer le Plain avec et sans rappel.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [main.py, ligne 232](../main.py#L232)** — lignes 232 à 232.

```python
        sigma = float(data["sigma"]) / 100
```

**Ouvrir [main.py, ligne 378](../main.py#L378)** — lignes 378 à 378.

```python
        res = analyse_autocall_plain(**arguments, first_call_date=premiere, has_autocall=has_autocall)
```

**Ouvrir [main.py, ligne 297](../main.py#L297)** — lignes 297 à 298.

```python
            "prix": round(res["prix"] * nominal, 4),
            "prix_pct": pct(res["prix"]),
```

**Ouvrir [templates/index.html, ligne 729](../templates/index.html#L729)** — lignes 729 à 732.

```javascript
      const response=await fetch('/api/autocall',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(payload),signal:controller.signal});
      const data=await response.json();if(!response.ok)throw new Error(data.error||'Calcul indisponible.');
      if(version!==acSelectionVersion)return;
      afficherAutocall(data);status.textContent='Calcul terminé.';
```

## 13. Faire la démonstration du Plain — Joseph

**À l’écran :** Application : Plain.

On prend trois ans, des constats semestriels et 4 % de coupon par constat. On garde les mêmes paramètres et la même graine pour comparer.

Je calcule d’abord avec rappel. Puis je décoche seulement cette case et je recalcule. Sans rappel, on doit retrouver trois ans de durée moyenne, cent pour cent à maturité et aucun rappel.

Si le seuil final est atteint, les six coupons font 24 %. Pour 1 000 investis, ça donne 1 240 à maturité. Ce montant futur et la valeur actuelle affichée ne sont pas la même chose.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [templates/index.html, ligne 253](../templates/index.html#L253)** — lignes 253 à 253.

```html
      <label class="check"><input id="ac-plain-autocall" type="checkbox" checked> Remboursement anticipé possible (autocall)</label>
```

**Ouvrir [templates/index.html, ligne 724](../templates/index.html#L724)** — lignes 724 à 724.

```javascript
    if(kind==='plain'){payload.has_autocall=$(prefix+'-autocall').checked;payload.first_call_months=payload.has_autocall?get('first'):0;}
```

**Pendant la vidéo :** privilégier ici la page de l’application ; le HTML est un repère complémentaire si vous voulez montrer la commande correspondante.

## 15. Terminer sur les limites — Joseph

**À l’écran :** Application : résultats.

Le modèle reste simplifié : le taux, le dividende et la volatilité sont constants. On ne prend pas en compte le smile, les frais de couverture ni le défaut de l’émetteur. Et les probabilités sont celles du modèle risque-neutre, pas des prévisions. Pour aller plus loin, on pourrait utiliser une courbe de taux et calibrer la volatilité sur les options.


**Code à montrer — indications non destinées à être lues :**

**Ouvrir [strutu_engine.py, ligne 39](../strutu_engine.py#L39)** — lignes 39 à 40.

```python
        croissance = (r - q - sigma**2 / 2) * dt
        choc = sigma * np.sqrt(dt) * Z
```

