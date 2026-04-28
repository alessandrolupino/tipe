# TIPE — Compression de données en temps réel pour l'acquisition industrielle

---

## Motivation et ancrage au thème

Dans un contexte industriel, un capteur (température, pression, vibration) émet un flux binaire continu à fréquence élevée. Ce flux doit être stocké localement pendant toute la durée d'une expérience — parfois plusieurs heures — avant d'être transmis ou analysé. La mémoire embarquée est limitée, et la bande passante de transmission est contrainte.

La compression sans perte est alors la seule option acceptable : chaque mesure doit pouvoir être **exactement reconstituée** après décompression, comme si elle n'avait jamais été compressée.

**Lien avec "Cycles, boucles"** : l'algorithme LZ78 repose sur une boucle de lecture du flux, construisant *itérativement* un dictionnaire de motifs. À chaque cycle de lecture, le dictionnaire s'enrichit ; le taux de compression converge asymptotiquement vers la limite théorique de Shannon. C'est cette dynamique cyclique qui fait l'objet de l'étude.

---

## Problématique

> **Dans quelle mesure un algorithme de compression à dictionnaire cyclique (LZ78) permet-il de comprimer fidèlement un flux binaire issu d'un capteur industriel, et comment la vitesse de convergence de son taux de compression vers la borne de Shannon dépend-elle de la structure statistique de la source ?**

---

## Cadre théorique

### 1. Entropie d'une source binaire

Soit une source binaire sans mémoire $S$ émettant le symbole $1$ avec probabilité $p$ et le symbole $0$ avec probabilité $1-p$. L'entropie de Shannon (en bits par symbole) est :

$$H(p) = -p \log_2 p - (1-p) \log_2 (1-p)$$

$H(p)$ atteint son maximum en $p = 0{,}5$ avec $H = 1$ bit/symbole, et tend vers $0$ lorsque $p \to 0$ ou $p \to 1$.

**Interprétation** : $H(p)$ est la borne inférieure théorique du nombre moyen de bits nécessaires pour encoder un symbole. Aucun algorithme de compression sans perte ne peut faire mieux en moyenne.

### 2. Théorème de compression de Shannon (1948)

Pour toute source stationnaire ergodique d'entropie $H$, il existe un code sans perte atteignant un taux moyen arbitrairement proche de $H$ bits/symbole, mais aucun code ne peut descendre en dessous.

Formellement, si $L_n$ est la longueur moyenne du code pour une séquence de $n$ symboles :

$$H \leq \frac{L_n}{n} < H + \varepsilon \quad \text{pour } n \text{ suffisamment grand}$$

### 3. L'algorithme LZ78 et sa boucle itérative

LZ78 (Ziv & Lempel, 1978) encode un flux par construction incrémentale d'un dictionnaire :

**Boucle principale :**
```
dictionnaire ← {ε : 0}   # dictionnaire initialisé avec le mot vide
i ← 0
tant que flux non vide :
    lire le plus long préfixe w connu dans le dictionnaire
    lire le symbole suivant c
    émettre (index(w), c)
    ajouter wc au dictionnaire avec un nouvel index
    i ← i + 1
```

**Propriété fondamentale** : Ziv et Lempel ont démontré en 1978 que pour toute source stationnaire :

$$\lim_{n \to \infty} \frac{L^{\text{LZ78}}_n}{n} = H$$

La convergence est *asymptotique* : pour un flux court (typique en industrie, quelques milliers de symboles), le taux réel $L_n/n$ est **supérieur à $H$**. Mesurer cet écart en fonction de $n$ et de $p$ est l'objet expérimental de ce TIPE.

### 4. Taux de compression

On définit le **taux de compression** $\tau$ par :

$$\tau = 1 - \frac{\text{taille compressée (bits)}}{\text{taille originale (bits)}}$$

Et l'**efficacité** de l'algorithme par rapport à la borne théorique :

$$\eta(n) = \frac{H(p)}{L^{\text{LZ78}}_n / n} \in [0, 1]$$

On cherche à modéliser $\eta(n, p)$ et à déterminer le seuil $n^*(p)$ à partir duquel $\eta > 0{,}95$.

---

## Dispositif expérimental simulé

### Modèle de capteur industriel

On modélise le flux d'un capteur de température par une **source de Markov d'ordre 1** :

$$P(X_{t+1} = 1 \mid X_t = x) = \begin{cases} \alpha & \text{si } x = 0 \\ 1 - \beta & \text{si } x = 1 \end{cases}$$

où $\alpha, \beta \in (0,1)$ caractérisent la "fluidité" du signal. L'entropie de cette source est :

$$H = -\pi_1 \left[\alpha \log_2 \alpha + (1-\alpha)\log_2(1-\alpha)\right] - \pi_0 \left[\beta \log_2 \beta + (1-\beta)\log_2(1-\beta)\right]$$

avec $\pi_1 = \frac{\alpha}{\alpha+\beta}$, $\pi_0 = \frac{\beta}{\alpha+\beta}$ (distribution stationnaire).

### Protocole

1. Générer des flux synthétiques de longueur $n \in \{10^2, 10^3, 10^4, 10^5\}$ symboles
2. Faire varier $p \in \{0{,}1,\ 0{,}2,\ \ldots,\ 0{,}5\}$
3. Appliquer LZ78, mesurer $L_n$ et calculer $\eta(n, p)$
4. Tracer les courbes de convergence et ajuster un modèle $\eta(n) \approx 1 - \frac{c(p)}{\log n}$
5. Décompresser et vérifier l'identité bit-à-bit avec le flux original

---

## Résultats attendus et valeur ajoutée

| Mesure | Valeur attendue |
|---|---|
| $\eta$ pour $n = 100$, $p = 0{,}5$ | $\approx 0{,}60$ — LZ78 très inefficace sur flux courts |
| $\eta$ pour $n = 10^5$, $p = 0{,}5$ | $\approx 0{,}92$ — convergence quasi-atteinte |
| Flux très redondant ($p = 0{,}1$) | $\eta$ élevé dès $n = 10^3$ |
| Vérification décompression | 100 % identique (compression sans perte) |

**La valeur ajoutée** : la plupart des études TIPE implémentent LZ78 et mesurent un taux global. Ici on modélise **la dynamique de convergence** de la boucle, ce qui répond directement au thème "Cycles, boucles" et apporte une perspective quantitative sur l'applicabilité réelle en contexte industriel.

---

## Ancrage au thème : Cycles, boucles

| Aspect | Lien avec le thème |
|---|---|
| Boucle de codage LZ78 | Chaque itération enrichit le dictionnaire → boucle productive |
| Convergence asymptotique | La boucle infinie converge vers $H$ : étude de la vitesse |
| Cycle acquisition/compression/décompression | Le flux est comprimé *en ligne* et restauré exactement |
| Chaîne de Markov | Modèle cyclique de dépendance temporelle des symboles |

---

## Références bibliographiques

1. **Shannon, C.E.** (1948). *A Mathematical Theory of Communication*. Bell System Technical Journal.
2. **Ziv, J. & Lempel, A.** (1978). *Compression of Individual Sequences via Variable-Rate Coding*. IEEE Transactions on Information Theory.
3. **Cover, T.M. & Thomas, J.A.** (2006). *Elements of Information Theory*. Wiley (2e éd.).
4. **Salomon, D.** (2007). *Data Compression: The Complete Reference*. Springer.

---
