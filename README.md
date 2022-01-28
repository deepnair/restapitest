## Express REST API testing with jest and supertest

This is based on the tutorial by TomDoesTech called [Testing Express REST API with jest and supertest](https://www.youtube.com/watch?v=r5L1XRZaCR0). This is based on an earlier project at [Express, Node and MongoDB example in Typescript](https://github.com/deepnair/typescriptnodemongodbexample)

### Objective

Test the User registration, Login process and the process of getting and creating a product from a REST API with jest and supertest.

## Approach

### Setup

1. Install all the necessary libraries for testing with the following command
    ```
    yarn add jest ts-jest supertest @types/jest @types/supertest mongodb-memory-server -D
    ```
1. Create a jest config file with the following command:
    ```
    yarn ts-jest config:init
    ```
1. In this file ensure the preset is ts-jest and testEnvironment is node. Then add verbose: true, testMatch: ["**/*.test.ts"], forceExit: true, clearMocks: true, resetMocks: true, restoreMocks: true. The verbose will show the results of each test, forceExit should be true to avoid unhandled tests being left, and all mocks should be cleared, reset and restored to avoid bleedover of the results of previous tests to the current one being run.
1. In the utils folder you will need to create a new file called server.ts. This will house a createServer function. This function will have the app from app.ts. When running tests, the app.listen() should not run. Therefore we will put everything to do with app into a function called createserver in the server.ts and export default it.
1. Cut out everything to do with "const app" from app.ts and put it in the createSever function in server.ts. Also bring in the relevant imports. Also bring in routes(app). Return app at the end of the createserver function. Finally export default the createServer function. 
1. Import the createServer function into app.ts. And run "npm run dev" or "yarn dev" to test if the server is still working fine. If not debug it.
1. Create a folder called \_\_tests_\_. 

### Structure of all tests
1. The structure of all tests is to mention what section is being tested in the outer most describe. For example "user" or "session".
1. This is followed by which route is being tested another describe which is nested in the outer most describe. For example "User Registration Route".
1. Within the nested describe, another describe will mention the specific test that is being done in the format "if the product exists". 
1. Finally in a nested "it", the assertion is made "should return a 200 status and the product in the body"

### Product Tests
1. Create a product.test.ts in the \_\_tests_\_ folder. All the product tests will be written in this file.
1. Create a const app and set it equal to createServer() which is imported from "../utils/server". This will be used by supertest throughout the file.
1. The product tests will be written using the mongoDB memory server for testing.
1. Import {MongoMemoryServer} from "mongodb-memory-server". 
1. Within the first describe create a beforeAll() that creates the MongoMemoryServer and then connect mongoose to the testing MongoDB database.
1. In an afterAll() disconnect the database and close the connection.
1. Run each test using the format const {statusCode, body} = await supertest(app) followed by the method and address, followed by anything that needs to be set, and what needs to be sent.
1. Finally write the relevant expect statements.
1. Write out any payloads that may be required for this. 
1. Since supertest always requires an await, naturally the function in "it" must be an async function.
1. The tests to be run with this are:
    1. Get product route
        1. If the product doesn't exist, it should return a 404 statusCode.
        1. If the product does exist, it should return a 200 statusCode and the body of the product.
    1. Create product route
        1. If the user isn't logged in, it should return a 403 statusCode.
        1. If the user is logged in, and you send a product, it should return a 200 and the body of the product.

### User Tests
1. Create a user.test.ts in the \_\_tests_\_ folder.
1. Create a const app and set it equal to createServer() which is imported from "../utils/server". This will be used by supertest throughout the file.
1. For user tests instead of using the mongodb-memory-server we will mock services using jest.spyOn.
1. In this a const createUserServiceMock can be set equal to jest.spyOn(ServiceName, "serviceFunction") where UserService is imported as follows: import * as UserService from "../service/user".
1. After the jest.spyOn another method is called on it. The ones we will be using will be mockReturnValueOnce(payLoad), and mockRejectedValueOnce('Error string here').
1. The mockReturnValueOnce and mockRejectedValueOnce will have to be ts-ignored with //@ts-ignore before the line on which they're typed since this is not production code.
1. In the expect we are to also check if the createUserServiceMock or other mocks were called.
1. The things to be tested in users are:
    1. User
        1. User Registration
            1. If the user's name, email and password are valid, it should return a 200 statusCode and the userPayload, the mock should also have been called with the payload.
            1. If the password and confirmation password do not match, it should return a 400 status code and the mock shouldn't have run since the schema would weed it out before it ever touches the service.
            1. If the service throws an error, it should return a 400 statusCode and the mock should have been called with the payload.
    1. Session
        1. Create a session
            1. If the email and password are valid, then it should return a 200 statusCode and an accessToken and refreshToken.

