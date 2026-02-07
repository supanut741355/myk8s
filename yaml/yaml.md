### YAML
is file use to represent **data of configuration**

key value pair
fruit: apple
meat: chicken


[array / list]
```
fruits:
- Orange
- apple
```

[Dictionary]
```
Banana:
  calories: 105
  fat: 0.4
  carb:27
```

YAML advance
```
Fruits:
  - banana:
      calories: 105
      fat: 0.4
      carb:27
  - apply:
      calories: 105
      fat: 0.4
      carb:27
    
```

###  Dictionary VS List VS List of dictionary

example of car

car has info: color blue, model corvette, tarnsmission manual, price $2000 write to YAML

using dictionary
```
color: blue
model: covette
transmission: manual
price: $20,000
```

using dictionary in dictionary
at model can split with model name and year
```
color: blue
model: 
  name: covette
  year: 1995
transmission: manual
price: $20,000
```



data:
- blue covette
- red covette
- black covette
- green covette

using list
```
-color: blue
  model: 
    name: covette
    year: 1995
  transmission: manual
  price: $20,000


- color: red
  model: 
    name: covette
    year: 1995
  transmission: manual
  price: $22,000

- color: black
  model: 
    name: covette
    year: 1995
  transmission: manual
  price: $23,000

- color: green
  model: 
    name: covette
    year: 1995
  transmission: manual
  price: $22,000
```
