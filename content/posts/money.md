---
title: "Building a simple money tracking mobile application with Corteza"
date: 2022-01-16T18:35:59+01:00
tags: [development,Flutter,Corteza,afternoon-project]
---

This project is the result of my experimentation with mobile development.
Do keep this in mind if you decide to follow my instructions to implement your mobile app.
The concepts are there, but some Flutter related specifics may not be that *by the book*.

The source code is available on [GitHub/tjerman/money](https://github.com/tjerman/money).

# What we will be building

We'll build simple money tracking application that holds a list of transactions (inbound and outbound).
These transactions can be accessed and added from inside Corteza and displayed inside our mobile app.

Here is a quick overview of what we'll do:

1. **Setup Corteza** to have an instance we can work with.
   We'll also need to do some magic to access our local instance from inside our mobile app.
2. **Setup the Low Code app** so that we'll be able to track our transactions.
3. **Setup an auth client** so that we'll be able to authenticate our mobile application
4. **Develop the mobile application**

# Setup Corteza

Setting up a new Corteza instance is as easy as eating pie.
All you need to do is **create a project folder** and **paste in some config files**.

Refer to the **File structure** section to see what the entire file structure will look like.
This is just to save a few minutes later on.

> **NOTE:** Additional DevOps and deploy related documentation is available in the [official documentation](https://docs.cortezaproject.org/corteza-docs/2021.9/devops-guide/index.html).

In the project repository (my Corteza configs reside inside the `/backend` root directory), we need two config files -- `/backend/.env` and `/backend/docker-compose.yaml`.

> **NOTE:** Pay attention to the `DOMAIN` variable; we'll be using a little trick so we can access our local Corteza from our mobile app.

Here is an example of the `/backend/.env`:

```
DOMAIN=somethingsomething.ngrok.io
VERSION=2021.9.6

# Just so we can use qwerty as my password
AUTH_PASSWORD_SECURITY=false

DB_DSN=corteza:corteza@tcp(db:3306)/corteza?collation=utf8mb4_general_ci

# So we can use a single container
HTTP_WEBAPP_ENABLED=true
ACTIONLOG_ENABLED=false

```

Here is an example of the `/backend/docker-compose.yaml`:

"`yaml
version: '3.5'

services:
  server:
    image: cortezaproject/corteza:${VERSION}
    restart: always
    env_file: [ .env ]
    depends_on: [ db ]
    ports: [ "127.0.0.1:18080:80" ]

  db:
    image: percona:8.0
    restart: always
    volumes: 
      - "dbdata:/var/lib/mysql"
    environment:
      MYSQL_DATABASE: corteza
      MYSQL_USER:     corteza
      MYSQL_PASSWORD: corteza
      MYSQL_RANDOM_ROOT_PASSWORD: random

    healthcheck: { test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"], timeout: 20s, retries: 10 }

volumes:
  dbdata:

```

Start Corteaz by running `docker-compose up -d' in the project directory inside `/backend`.
You can check if everything is ok by running `docker-compose ps` in the same directory.

```
▶ docker-compose ps
      Name                    Command                  State                Ports         
------------------------------------------------------------------------------------------
backend_db_1       /docker-entrypoint.sh mysqld     Up (healthy)   3306/tcp, 33060/tcp    
backend_server_1   ./bin/corteza-server serve-api   Up (healthy)   127.0.0.1:18080->80/tcp
```

# Expose Corteza to the internet

If you don't have your server, you'll need to make somehow Corteza accessible from the internet.
I'll be using [ngrok](https://ngrok.com/), but you are free to use whatever you wish.

If you're using ngrok, run  `ngrok http 18080`.
Navigate to the tunnel URL to see if everything works as expected.

> **TIP:** You can use [Planet Crust cloud](https://www.planetcrust.com/price/cloud/) to setup a Corteza instance.
> **Disclosure** I am a core Corteza contributor, and I work with Planet Crust.

# Setup the Low Code app

After deploying our Corteza instance, we need to create a Low Code app.
We'll need to create a **namespace**, **a module**, and a **page**.

Detailed instructions on Low Code configuration are available in the [official documentation](https://docs.cortezaproject.org/corteza-docs/2021.9/integrator-guide/compose-configuration/index.html).

> **TIP:** The resulting configuration is available <a href="/money/money.zip">here</a>.
> Refer to the [official documentation](https://docs.cortezaproject.org/corteza-docs/2021.9/integrator-guide/compose-configuration/import-export.html) on how you can import/export namespaces.

To create the namespace, navigate to the Low Code application

![Screenshot](/money/app-selector-compose.png)

click on the "create a new namespace" button

![Screenshot](/money/namespaces.png)

fill in the information and click on "save and close"

![Screenshot](/money/namespace-create.png)

Next, open up the namespace

![Screenshot](/money/namespace-created.png)

navigate to the admin panel.

![Screenshot](/money/namespace-empty.png)

In the admin panel, navigate to "modules", and click on the "new module" button

![Screenshot](/money/module-list.png)

define the module and click on "save".

![Screenshot](/money/module-create.png)

Click on the "create record page" so that we'll be able to interact with the module (create and edit transactions).

![Screenshot](/money/module-create-rp.png)

We'll use the default record page configuration for now, so click on the "save and close" button (we can get back and change it later on).

![Screenshot](/money/transaction-record-page.png)

Lastly, we need to define our home page to access our logged transactions.
In the side navigation, click on "pages", fill out the page title and click on the "create page" button.

![Screenshot](/money/page-list.png)

Open the page builder for the home page and add a simple record list to show the list of logged transactions.
Once finished, click on the "save and close" button.

![Screenshot](/money/homepage.png)

Navigate to the home page and add a few transactions to have something to work with.

![Screenshot](/money/transaction-records.png)

# Setup an auth client

Corteza uses OAuth2 for authentication, which requires us to define an auth client first and then initialize the OAuth2 flow from inside the mobile application.

Refer to the [official documentation](https://docs.cortezaproject.org/corteza-docs/2021.9/administrator-guide/authentication/index.html) for notes on auth clients.

To create an auth client, navigate to the admin area application

![Screenshot](/money/app-selector-admin.png)

navigate to the auth clients sub-page and click on "new".

![Screenshot](/money/auth-client-list.png)

Fill in the information and click on the "submit" button.

You can leave most things on the default value, but make sure to fill in the redirect URI, select the "grant type = authorization_code", and tick all of those checkboxes.

The redirect URI value doesn't matter; use my value (`com.tjerman.money`) and adjust it to fit your mobile application.

![Screenshot](/money/auth-client-create.png)


# Flutter boilerplate

## Create the app

Run `flutter create --org com.tjerman money` to create your flutter app.
Make sure to adjust the org to match the value you used for the redirect URI.

## File structure

This is the file structure I decided on using for this project:

* `/backend` contains my Corteza configuration files discussed in the **Setup Corteza** section,
* `/models` contains all of the class definitions for things we'll use through the application,
* `/screens` contains all of the screens defined in the application,
* `/utilities` contains all of the generic utility code such as an API client and a state manager,
* `/widgets` contains all of the widget definitions our screens will be using.

## Packages

These are the packages I have defined; they are defined in `/pubspec.yaml`:

"`yaml
dependencies:
  flutter:
    sdk: flutter

  http: ^0.12.1 # So I can make HTTP stuff for accessing Corteza API
  flutter_appauth: ^0.9.1 # To implement authentication
  cupertino_icons: ^1.0.2
  flutter_spinkit: ^5.1.0 # To have spinners that indicate loading
  flutter_dotenv: ^5.0.2 # To access the vars defined in the .env file
```

## Assets

For now, the only asset we'll need is the `.env` file, so that gets defined in the `/pubspec.yaml` file:

"`yaml
flutter:
  # ...
  assets:
    - .env
  # ...
```

## Android-specific configuration

We need to set `appAuthRedirectScheme` for the OAuth2 flow.
This goes in the `/android/app/build.gradle` file.

"`gradle
android {
    // ...
    defaultConfig {
        // ...
        // We need to change the minSDK version for the auth stuff
        minSdkVersion 18
        // ...
        manifestPlaceholders = [
            // Don't forget to change it to the value you've used elsewhere
            'appAuthRedirectScheme': 'com.tjerman.money'
        ]
        // ...
    }
    // ...
}
```

# Mobile app

This is the plan:

1. We need to have a login page that will do the OAuth2 flow with Corteza.
2. We need to save the credentials and make them available to the rest of the application.
3. We need to define a TL;DR screen to show some basic bits, such as a list of transactions and a nice welcome message.

The entire source code is available in the [GitHub repository](https://github.com/tjerman/money), so we won't go too much into details.
Here is the gist of it.

## Login screen

To implement the login, we need to do a few things:
1. Prepare a new screen `/screens/LoginScreen.dart`
2. define a new `/login` route inside `main.dart`
3. define a little home-brew state class inside `utilities/State.dart`
4. implement the OAuth2 flow inside the login screen (`/screens/LoginScreen.dart`)

We will use our little home-brew state to keep our tokens available for use.
In an actual application, you would use a state manager or the local file system to persist them through restarts.

Our state class looks like this:

```dart
// This is all we'll need for now; we can extend it later down the line
class AppState {
  String accessToken = '';
  String refreshToken = '';
  String userID = '';
}

// Prepare a global singleton just so that we can access it from where ever we wish
var state = AppState();

```

Then to implement the OAuth2 flow, inside `/screens/LoginScreen.dart`, all we need to do is initialize `FlutterAppAuth` and call the `authorizeAndExchangeCode` method on it

Login related configs are defined in the `.env` file; these are the options:
* `DOMAIN` is the value you've defined for your Flutter app (for example, `com.tjerman.money`).
* `AUTH_CLIENT_ID` is the auth client ID obtained from the admin panel.
  The auth client ID is the long number in the URL (for example, `266991625534701571`).
* `AUTH_CLIENT_SECRET` is the secret value obtained in the auth client in the admin panel.
* `AUTH_REDIRECT_URL` the value tells where Corteza should redirect after the auth is completed.
  The value has to match the one defined in the auth client.
* `API_DOMAIN` the value defines where Corteza API is located.

## TL;DR screen

To implement the TL;DR screen, we need to do a few things:
1. Prepare a new screen `/screens/TLDRScreen.dart`
2. define a new `/tldr` route inside `main.dart`
3. define a Corteza SDK inside `/utilities/CortezaSDK.dart`
4. prepare a widget to show basic user info
5. prepare a widget to show a list of transactions

### Corteza SDK

We will prepare a little SDK to handle API requests and prepare typed responses to make our life easier.

Since we only need a few methods, I decided to write them down by hand.
If I'd need something larger, I would consider [generating the code the same way we do for Node.js](https://github.com/cortezaproject/corteza-js/blob/2022.3.x/tools/codegen/corteza-api-client.js).

If we glance over the SDK:

```dart
CortezaSDK csdk = CortezaSDK(dotenv.env['API_DOMAIN']!);

class CortezaSDK {
  // Hold base Corteza API url so we don't need to provide it every time.
  // The value won't change so we don't need to have it dynamic.
  String _baseURL;
  CortezaSDK(this._baseURL);

  // A method to construct the URL to fetch the user
  Uri userURL(String userID) {
    return Uri.parse("https://$_baseURL/api/system/users/$userID");
  }

  // A method to fetch the user and return a structured response
  Future<User> fetchUser() async {
    var raw = await http.get(
      userURL(
        state.userID,
      ),
      headers: baseHeaders(),
    );

    var response = parseResponse(raw);

    return User(
      response['email']!, 
      response['username']!, 
      response['handle']!, 
      response['name']!,
    );
  }

  // ...
}
```

You can apply the same pattern for the other operations your SDK should do.
I then make a global singleton (`csdk`) to access these methods from anywhere easily.

If you're making an alternative application to the native Corteza web application, the SDK should implement the primitive operations, such as listing records, creating users, and rendering templates.

If you're making an application that uses Corteza as a back-end you can implement the operations your application needs (what we've done here).

In regards to the `.env` we need the following variables:
* `API_DOMAIN` the value defines where Corteza API is located.
* `COMPOSE_NAMESPACE_ID` the value defines the namespace ID of our Low Code app
* `COMPOSE_MODULE_TRANSACTION_ID` the value defines the module ID of where we store the transactions.

> **TIP:** Defining things like moduleID and namespaceID inside the `.env` might be a little lame for larger projects.
> We usually define a helper to construct an index of available modules, which we can then reference by the handle.
> Depending on the application, the latter approach could be better.

### TL;DR screen

The screen is defined in the `screens/TLDRScreen.dart` file, which defines the base layout, invokes the SDK to fetch the data, and shows the data in the widgets.

We have two widgets:
* `/widgets/Profile.dart` to show the user info, such as the profile picture and a welcome message, and
* `/widgets/Transactions.dart` to show the list of registered transactions.

The TL;DR screen uses `FutureBuilders` to allow async data fetch to show the UI before the data is actually obtained.

# The result

This is what the resulting mobile application looks like.

When the application starts, we are prompted to log in through Corteza.
You enter your login credentials and tap on login.

<img src="/money/mobile-login.jpg" style="max-height: 500px" />

After the login is successful, you are redirected to the TL;DR screen, where we can see our basic user info as well as the transactions we created earlier

<img src="/money/mobile-tldr.jpg" style="max-height: 500px" />

# TODO

This was just a weekend afternoon project to get a taste of Flutter and mobile development.

Here are a few improvements I can see:

* **Error handling**: I didn't do much error handling, so some errors would crash the whole application.
  Errors should be handled appropriately, logged, and displayed to the user.
* **Live data**: Currently, data won't get automatically refreshed if it is changed in Corteza.
  Corteza currently doesn't provide WebSockets to indicate changes, so we could do some pooling until we implement it.
* **Send notifications**: We could add email or push notifications to notify users about special events.
* **Option to log transactions**: We could add an input inside the mobile application to add new transactions.
