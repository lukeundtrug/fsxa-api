# FSXA-API
The FSXA-API is an interface handling the data coming from the FirstSpirit 
[CaaS](https://docs.e-spirit.com/module/caas/CaaS_Product_Documentation_EN.html) and the 
Navigation Service. The data is processed and transformed to perfectly suit the needs of 
the [FSXA-Pattern-Library](https://github.com/e-Spirit/fsxa-pattern-library).

- [1. About the FSXA](#about-the-fsxa)
- [2. Legal Notices](#legal-notices)
- [3. Methods](#methods)
  - [Constructor](#constructor)
  - [setConfiguration](#setconfiguration)
  - [config](#config)
  - [buildAuthorizationHeaders](#buildauthorizationheaders)
  - [buildCaaSURL](#buildcaasurl)
  - [buildNavigationServiceUrl](#buildnavigationserviceurl)
  - [fetchElement](#fetchelement)
  - [fetchByFilter](#fetchbyfilter)
  - [fetchProjectProperties](#fetchprojectproperties)
- [4. Filter](#filter)
  - [LogicalFilter](#logical-query-operators)
  - [ComparisonFilter](#comparison-query-operators)
  - [ArrayFilter](#array-query-operators)
- [5. Disclaimer](#disclaimer)

### About the FSXA
The FirstSpirit Experience Accelerator (FSXA) is the hybrid solution of a digital 
experience platform, combining a headless approach with enterprise capabilities. 
If you are interested in the FSXA check this 
[Overview](https://docs.e-spirit.com/module/fsxa/overview/benefits-hybrid/index.html). You can order 
a demo [online](https://www.e-spirit.com/us/specialpages/forms/on-demand-demo/).

## Legal Notices
FSXA-API is a product of [e-Spirit AG](http://www.e-spirit.com), Dortmund, Germany.
The FSXA-API is subject to the Apache-2.0 license.

## Methods

In this section all available methods are explained and provided with an example.

### Constructor

To be able to use the FSXA-API, a new object must be created.
How you create the object depends on how you want to use the FSXA-API.

If you want to use the information provided by the CaaS in your frontend, you have to use the `proxy` mode.  
If you want to use it in your server, you have to use the `remote` mode.

However, to have a fully running application, it makes most sense to use the FSXA-API in your server and in your frontend.

In all cases you have to specify the content mode, the config data and optionally the log level.

The config mode can be `preview` or `release`. It depends on which information you want to get.
<br />
There is an enum to use these modes.
<br />
`FSXAContentMode.PREVIEW` for `preview`
<br />
`FSXAContentMode.RELEASE` for `release`


The config data depends on which in which mode you want to run the FSXA-API.

If you want to use the `remote` mode, you have to specify all authorization keys:
````typescript
{
    mode: "remote",
    config: {
            apiKey: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            navigationService: "https://your.navigation-service.url/navigation",
            caas: "https://your.caas.url",
            projectId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            tenantId: "your-tenant-id"
    }
}
````

If you want to use the `proxy` mode, you have to specify the baseURL:
```typescript
{
    mode: 'proxy',
    baseUrl: 'http://localhost:3001/api'
}
```

The log level can be:
`0` = Info
`1` = Log
`2` = Warning
`3` = Error
`4` = None. The default is set to `3`.

Here is an example how the FSXA-API could be used with an [Express.js](https://expressjs.com/) backend.
Make sure you have `cross-fetch`, `express`, `cors`, `lodash` and of course `fsxa-api` installed.
```typescript
require("cross-fetch/polyfill")
const express = require('express')
const {FSXAApi, FSXAContentMode, } = require("fsxa-api");
const expressIntegration = require("fsxa-api/dist/lib/integrations/express").default
const cors = require("cors")

const app = express()

const remoteApi = new FSXAApi(
    FSXAContentMode.PREVIEW,
    {
        mode: "remote",
        config: {
            apiKey: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            navigationService: "https://your.navigation-service.url/navigation",
            caas: "https://your.caas.url",
            projectId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            tenantId: "your-tenant-id"
        }
    },
    3
)

app.use(cors())
app.use("/api", expressIntegration({ api: remoteApi }))

app.listen(3001, () => {
  console.log('Listening at http://localhost:3001')
})
```

Here is an example of the corresponding usage in a frontend application:

```typescript
const fsxaApi = new FSXAApi(
    FSXAContentMode.PREVIEW,
    {
        mode: 'proxy',
        baseUrl: 'http://localhost:3001/api'
    },
    1
)
```

### setConfiguration

The configuration of the FSXA-API can be set via a method.

The config mode can be `preview` or `release`. It depends on which information you want to get.
<br />
There is an enum to use these modes.
<br />
`FSXAContentMode.PREVIEW` for `preview`
<br />
`FSXAContentMode.RELEASE` for `release`

The config data depends on which in which mode you want to run the FSXA-API.

If you want to use the `remote` mode, you have to specify all authorization keys:
````typescript
{
    mode: "remote",
    config: {
            apiKey: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            navigationService: "https://your.navigation-service.url/navigation",
            caas: "https://your.caas.url",
            projectId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            tenantId: "your-tenant-id"
    }
}
````

If you want to use the `proxy` mode, you have to specify the baseURL:
```typescript
{
    mode: 'proxy',
    baseUrl: 'http://localhost:3001/api'
}
```

Example:
```typescript
fsxaApi.setConfiguration({
  FSXAContentMode.RELEASE,
  {
    mode: "remote",
        config: {
            apiKey: "xxxx-xxxx-xxxx-xxxx",
            navigationService: "https://your.navigation.service",
            caas: "https://your.caas.service",
            projectId: "xxx-xxxx-xxx",
            tenantId: "your-tentant-id"
        }
  }
})
```

### config

Returns the current configuration when the mode is set to `remote`.
<br />
If the mode is set to `proxy`, this method returns `null`.

Example:
```typescript
fsxaApi.config()
```

### buildAuthorizationHeaders

Returns the build authorization header in the following format when mode is set to `remote`: 
```typescript
{ authorization: 'apikey="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"' }
```

Returns an empty object when mode is set to `proxy`.

Example:
```typescript
fsxaApi.buildAuthorizationHeaders()
```
 
### buildCaaSURL 

Returns the build CaaS url when mode is set to `remote`:

Returns an empty string when mode is set to `proxy`.

Example:
```typescript
fsxaApi.buildCaaSUrl()
```


### buildNavigationServiceUrl

Returns the build navigation-service url when mode is set to `remote`:

Returns an empty string when mode is set to `proxy`.

Example:
```typescript
fsxaApi.buildNavigationServiceUrl()
```

### fetchElement

Returns the corresponding CaaS data entry. 

Expects as input parameter an id, which is described in CaaS as 'identifier' and a language abbreviation.
<br />
Optionally additional parameters can be passed that will be appended to the CaaS-Url. Be aware that the response is not mapped, if you pass the keys-parameter. 

Example:
```typescript
fsxaApi.fetchElement(
    "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "en-EN"
)
```

### fetchByFilter
Returns the matching CaaS data entries.

Expects as input parameter an array of filters and a language abbreviation.
Optionally a page number, page size and additional parameters can be passed.

Several filter objects can be specified in the filter array, the results will then correspond to all specified filters.

One filter object must have a:
<br />
`field` which specifies the searched key,
<br />
`operator` which specifies the search operation,
<br />
`value` which specifies the value that is looked for. 

[More information to the filters](#filter)

In this example we search for all elements with the `fsType` equals `Example`. We want the `2nd` page with a maximum of `50` entries. However, we do not want the `identifier` to appear:
```typescript
fsxaApi.fetchByFilter(
    [
      {
        field: "fsType",
        operator: ComparisonQueryOperatorEnum.EQUALS,
        value: "Example"
      }
    ],
    "en",
    2,
    50,
    {"keys": JSON.stringify({"identifier": 0})}
)
```


### fetchProjectProperties

Returns the project properties of the given language.

Expects as input parameter the language abbreviation.

ATTENTION: Works only with CaaSConnect module version 3 onwards.

Example:
```typescript
fsxaApi.fetchProjectProperties("en_EN")
```

## Filter

To customize your queries in the [fetchByFilter](#fetchbyfilter) method with these operations. For more info you can click on the respective links.

### Logical Query Operators
[MongoDB Documentation](https://docs.mongodb.com/manual/reference/operator/query-logical/)

| Enum | Operation |
| --- | --- |
|LogicalQueryOperatorEnum.AND | $and |
|LogicalQueryOperatorEnum.NOT | $not |
|LogicalQueryOperatorEnum.NOR | $nor |
|LogicalQueryOperatorEnum.OR | $or |

### Comparison Query Operators
[MongoDB Documentation](https://docs.mongodb.com/manual/reference/operator/query-comparison/)

| Enum | Operation |
| --- | --- |
|LogicalQueryOperatorEnum.GREATER_THAN_EQUALS | $gte |
|LogicalQueryOperatorEnum.GREATER_THAN | $gt | 
|LogicalQueryOperatorEnum.EQUALS | $eq | 
|LogicalQueryOperatorEnum.IN | $in | 
|LogicalQueryOperatorEnum.LESS_THAN | $lt | 
|LogicalQueryOperatorEnum.LESS_THAN_EQUALS | $lte |
|LogicalQueryOperatorEnum.NOT_EQUALS | $ne |
|LogicalQueryOperatorEnum.NOT_IN | $nin |


### Array Query Operators
[MongoDB Documentation](https://docs.mongodb.com/manual/reference/operator/query-array/)

| Enum | Operation |
| --- | --- |
|ArrayQueryOperatorEnum.ALL | $all |

## Disclaimer
This document is provided for information purposes only.
e-Spirit may change the contents hereof without notice. 
This document is not warranted to be error-free, nor subject to any 
other warranties or conditions, whether expressed orally or 
implied in law, including implied warranties and conditions of 
merchantability or fitness for a particular purpose. e-Spirit 
specifically disclaims any liability with respect to this document 
and no contractual obligations are formed either directly or 
indirectly by this document. The technologies, functionality, services, 
and processes described herein are subject to change without notice.
