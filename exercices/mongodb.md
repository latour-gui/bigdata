# MongoDB

## Cours

### Création collection avec contrainte

Réaliser la base de données permettant le stockage et le traitement d’un client et ses commandes de produits.

- Le nom de la base de données est libre.
- Cette base de données doit contenir deux collections:
  - clients
    - NOM
    - ADRESSE
    - LOCALITE
    - CAT
    - COMPTE
    - COMMANDES
      - NCOM
      - DATECOM
      - DETAIL
        - PRODUIT
        - QCOM
  - produits
    - NPROD
    - LIBELLE
    - PRIX
    - DATELASTMODIF
    - QSTOCK

Création de la base de données : `use testclient`

```Javascript
db.createCollection("produits", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["NPROD", "LIBELLE", "PRIX", "DATELASTMODIF", "QSTOCK"],
            properties: {
                NPROD: {
                    bsonType: "string",
                    description: "idk"
                },
                LIBELLE: {
                    bsonType: "string",
                    description: "libele i guess"
                },
                PRIX: {
                    bsonType: "double",
                    minimum: 0.0,
                    description: "The price of the product"
                },
                DATELASTMODIF: {
                    bsonType: "date"
                },
                QSTOCK: {
                    bsonType: "int"
                }
            }
        }
    }
})

db.createCollection("clients", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["NOM", "ADRESSE", "LOCALITE", "CAT", "COMPTE"],
            properties: {
                NOM: {
                    bsonType: "string"
                },
                ADRESSE: {
                    bsonType: "string"
                },
                LOCALITE: {
                    bsonType: "string"
                },
                CAT: {
                    bsonType: "string"
                },
                COMPTE: {
                    bsonType: "string"
                },
                COMMANDES: {
                    bsonType: "array",
                    items: {
                        bsonType: "object",
                        required: ["NCOM", "DATECOM", "DETAIL"],
                        properties: {
                            NCOM: {
                                bsonType: "string"
                            },
                            DATECOM: {
                                bsonType: "date"
                            },
                            DETAIL: {
                                bsonType: "object",
                                required: ["PRODUIT", "QCOM"],
                                properties: {
                                    PRODUIT: {
                                        bsonType: "objectId"
                                    },
                                    QCOM: {
                                        bsonType: "int"
                                    }
                                }
                            }
                        }
                    }
                }

            }
        }
    }
})
```

## Moodle

### Récupérer tous clients n'étant pas liés à une société

```js
db.customers.find({
  $or: [{ Company: null }, { Company: { $exists: false } }],
})
```

### Lister toutes les villes où des clients travaillant pour les sociétés "Yodo" et "Kare" vivent

```js
db.customers.find(
  {
    Company: { $in: ["Yodo", "Kare"] },
  },
  { _id: false, City: true }
)
```

### Compter le nombre de produit qui sont vendus à un prix supérieur à 50

```js
db.products.count({ price: { $gt: 50.0 } }) // BEST
db.products.find({ price: { $gt: 50.0 } }).count() // WORKING BUT LAME
```

### Lister les noms et prénoms des clients possédant une adresse mail terminant par @patch.com

```js
db.customers.find(
  { email: { $regex: ".*@patch.com$" } },
  { _id: false, first_name: true, last_name: true }
)
```

### Trouver les produits de couleurs mauves ("Purple") coûtant plus de 20

```js
db.products.find({ $and: [{ color: "Purple" }, { price: { $gt: 20 } }] })
```

### Afficher les nom et prénom d'un client de l'état MI

```js
db.customers.findOne(
  { State: "MI" },
  { _id: false, first_name: true, last_name: true }
)
```

### Compter les clients par état

```js
db.customers.aggregate([{ $group: { _id: "$State", count: { $sum: 1 } } }])
```

### Compter le nombre de femmes et d'hommes

```js
db.customers.aggregate([{ $group: { _id: "$gender", count: { $sum: 1 } } }])
```

### Compter le nombre d'individus par société

```js
db.customers.aggregate([{ $group: { _id: "$Company", count: { $sum: 1 } } }])
```

### Comptez le nombre d'hommes et de femmes par société

```js
db.customers.aggregate([
  {
    $group: {
      _id: { societe: "$Company", sexe: "$gender" },
      count: { $sum: 1 },
    },
  },
  {
    $project: {
      _id: false,
      company: "$_id.societe",
      gender: "$_id.sexe",
      count: true,
    },
  },
])
```

### Compter le nombre de commande pour un client (aléatoire)

```js
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      count: { $sum: 1 },
    },
  },
])
```

```js
// source : prof
db.embeddedOrders
  .aggregate([
    { $limit: 1 },
    {
      $project: {
        first_name: 1,
        last_name: 1,
        nbOrders: { $size: "$orders" },
      },
    },
  ])
  .pretty()
```

### Pour chaque client de la société "Yodo", compter le nombre de commandes

```js
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customer",
    },
  },
  {
    $match: { "customer.Company": { $eq: "Yodo" } },
  },
  {
    $group: {
      _id: "$customerId",
      count: { $sum: 1 },
    },
  },
])
```

```js
// source : prof
db.embeddedOrders
  .aggregate(
    { $match: { Company: "Yodo" } },
    {
      $project: {
        first_name: 1,
        last_name: 1,
        nbOrders: { $size: "$orders" },
      },
    }
  )
  .pretty()
```

### Pour le client \_id:2, compter le nombre de fois que chaque produit a été commandé, ainsi que sa quantité

Exemple de structure de résultat: `{_id: <NOM_PRODUIT>, nbCommandes: <INTEGER>, qte: <INTEGER>}`

```js
// 35 sec
db.lineitems.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "orderId",
      foreignField: "_id",
      as: "order",
    },
  },
  {
    $unwind: "$order",
  },
  {
    $match: {
      "order.customerId": { $eq: 2 },
    },
  },
  {
    $lookup: {
      from: "products",
      localField: "prodId",
      foreignField: "_id",
      as: "product",
    },
  },
  {
    $unwind: "$product",
  },
  {
    $group: {
      _id: "$product.productName",
      nbCommandes: {
        $sum: 1,
      },
      qte: {
        $sum: "$itemCount",
      },
    },
  },
])
```

```js
// source : prof
db.embeddedOrders
  .aggregate(
    { $match: { _id: 2 } },
    {
      $unwind: {
        path: "$orders",
      },
    },
    {
      $unwind: {
        path: "$orders.lineItems",
      },
    },
    {
      $project: {
        orderId: "$orders.orderId",
        productName: "$orders.lineItems.productName",
        qty: "$orders.lineItems.itemCount",
      },
    },
    {
      $group: {
        _id: "$productName",
        nbCommandes: { $sum: 1 },
        qte: { $sum: "$qty" },
      },
    },
    {
      $sort: {
        nbCommandes: -1,
      },
    }
  )
  .pretty()
```

### Touvez le produit le plus commandé par des femmes

```js
// 1 min 30 sec
db.lineitems.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "orderId",
      foreignField: "_id",
      as: "order",
    },
  },
  {
    $unwind: "$order",
  },
  {
    $lookup: {
      from: "customers",
      localField: "order.customerId",
      foreignField: "_id",
      as: "customer",
    },
  },
  {
    $unwind: "$customer",
  },
  {
    $match: { "customer.gender": "Female" },
  },
  {
    $lookup: {
      from: "products",
      localField: "prodId",
      foreignField: "_id",
      as: "product",
    },
  },
  {
    $unwind: "$product",
  },
  {
    $group: {
      _id: "$product.productName",
      nbCommandes: {
        $sum: 1,
      },
    },
  },
  {
    $sort: { nbCommandes: -1 },
  },
  {
    $limit: 1,
  },
  {
    $project: { _id: false, best_selling_female_item: "$_id" },
  },
])
```

```js
// source : prof
db.embeddedOrders
  .aggregate(
    { $match: { gender: "Female" } },
    {
      $unwind: {
        path: "$orders",
      },
    },
    {
      $unwind: {
        path: "$orders.lineItems",
      },
    },
    {
      $project: {
        orderId: "$orders.orderId",
        productName: "$orders.lineItems.productName",
        qty: "$orders.lineItems.itemCount",
      },
    },
    {
      $group: {
        _id: "$productName",
        nbCommandes: { $sum: 1 },
        qtyOrdered: { $sum: "$qty" },
      },
    },
    {
      $sort: {
        qtyOrdered: -1,
      },
    },
    { $limit: 1 }
  )
  .pretty()
```

### Calculez le chiffre d'affaires par État

```js
db.embeddedOrders.mapReduce(
  function () {
    this.orders.forEach((order) => {
      order.lineItems.forEach((item) => {
        emit(this.State, item.price)
      })
    })
  },
  function (country, product_prices) {
    return Array.sum(product_prices)
  },
  { out: "receipe_by_state" }
)
db.receipe_by_state.find().sort({ value: -1 }).pretty()
```

```js
// source : prof
db.embeddedOrders
  .aggregate(
    {
      $unwind: {
        path: "$orders",
      },
    },
    {
      $unwind: {
        path: "$orders.lineItems",
      },
    },
    {
      $project: {
        State: 1,
        amount: {
          $multiply: ["$orders.lineItems.itemCount", "$orders.lineItems.price"],
        },
      },
    },
    {
      $group: {
        _id: "$State",
        totalAmount: { $sum: "$amount" },
      },
    },
    {
      $sort: {
        totalAmount: -1,
      },
    }
  )
  .pretty()
```

### Pour chaque client vivant à Orlando, déterminer combien de commandes contiennent le produit "Chips - Miss Vickies"

```js
db.embeddedOrders.aggregate([
  {
    $match: { City: "Orlando" },
  },
  {
    $unwind: "$orders",
  },
  {
    $unwind: "$orders.lineItems",
  },
  {
    $match: { "orders.lineItems.productName": "Chips - Miss Vickies" },
  },
  {
    $group: {
      _id: { $concat: ["$first_name", " ", "$last_name"] },
      count: { $sum: 1 },
    },
  },
])
```
