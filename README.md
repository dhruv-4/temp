
## Context

This is about this epic: [ARCH-260 Next Generation Logging and Tracing](https://comtravo.atlassian.net/browse/ARCH-260) .
The overall aim to improve logging and debugging.
This is how we came up with the steps
1. Standardise logs
2. Introduce a tool to visualize logs
3. Distributed tracing

Will try to go through each steps a bit

## Standardise logs
The first step we determined will be to standard logs


## Solution

<details>
  <summary>1. Updating logging object</summary>
  
  This could be what I think would be helpful object
Response Error Object

```ts
{
  application: string,
  statusCode: number,
  message: string,
  level:  'info' | 'error' | 'warn' ,
  environment: 'prod',
  request: {
    url:  string,
    headers: {}
  },
  response: {
    statusCode: number,
    message: string,
    headers: {}
  },
  responseTime: string
}
```

Info log

```ts
{
  application: string,
  message: string,
  level:  'info',
  environment: 'prod'
}
```

Error log

```ts
{
  application: string,
  message: string,
  level:  'error',
  environment: 'prod'
}
```

Heres an example of what the change could look like, (before vs after)
<img width="1635" alt="unauthorized_error" src="https://user-images.githubusercontent.com/75316673/127314808-a404b677-204a-4556-9f90-722b358e1500.png">

We checked out replacing `bunyan` with `pinojs` for this.You can check out this https://github.com/comtravo/ct-backend/pull/11487
Using pinojs since its faster and has an ecosymtem of logging around it.So we woyuld be able to keep the logging format similar
</details>

<details>
  <summary>2. Adding environment to logs</summary>
  
  Currently we index logs by environment, this means we have different sources for different environments.This also means it makes it difficult to change sources.If we add environment to logs and index together, we could easily switch environments in logs from the logs itself
![image](https://user-images.githubusercontent.com/75316673/127350174-3a631dd1-2a8b-4eb4-8145-3ff96adf70b4.png)

</details>

<details>
  <summary>3. Source for logs</summary>
  
  For alignment and ease, we could still keep using elasticserahc as our source for different tool
</details>

<details>
  <summary>4. Getting rid of debug lib</summary>
  
  Currently we use a package called `debug`, which allows us to add a env variable `DEBUG: *` and once this is set these log are used for debugging lambdas and services.
  
  We could instead make use [pino-debug](https://github.com/pinojs/pino-debug). This would allow us make use of the same library and make use of the same library like `logger.debug()`
</details>

<details>
  <summary>5. Getting rid of <b>ct_inspector</b></summary>
  
  Currently we use `ct_inspector` to log and also track the amount of time for third party.
  This is also used to in this [flight search dashboard](https://grafana.prod.comtravo.com/d/4xPM7fGr8/flight-search-api-details?orgId=1&refresh=5m)
![image](https://user-images.githubusercontent.com/75316673/127357499-f42e2e51-f196-4834-8627-223428d40635.png)


  A idea from Puneeth was to use timeseries db for this.
</details>

<details>
  <summary>6. Redacting customer info before using a 3rd party tool</summary>
  
  Currently when swagger validation fails it logs the entire object that failed. This also includes stuff like `booking.guest_travelers` and `booking.booker` which has all info like email and phone number.
  So we should try to redact these before actually using a 3rd party tool for visualization.

  It becomes really easy with pinojs
  
  ```ts
    redact: {
      paths: [
        'req.headers.authorization',
        'request_data.body.booker',
        'request_data.body.guest_travelers'
      ],
      remove: true
    },
  ```
</details>

## Introduce a tool to visualise logs
Tools in consideration now
1. [Sentry](https://sentry.io/welcome/)
2. [Jaeger Tracing](https://www.jaegertracing.io/)

Things to consider when choosing this
- Should we keep using existing ELK stack or use S3 
- We also want a different way for alerting, sending all messages to slack is not really scalable
- Distributed tracing, although the last step would still be important to consider.This is where we would be able to see the flow of a request through the system, with some kind of correlation id
- Some dashboard like this may give you a final outcome https://grafana.infra.comtravo.com/d/BcyIAPz7k/test-dashboard-playground?orgId=1&refresh=10s
![image](https://user-images.githubusercontent.com/75316673/127340128-82f194d4-b7d2-413a-9d24-f5f178587ccf.png)


