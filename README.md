# Keycloak with Kong - Local development environment

This repository is intended to facilitate the creation of a dockerized development environment of Keycloak and Kong using Docker Swarm. It contains a custom kong image with the kong-oidc plugin already installed and a docker-compose file with all the necessary services. DO NOT use in production.

## Configuration steps

### 1. Build custom Kong image

```
docker build -t kong:1.4.0-alpine-oidc kong/
```

### 2. Deploy stack

```
docker stack deploy -c docker-compose.yml kk-stk
```

Give the stack a few minutes to initialize. You can check the status of each service running the command `docker service ls `.
NOTE: The service `kong-migration ` will run once to initialize the database and will NOT be restarted after that.

### 3. Configure Kong with Konga
#### 3.1 Connect Kong with Konga
1. Go to http://localhost:1337 and create your user account in Konga.
  ![Konga user creation](/images/konga-user-creation.png)
2. Login with your newly created user account.
3. Add your Kong service to Konga using the Admin API port number.
  * **Name:** Local
  * **Kong Admin URL:** http://kong:8001
  ![Konga add Kong admin](/images/konga-kong-connection.png)
#### 3.2 Add a mocking endpoint(Optional)
1. In Konga go to `Services` and click the button `+ ADD NEW SERVICE`.
2. Fill in the `Url` and `Path` fields with the following information and then click `SUBMIT SERVICE`.
   * **Name:** mocking(Optional)
   * **Url:** http://mockbin.org
   * **Path:** /request
   ![Konga add new service](/images/konga-kong-new-service.png)
3. Click your newly created service, go to `Routes` then `+ ADD ROUTE`.
4. Add `/mock` in the `Paths` field, hit the `enter` key(on your keyboard) and click `SUBMIT ROUTE`.
   ![Konga add new route](/images/konga-kong-new-route.png)
5. You can access your newly created endpoint by typing http://localhost:8000/mock on your browser.

### 4. Setup Kong to work with Keycloak
On your web browser go to http://localhost:8180, click on `Administration Console` and login with the credentials specified in the `docker-compose.yml` file.

#### 4.1 Add a client for Kong
1. Go to `Clients` on the left menu and click the button `Create` on the top right corner.
2. Fill in the `Client ID` field with `local-kong` and click  `Save`.
   ![Keycloak add client](/images/keycloak-add-client.png)
3. On the `Settings` tab fill in the following fields then click `Save`:
   * **Access Type:** confidential
   * **Root URL:** http://localhost:8000
   * **Valid Redirect URLs:** /mock*
  ![Keycloak add client2](/images/keycloak-add-client2.png)
4. Go to the `Credentials` tab and copy the `secret` since we are going to need it soon.

#### 4.2 Create an User in Keycloak
1. Go to `Users` on the left menu and click the button `Add user` on the top right corner.
2. On the `Add user` screen set a username and make sure you turn on the `Email Verified` option then click `Save`.
   ![Keycloak add user](/images/keycloak-new-user.png)
3. Go to the `Credentials` tab and set a password for your newly created user. Make sure the `Temporary` option is turned off.
   ![Keycloak user set pass](/images/keycloak-user-set-pass.png)

#### 4.3 Connect Keycloak and Kong
1. Go to `Services` on the left menu and click the `mocking` service you created on step 3.2.2.
2. Go to the `Routes` tab and select the route you created on step 3.2.4.
3. Go to the `Plugins` tab and click the `+ ADD PLUGIN` button.
4. In the `Others` tab click the `ADD PLUGIN` button under the `Oidc` option.
5. Fill in the following fields then click the `ADD PLUGIN` button:
   * **realm:** master
   * **client id:** local-kong
   * **discovey:** http://HOST_IP:8180/auth/realms/master/.well-known/openid-configuration (Replace HOST_IP with your local IP address)
   * **secret:** Paste the `secret` you copied on step 4.1.4.
  ![Keycloak add plugin](/images/keycloak-add-plugin.png)
That's it! Now when you try to access your mocking endpoint (http://localhost:8000/mock) Kong should redirect you to authenticate with Keycloak.