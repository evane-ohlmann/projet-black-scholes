# Valorisation d'options et application à une garantie plancher décès

> Projet personnel **en cours**, réalisé pour approfondir le cours *Introduction à l'évaluation des produits financiers* (M1 Actuariat, ISUP, 2026-2027).
> L'objectif est de mettre en œuvre les résultats du cours en Python, de les vérifier, puis de les appliquer à un exemple simple d'assurance vie.

---

## Objectif

Le cours montre comment donner un prix à une option sous l'hypothèse d'absence d'opportunité d'arbitrage (AOA) : d'abord dans le modèle binomial, puis dans le modèle log-normal, qui mène à la formule de Black-Scholes.

Ce projet :

1. **code** la formule de Black-Scholes et le modèle binomial à n périodes ;
2. **vérifie** que le prix binomial converge vers le prix de Black-Scholes quand n augmente, comme l'annonce le cours ;
3. **applique** ces outils à un exemple d'assurance vie : le coût d'une garantie qui protège l'épargne d'un assuré en cas de décès.

## Cadre et modèle

Le projet se place dans le cadre simplifié du cours. Ces hypothèses rendent les calculs possibles ; leurs limites sont discutées à la fin.

### Le marché

- **Marché sans friction** : pas de frais ni de taxes, même prix à l'achat et à la vente, ventes à découvert autorisées, même taux pour prêter et pour emprunter.
- **Absence d'opportunité d'arbitrage (AOA)** : il est impossible de gagner de l'argent sans risque à partir d'une mise nulle. Conséquence : si un portefeuille reproduit exactement les flux d'un produit, il a le même prix que lui. Le prix d'une option est donc le **coût du portefeuille qui la réplique**.
- **Deux actifs seulement** :
    - un **actif sans risque**, rémunéré au taux r constant ;
    - une **action** qui ne verse pas de dividende.

### Le modèle d'évolution de l'action

- **Modèle binomial à n périodes** : à chaque période, le prix de l'action est multiplié soit par u (hausse), soit par d (baisse). L'AOA impose u > 1 + r > d. Toute option européenne peut alors être répliquée avec l'action et l'actif sans risque, et son prix est l'espérance de son payoff actualisé sous la **probabilité risque-neutre** p̃ = (1 + r − d)/(u − d). Ce prix ne dépend pas de la probabilité réelle que l'action monte.
- **Modèle log-normal**, limite du modèle binomial quand le nombre de périodes tend vers l'infini :

  ```
  S_T = S0 · exp((r − σ²/2) · T + σ · √T · Z),   Z de loi normale centrée réduite
  ```

  La **volatilité σ** mesure l'ampleur des variations de l'action. Elle est supposée **constante**, comme le taux r. C'est dans ce modèle que s'obtient la formule de Black-Scholes.

### L'exemple d'assurance vie (étape 3)

- **Le contrat** : l'épargne de l'assuré est placée sur les marchés. Sa valeur est modélisée comme le prix S_t de l'action du modèle log-normal.
- **La garantie** : si l'assuré décède, l'assureur verse au moins le montant K investi au départ. S'il décède pendant l'année t, il complète donc de (K − S_t)⁺, avec un paiement supposé en fin d'année.
- **Le décès** : les probabilités de décès viennent d'une table de mortalité française (TH 00-02 pour les hommes, TF 00-02 pour les femmes). On suppose qu'elles ne dépendent pas des marchés.
- **Une différence importante avec les options** : le risque de décès ne peut pas être répliqué avec l'action et l'actif sans risque. On utilise donc directement les probabilités de la table, et non une probabilité risque-neutre.

## Contenu du dépôt

| Fichier | Contenu |
|---|---|
| `ProjetBS.ipynb` | Le notebook du projet, en trois étapes : formule de Black-Scholes et tests ; modèle binomial et convergence vers Black-Scholes ; coût de la garantie en cas de décès et sensibilités |
| `bonus_monte_carlo.ipynb` | Simulation Monte Carlo (prolongement en cours) |
| `table_mortalite.csv` | Table de mortalité TH 00-02 (hommes), colonnes `age` et `lx` |
| `table_mortalite_TF.csv` | Table de mortalité TF 00-02 (femmes), même format |
| `figures/` | Graphiques produits par le notebook |

## Démarche

### Étape 1 : la formule de Black-Scholes

Dans le modèle log-normal, les prix du Call et du Put européens de strike K et de maturité T sont :

```
Call = S0 · N(d+) − K · e^(−rT) · N(d−)
Put  = K · e^(−rT) · N(−d−) − S0 · N(−d+)
d±   = 1/(σ√T) · log(S0 · e^(rT) / K) ± ½ · σ√T
```

Ces formules servent de **référence** pour les étapes suivantes.

### Étape 2 : le modèle binomial et sa convergence

Dans le modèle binomial à n périodes, le prix d'une option de payoff Ψ(S_T) est :

```
C0 = (1 + r)^(−n) · Σ_{k=0}^{n} C(n,k) · p̃^k · (1 − p̃)^(n−k) · Ψ(S0 · u^k · d^(n−k)),   avec p̃ = (1 + r − d)/(u − d)
```

Avec u_n = e^(σ√(T/n)), d_n = e^(−σ√(T/n)) et un taux rT/n par période, ce prix converge vers celui de Black-Scholes quand n tend vers l'infini. Le notebook trace cette convergence pour n allant de 1 à 200.

### Étape 3 : la garantie en cas de décès

**L'exemple.** Un assuré de 60 ans place 10 000 €. S'il décède dans les 10 ans, l'assureur garantit que ses proches recevront au moins K = 10 000 €.

**Le raisonnement.** Si le décès a lieu pendant l'année t, l'assureur verse le complément (K − S_t)⁺ : c'est le payoff d'un Put de maturité t. La garantie revient donc à un ensemble de 10 Puts, un par année, chacun pondéré par la probabilité de décéder cette année-là :

```
Coût de la garantie = Σ_{t=1}^{10} P(décès pendant l'année t) × Put(K, t)
P(décès pendant l'année t) = (l_{x+t−1} − l_{x+t}) / l_x
```

où l_x est le nombre de personnes encore en vie à l'âge x dans la table de mortalité.

## Résultats

Paramètres : S0 = K = 100 et r = 5 % (étapes 1 et 2) ; S0 = K = 10 000 € et r = 3 % (étape 3) ; σ = 20 % partout.

### Black-Scholes et modèle binomial

| | Black-Scholes | Binomial, n = 1 000 |
|---|---|---|
| Call (K = 100, T = 1 an) | 10,4506 | 10,4485 |
| Put (K = 100, T = 1 an) | 5,5735 | 5,5716 |

L'écart est inférieur à 0,01 pour n = 1 000. Le prix binomial oscille autour du prix de Black-Scholes, de moins en moins à mesure que n augmente.

![Convergence du prix binomial vers Black-Scholes](figures/convergence_binomial.png)

### Coût de la garantie (assuré de 60 ans, horizon 10 ans)

| Table de mortalité | Probabilité de décéder dans les 10 ans | Coût de la garantie |
|---|---|---|
| TH 00-02 (hommes) | 15,80 % | 159,57 €, soit 1,60 % du montant placé |
| TF 00-02 (femmes) | 6,77 % | 68,58 €, soit 0,69 % du montant placé |

Le coût vient surtout des dernières années : 7,40 € pour l'année 1, 22,96 € pour l'année 10. La probabilité de décès augmente avec l'âge, et un Put de maturité plus longue vaut plus cher.

### Sensibilités (table TH 00-02)

| Paramètre | Valeurs | Coût de la garantie |
|---|---|---|
| Volatilité σ | 10 % / 15 % / 20 % / 25 % / 30 % | 45 € / 100 € / 160 € / 221 € / 282 € |
| Âge de l'assuré | 40 / 50 / 60 / 70 / 80 ans | 38 € / 78 € / 160 € / 344 € / 686 € |
| Taux sans risque r | 0 % / 1 % / 3 % / 5 % | 296 € / 243 € / 160 € / 103 € |

La garantie coûte plus cher quand les marchés sont plus volatils, quand l'assuré est plus âgé et quand les taux sont bas.

![Sensibilité à la volatilité](figures/sensibilite_sigma.png)
![Sensibilité à l'âge](figures/sensibilite_age.png)

## Limites

| Simplification | Ce qui se passe en réalité |
|---|---|
| Volatilité et taux constants | Ils varient dans le temps |
| Marché sans friction | Acheter et vendre coûte des frais |
| Contrat simplifié | Un vrai contrat comporte des frais, et l'assuré peut retirer son argent avant son décès |
| Décès en fin d'année | Le décès peut survenir à tout moment de l'année |
| Décès indépendants des marchés | Hypothèse raisonnable, mais simplificatrice |

Le coût obtenu est un **coût moyen**, calculé dans un cadre simplifié : il sert à comprendre ce qui fait varier le prix d'une telle garantie, pas à fixer le prix d'un contrat réel.

## Prochaine étape

Calculer les mêmes prix par **simulation Monte Carlo** : simuler un grand nombre de valeurs de S_T dans le modèle log-normal, puis prendre la moyenne actualisée des payoffs, avec un intervalle de confiance. La fonction de simulation est écrite ; il reste le calcul du prix et l'étude de la convergence.

## Exécution

Prérequis : Python 3 avec NumPy, SciPy, pandas, Matplotlib et Jupyter.

```bash
pip install numpy scipy pandas matplotlib jupyter
jupyter notebook
```

Ouvrir `ProjetBS.ipynb`, puis exécuter toutes les cellules dans l'ordre. Le fichier `table_mortalite.csv` doit se trouver dans le même dossier que le notebook.

## Références

- Polycopié du cours *Introduction à l'évaluation des produits financiers* (UM4ACT03, M1 Actuariat, Sorbonne Université - ISUP), en particulier les chapitres 4 (produits optionnels), 5 (modèle binomial) et 6 (formule de Black-Scholes).
- J. C. Cox, S. A. Ross, M. Rubinstein, *Option pricing: A simplified approach*, Journal of Financial Economics, 1979.
- F. Black, M. Scholes, *The pricing of options and corporate liabilities*, Journal of Political Economy, 1973.
- Tables de mortalité françaises TH 00-02 et TF 00-02.
