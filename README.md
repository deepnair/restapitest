## Express, Node, and MongoDB Example in Typescript

This is based on the tutorial by TomDoesTech called [Build a REST API with Node.js, Express, TypeScript, MongoDB & Zod](https://www.youtube.com/watch?v=BWUi6BS9T5Y).

### Objective

Create a user, create a way to login, get sessions, and logout. Then create a product that you can get, update and delete. You can only update and delete the product if you are the user that created the product.

## Structure
___
1. It will follow a RCSM pattern with Routes leading to a Controller which calls a service, that then uses the model to connect to the database layer.
1. A 'utils' folder will be creted for utilities. A logger to log info and errors with timestamps will be created, a connect.ts will be created to connect to the database and a jwt.utils.ts will be created to write the code to sign and verify jason web tokens which will be used for authorization.
1. A middleware folder will be created for a deserializeUser.ts which will take the accessToken and the refreshToken from the request Headers and verify them, finds the user associated with the token and assigns the user to res.locals.user. Then a requireUser.ts will also be created which requires there to be be a res.locals.user for access to parts of the site that requires someone to be logged in.
1. A config folder will be created with a default.ts which contains several variables that go into the server that you don't necessarily want being public like the privateKey, publicKey, port, accesTokenTimeToLive, refreshTokenTimeToLive.

## Note
This repository does not have the config folder and the node_modules folder. npm install will install all the node modules. But for the config, you'll have to read the approach and put the relevant variables and data in the default.ts file.

## Approach

### Setup & Utilities
___

1. Create a folder for the program by using mkdir in your terminal.
1. Cd into the folder and then use npm init -y to create a package.json for your node environment.
1. Then run the following commands to install all the necessary packages for the node express mongoDB server
    ```
    yarn add express zod config cors express mongoose pino pino-pretty dayjs bcrypt jsonwebtoken lodash nanoid
    ```
    Then run the following to add the developer dependencies and the types for those packages
    ```
    yarn add @types/body-parser @types/config @types/cors @types/express @types/node @types/pino @types/mongoose @types/bcrypt @types/jsonwebtoken @types/lodash @types/nanoid ts-node-dev typescript -D
    ``` 
1. Start the database part by setting up the MongoDB server on Atlas (just google Mongodb Atlas). You can create a free account, and then start a basic database (should be free). 
1. Make sure you make the access from anywhere option active in the Network access tab. Then go to the connect button on the homepage of atlas for the cluster and then click the connect your application option and copy the string where the option is Node.js. 
1. Come to your developer folder mkdir and create a folder called config. Within it create a file called default.ts by using the touch default.ts command.
1. Within this default.ts create an object that you export as default with the url that you copied earlier from Atlas as 'mongo_uri'. It should look something like:
    ```
    export default {
        mongo_uri: 'mongodb+srv://username:<password>@cluster0.11io9.mongodb.net/databasename?retryWrites=true&w=majority',
    }
    ```
    Change the username, password and insert your databasename sections to the values that you want. You would have set the username and password in the database access part of your atlas while setting up your database. Else go to atlas and set that up.
1. Run the following commands to generate private and public keys in your terminal:
    ```
    openssl genrsa -out private.pem 2048
    ```
    Then run:
    ```
    openssl rsa -in private.pem pubout -out public.pem
    ```
    This will create two files a private.pem which will have your privateKey and a public.pem which will have your private key. You'll need this to generate and verify your jason web tokens respectively.
1. Copy the private key and put it as a property within the object exported from the default.ts file in the config folder. Do the same with the public key and then delete the private.pem and the public.pem files.
1. Add the port number that you'll use in connecting to your database. Have this be 4000 at the beginning. Put this in as a property in the default.ts file. Also add an accessTokenTtl (time the access jwt token lives): "15m" should be good. Access tokens should be short in duration. And a refreshTokenTtl of longer duration like "1y".
1. Also create a property called saltWorkFactor and set it to the number 10. This will be used to hash your passwords when storing it in your database.
1. Within the root folder of the server, create a utils folder and within it create a file called logger.ts. This will be used to log things in the server but with a timestamp.
1. Import logger from "pino" and dayjs from "dayjs". Create a const log that has a base, prettyPrint and timestamp property. Note that timestamp takes a function which in the return uses the dayjs().format() and the base takes an object with a property of pid: which is set to false. prettyPrint takes a boolean which will be true. Then export default log.
1. Within the same utils folder create a file called connect.ts. This will be used to connect to the database. This file should have a connectDb function.
1. Import log from "../logger" and mongoose from "mongoose".
1. connectDb is an async function that takes a url. Then within a try catch it awaits mongoose.connect(url). Then log.info that the database is connected or in the catch log.errors that the database could not be connected.
1. Be sure to export default connectDb from the connect.ts.
1. Now create the utility to sign and verify the jasonwebtokens called jwt.utils.ts within the utils folder.
1. This will have two functions a signJwt function and a verifyJwt function, both of which are exported.
1. This requires import of config from "config", jwt from "jsonwebtoken" and log from "../logger".
1. First get the privateKey and the publicKey from config using config.get\<string>("privateKey") and likewise for the publickey and assing them to variables. 
1. Then write the signJwt function that takes a (object: Object, options: jwt.SignOptions | undefined). This function returns a jwt.sign(object, privateKey, {...(options && options), algorithm: 'RSA256'}) within a try catch.
1. Then write a verifyJwt that takes a token:string with a try catch in it. const decoded is jwt.verify(token). Then return an object with the parameters. valid: true, expired: false, decoded. In the catch return an object with valid: false, expired: e.message === 'jwt expired' and decoded: null.
1. That completes all the utils.

### Create app.ts, create routes.ts and do a healthcheck
___
1. Create an app.ts. Within this create a 

### Create User
___
1. Create a folder called model. Within it create a file called user.model.ts. This will have a userSchema which is a mongoose schema which will feed into the mongoose Model. Before the schema is fed into the mongoose Model, the password will be hashed and comparePassword method will be added to the schema. The file will also have two typescript interfaces UserInput and UserDocument. UserInput has the inputs that will be put into create a user and the UserDocument will have the additional createdAt, updatedAt and comparePassword functions. 
1. Import mongoose from "mongoose", bcrypt from "bcrypt" and config from "config".
1. Create the two typescript interfaces. UserDocument will extend UserInput and mongoose.Document. The comparePassword method will take candidatePassword of type string and return a Promise\<Boolean>.
1. Note that typescript interfaces take primitive types like string and number in lowercase (non primitives like Date can be capitalized), but in mongoose Schema that case of String and Number are capitals.
1. For the mongoose schema, you need to create a "new mongoose.Schema. Within the schema each userinput category is a property that is set equal to an object with a property of type, required, unique, default etc.
1. Next we hash the password by doing userSchema.pre("save", async function(next)). NOTE: The async function must not be an arrow function (because arrow functions have lexical scoping and regular functions will take the parent object for "this".). Within the object const user is this as UserDocument. Then if the !user.isModified("password"), then next(). Then create a salt variable that uses bcrypt and the saltWorkFactor from bcrypt to create a salt. Then in the next using bcrypt.hashSync create a hashed. Then replace the password with hashed and then return next().
1. Create the comparePassword method with userSchema.methods.comparePassword and set it equal to a function (not arrow function). In the function set const user = this as UserDocument. Uses bcrypt.compare(candidatePassword, user.password). Attach a catch to it and return false.
1. Next create a folder called service. Create a file called user.service.ts. The purpose of this is to make a creatUser, validatePassword and findUser function. A userType also needs to be made for the input in the createUser function.
1. In the createUser function in a try, catch call the create method on the UserModel imported from "../model/user.model". return omit(user, "password") with omit imported from lodash so that the returned object doesn't contain the password entered.
1. In the catch segment, throw a new Error(e). This could be caught later in the controller.
1. The verifyPassword method takes an object with email and password. First use the UserModel to findone to see if a user exits, if they can then use the comparePassword method from userModel to see if it's valid. If it is use omit from lodash to return the user without the password.
1. Use the findOne to findUser and return a lean version of it in the findUser method. It takes a query of type FilterQuery (from mongoose) which takes a \<UserDocument>. The lean version returns a POJO (Plain old javascript object) because it will be called by another function.  
1. Make sure every function in the service is async and exported or the code will malfunction.
1. Create a folder called controller and create a user.controller.ts within it. It will have just one function called createUserHandler. 
1. This function takes a req: of type Request<{}, {}, UserInput> where the first is the params, the second is the response body and the third is the request body. Within a try, catch call the createUser function from user.service. If user then res.send(user). In the catch res.status(403).send(e). Export the createUserHandler and make sure it's an async function. Always await when the userModel has been called.
1. Create a folder called schema and create a user.schema.ts. In it create a variable called createSessionSchema that takes an object imported from zod. The object has a body property which also takes an object. From it there is email, which is of string (imported from zod) which takes an object with required_error. Then a .email() with the error mentioned within. Also include a password property and export this.
1. Create a folder called middleware and a file within called validateResource.ts. Create a validateResource function which takes schema of type AnyZodObject imported from Zod and that in turn takes a req, res, and next. Then within a try catch do schema.parse({}) the body, params, and query of req then return next. If catch, res.status(400).send(\`Error: ${e}`).
1. Export default validateResource.

