# express-adding-models

#### CRUD App with Mongoose

## Initialize a directory

1. `mkdir models`
1. `touch models/index.js`

## Connect Express to Mongo

1. `npm i mongoose`
1. Inside index.js, inside of models folder.

```javascript
const mongoose = require('mongoose');

const connectionString = 'mongodb://localhost:27017/fruit';

mongoose.connect(connectionString, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
  useFindAndModify: false,
});

mongoose.connection.on('connected', () => {
  console.log(`Mongoose connected to ${connectionString}`);
});

// Export Models
module.exports = {
  Fruit: require('./Fruit'),
};
```

## Create Fruits Model

1. `touch models/Fruit.js`
1. Create the fruit schema

```javascript
const mongoose = require('mongoose');

const fruitSchema = new mongoose.Schema({
  name: { type: String, required: true },
  color: { type: String, required: true },
  readyToEat: Boolean,
});

const Fruit = mongoose.model('Fruit', fruitSchema);

module.exports = Fruit;
```

## Have Create Route Create data in MongoDB

Inside of the `fruitsController.js` file.

- require `./models/index.js`

```javascript
const db = require('../models/index.js');

//... and then farther down the file

router.post('/', (req, res) => {
  // Convert the data to the correct format
  if (req.body.readyToEat === 'on') {
    req.body.readyToEat = true;
  } else {
    req.body.readyToEat = false;
  }

  // Add that new fruit data into our database
  db.Fruit.create(req.body, (err) => {
    if (err) return console.log(err);

    res.redirect('/fruits');
  });
})
```

## Have Index Route Render All Fruits

```javascript
router.get('/', (req, res) => {
  db.Fruit.find({}, (err, allFruits) => {
    if (err) return console.log(err);

    res.render('index.ejs', { allFruits: allFruits });
  });
});
```

Update the ejs file:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
<body>
    <h1>Here are all the fruits:</h1>

    <ul>
        <% for(let i = 0; i < allFruits.length; i++) { %>
            <li>
                <a href="/fruits/<%= allFruits[i]._id %>">
                    <h2><%= allFruits[i].name %></h2>
                </a>
        
                <form action="/fruits/<%= allFruits[i]._id %>?_method=DELETE" method="POST">
                    <input type="submit" value="Delete this Fruit">
                </form>
        
                <a href="/fruits/<%= allFruits[i]._id %>/edit">Edit this Fruit</a>
            </li>
        <% } %>
    </ul>
</body>
</html>
```

## Show Route

```javascript
router.get('/:fruitId', (req, res) => {
  db.Fruit.findById(req.params.fruitId, (err, foundFruit) => {
    if (err) return console.log(err);

    res.render('show.ejs', { oneFruit: foundFruit });
  })
})
```

## Make the Delete Route Delete the Model from MongoDB

Also, have it redirect back to the fruits index page when deletion is complete

```javascript
router.delete('/:fruitId', (req, res) => {
  db.Fruit.findByIdAndDelete(req.params.fruitId, (err) => {
    if (err) return console.log(err);

    res.redirect('/fruits');
  });
})
```

## Update Edit Route with MongoDB

First the route:

```javascript
router.get('/:fruitId/edit', (req, res) => {
  db.Fruit.findById(req.params.fruitId, (err, foundFruit) => {
    if (err) return console.log(err);
    
    res.render('edit.ejs', { oneFruit: foundFruit });
  })
});
```

## Update the edit.ejs Template

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body>
    <h1>Edit the <%= oneFruit.name %></h1>
    <form action="/fruits/<%= oneFruit._id %>?_method=PUT" method="POST">
        <label for="name">Name:</label>
        <input name="name" type="text" value="<%= oneFruit.name %>">
        <br>
        <label for="color">Color:</label>
        <input name="color" type="text" value="<%= oneFruit.color %>">
        <br>
        <label for="readyToEat">Ready To Eat?</label>
        <input name="readyToEat" type="checkbox" <%= oneFruit.readyToEat ? "checked" : "" %>>
        <br>
        <input type="submit" value="Update Fruit">
    </form>
  </body>
</html>
```

## Make the PUT Route Update the Model in MongoDB

```javascript
router.put('/:fruitId', (req, res) => {
  // 1. Convert the data to the correct format
  if (req.body.readyToEat === 'on') {
    req.body.readyToEat = true;
  } else {
    req.body.readyToEat = false;
  }
  
  // 2. Update the data in database
  db.Fruit.findOneAndUpdate(req.params.fruitId, req.body, (err, updatedFruit) => {
    if (err) return console.log(err);

    // 3. Redirect to the show page for that particular fruit
    res.redirect('/fruits/' + req.params.fruitId);
  });
})
```
