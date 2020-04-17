# express-adding-models

#### CRUD App with Mongoose 

## Initialize a directory

1. `mkdir models`
1. `touch models/fruit.js



## Connect Express to Mongo

1. `npm install mongoose --save`
1. Inside db.js, inside of db folder.

```javascript
const mongoose = require('mongoose');

const connectionString = 'mongodb://localhost/fruit';

mongoose.connect(connectionString, { useNewUrlParser: true,
                                     useUnifiedTopology: true,
                                     useCreateIndex: true,
                                     useFindAndModify: false
                                    });


mongoose.connection.on('connected', () => {
  console.log(`Mongoose connected to ${connectionString}`);
});

mongoose.connection.on('disconnected', () => {
  console.log(`Mongoose is disconnected`);
});

mongoose.connection.on('error', (err) => {
  console.log(err, 'mongoose error');
});
```

- require `./db/db` inside of `server.js`

## Create Fruits Model

1. `mkdir models`
1. `touch models/fruits.js`
1. Create the fruit schema

```javascript
const mongoose = require('mongoose');

const fruitSchema = new mongoose.Schema({
    name:  { type: String, required: true },
    color:  { type: String, required: true },
    readyToEat: Boolean
});

const Fruit = mongoose.model('Fruit', fruitSchema);

module.exports = Fruit;
```

## Have Create Route Create data in MongoDB

Inside server.js:

```javascript
const Fruit = require('./models/fruits.js');
//... and then farther down the file
app.post('/fruits/', (req, res)=>{
    if(req.body.readyToEat === 'on'){ //if checked, req.body.readyToEat is set to 'on'
        req.body.readyToEat = true;
    } else { //if not checked, req.body.readyToEat is undefined
        req.body.readyToEat = false;
    }
    Fruit.create(req.body, (error, createdFruit)=>{
        res.send(createdFruit);
    });
});
```


## Have Index Route Render All Fruits

```javascript
app.get('/fruits', (req, res)=>{
    Fruit.find({}, (error, allFruits)=>{
        res.render('index.ejs', {
            fruits: allFruits
        });
    });
});
```

Update the ejs file:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>Fruits index page</h1>
        <ul>
            <% for(let i = 0; i < fruits.length; i++){ %>
                <li>
                    The <%=fruits[i].name; %> is  <%=fruits[i].color; %>.
                    <% if(fruits[i].readyToEat === true){ %>
                        It is ready to eat
                    <% } else { %>
                        It is not ready to eat
                    <% } %>
                </li>
            <% } %>
        </ul>
    </body>
</html>
```



## Have Create Route redirect to Index After Fruit Creation

Inside the create route

```javascript
Fruit.create(req.body, (error, createdFruit)=>{
    res.redirect('/fruits');
});
```

##  Show Route

```javascript
app.get('/fruits/:id', (req, res)=>{
    Fruit.findById(req.params.id, (err, fruit)=>{
        res.send(fruit);
    });
});
```

## Have Index Page Link to Show Route

```html
<li>
    The <a href="/fruits/<%=fruits[i].id; %>"><%=fruits[i].name; %></a> is  <%=fruits[i].color; %>.
    <% if(fruits[i].readyToEat === true){ %>
        It is ready to eat
    <% } else { %>
        It is not ready to eat
    <% } %>
</li>
```

## Create show.ejs


Render the ejs

```javascript
app.get('/fruits/:id', (req, res)=>{
    Fruit.findById(req.params.id, (err, foundFruit)=>{
        res.render('show.ejs', {
            fruit:foundFruit
        });
    });
});
```


## Make the Delete Route Delete the Model from MongoDB

Also, have it redirect back to the fruits index page when deletion is complete

```javascript
app.delete('/fruits/:id', (req, res)=>{
    Fruit.findByIdAndDelete(req.params.id, (err, deletedFruit)=>{
        res.redirect('/fruits');//redirect back to fruits index
    });
});
```

## Create a link to an edit route

In your `index.ejs` file:

```html
<a href="/fruits/<%=fruits[i].id; %>/edit">Edit</a>
```

## Update Edit Route with MongoDB

First the route:

```javascript
app.get('/fruits/:id/edit', (req, res)=>{
    Fruit.findById(req.params.id, (err, foundFruit)=>{ //find the fruit
        res.render(
    		'edit.ejs',
    		{
    			fruit: foundFruit //pass in found fruit
    		}
    	);
    });
});
```

Now the EJS:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
        <link rel="stylesheet" href="/css/app.css">
    </head>
    <body>
        <h1>Edit Fruit Page</h1>
        <form method="POST" action="/fruits/<%=fruit._id%>?_method=PUT">
            <!--  NOTE: the form is pre-populated with values for the server-->
            Name:   <input type="text" name="name" value="<%=fruit.name%>"/><br/>
            Color:  <input type="text" name="color" value="<%=fruit.color%>"/><br/>
            Is Ready To Eat:
                    <input type="checkbox" name="readyToEat"
                        <% if(fruit.readyToEat === true){ %>
                      checked
                        <% } %>
                    />
                    <br/>



            <input type="submit" name="" value="Submit Changes"/>
        </form>
    </body>
</html>

```


## Make the PUT Route Update the Model in MongoDB

```javascript
app.put('/fruits/:id', (req, res)=>{
    if(req.body.readyToEat === 'on'){
        req.body.readyToEat = true;
    } else {
        req.body.readyToEat = false;
    }
    Fruit.findOneAndUpdate(req.params.id, req.body, (err, updatedModel)=>{
          res.redirect('/fruits');
    });
});
```


