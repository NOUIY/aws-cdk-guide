# Tokens and the AWS CDK<a name="tokens"></a>

*Tokens* represent values that can only be resolved at a later time in the [app lifecycle](deploy.md#deploy-how-synth-app)\. For example, the name of an Amazon Simple Storage Service \(Amazon S3\) bucket that you define in your CDK app is only allocated when the AWS CloudFormation template is synthesized\. If you print the `bucket.bucketName` attribute, which is a string, you will see that it contains something like the following:

```
${TOKEN[Bucket.Name.1234]}
```

This is how the AWS CDK encodes a token whose value is not yet known at construction time, but will become available later\. The AWS CDK calls these placeholders *tokens*\. In this case, it's a token encoded as a string\.

You can pass this string around as if it was the name of the bucket\. In the following example, the bucket name is specified as an environment variable to an AWS Lambda function\.

------
#### [ TypeScript ]

```
const bucket = new s3.Bucket(this, 'amzn-s3-demo-bucket');

const fn = new lambda.Function(stack, 'MyLambda', {
  // ...
  environment: {
    BUCKET_NAME: bucket.bucketName,
  }
});
```

------
#### [ JavaScript ]

```
const bucket = new s3.Bucket(this, 'amzn-s3-demo-bucket');

const fn = new lambda.Function(stack, 'MyLambda', {
  // ...
  environment: {
    BUCKET_NAME: bucket.bucketName
  }
});
```

------
#### [ Python ]

```
bucket = s3.Bucket(self, "amzn-s3-demo-bucket")

fn = lambda_.Function(stack, "MyLambda",
        environment=dict(BUCKET_NAME=bucket.bucket_name))
```

------
#### [ Java ]

```
final Bucket bucket = new Bucket(this, "amzn-s3-demo-bucket");

Function fn = Function.Builder.create(this, "MyLambda")
        .environment(java.util.Map.of(    // Map.of requires Java 9+
            "BUCKET_NAME", bucket.getBucketName()))
        .build();
```

------
#### [ C\# ]

```
var bucket = new s3.Bucket(this, "amzn-s3-demo-bucket");
        
var fn = new Function(this, "MyLambda", new FunctionProps {
    Environment = new Dictionary<string, string>
    {
        ["BUCKET_NAME"] = bucket.BucketName
    }
});
```

------

When the AWS CloudFormation template is finally synthesized, the token is rendered as the AWS CloudFormation intrinsic `{ "Ref": "amzn-s3-demo-bucket" }`\. At deployment time, AWS CloudFormation replaces this intrinsic with the actual name of the bucket that was created\.

**Topics**
+ [Tokens and token encodings](#tokens_encoding)
+ [String\-encoded tokens](#tokens_string)
+ [List\-encoded tokens](#tokens_list)
+ [Number\-encoded tokens](#tokens_number)
+ [Lazy values](#tokens_lazy)
+ [Converting to JSON](#tokens_json)

## Tokens and token encodings<a name="tokens_encoding"></a>

Tokens are objects that implement the [IResolvable](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.IResolvable.html) interface, which contains a single `resolve` method\. The AWS CDK calls this method during synthesis to produce the final value for the AWS CloudFormation template\. Tokens participate in the synthesis process to produce arbitrary values of any type\.

**Note**  
You'll rarely work directly with the `IResolvable` interface\. You will most likely only see string\-encoded versions of tokens\.

Other functions typically only accept arguments of basic types, such as `string` or `number`\. To use tokens in these cases, you can encode them into one of three types by using static methods on the [cdk\.Token](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html) class\.
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrstringvalue-options](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrstringvalue-options) to generate a string encoding \(or call `.toString()` on the token object\)
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrlistvalue-options](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrlistvalue-options) to generate a list encoding
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrnumbervalue](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrnumbervalue) to generate a numeric encoding

These take an arbitrary value, which can be an `IResolvable`, and encode them into a primitive value of the indicated type\.

**Important**  
Because any one of the previous types can potentially be an encoded token, be careful when you parse or try to read their contents\. For example, if you attempt to parse a string to extract a value from it, and the string is an encoded token, your parsing fails\. Similarly, if you try to query the length of an array or perform math operations with a number, you must first verify that they aren't encoded tokens\.

To check whether a value has an unresolved token in it, call the `Token.isUnresolved` \(Python: `is_unresolved`\) method\.

The following example validates that a string value, which could be a token, is no more than 10 characters long\. 

------
#### [ TypeScript ]

```
if (!Token.isUnresolved(name) && name.length > 10) {
  throw new Error(`Maximum length for name is 10 characters`);
}
```

------
#### [ JavaScript ]

```
if ( !Token.isUnresolved(name) && name.length > 10) {
  throw ( new Error(`Maximum length for name is 10 characters`));
}
```

------
#### [ Python ]

```
if not Token.is_unresolved(name) and len(name) > 10:
    raise ValueError("Maximum length for name is 10 characters")
```

------
#### [ Java ]

```
if (!Token.isUnresolved(name) && name.length() > 10)
    throw new IllegalArgumentException("Maximum length for name is 10 characters");
```

------
#### [ C\# ]

```
if (!Token.IsUnresolved(name) && name.Length > 10)
    throw new ArgumentException("Maximum length for name is 10 characters");
```

------

If **name** is a token, validation isn't performed, and an error could still occur in a later stage in the lifecycle, such as during deployment\.

**Note**  
You can use token encodings to escape the type system\. For example, you could string\-encode a token that produces a number value at synthesis time\. If you use these functions, it's your responsibility to make sure that your template resolves to a usable state after synthesis\.

## String\-encoded tokens<a name="tokens_string"></a>

String\-encoded tokens look like the following\.

```
${TOKEN[Bucket.Name.1234]}
```

They can be passed around like regular strings, and can be concatenated, as shown in the following example\.

------
#### [ TypeScript ]

```
const functionName = bucket.bucketName + 'Function';
```

------
#### [ JavaScript ]

```
const functionName = bucket.bucketName + 'Function';
```

------
#### [ Python ]

```
function_name = bucket.bucket_name + "Function"
```

------
#### [ Java ]

```
String functionName = bucket.getBucketName().concat("Function");
```

------
#### [ C\# ]

```
string functionName = bucket.BucketName + "Function";
```

------

You can also use string interpolation, if your language supports it, as shown in the following example\.

------
#### [ TypeScript ]

```
const functionName = `${bucket.bucketName}Function`;
```

------
#### [ JavaScript ]

```
const functionName = `${bucket.bucketName}Function`;
```

------
#### [ Python ]

```
function_name = f"{bucket.bucket_name}Function"
```

------
#### [ Java ]

```
String functionName = String.format("%sFunction". bucket.getBucketName());
```

------
#### [ C\# ]

```
string functionName = $"${bucket.bucketName}Function";
```

------

Avoid manipulating the string in other ways\. For example, taking a substring of a string is likely to break the string token\.

## List\-encoded tokens<a name="tokens_list"></a>

List\-encoded tokens look like the following:

```
["#{TOKEN[Stack.NotificationArns.1234]}"]
```

The only safe thing to do with these lists is pass them directly to other constructs\. Tokens in string list form cannot be concatenated, nor can an element be taken from the token\. The only safe way to manipulate them is by using AWS CloudFormation intrinsic functions like [Fn\.select](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-select.html)\.

## Number\-encoded tokens<a name="tokens_number"></a>

Number\-encoded tokens are a set of tiny negative floating\-point numbers that look like the following\.

```
-1.8881545897087626e+289
```

As with list tokens, you cannot modify the number value, as doing so is likely to break the number token\. The only allowed operation is to pass the value around to another construct\. 

## Lazy values<a name="tokens_lazy"></a>

In addition to representing deploy\-time values, such as AWS CloudFormation [parameters](parameters.md), tokens are also commonly used to represent synthesis\-time lazy values\. These are values for which the final value will be determined before synthesis has completed, but not at the point where the value is constructed\. Use tokens to pass a literal string or number value to another construct, while the actual value at synthesis time might depend on some calculation that has yet to occur\.

You can construct tokens representing synth\-time lazy values using static methods on the `Lazy` class, such as [Lazy\.string](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Lazy.html#static-stringproducer-options) and [Lazy\.number](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Lazy.html#static-numberproducer)\. These methods accept an object whose `produce` property is a function that accepts a context argument and returns the final value when called\.

The following example creates an Auto Scaling group whose capacity is determined after its creation\.

------
#### [ TypeScript ]

```
let actualValue: number;

new AutoScalingGroup(this, 'Group', {
  desiredCapacity: Lazy.numberValue({
    produce(context) {
      return actualValue;
    }
  })
});

// At some later point
actualValue = 10;
```

------
#### [ JavaScript ]

```
let actualValue;

new AutoScalingGroup(this, 'Group', {
  desiredCapacity: Lazy.numberValue({
    produce(context) {
      return (actualValue);
    }
  })
});

// At some later point
actualValue = 10;
```

------
#### [ Python ]

```
class Producer:
    def __init__(self, func):
        self.produce = func

actual_value = None
          
AutoScalingGroup(self, "Group", 
    desired_capacity=Lazy.number_value(Producer(lambda context: actual_value))
)
    
# At some later point
actual_value = 10
```

------
#### [ Java ]

```
double actualValue = 0;

class ProduceActualValue implements INumberProducer {

    @Override
    public Number produce(IResolveContext context) {
        return actualValue;
    }
}

AutoScalingGroup.Builder.create(this, "Group")
    .desiredCapacity(Lazy.numberValue(new ProduceActualValue())).build();

// At some later point
actualValue = 10;
```

------
#### [ C\# ]

```
public class NumberProducer : INumberProducer
{
    Func<Double> function;
        
    public NumberProducer(Func<Double> function)
    {
        this.function = function;
    }

    public Double Produce(IResolveContext context)
    {
        return function();
    }
}

double actualValue = 0;

new AutoScalingGroup(this, "Group", new AutoScalingGroupProps
{
    DesiredCapacity = Lazy.NumberValue(new NumberProducer(() => actualValue))
});

// At some later point
actualValue = 10;
```

------

## Converting to JSON<a name="tokens_json"></a>

Sometimes you want to produce a JSON string of arbitrary data, and you may not know whether the data contains tokens\. To properly JSON\-encode any data structure, regardless of whether it contains tokens, use the method [stack\.toJsonString](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html#towbrjsonwbrstringobj-space), as shown in the following example\.

------
#### [ TypeScript ]

```
const stack = Stack.of(this);
const str = stack.toJsonString({
  value: bucket.bucketName
});
```

------
#### [ JavaScript ]

```
const stack = Stack.of(this);
const str = stack.toJsonString({
  value: bucket.bucketName
});
```

------
#### [ Python ]

```
stack = Stack.of(self)
string = stack.to_json_string(dict(value=bucket.bucket_name))
```

------
#### [ Java ]

```
Stack stack = Stack.of(this);
String stringVal = stack.toJsonString(java.util.Map.of(    // Map.of requires Java 9+
        put("value", bucket.getBucketName())));
```

------
#### [ C\# ]

```
var stack = Stack.Of(this);
var stringVal = stack.ToJsonString(new Dictionary<string, string>
{
    ["value"] = bucket.BucketName
});
```

------