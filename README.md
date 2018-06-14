# SignalFx C# Lambda Wrapper

SignalFx C# Lambda Wrapper.

## Usage

The SignalFx C# Lambda Wrapper is a wrapper around an Lambda Function, used to instrument execution of the function and send metrics to SignalFx.

### Install via NuGet
Add the following package reference to your `.csproj` or `function.proj`
```xml
  <PackageReference Include="signalfx-lambda-functions" Version="1.0.0"/>
```


### Using the Metric Wrapper

Create a MetricWrapper with the ExecutionContext
Wrap your code in try-catch-finally, disposing of the wrapper finally.
```cs
using signalfxlambdawrapper

...

    [FunctionName("HttpTrigger")]
		public static IActionResult Run([HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]HttpRequest req, TraceWriter log, ExecutionContext context)
        {
            log.Info("C# HTTP trigger function processed a request.");
            MetricWrapper wrapper = new MetricWrapper(context);
            try { 
                ...
                // your code
                ...
                return ResponseObject
            } catch (Exception e) {
              wrapper.Error();
            } finally {
              wrapper.Dispose();
            }

        }
```

### Environment Variable
Set the Lambda Function environment variables as follows:

1) Set authentication token:
```
 SIGNALFX_AUTH_TOKEN=signalfx token
```
2) Optional parameters available:
```
 SIGNALFX_API_HOSTNAME=[pops.signalfx.com]
 SIGNALFX_API_PORT=[443]
 SIGNALFX_API_SCHEME=[https]
 SIGNALFX_SEND_TIMEOUT=milliseconds for signalfx client timeout [2000]
```

### Metrics and dimensions sent by the wrapper
The Lambda wrapper sends the following metrics to SignalFx:

| Metric Name  | Type | Description |
| ------------- | ------------- | ---|
| function.invocations  | Counter  | Count number of Lambda invocations|
| function.cold_starts  | Counter  | Count number of cold starts|
| function.errors  | Counter  | Count number of errors from underlying Lambda handler|
| function.duration  | Gauge  | Milliseconds in execution time of underlying Lambda handler|

The Lambda wrapper adds the following dimensions to all data points sent to SignalFx:

| Dimension | Description |
| ------------- | ---|
| lambda_arn  | ARN of the Lambda function instance |
| aws_region  | AWS Region  |
| aws_account_id | AWS Account ID  |
| aws_function_name  | AWS Function Name |
| aws_function_version  | AWS Function Version |
| aws_function_qualifier  | AWS Function Version Qualifier (version or version alias if it is not an event source mapping Lambda invocation) |
| event_source_mappings  | AWS Function Name (if it is an event source mapping Lambda invocation) |
| aws_execution_env  | AWS execution environment (e.g. AWS_Lambda_java8) |
| function_wrapper_version  | SignalFx function wrapper qualifier (e.g. signalfx-lambda-0.0.5) |
| metric_source | The literal value of 'lambda_wrapper' |

### Sending a custom metric from the Lambda Function
```cs
using com.signalfuse.metrics.protobuf;

// construct a data point
DataPoint dp = new DataPoint();

// use Datum to set the value
Datum datum = new Datum();
datum.intValue = 1;

// set the name, value, and metric type on the datapoint

dp.metric = "metric_name";
dp.metricType = MetricType.GAUGE;
dp.value = datum;

// add custom dimension
Dimension dim = new Dimension();
dim.key = "applicationName";
dim.value = "CoolApp";
dp.dimensions.Add(dim);

// send the metric
MetricSender.sendMetric(dp);
```

### Testing locally.
1) Follow the Lambda instructions to run functions locally https://aws.amazon.com/about-aws/whats-new/2017/08/introducing-aws-sam-local-a-cli-tool-to-test-aws-lambda-functions-locally/

2) Install the NuGet package above, run as normal.


## License

Apache Software License v2. Copyright © 2014-2017 SignalFx