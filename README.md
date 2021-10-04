# Authentication and authorisation workshop

This workshop targets students of EPITA SIGL 2022.

You will integrate an IDP called [Auth0](https://auth0.com/) to Garlaxy.

## Step 1: Setup your account on Auth0

- Go to [Auth0](https://auth0.com/auth/login) and login with your github account.
- Create a new EU dev tenant with the format `garlaxy-groupe-XX` (e.g. `garlaxy-groupe-13` for group 13)
![tenant.png](images/tenant.png)

To use Auth0 service in yours app, you need credentials.

To get them, go to the `Applications` section in Auth0 dashboard.

A default `Application` is already available but we will create a new one called Arla:

- Click on `Create Application` button
- Name it `Arla` and select `Single Page Web Application`
- Click on Create

> Note: The dashboard propose you to see a Quick Start with the different technology.
> Feel free to check available Quick Start for your favourite tech.
> 
> We adapted a bit the code provided from [React Quick Start](https://auth0.com/docs/quickstart/spa/react/01-login) for today's workshop.

- You have credentials available in the `Settings` tab of your `Arla` `Application`.
- You need to configure the Callback URLs, Logout URLs, Allowed Web Origins. Put these URLs (with comma sperator), replacing XX by your group number:
  - http://localhost:3000             (use for local dev environment)
  - http://localhost:3030
  - https://groupeXX.arla-sigl.fr     (use for production environment)
  - https://api.groupeXX.arla-sigl.fr

- Save changes

Before moving to code changes, let's create a test user in your Auth0 Application:
- From your Auth0 dashboard, move to user managment
![user-managment.png](images/user-managment.png)
- Click on `Add User` with credentials you'll remember and can share to your group
![add-user.png](images/add-user.png)


## Step 2: Integrate Auht0 login to your frontend

**Objective**: Allow only authenticated users to access Garlaxy.

### Install the Auth0 React SDK

Auth0 provide an easy to use sdk for react.

First you need to install [Auth0 React SDK](https://www.npmjs.com/package/@auth0/auth0-react), in your `frontend/` directory:

```sh
# from frontend/
nvm use v16
npm i --save @auth0/auth0-react
```

You can now import the sdk and start to configure the `Auth0Provider` component.

> Note: Under the hood, the Auth0 React SDK uses React Context to manage the authentication state of your users.
> One way to integrate Auth0 with your React app is to wrap your root component 
> with an Auth0Provider that you can import from the SDK.

In your `frontend/src/App.js`, you will add:
- `import { Auth0Provider } from "@auth0/auth0-react";`
- `Auth0Provider` provided by Auth0; configured with your Auth0 Arla application's settings

```javascript
// inside frontend/src/App.js
//...
import { Auth0Provider } from "@auth0/auth0-react";

function App() {
  const [state, dispatch] = React.useReducer(reducer, initialState);
  return (
    <Auth0Provider 
      clientId="<YOUR CLIENT ID>"
      domain="garlaxy-groupe-XX.eu.auth0.com"
      redirectUri={window.location.origin}
      cacheLocation="localstorage"
    >
      <GarlaxyContext.Provider value={{ state, dispatch }}>
        <Header></Header>
        <Content></Content>
        <Footer></Footer>
      </GarlaxyContext.Provider>
    </Auth0Provider>
  );
}

```

> Note: The Auth0Provider component takes the following props:
> - `YOUR_DOMAIN` and `YOUR_CLIENT_ID`: The values of these properties correspond to the "Domain" and "Client ID" values 
> present under the "Settings" of the single-page application that you registered with Auth0.
>
> - `redirectUri`: The URL to where you'd like to redirect your users after they authenticate with Auth0.
> 
> `Auth0Provider` stores the authentication state of your users and the state of the SDK — whether Auth0 is ready to use or not.
> It also exposes helper methods to log in and log out your users, which you can access using the useAuth0() hook.

### Add Login to your Application

The Authenticated is here to automaticly redirect the user to Auth0 when it's not logged.

- Create a file `frontend/src/Authenticated.js` with:

```jsx
import React from "react";
import { useAuth0 } from "@auth0/auth0-react";

export function Authenticated({ children }) {
  const { loginWithRedirect, user, isLoading } = useAuth0();

  React.useEffect(() => {
    const redirect = async () => {
      if (!user && !isLoading) {
        await loginWithRedirect();
      }
    };
    redirect();
  }, [isLoading]);
  
  return isLoading ? <span>Loading ...</span> : <>{children}</>;
};
```

> Note: This means that:
> If user is not authenticated, you are redirecting user to login page.
> If user is logged in, you are rendering child components.

### Add Logout to your Application

The `useAuth0` react hook is providing a `logout` method for the user to terminate her/his session.

- Create a new file `frontend/src/Logout.js` with:

```jsx
import React from "react";
import { useAuth0 } from "@auth0/auth0-react";

export function Logout(){
  const { logout } = useAuth0();
  return (
    <a
      className="logout"
      onClick={() => {
        logout({ returnTo: document.location.origin });
      }}
    >
      Déconnexion
    </a>
  );
};
```

This will display the user email you've logged in with and display a `Logout` button that should redirect you to the main page.

### Add "Déconnexion" to your header

**Objective**: Create a small `Profile` component in your header, where you will add your logout button.

- Create a new file `frontend/src/Profile.js` with:
```jsx
// inside frontend/src/Profile.js
import React from "react";
import { useAuth0 } from "@auth0/auth0-react";
import { Logout } from "./Logout";

export function Profile() {
  const { user } = useAuth0();
  const userMail = user ? user.email : "-";
  return (
    <div className="profile">
      <span>{userMail}</span>
      <Logout />
    </div>
  );
}
```  
- Add profile to `frontend/src/Header.js` so it looks like:
```jsx
// inside frontend/src/Header.js
import React from "react";
import { Profile } from "./Profile";

export function Header() {
  return (
    <div className="header">
      <p className="garlaxy-title text-centered">GARLAXY</p>
      <Profile />
    </div>
  );
}
```
- Add css properties for normal inside `frontend/src/App.css`:
```css
.profile {
  display: flex;
  align-items: center;
}

.logout {
  cursor: pointer;
  text-decoration: underline;
  margin-left: 1rem;
}
```
- Add css properties for small screen inside `frontend/src/App.css`:
```css
/* RESPONSIVE PROPERTIES */
@media screen and (max-width: 768px) {
  /* ... */ 
  .header {
    flex-direction: column;
    height: auto;
  }
  /* ... */
}
```

You should be all set! 

Start your frontend code
```sh
# from frontend/
npm start
```

And you should have your Garlaxy behind a login page, on [locahost:3000](http://localhost:3000)

## Step 3: Secure your API

**Objective**: Only user who are logged in can consume your "Ressource" web API.

### Create your API in Auth0

You need to create credentials for your web API in the same Auth0 tenant:

- go to your Dashboard > API > Create new API
![api-menu.png](images/api-menu.png)
- Set the following fields (adapt for your group name):
  - Name: arla-groupe-XX-api
  - Identifier: https://api.groupeXX.arla-sigl.fr
  - Leave default signing algorithm: RS256

You should see quick-start and your API created with an identifier; something like:
![create-api.png](images/create-api.png)

From your `backend/` code, adapt with the code Auth0 provides in its API QuickStart > NodeJS section:

- install dependencies

```sh
# from backend/
nvm use v16
npm install --save express-jwt jwks-rsa
```

- Adapt the following code `jwtCheck` variable with your own group's context. Verify your credentials by looking in your Auth0 API QuickStart NodeJS section (like screenshot above):

```js
// src/server.js
//...
const jwt = require('express-jwt');
const jwks = require('jwks-rsa');
//...

var jwtCheck = jwt({
  secret: jwks.expressJwtSecret({
    cache: true,
    rateLimit: true,
    jwksRequestsPerMinute: 5,
    jwksUri: "https://garlaxy-groupe-XX.eu.auth0.com/.well-known/jwks.json",
  }),
  audience: "https://api.groupeXX.arla-sigl.fr",
  issuer: "https://garlaxy-groupe-XX.eu.auth0.com/",
  algorithms: ["RS256"],
});

app.use(jwtCheck);

// ...
```

That's it! You've secured your API. `jwtCheck` is a middleware that will check if the `Bearer` token is valid on **all incomming requests to any routes**.

You can try out to curl your API again, you should get an `Unauthorized` message:

```sh
# make sure to run your api from a terminal session;
# from backend/ 
# nvm use v16
# node src/server.js

curl 'http://localhost:3030/v1/resource?page=1&limit=5'
# You should get an HTML containing:
# UnauthorizedError: No authorization token was found
```

And if you wanna verify if giving a fake token would work, you can try (and make sure it fails!):

```sh
curl -H "Authorization: Bearer AFakeT0kenThatMeansNothingButIAmStillTrying" 'http://localhost:3030/v1/resource?page=1&limite=5'
# You should get an HTML containing:
# UnauthorizedError: jwt malformed
```

The only way to get a correct token is if you are authenticated thru the login page of frontend.

## Step 4: Call your Secured API

**Objective**: Make authenticated calls to your web API, from your frontend code.

You may have noticed the usage of [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) to consume your web API inside `frontend/src/Content.js`.

In this step, you will:
- Setup your audience
- Adapt fetch to include a secured token in request's headers

### Adapt your frontend component in the frontend

Set the `audience` property of your `<Auth0Provider />` to your `API identifier` from your newly created API in Auth0:
![api-audience.png](images/api-audience.png)
```jsx
// inside frontend/src/App.js
// ...
<Auth0Provider
    // ...
    audience={/* Your API Identifier */}
    cacheLocation="localstorage"
  >
```

This will make sure the token you get after authentication has access your API.

### Adapt fetch

You will use the provide method `getAccessTokenSilently` from the `useAuth0` react hook provided by Auth0 SDK.

In your `frontend/src/Content.js`, add the following changes:

```js
// From a new file inside frontend
// frontend/src/Content.js
// ...
import { useAuth0 } from "@auth0/auth0-react";
// ...
export function Content() {
  const { dispatch } = React.useContext(GarlaxyContext);
  const { getAccessTokenSilently } = useAuth0();

  React.useEffect(() => {
    async function getResources() {
      const token = await getAccessTokenSilently();
      const apiResponse = await fetch(
        "https://localhost:3030/v1/resource?page=1&limit=10",
        { headers: { Authorization: `Bearer ${token}` } }
      );
      const resourcesDocument = await apiResponse.json();
      // ...
```

You are now passing an Authorization header with a JWT token, that you got from the moment you've authenticated to the app with the correct audience.

> Note: You may have noticed `async` and `await` keywords. It's syntaxic sugar to replace many chain callbacks using the classic promise's `then` and `catch` keyword.

# Step 5: Adapt frontend config to manage both localhost and production domains

**Objective**: Use localhost:3030 when running frontend for development and https://api.groupeXX.arla-sigl.fr when building for production.

Create-react-app, the tool you've used to create your project, offers [a way to add custom environment variables when building your frontend](https://create-react-app.dev/docs/adding-custom-environment-variables/).

From your `frontend/src/Content.js` file where you call `fetch`:

```js
// inside frontend/src/Content.js

export function Content() {
  // ...
  React.useEffect(() => {
    async function getResources() {
      // ...
      const apiResponse = await fetch(
        `${process.env.REACT_APP_API_HOST}/v1/resource?page=1&limit=10`,
        { headers: { Authorization: `Bearer ${token}` } }
      );
      // ...
```

And adapt your `npm start` script (replace `groupeXX` by your group number):
```json
{
  // ...
  "scripts": {
    "start": "REACT_APP_API_HOST=http://localhost:3030 node scripts/start.js",
    "build": "REACT_APP_API_HOST=https://api.groupeXX.arla-sigl.fr node scripts/build.js",
    // ...
}
```

You should be all set:
- When running `npm run build` inside your docker container, you are setting fetch to call the web API on `https://api.groupeXX.arla-sigl.fr`
- When running `npm start` (or `npm run start`) from your terminal, you are setting fetch to call the web API on `http://localhost:3030`

Just commit/push your changes and your version with login and secured API should be live, congrats!
