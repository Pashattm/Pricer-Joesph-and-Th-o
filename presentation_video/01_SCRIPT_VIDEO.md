# Script oral — Théo et Joseph

Version plus naturelle : les phrases servent de repères. Vous pouvez changer les mots et vous arrêter pour montrer le code. Les indications « À l’écran » ne se lisent pas. Prévoir environ **15 à 18 minutes**, selon les manipulations et le rythme, à vérifier en répétition.

## 1. Présenter le projet — Théo

**À l’écran :** Application.

On va vous présenter notre pricer de produits structurés. On a deux produits : le Phoenix et le Plain, avec ou sans rappel. On va vous montrer comment on passe des données de marché au prix, puis faire un calcul dans l’application.

## 2. Montrer les fichiers — Théo

**À l’écran :** main.py : imports en haut du fichier.

Pour se repérer, main.py fait le lien entre la page et les calculs. Market data récupère les données, volatility calcule la volatilité, et strutu_engine contient la simulation et les règles des produits. Ici, les imports permettent de réutiliser les fonctions de ces fichiers.

## 3. Récupérer les cours — Théo

**À l’écran :** market_data_engine.py : get_historical_prices et get_vol_historique.

Ici, on récupère les cours sur Yahoo. On garde les clôtures ajustées et on retire les valeurs manquantes. Ensuite, dans get_vol_historique, on passe ces cours à historical_volatility. Cette fonction est importée depuis volatility_engine : on ne refait donc pas le calcul ici. On multiplie juste le résultat par cent pour l’afficher en pourcentage. Joseph va vous montrer ce qu’il y a dans cette fonction.

## 4. Calculer la volatilité — Joseph

**À l’écran :** volatility_engine.py : log_returns et historical_volatility.

Donc ici, on reçoit les cours que Théo vient de montrer. On commence par calculer les rendements. Avec prices[1:] et prices[:-1], on met chaque cours en face du précédent, puis on prend le logarithme du ratio. Avec trois cours, on obtient deux rendements.

Ensuite, np.std calcule leur dispersion. Le ddof=1, c’est pour utiliser le nombre de rendements moins un dans le calcul de l’écart-type. Et on multiplie par racine de 252 pour passer d’une volatilité journalière à une volatilité annuelle.

La fonction renvoie un décimal : 0,25, c’est 25 %. Il faut au moins trois cours pour ce calcul. Dans market data, on en demande vingt : c’est un minimum supplémentaire pour l’application.

Cette volatilité va devenir sigma dans le simulateur. Théo, tu peux montrer où elle intervient ?

## 5. Simuler les cours — Théo

**À l’écran :** strutu_engine.py : simulate_observation_dates.

On retrouve sigma ici. Le tableau prix contient une ligne par scénario, et une colonne par date, plus le cours de départ. Les deux-points dans prix[:, i], ça veut dire qu’on prend tous les scénarios à cette date.

À chaque étape, on calcule le temps écoulé, dt, puis on tire un choc aléatoire par scénario avec standard_normal. Le prochain cours dépend du taux, du dividende, de la volatilité et de ce choc. On simule sous une mesure risque-neutre, avec une dérive r moins q : ce n’est pas une prévision du cours de l’action.

Cette fonction renvoie seulement les cours. Ensuite, on applique les règles du Phoenix ou du Plain dessus. Le simulateur est commun aux deux.

## 6. Appliquer les règles du Phoenix — Théo

**À l’écran :** strutu_engine.py : _simuler_phoenix.

Pour le Phoenix, on regarde d’abord quels scénarios sont encore en vie. C’est actifs = ~rappele : on écarte ceux qui sont déjà terminés.

Ensuite, on teste le seuil du coupon. Sans mémoire, si le seuil n’est pas atteint, le coupon est perdu. Avec mémoire, on le garde dans coupons_en_attente. Dès qu’un constat est favorable, on paie le stock, puis on le remet à zéro.

Par exemple, avec un coupon de 4 %, si on manque le premier mais qu’on touche le deuxième, on reçoit 8 % avec mémoire, contre 4 % sans mémoire.

Si le produit est rappelé avant maturité, on rend aussi le capital et on arrête ce scénario. Sinon, le capital est traité à la fin selon la barrière de protection. Joseph va montrer la différence avec le Plain, où les coupons sont payés d’un coup.

## 7. Expliquer le Plain — Joseph

**À l’écran :** strutu_engine.py : _flux_autocall.

Dans le Plain, on ne verse pas les coupons à chaque date. On attend la sortie du produit.

Ici, rappel_du sélectionne les scénarios qui peuvent être rappelés. Pour ceux-là, on met de côté le montant des coupons et la date du paiement. Le coupon vaut coupon fois k+1, parce que k commence à zéro. Au deuxième constat, ça fait donc deux coupons : avec 4 %, on rend 100 % du capital plus 8 %.

Les crochets permettent de modifier seulement les scénarios concernés. Et on les marque comme rappelés pour ne plus les payer ensuite.

## 8. Désactiver le rappel — Joseph

**À l’écran :** strutu_engine.py : has_autocall et coupon_final_du.

La case de rappel devient has_autocall dans le moteur. Si elle est décochée, on ne passe plus dans le bloc de rappel. Le produit reste donc jusqu’à maturité.

Par contre, on garde la condition finale des coupons. Avec six constats à 4 %, on reçoit 24 % si le seuil final est atteint, sinon aucun coupon. Un bon constat au début ne suffit pas à gagner un coupon. C’est bien le niveau final qui compte dans ce mode.

Théo, tu peux préciser l’autre case, celle de la protection ?

## 9. Distinguer les deux cases — Théo

**À l’écran :** strutu_engine.py : remboursement et perte_capital.

Oui, les deux cases sont indépendantes. Le rappel décide si on peut sortir avant la fin. La protection conditionnelle décide ce qu’on récupère à maturité.

Le tableau remboursement commence à un : ça représente tout le nominal. Si le cours finit sous la barrière de protection, on remplace ce montant par S_T divisé par S0. Donc avec un départ à 100 et une arrivée à 50, on récupère la moitié du capital. À la barrière exactement, le nominal est remboursé.

Si on désactive cette perte conditionnelle, le code rend toujours le nominal. Joseph, à partir de ces paiements, comment on obtient le prix aujourd’hui ?

## 10. Calculer la valeur actuelle — Joseph

**À l’écran :** strutu_engine.py : _analyse_autocall.

On récupère les montants et les dates de paiement calculés juste avant. On additionne le capital et les coupons, puis on les multiplie par exp de moins r fois la date de paiement. C’est l’actualisation.

Ensuite, np.mean fait la moyenne de tous les scénarios. C’est cette moyenne qui donne le prix. Le moteur travaille pour un nominal de un : 0,95 signifie 95 %, donc 950 pour 1 000 investis.

Le capital est déjà dans ce calcul. On n’a donc pas à ajouter un zéro-coupon à côté.

Juste après, on calcule l’erreur type et l’intervalle de confiance. Ça mesure la précision de la simulation, pas si toutes les hypothèses du modèle sont bonnes. Théo va maintenant relier ces résultats à ce qu’on voit à l’écran.

## 11. Lire les résultats — Théo

**À l’écran :** Application et _analyse_autocall.

Le moteur renvoie aussi les statistiques des mêmes scénarios. Par exemple, date_rappel égal à moins un, ça veut dire jamais rappelé. La moyenne de cette condition donne la proportion qui arrive à maturité.

La probabilité de perte concerne le capital, avant les coupons. Et la marge affichée, c’est simplement cent moins le prix : ce n’est pas un bénéfice net.

Le graphique, lui, montre ce qu’on reçoit à maturité si le produit n’a pas été rappelé. Ce sont des montants futurs, pas le prix aujourd’hui. Pour le Phoenix, il ne montre que le dernier coupon, pas les coupons déjà reçus ni tout l’effet mémoire.

## 12. Relier le calcul à la page — Théo

**À l’écran :** main.py : api_phoenix et api_autocall ; index.html : fetch.

On revient dans main.py. La page envoie les paramètres ici. On vérifie les données et on convertit les unités : par exemple, 25 dans le formulaire devient 0,25 pour sigma.

Pour le Plain, on lit aussi has_autocall et on le transmet à la fonction que Joseph a montrée. Les deux étoiles devant arguments servent à passer les valeurs du dictionnaire comme paramètres nommés.

Après le calcul, on renvoie les résultats en JSON. Dans le HTML, fetch récupère la réponse, puis les fonctions d’affichage mettent à jour les indicateurs et le graphique. Le JavaScript gère donc les échanges et l’affichage ; les calculs financiers restent en Python.

Je montre rapidement la case mémoire du Phoenix. Ensuite Joseph va comparer le Plain avec et sans rappel.

## 13. Faire la démonstration du Plain — Joseph

**À l’écran :** Application : Plain.

On prend trois ans, des constats semestriels et 4 % de coupon par constat. On garde les mêmes paramètres et la même graine pour comparer.

Je calcule d’abord avec rappel. Puis je décoche seulement cette case et je recalcule. Sans rappel, on doit retrouver trois ans de durée moyenne, cent pour cent à maturité et aucun rappel.

Si le seuil final est atteint, les six coupons font 24 %. Pour 1 000 investis, ça donne 1 240 à maturité. Ce montant futur et la valeur actuelle affichée ne sont pas la même chose.

## 14. Montrer les tests — Théo

**À l’écran :** tests/test_api.py et tests/test_engine.py.

On a aussi des tests sur des cas dont on connaît les paiements. On vérifie les conversions, les barrières et les erreurs de paramètres. Il y a notamment un cas qui vérifie qu’en mode sans rappel, un constat intermédiaire favorable ne crée pas de coupon acquis. Les quatorze tests passent dans cette version.

## 15. Terminer sur les limites — Joseph

**À l’écran :** Application : résultats.

Le modèle reste simplifié : le taux, le dividende et la volatilité sont constants. On ne prend pas en compte le smile, les frais de couverture ni le défaut de l’émetteur. Et les probabilités sont celles du modèle risque-neutre, pas des prévisions. Pour aller plus loin, on pourrait utiliser une courbe de taux et calibrer la volatilité sur les options.

