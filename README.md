# Backend exercise

## Requirements

We want to develop an API for a T-Shirt e-commerce site. For that we are using a NoSQL database, so we are interested on its advantages, like model denormalization. Thus, we would like to get a denormalized design that focuses on getting the necessary data with the least possible queries, or involving the least amount of lookups (a lookup in MongoDB is the equivalent of a left outer join)


* An admin will be able to create and update products (delete is not required)
* A user can obtain a list of products and filter them by color and size
* A user can get the detail of a particular product, including all its colors and sizes with the available stock of each combination
* A user can add products to his shopping cart, that if it's previously authenticated 
* The user can check his shopping cart, being able to modify the quantities of each product in it and also being able to delete them
* The user can buy the products in his cart. That should create an order that could be thought as an invoice. This doesn't involve making any payments, nor integrating with any payments service. Payments are out of the scope
* After checkout, the user should get a confirmation email with the order number
* When the order is created, the dispatch area from the T-Shirt company should be notified of this to prepare the package and ship it. For this, the order should be sent to message queue
* A process will consume from that queue, getting an order and appending it to a CSV file

## Technical details
The base project provided is what must be used to complete this exercise. It implements a REST API using NodeJS, Express and MongoDB.

The API already implements the User model, including authentication endpoints. There are also incomplete Product and CartItem models, including their unit tests. This should be a starting point and an example of the style, the way of organizing the code and also the  expectations about how the app should be tested

About the design of the requested features, it must be considered that we aim to create an API that should be able to handle intensive use. Because of that, the design should focus on model denormalization, trying to minimize both quantity and complexity of database queries.

For the message queue, we are using a docker image that emulates an SQS queue from Amazon Web Services. The aws-sdk should be used to generate the message in the API and there should be a separate script that consumes the messages and process them when executed.

## Endpoints 
#### Products
* Create product (admin)
* Update product (admin)
* Get product list (optional filters by color and / or size)
* Get product by id showing all colors and sizes available. If a color/size is not available it should not be listed

#### Cart
* Add to cart
* Empty cart
* Remove cart item
* Update cart item quantity

#### Order
* Place order
* Get own orders
* Get all orders (admin)

## Installation

First step is to setup Vagrant and VirtualBox. This is for running a virtual ubuntu box.

* [VirtualBox](https://www.virtualbox.org)
* [Vagrant](http://vagrantup.com)

#### How to build the Virtual Machine

```
host $ cd vagrant
host $ vagrant up
```

After the installation has finished, you can access the virtual machine with

```
host $ vagrant ssh
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.13.0-55-generic x86_64)
...
vagrant@lateral-dev-box:~$
```

You can increase the performance with Vagrant's NFS synced folders.

With an NFS server installed (already installed on Mac OS X), uncomment the following from the Vagrantfile:

```ruby
config.vm.network :private_network, ip: '192.168.50.77'
config.vm.synced_folder "/path/to/your/folder", "/home/vagrant/projects", nfs: true
```

## Run the App
First you will need to install all the dependencies
```sh
$ cd backend-exercise
$ npm install
```

After that, you will need to set the database host, base URL, Sendgrid API key and S3 keys in a `.env` file, so they are accesible by environment variables.
You can use as an example the `.env.test` file, used for tests

If S3 keys and/or bucket name are not set no errors will be thrown, but the files will not upload at all (for example in profile picture upload).
Also, don't forget to run the mongoDB server.

Finally, you can run the server by doing:

```sh
$ node server.js
```

## Apidoc

The apidoc is created with grunt post-install script. It can be accessed through **http://localhost:8085/apidoc**

## Tests

Tests are written with [Mocha](http://mochajs.org/), [Chai](http://chaijs.com/), [Supertest](https://github.com/visionmedia/supertest/) and [Nock](https://github.com/pgte/nock).

```sh
$ npm test
```

We also use [Factory Girl](https://github.com/aexmachina/factory-girl) and [Faker.js](https://github.com/marak/Faker.js/) to create model instances. If you need to define new factories add them inside the register function present on **/test/factories.js** file.

```javascript
...
var register = function() {
    // User factory
    factory.define('user', User, {
        email: function() {
            return faker.internet.email();
        },
        password: faker.internet.password(),
        firstname: faker.name.firstName(),
        lastname: faker.name.lastName()
    });
}
...
```

## Directory Structure

```
backend-exercise
│   .gitignore
│   package.json
│   README.md
│   server.js
|   .env
|   .env.test
│
└───app
    │   ...
    │
└───node-modules
    │   ...
    │
└───test
    │   ...
└───apidoc
    │   ...
```

* **App Folder** -> Backend logic.
* **Test Folder** -> Unit Tests.
* **Apidoc Folder** -> Auto-generated API documentation.
* **.env** -> Backend configuration depending on the environment (ouside the repo, you must create the file).
* **.env.test** -> Environment variables for tests.
* **server.js** -> Express server.

## App

```
app
└───handlers
    │   usersHandler.js
    │   ...
    │
└───helpers
    │   mailer.js
    │   ...
    │
└───middleware
    │   auth.js
    │   ...
    │
└───models
    │   user.js
    │   ...
└───routes
    │   routes.js
```

* **Handlers Folder** -> Request handlers executed in each route in **routes.js**. New handlers must be registered in **server.js** file:

```javascript
...
// Request Handlers
var handlers = {
    users: require('./app/handlers/usersHandler'),
    // Add new one here
};
...
```

* **Helpers Folder** -> Shared functions within the backend
* **Middleware Folder** -> Express middleware. Add new middleware to the Express App in **server.js** or **routes.js**. The **auth.js** file cotains a middleware to authenticate routes with the authentication token. If authentication succeeds, it saves the current user in the request object
* **Models Folder** -> Mongoose models
* **Routes Folder** -> Express routes


Happy coding!

