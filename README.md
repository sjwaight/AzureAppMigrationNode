# Migrating your apps to Azure

> Note: the original source of this project can be found in the [GitHub repository](https://github.com/microsoft/IgniteTheTour/tree/master/DEV%20-%20Building%20your%20Applications%20for%20the%20Cloud/DEV10/src/product-service) for Microsoft Ignite | The Tour 2018.

## NodeJS - Product Service

This project requires Node.js 8+ as well CosmosDB or MongoDB running somewhere.

1. Create a file called `.env` in the root folder of this project.
2. Put these variables in it:
   - `DB_CONNECTION_STRING` (Required) — Cosmos DB or Mongo DB Connection URL – make sure it has the db name in it. If you're pulling from Cosmos DB, make sure to add a `/tailwind` (or whatever you called it in `DB_NAME`) after the port but before the query string.

   - `DB_NAME` (Optional) — Name of DB in CosmosDB or MongoDB. default `tailwind`
   - `COLLECTION_NAME` (Optional) — Name of collection in db. default `inventory`

   - `ITEMS_AMOUNT` (Optional) — If running the `populate:mongodb` script, how many total items to fill the DB with. default `10000`
   - `IMAGE_SIZE` (Optional) — If running the `populate:mongodb` script, what size of images to use. default: `250`
   - `PG_CONNECTION_STRING` (Optional) — If running the `populate:mongodb` script, an optional param to draw data from a Postgres database first before filling the rest of the MongoDB/CosmosDB instance with random data. optional
   - `PORT` (Optional) — What port to start the service on. default `8000` – don't add this to your .env unless you _definitely_ need it
   - `HOSTNAME` (Optional) — What hostname the server is hosted on. default: `localhost` – don't add this to your .env unless you _definitely_ need it
   - `CORS` (Optional) - Which URL would you like to restrict CORS to? By default it is set to '\*'.
   - `APPINSIGHTS_INSTRUMENTATIONKEY` (Optional) - Applications Insights key (enables Application Insights).

**OR**

1. Deploy an Azure Key Vault instance and define all of the items above as Secrets. 
2. In your `.env` file, put the following variables:

   - `KEYVAULT_URI` — URI of your Azure Key Vault

   - `KEYVAULT_ID` — ID of your Service Principal which will access Azure Key Vault
   - `KEYVAULT_SECRET` — Password of your Service Principal which will access your Azure Key Vault.
   
## To Populate ComsosDB or MongoDB with fake data

### Randomly generate large quantities of products
1. In the root folder run `npm install`
1. If you need to pull from a PostgreSQL DB, make sure you have a connection string for it in your `.env`
1. Drop the previous collection if you're re-using a collection
1. Run `npm run populate:mongodb`.

### Seed database with "official" Tailwind products
1. Set `SEED_DATA` = `true`, `DB_CONNECTION_STRING` = valid connection string in your `.env` file.
1. `npm start` to run the app.

## To Develop

1. In the root folder rnu `npm install`
1. `npm run dev`
1. Project will be running on host and port you specified, defaults to http://localhost:8000
1. Adhere to the Prettier and ESLint rules before checking in.

## To Run in Production

1. In the root folder run `npm install`
1. `npm start`
1. Project will be running on host and port you specified.

## To Deploy To Azure

### From Azure CLI

1. Create a new Resource Group with a data center location. You can substitute "westus" here for any other data center location.

   For a list of locations, run `az account list-locations`.

   ```
   az group create -n {Your Resource Group Name} -l westus
   ```

1. Create a new Service Plan. Select the S1 tier. This tier performs faster for demos. Also make sure you specify that you want a Linux server.
   ```
   az appservice plan create -n {Your Service Plan Name} --sku S1 --is-linux -g {Your Resource Group Name}
   ```
1. Create a new App Service site to hose the Product Service API. Specify a Node version to use as well. At the time of this writing, the current LTS version of Node in Azure is 8.11.

   For a complete list of supported Node versions, run `az webapp list-runtimes`.

   ```
   az webapp create -n {Your App Name} -p {Your Service Plan Name} -g {Your Resource Group Name} --runtime 'node|8.11'
   ```

1. Zip up all of the project files, excluding the `node_modules` folder. An `npm install` will be run by Azure after the zip is deployed. Make sure you are in the project directory when you run this.

   ```
   zip -r Deployment . -x "node_modules/**/*"
   ```

1. Deploy the zip file to Azure

   ```
   az webapp deployment source config-zip -g {Your Resource Group Name} -name {Your App Name} --src Deployment.zip
   ```

1. Add an application setting that points to your Products Cosmos DB database instance.

   ```
   az webapp config appsettings set -n {Your App Name} --settings "COSMOSDB_OR_MONGODB_CONNECTION_STRING={Your Cosmos DB Connection String}" -g {Your Resource Group}
   ```

1. Open the app in a browser...
   ```
   az webapp browse -n {Your App Name} -g {Your Resource Group Name}
   ```
   ...and then navigate to `api/products` to see some data.
