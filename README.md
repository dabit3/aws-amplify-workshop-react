# Building Web Applications with AWS Amplify

In this workshop we'll learn how to build cloud-enabled web applications with React & [AWS Amplify](https://aws-amplify.github.io/).

![](https://imgur.com/IPnnJyf.jpg)

### Topics we'll be covering:

- [Authentication](https://github.com/dabit3/aws-amplify-workshop-react#adding-authentication)
- [Serverless Functions](https://github.com/dabit3/aws-amplify-workshop-react#adding-a-serverless-function)
- [REST API with a Lambda Function](https://github.com/dabit3/aws-amplify-workshop-react#adding-a-rest-api)
- [GraphQL API with AWS AppSync](https://github.com/dabit3/aws-amplify-workshop-react#adding-a-graphql-api)
- [Adding Storage with Amazon S3](https://github.com/dabit3/aws-amplify-workshop-react#working-with-storage)
- [Analytics](https://github.com/dabit3/aws-amplify-workshop-react#adding-analytics)
- [Multiple Environments](https://github.com/dabit3/aws-amplify-workshop-react#working-with-multiple-environments)
- [Deploying via the Amplify Console](https://github.com/dabit3/aws-amplify-workshop-react#deploying-via-the-amplify-console)
- [Adding Fine-grained Authorization to the GraphQL API](https://github.com/dabit3/aws-amplify-workshop-react#adding-fine-grained-authorization-to-the-graphql-api)
- [Removing / Deleting Services](https://github.com/dabit3/aws-amplify-workshop-react#removing-services)

## Redeeming our AWS Credit   
1. Visit the [AWS Console](https://console.aws.amazon.com/console).
2. In the top right corner, click on __My Account__.
![](dashboard1.jpg)
3. In the left menu, click __Credits__.
![](dashboard2.jpg)

## Getting Started - Creating the React Application

To get started, we first need to create a new React project & change into the new directory using the [Create React App CLI](https://github.com/facebook/create-react-app).

If you already have this installed, skip to the next step. If not, either install the CLI & create the app or create a new app using npx:

```bash
npm install -g create-react-app
create-react-app my-amplify-app
```

Or use npx (npm 5.2 & later) to create a new app:

```bash
npx create-react-app my-amplify-app
```

Now change into the new app directory & install the AWS Amplify & AWS Amplify React libraries:

```bash
cd my-amplify-app
npm install --save aws-amplify aws-amplify-react uuid
# or
yarn add aws-amplify aws-amplify-react uuid
```

## Installing the CLI & Initializing a new AWS Amplify Project

### Installing the CLI

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```

Now we need to configure the CLI with our credentials:

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __us-east-1__
- Specify the username of the new IAM user: __amplify-workshop-user__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
  accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
  secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __amplify-workshop-user__

### Initializing A New Project

```bash
amplify init
```

- Enter a name for the project: __amplifyreactapp__
- Enter a name for the environment: __local__
- Choose your default editor: __Visual Studio Code (or your default editor)__   
- Please choose the type of app that you're building __javascript__   
- What javascript framework are you using __react__   
- Source Directory Path: __src__   
- Distribution Directory Path: __build__   
- Build Command: __npm run-script build__   
- Start Command: __npm run-script start__   
- Do you want to use an AWS profile? __Y__
- Please choose the profile you want to use: __amplify-workshop-user__

Now, the AWS Amplify CLI has iniatilized a new project & you will see a new folder: __amplify__ & a new file called `aws-export.js` in the __src__ directory. These files hold your project configuration.

## Adding Authentication

To add authentication, we can use the following command:

```sh
amplify add auth
```
- Do you want to use default authentication and security configuration?  __Default configuration__
- How do you want users to be able to sign in when using your Cognito User Pool? __Username__
- What attributes are required for signing up? __Email__ (keep default)

Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push
```

To view the services created by Amplify at any time you can run the `console` command & choose the feature you'd like to view:

```sh
amplify console
```

### Configuring the React applicaion

Now, our resources are created & we can start using them!

The first thing we need to do is to configure our React application to be aware of our new AWS Amplify project. We can do this by referencing the auto-generated `aws-exports.js` file that is now in our src folder.

To configure the app, open __src/index.js__ and add the following code below the last import:

```js
import Amplify from 'aws-amplify'
import config from './aws-exports'
Amplify.configure(config)
```

Now, our app is ready to start using our AWS services.

### Using the withAuthenticator component

To add authentication, we'll go into __src/App.js__ and first import the `withAuthenticator` HOC (Higher Order Component) from `aws-amplify-react`:

### src/App.js

```js
import { withAuthenticator } from 'aws-amplify-react'
```

Next, we'll wrap our default export (the App component) with the `withAuthenticator` HOC:

```js
export default withAuthenticator(App, { includeGreetings: true })
```

```sh
# run the app

npm start
```

Now, we can run the app and see that an Authentication flow has been added in front of our App component. This flow gives users the ability to sign up & sign in.

> To view the new user that was created in Cognito, go back to the dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

### Accessing User Data

We can access the user's info now that they are signed in by calling `Auth.currentAuthenticatedUser()`.

### src/App.js

```js
import { Auth } from 'aws-amplify'

class App extends React.Component {
  async componentDidMount() {
    const user = await Auth.currentAuthenticatedUser()
    console.log('user info:', user.signInUserSession.idToken.payload)
    console.log('username:', user.username)
  }

  render() {
    return (
      // existing code
      <div className="App">
       // rest of code
      </div>
    )
  }
}
```

### Custom authentication strategies

The `withAuthenticator` component is a really easy way to get up and running with authentication, but in a real-world application we probably want more control over how our form looks & functions.

Let's look at how we might create our own authentication flow.

To get started, we would probably want to create input fields that would hold user input data in the state. For instance when signing up a new user, we would probably need 4 user inputs to capture the user's username, email, password, & phone number.

To do this, we could create some initial state for these values & create an event handler that we could attach to the form inputs:

```js
// initial state
state = {
  username: '', password: '', email: '', phone_number: ''
}

// event handler
onChange = (event) => {
  this.setState({ [event.target.name]: event.target.value })
}

// example of usage with input
<input
  name='username'
  placeholder='username'
  onChange={this.onChange}
/>
```

We'd also need to have a method that signed up & signed in users. We can use the Auth class to do thi. The Auth class has over 30 methods including things like `signUp`, `signIn`, `confirmSignUp`, `confirmSignIn`, & `forgotPassword`. Thes functions return a promise so they need to be handled asynchronously.

```js
// import the Auth component
import { Auth } from 'aws-amplify'

// Class method to sign up a user
signUp = async() => {
  const { username, password, email, phone_number } = this.state
  try {
    await Auth.signUp({ username, password, attributes: { email, phone_number }})
  } catch (err) {
    console.log('error signing up user...', err)
  }
}
```

## Adding a Serverless Function

### Adding a basic Lambda Function

To add a serverless function, we can run the following command:

```sh
amplify add function
```

> Answer the following questions

- Provide a friendly name for your resource to be used as a label for this category in the project: __basiclambd
a__
- Provide the AWS Lambda function name: __basiclambda__
- Choose the function template that you want to use: __Hello world function__
- Do you want to edit the local lambda function now? __Y__

> This should open the function package located at __amplify/backend/function/basiclambda/src/index.js__.

Edit the function to look like this, & then save the file.

```js
exports.handler = function (event, context) {
  console.log('event: ', event)
  const body = {
    message: "Hello world!"
  }
  const response = {
    statusCode: 200,
    body
  }
  context.done(null, response);
}
```

Next, we can test this out by running:

```sh
amplify function invoke basiclambda
```

- Provide the name of the script file that contains your handler function: index.js
-  Provide the name of the handler function to invoke: handler

You'll notice the following output from your terminal:

```sh
Running "lambda_invoke:default" (lambda_invoke) task

event:  { key1: 'value1', key2: 'value2', key3: 'value3' }

Success!  Message:
------------------
{"statusCode":200,"body":{"message":"Hello world!"}}

Done.
Done running invoke function.
```

Where is the event data coming from? It is coming from the values located in event.json in the function folder (__amplify/backend/function/basiclambda/src/event.json__). If you update the values here, you can simulate data coming arguments the event.

Feel free to test out the function by updating `event.json` with data of your own.

## Adding a REST API

To add a REST API, we can use the following command:

```sh
amplify add api
```

> Answer the following questions

- Please select from one of the above mentioned services __REST__   
- Provide a friendly name for your resource that will be used to label this category in the project: __amplifyrestapi__   
- Provide a path, e.g. /items __/pets__   
- Choose lambda source __Create a new Lambda function__   
- Provide a friendly name for your resource that will be used to label this category in the project: __amplifyrestapilambda__   
- Provide the Lambda function name: __amplifyrestapilambda__   
- Please select the function template you want to use: __Serverless express function (Integration with Amazon API Gateway)__   
- Do you want to edit the local lambda function now? __Y__   

> Update the existing `app.get('/pets') route with the following:
```js
app.get('/pets', function(req, res) {
  // Add your code here
  // Return the API Gateway event and query string parameters for example
  const pets = [
    'Spike', 'Zeus', 'Butch'
  ]
  res.json({
    success: 'get call succeed!',
    url: req.url,
    pets
  });
});
```

- Restrict API access __Y__
- Who should have access? __Authenticated users only__
- What kind of access do you want for Authenticated users __read/write__
- Do you want to add another path? (y/N) __N__     

> Now the resources have been created & configured & we can push them to our account: 

```bash
amplify push
```

### Interacting with the new API

Now that the API is created we can start sending requests to it & interacting with it.

Let's request some data from the API:

```js
// src/App.js

import { API } from 'aws-amplify'

// create initial state
state = { pets: [] }

// fetch data at componentDidMount
componentDidMount() {
  this.getData()
}
getData = async() => {
  try {
    const data = await API.get('amplifyrestapi', '/pets')
    console.log('data from Lambda REST API: ', data)
    this.setState({ pets: data.pets })
  } catch (err) {
    console.log('error fetching data..', err)
  }
}

// implement into render method
{
  this.state.pets.map((p, i) => (
    <p key={i}>{p}</p>
  ))
}
```

### Fetching data from another API in a Lambda function.

Next, let's configure the REST API to add another endpoint that will fetch data from an external resource.

First, we'll need to configure the API to know about the new path:

```sh
amplify configure api
```

- Please select from one of the below mentioned services __REST__
- Please select the REST API you would want to update __amplifyrestapi__
- What would you like to do __Add another path__
- Provide a path (e.g., /items) __/people__
- Choose a Lambda source __Use a Lambda function already added in the current Amplify project__
- Choose the Lambda function to invoke by this path __amplifyrestapilambda__
- Restrict API access __Yes__
- Who should have access? __Authenticated users only__
- What kind of access do you want for Authenticated users __read/write__
- Do you want to add another path? __No__

The next thing we need to do is install `axios` in our Lambda function folder.

Navigate to __amplify/backend/function/<FUNCTION_NAME>/src__ and install __axios__:

```sh
yarn add axios

# or

npm install axios
```

Next, in __amplify/backend/function/<FUNCTION_NAME>/src/app.js__, let's add a new endpoint that will fetch a list of people from the [Star Wars API](https://swapi.co/).

```js
// require axios
var axios = require('axios')

// add new /people endpoint
app.get('/people', function(req, res) {
  axios.get('https://swapi.co/api/people/')
    .then(response => {
      res.json({
        people: response.data.results,
        success: 'get call succeed!',
        url: req.url
      });
    })
    .catch(err => {
      res.json({
        error: 'error fetching data'
      });
    })
});
```

Now we can add a new function called getPeople that will call this API:

```js
componentDidMount() {
  this.getData()
  this.getPeople() // new
}

getPeople = async() => {
  try {
    const data = await API.get('amplifyrestapi', '/people')
    console.log('data from new people endpoint:', data)
  } catch (err) {
    console.log('error fetching data..', err)
  }
}
```

## Adding a GraphQL API

To add a GraphQL API, we can use the following command:

```sh
amplify add api
```

Answer the following questions

- Please select from one of the above mentioned services __GraphQL__   
- Provide API name: __GraphQLPets__   
- Choose an authorization type for the API __API key__   
- Do you have an annotated GraphQL schema? __N__   
- Do you want a guided schema creation? __Y__   
- What best describes your project: __Single object with fields (e.g. “Todo” with ID, name, description)__   
- Do you want to edit the schema now? (Y/n) __Y__   

> When prompted, update the schema to the following:   

```graphql
type Pet @model {
  id: ID!
  clientId: ID
  name: String!
  description: String!
}
```

> Next, let's push the configuration to our account:

```bash
amplify push
```

- Do you want to generate code for your newly created GraphQL API __Y__
- Choose the code generation language target: __javascript__
- Enter the file name pattern of graphql queries, mutations and subscriptions: __(src/graphql/**/*.js)__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions? __Y__
- Enter maximum statement depth [increase from default if your schema is deeply nested] __2__

To view the services created by Amplify at any time you can run the `console` command & choose the feature you'd like to view:

```sh
amplify console
```

### Adding mutations from within the AWS AppSync Console

In the AWS AppSync console, open your API & then click on Queries.

Execute the following mutation to create a new pet in the API:

```graphql
mutation createPet {
  createPet(input: {
    name: "Zeus"
    description: "Best dog in the western hemisphere"
  }) {
    id
  }
}
```

Now, let's query for the pet:

```graphql
query listPets {
  listPets {
    items {
      id
      name
      description
    }
  }
}
```

We can even add search / filter capabilities when querying:

```graphql
query listPets {
  listPets(filter: {
    description: {
      contains: "dog"
    }
  }) {
    items {
      id
      name
      description
    }
  }
}
```

### Interacting with the GraphQL API from our client application - Querying for data

Now that the GraphQL API is created we can begin interacting with it!

The first thing we'll do is perform a query to fetch data from our API.

To do so, we need to define the query, execute the query, store the data in our state, then list the items in our UI.

### src/App.js

```js
import React from 'react'
import './App.css'

// imports from Amplify library
import { API, graphqlOperation } from 'aws-amplify'
// import query
import { listPets as ListPets } from './graphql/queries'

class App extends React.Component {
  // create initial state
  state = {
    pets: []
  }
  // execute the query in componentDidMount
  async componentDidMount() {
    try {
      const pets = await API.graphql(graphqlOperation(ListPets))
      console.log('pets:', pets)
      this.setState({
        pets: pets.data.listPets.items
      })
    } catch (err) {
      console.log('error fetching pets...', err)
    }
  }
  // implement into render method
  render() {
    return (
      <div className="App">
        <h2>Amplify App</h2>
        {
          this.state.pets.map((pet, index) => (
            <div key={index}>
              <h3>{pet.name}</h3>
              <p>{pet.description}</p>
            </div>
          ))
        }
      </div>
    );
  }
}

export default App
```

## Performing mutations

 Now, let's look at how we can create mutations.

```js
// import uuid to create a unique client ID
import uuid from 'uuid/v4'

// import the mutation
import { createPet as CreatePet } from './graphql/mutations'

const CLIENT_ID = uuid()

// update initial state
state = {
  name: '', description: '', pets: []
}

createPet = async() => {
  const { name, description } = this.state
  if (name === '' || description === '') return
  const pet = {
    name, description, clientId: CLIENT_ID
  }
  const updatedPetArray = [...this.state.pets, pet]
  this.setState({ pets: updatedPetArray })
  try {
    await API.graphql(graphqlOperation(CreatePet, { input: pet }))
    console.log('item created!')
  } catch (err) {
    console.log('error creating pet...', err)
  }
}

// change state then user types into input
onChange = (event) => {
  this.setState({
    [event.target.name]: event.target.value
  })
}

// add UI with event handlers to manage user input
<input
  name='name'
  onChange={this.onChange}
  value={this.state.name}
/>
<input
  name='description'
  onChange={this.onChange}
  value={this.state.description}
/>
<button onClick={this.createPet}>Create Pet</button>
```

### GraphQL Subscriptions

Next, let's see how we can create a subscription to subscribe to changes of data in our API.

To do so, we need to define the subscription, listen for the subscription, & update the state whenever a new piece of data comes in through the subscription.

```js
// import the subscription
import { onCreatePet as OnCreatePet } from './graphql/subscriptions'

// define the subscription in the class
subscription = {}

// subscribe in componentDidMount
componentDidMount() {
  this.subscription = API.graphql(
    graphqlOperation(OnCreatePet)
  ).subscribe({
      next: (eventData) => {
        console.log('eventData', eventData)
        const pet = eventData.value.data.onCreatePet
        if (pet.clientId === CLIENT_ID) return
        
        const pets = [ ...this.state.pets, pet]
        this.setState({ pets })
      }
  })
}

componentWillUnmount() {
  this.subscription.unsubscribe()
}
```

### Adding Basic Authorization to the GraphQL API

To add authorization to the API, we can re-configure the API to use our cognito identity pool. To do so, we can run `amplify configure api`:

```sh
amplify configure api
```
Please select from one of the below mentioned services: __GraphQL__
Choose an authorization type for the API: __Amazon Cognito User Pool__

Next, we'll run `amplify push`:

```sh
amplify push
```

- Do you want to update code for your updated GraphQL API __N__

Now, we can only access the API with a logged in user.

## Working with Storage

To add storage, we can use the following command:

```sh
amplify add storage
```

> Answer the following questions   

- Please select from one of the below mentioned services __Content (Images, audio, video, etc.)__
- Please provide a friendly name for your resource that will be used to label this category in the
 project: __YOURAPINAME__
- Please provide bucket name: __YOURUNIQUEBUCKETNAME__
- Who should have access: __Auth users only__
- What kind of access do you want for Authenticated users __read/write__   


```sh
amplify push
```

Now, storage is configured & ready to use.

What we've done above is created configured an Amazon S3 bucket that we can now start using for storing items.

For example, if we wanted to test it out we could store some text in a file like this:

```js
import { Storage } from 'aws-amplify'

// create function to work with Storage
addToStorage = () => {
  Storage.put('javascript/MyReactComponent.js', `
    import React from 'react'
    const App = () => (
      <p>Hello World</p>
    )
    export default App
  `)
    .then (result => {
      console.log('result: ', result)
    })
    .catch(err => console.log('error: ', err));
}

// add click handler
<button onClick={this.addToStorage}>Add To Storage</button>
```

This would create a folder called `javascript` in our S3 bucket & store a file called __MyReactComponent.js__ there with the code we specified in the second argument of `Storage.put`.

To view the services created by Amplify at any time you can run the `console` command & choose the feature you'd like to view:

```sh
amplify console
```

If we want to read everything from this folder, we can use `Storage.list`:

```js
readFromStorage = () => {
  Storage.list('javascript/')
    .then(data => console.log('data from S3: ', data))
    .catch(err => console.log('error'))
}
```

If we only want to read the single file, we can use `Storage.get`:

```js
readFromStorage = () => {
  Storage.get('javascript/MyReactComponent.js')
    .then(data => console.log('data from S3: ', data))
    .catch(err => console.log('error'))
}
```

If we wanted to pull down everything, we can use `Storage.list`:

```js
readFromStorage = () => {
  Storage.list('')
    .then(data => console.log('data from S3: ', data))
    .catch(err => console.log('error'))
}
```

### Working with images

Working with images is also easy:

```js
class S3ImageUpload extends React.Component {
  onChange(e) {
      const file = e.target.files[0];
      Storage.put('example.png', file, {
          contentType: 'image/png'
      })
      .then (result => console.log(result))
      .catch(err => console.log(err));
  }

  render() {
      return (
          <input
              type="file" accept='image'
              onChange={(e) => this.onChange(e)}
          />
      )
  }
}

```

We can even use the S3Album component, one of a few components in the AWS Amplify React library to create a preconfigured photo picker:

```js
import { S3Album, withAuthenticator } from 'aws-amplify-react'

class App extends Component {
  render() {
    return (
      <div className="App">
        <S3Album path={''} picker />
      </div>
    );
  }
}
```

## Adding Analytics

To add analytics, we can use the following command:

```sh
amplify add analytics
```

> Next, we'll be prompted for the following:

- Provide your pinpoint resource name: __amplifyanalytics__   
- Apps need authorization to send analytics events. Do you want to allow guest/unauthenticated users to send analytics events (recommended when getting started)? __Y__   
- overwrite YOURFILEPATH-cloudformation-template.yml __Y__

### Recording events

Now that the service has been created we can now begin recording events.

To record analytics events, we need to import the `Analytics` class from Amplify & then call `Analytics.record`:

```js
import { Analytics } from 'aws-amplify'

state = {username: ''}

async componentDidMount() {
  try {
    const user = await Auth.currentAuthenticatedUser()
    this.setState({ username: user.username })
  } catch (err) {
    console.log('error getting user: ', err)
  }
}

recordEvent = () => {
  Analytics.record({
    name: 'My test event',
    attributes: {
      username: this.state.username
    }
  })
}

<button onClick={this.recordEvent}>Record Event</button>
```

## Working with multiple environments

You can create multiple environments for your application in which to create & test out new features without affecting the main environment which you are working on.

When you create a new environment from an existing environment, you are given a copy of the entire backend application stack from the original project. When you make changes in the new environment, you are then able to test these new changes in the new environment & merge only the changes that have been made since the new environment was created back into the original environment.

Let's take a look at how to create a new environment. In this new environment, we'll re-configure the GraphQL Schema to have another field for the pet owner.

First, we'll initialize a new environment using `amplify env add`:

```sh
amplify env add

> Do you want to use an existing environment? No
> Enter a name for the environment: apiupdate
> Do you want to use an AWS profile? Y
> Please choose the profile you want to use: amplify-workshop-profile
```

Once the new environment is initialized, we should be able to see some information about our environment setup by running:

```sh
amplify env list

| Environments |
| ------------ |
| local        |
| *apiupdate   |
```

Now we can update the GraphQL Schema in `amplify/backend/api/GraphQLPets/schema.graphql` to the following (adding the `owner` field):

```graphql
type Pet @model {
  id: ID!
  clientId: ID
  name: String!
  description: String!
  owner: String
}
```

Now, we can create this new stack by running `amplify push`:

```sh
amplify push
```

After we test it out, we can now merge it into our original local environment:

```sh
amplify env checkout local
```

Next, run the `status` command:

```sh
amplify status
```

You should now see an __Update__ operation:

```sh
Current Environment: local

| Category | Resource name   | Operation | Provider plugin   |
| -------- | --------------- | --------- | ----------------- |
| Api      | GraphQLPets     | Update    | awscloudformation |
| Auth     | cognito75a8ccb4 | No Change | awscloudformation |
```

To deploy the changes, run the push command:

```sh
amplify push
```

- Do you want to update code for your updated GraphQL API? __Y__
- Do you want to generate GraphQL statements? __Y__

Now, the changes have been deployed & we can delete the apiupdate environment:

```sh
amplify env remove apiupdate

Do you also want to remove all the resources of the environment from the cloud? Y
```

Now, we should be able to run the `list` command & see only our main environment:

```sh
amplify env list
```

## Deploying via the Amplify Console

For hosting, we can use the [Amplify Console](https://aws.amazon.com/amplify/console/) to deploy the application.

The first thing we need to do is [create a new GitHub repo](https://github.com/new) for this project. Once we've created the repo, we'll copy the URL for the project to the clipboard & initialize git in our local project:

```sh
git init

git remote add origin git@github.com:username/project-name.git

git add .

git commit -m 'initial commit'

git push origin master
```

Next we'll visit the Amplify Console in our AWS account at [https://eu-west-1.console.aws.amazon.com/amplify/home](https://eu-west-1.console.aws.amazon.com/amplify/home).

Here, we'll click __Get Started__ to create a new deployment. Next, authorize Github as the repository service.

Next, we'll choose the new repository & branch for the project we just created & click __Next__.

In the next screen, we'll create a new role & use this role to allow the Amplify Console to deploy these resources & click __Next__.

Finally, we can click __Save and Deploy__ to deploy our application!

Now, we can push updates to Master to update our application.

## Adding Fine-grained Authorization to the GraphQL API

Let's how how we can access the user's identity in the resolver. To do so, we'll first need to store the user's identity in the database table as userId & add a new index on the table to query for this user ID.

__Adding an index to the table__

Next, we'll want to add a new GSI (global secondary index) in the table. We do this so we can query on the index to gain new data access pattern.

To add the index, open the [AppSync Console](https://console.aws.amazon.com/appsync/home), choose your API & click on __Data Sources__. Next, click on the data source link.

From here, click on the __Indexes__ tab & click __Create index__.

For the __Partition key__, input `userId` to create a `userId-index` Index name & click __Create index__.

Next, we'll update the resolver for adding pets & querying for pets.

#### Updating the resolvers

In the folder __amplify/backend/api/GraphQLPets/resolvers__, create the following two resolvers:

__Mutation.createPet.req.vtl__ & __Query.listPets.req.vtl__.

__Mutation.createPet.req.vtl__

```vtl
$util.qr($context.args.input.put("createdAt", $util.time.nowISO8601()))
$util.qr($context.args.input.put("updatedAt", $util.time.nowISO8601()))
$util.qr($context.args.input.put("__typename", "Pet"))
$util.qr($context.args.input.put("userId", $ctx.identity.sub))

{
  "version": "2017-02-28",
  "operation": "PutItem",
  "key": {
      "id":     $util.dynamodb.toDynamoDBJson($util.defaultIfNullOrBlank($ctx.args.input.id, $util.autoId()))
  },
  "attributeValues": $util.dynamodb.toMapValuesJson($context.args.input),
  "condition": {
      "expression": "attribute_not_exists(#id)",
      "expressionNames": {
          "#id": "id"
    }
  }
}
```

__Query.listPets.req.vtl__

```vtl
{
    "version" : "2017-02-28",
    "operation" : "Query",
    "index" : "userId-index",
    "query" : {
        "expression": "userId = :userId",
        "expressionValues" : {
            ":userId" : $util.dynamodb.toDynamoDBJson($ctx.identity.sub)
        }
    }
}
```

Next, run the push command again to update the API:

```sh
amplify push
```

> Now that we've added authorization to the API, we will have to log in if we would like to perform queries in the AppSync Console. To log in, find the `aws_user_pools_web_client_id` from `aws-exports.js` & log in using your `username` & `password`.

Now when we create new pets the `userId` field will be populated with the `userId` of the logged-in user.

When we query for the pets, we will only receive the data for the items that we created ourselves.

```graphql
query listPets {
  listPets {
    items {
      id
      name
      description
    }
  }
}
```

#### Creating custom resolvers (Advanced)

Now let's say we want to define & use a custom GraphQL operation & create corresponding resolvers that do not yet exist? We can also do that using Amplify & the local environment.

Let's create a query & resolvers that will query for __all__ pets in the API, similar to the functionality we had before changing the `listPets` resolver.

To do so, we need to do three things:

1. Define the operations we'd like to have available in our schema (add queries, mutations, subscriptions to __schema.graphql__).

To do so, update __amplify/backend/api/GraphQLPets/schema.graphql__ to the following:

```graphql
type Pet @model {
  id: ID!
  clientId: ID
  name: String!
  description: String!
}

type ModelPetConnection {
  items: [Pet]
  nextToken: String
}

type Query {
  listAllPets(limit: Int, nextToken: String): ModelPetConnection
}
```

2. Create the request & response mapping templates in __amplify/backend/api/GraphQLPets/resolvers__.

__Query.listAllPets.req.vtl__

```vtl
{
    "version" : "2017-02-28",
    "operation" : "Scan",
    "limit": $util.defaultIfNull(${ctx.args.limit}, 20),
    "nextToken": $util.toJson($util.defaultIfNullOrBlank($ctx.args.nextToken, null))
}
```

__Query.listAllPets.res.vtl__

```vtl
{
    "items": $util.toJson($ctx.result.items),
    "nextToken": $util.toJson($util.defaultIfNullOrBlank($context.result.nextToken, null))
}
```

3. Update __amplify/backend/api/GraphQLPets/stacks/CustomResources.json__ with the definition of the custom resource.

Update the `Resources` field in __CustomResources.json__ to the following:

```json
{
  ...rest of template,
  "Resources": {
    "QueryListAllPetsResolver": {
      "Type": "AWS::AppSync::Resolver",
      "Properties": {
        "ApiId": {
          "Ref": "AppSyncApiId"
        },
        "DataSourceName": "PetTable",
        "TypeName": "Query",
        "FieldName": "listAllPets",
        "RequestMappingTemplateS3Location": {
          "Fn::Sub": [
            "s3://${S3DeploymentBucket}/${S3DeploymentRootKey}/resolvers/Query.listAllPets.req.vtl",
            {
              "S3DeploymentBucket": {
                "Ref": "S3DeploymentBucket"
              },
              "S3DeploymentRootKey": {
                "Ref": "S3DeploymentRootKey"
              }
            }
          ]
        },
        "ResponseMappingTemplateS3Location": {
          "Fn::Sub": [
            "s3://${S3DeploymentBucket}/${S3DeploymentRootKey}/resolvers/Query.listAllPets.res.vtl",
            {
              "S3DeploymentBucket": {
                "Ref": "S3DeploymentBucket"
              },
              "S3DeploymentRootKey": {
                "Ref": "S3DeploymentRootKey"
              }
            }
          ]
        }
      }
    }
  },
  ...rest of template,
}
```

Now that everything has been updated, run the push command again:

```sh
amplify push
```

Now, we should have a new query available in our AppSync API to list all pets.

## Removing Services

If at any time, or at the end of this workshop, you would like to delete a service from your project & your account, you can do this by running the `amplify remove` command:

```sh
amplify remove auth

amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.

## Deleting entire project

```sh
amplify delete
```