# terraform-testing-with-terratest

# Terraform Testing with Terratest
## Theory, Fundamentals and Simple Production-Style Hands-On Demo

---

# 1. What is Terraform Testing?

Terraform testing means automatically checking whether our Terraform code behaves as expected.

Think about normal application development.

A developer writes code:

```text
Application Code
      |
      v
      Tests
      |
      v
   Deploy
```

We can apply the same concept to Infrastructure as Code:

```text
Terraform Code
      |
      v
Terraform Tests
      |
      v
Create Infrastructure
      |
      v
Verify Infrastructure
      |
      v
Destroy Test Infrastructure
```

The goal is simple:

> Before trusting Terraform code in production, automatically verify that it creates the infrastructure we expect.

For example, if we have a Terraform module that creates an S3 bucket, we may want to verify:

```text
Bucket exists
Versioning is enabled
Encryption is enabled
Required tags exist
Public access is blocked
```

Instead of manually checking these things every time, we automate them.

---

# 2. Why Do We Need Terraform Testing?

Terraform code can be valid from a syntax perspective and still create infrastructure that is incorrect.

For example:

```hcl
resource "aws_s3_bucket" "app_bucket" {
  bucket = var.bucket_name
}
```

Terraform may successfully execute this configuration.

But perhaps our production standard says:

```text
Versioning = Required
Encryption = Required
Environment tag = Required
Public access = Blocked
```

Terraform can successfully create the bucket while our production requirements are still not satisfied.

This is where testing becomes useful.

---

# 3. Simple Real-World Example

Imagine an enterprise has a reusable S3 Terraform module.

Application teams use the same module:

```text
Application A
     |
     v
S3 Terraform Module

Application B
     |
     v
S3 Terraform Module

Application C
     |
     v
S3 Terraform Module
```

Instead of manually testing every deployment, we can write automated tests for the module.

For example:

```text
Test 1
Bucket should exist

Test 2
Versioning should be enabled

Test 3
Encryption should be enabled

Test 4
Required tags should exist
```

Whenever somebody changes the module, the tests run again.

This helps prevent accidental breaking changes.

---

# 4. Terraform Testing Landscape

There is not just one type of Terraform testing.

A good way to explain it in an interview is to divide testing into multiple levels.

## Level 1: Formatting

```bash
terraform fmt -check
```

This checks whether the Terraform code follows the expected formatting.

It is not really an infrastructure test.

It is a code-quality check.

---

# 5. Terraform Validate

Command:

```bash
terraform validate
```

This checks whether the Terraform configuration is structurally valid.

For example:

```text
Syntax problems
Invalid references
Invalid argument names
Invalid configuration structure
```

Example:

```bash
terraform init
terraform validate
```

A successful result may look like:

```text
Success! The configuration is valid.
```

Important interview point:

> `terraform validate` does not prove that the infrastructure works correctly in AWS.

It mainly validates the Terraform configuration itself.

---

# 6. Terraform Plan

Command:

```bash
terraform plan
```

This tells us what Terraform intends to change.

For example:

```text
Plan:

+ create S3 bucket
+ enable versioning
+ enable encryption
```

Plan is extremely useful, but:

> `terraform plan` by itself is not a complete infrastructure test.

It tells us what Terraform intends to do.

It does not necessarily prove that the final infrastructure behaves correctly.

---

# 7. Terraform Test

Modern Terraform also has a native testing framework.

The command is:

```bash
terraform test
```

Terraform test files normally use:

```text
.tftest.hcl
```

and are commonly placed under:

```text
tests/
```

Example:

```text
terraform-project/
├── main.tf
├── variables.tf
├── outputs.tf
└── tests/
    └── s3.tftest.hcl
```

Terraform can execute test runs using `plan` or `apply`.

With apply-based tests, Terraform can create temporary infrastructure, execute assertions, and clean up the test infrastructure afterward.

A very simple conceptual example is:

```hcl
run "create_bucket" {
  command = apply

  assert {
    condition     = aws_s3_bucket.app_bucket.id != ""
    error_message = "Bucket was not created."
  }
}
```

Terraform's native testing framework can also use mocking features, which can allow some tests to run without actually creating cloud infrastructure.

---

# 8. So Where Does Terratest Fit?

This is one of the most important interview questions.

Terratest is an open-source Go library for testing infrastructure code.

It provides helpers for testing things such as:

```text
Terraform
AWS
Azure
GCP
Kubernetes
Docker
Helm
HTTP endpoints
SSH
```

Terratest uses Go's testing framework.

Conceptually:

```text
Go Test
   |
   v
Terratest
   |
   v
Terraform
   |
   v
AWS
   |
   v
Assertions
```

Terratest can run Terraform commands such as:

```text
terraform init
terraform apply
terraform output
terraform destroy
```

and then use AWS APIs or other mechanisms to verify the infrastructure.


---

# 9. Terraform Test vs Terratest

This is another strong interview topic.

## Terraform Native Testing

Terraform provides:

```bash
terraform test
```

Advantages:

```text
Built into Terraform
Easy to start
HCL-based tests
Good for testing Terraform module logic
Supports plan-based tests
Supports apply-based tests
Supports mocking
```

## Terratest

Terratest uses:

```text
Go
```

Advantages:

```text
Real infrastructure testing
AWS API validation
HTTP validation
SSH validation
Kubernetes validation
More programmable assertions
Useful for end-to-end testing
```

A simple way to remember the difference:

```text
terraform test
      |
      v
Test Terraform configuration and behavior

Terratest
      |
      v
Use Go to test Terraform + real infrastructure behavior
```

A strong interview answer is:

> I would use Terraform native testing for testing module logic and configuration behavior, and I would use Terratest when I need more programmable integration or end-to-end testing against real infrastructure.

---

# 10. What Exactly Should We Test?

For Terraform modules, we should test important behaviors rather than every single line of Terraform code.

For example, for an S3 module:

```text
Does the bucket exist?

Is versioning enabled?

Is encryption enabled?

Are mandatory tags present?

Is public access blocked?
```

For an EC2 module:

```text
Does the instance exist?

Is the expected AMI being used?

Is the correct instance type configured?

Does it have the expected security group?

Are required tags present?
```

For an EKS module:

```text
Does the cluster exist?

Are the expected node groups present?

Are the expected Kubernetes resources available?

Can we communicate with the cluster?
```

For a networking module:

```text
Does the VPC exist?

Are the expected subnets created?

Are route tables configured correctly?

Are the expected CIDR ranges present?
```

---

# 11. A Very Important Concept: Test Behavior, Not Terraform Lines

We generally do not want tests like:

```text
Line 17 should exist.

Line 18 should contain this value.
```

Instead, test the expected behavior.

For example:

```text
Expected:

Production S3 bucket must have versioning enabled.
```

The test should verify:

```text
AWS
 |
 +-- S3 bucket
       |
       +-- Versioning = Enabled
```

This is much more useful.

---

# 12. Unit Testing vs Integration Testing

This distinction is very important in interviews.

## Unit Testing

Unit testing generally tries to test logic in isolation.

For Terraform, this can mean testing configuration behavior without actually creating all the cloud infrastructure.

Example:

```text
Input:
environment = "prod"

Expected:
production-specific configuration should be selected
```

Terraform's native testing and mocking capabilities can be useful for this type of testing.

---

## Integration Testing

Integration testing verifies that Terraform actually works with the cloud provider.

Example:

```text
Terraform
   |
   v
AWS
   |
   v
S3 bucket created
   |
   v
Terratest verifies bucket
```

This is where Terratest becomes particularly useful.

Terratest can apply real Terraform infrastructure and then query AWS APIs to verify the result.

---

# 13. Acceptance / End-to-End Testing

An even higher level of testing is:

```text
Terraform
   |
   v
AWS infrastructure
   |
   v
Application
   |
   v
HTTP/API request
   |
   v
Expected response
```

For example:

```text
Terraform creates ALB
       |
       v
Application is deployed
       |
       v
Get Load Balancer DNS
       |
       v
HTTP request
       |
       v
HTTP 200 expected
```

That is a very strong production-style Terratest use case.

Terratest provides helpers for HTTP requests, AWS APIs, SSH and other infrastructure-related verification.

---

# 14. The Basic Terratest Workflow

The basic Terratest lifecycle is:

```text
1. Prepare Terraform
        |
        v
2. terraform init
        |
        v
3. terraform apply
        |
        v
4. Read Terraform outputs
        |
        v
5. Query AWS / application
        |
        v
6. Assert expected result
        |
        v
7. terraform destroy
```

In Go this commonly looks conceptually like:

```go
terraform.InitAndApply()

resource := terraform.Output()

assert(resource)

terraform.Destroy()
```

Terratest's official examples follow this general pattern.

---

# 15. Why Cleanup Is Extremely Important

Terratest often creates real cloud resources.

For example:

```text
EC2
RDS
S3
ALB
VPC
EKS
```

If the test finishes but infrastructure is not destroyed, you can end up with:

```text
AWS resources
+
AWS bill
```

Therefore, cleanup is extremely important.

A common Terratest pattern is:

```go
defer terraform.Destroy(t, terraformOptions)
```

The idea is:

> No matter whether the test succeeds or fails, try to destroy the infrastructure created by the test.

This is one of the most important things to explain in an interview.

Terraform's own test framework also warns that apply-based tests can create real infrastructure and therefore can incur cloud costs.

---

# 16. Production-Style Hands-On Demo

We will create a simple production-style S3 bucket.

Imagine that this bucket is used by an application to store:

```text
Application artifacts
Application exports
Backup files
Log archives
```

Our production requirements are:

```text
1. Bucket must exist
2. Versioning must be enabled
3. Server-side encryption must be enabled
4. Environment tag must exist
5. ManagedBy tag must exist
```

Then we will write a Terratest test that:

```text
Creates the bucket
      |
      v
Checks the bucket exists
      |
      v
Checks versioning
      |
      v
Checks encryption
      |
      v
Checks tags
      |
      v
Destroys the bucket
```

---

# 17. Prerequisites

You need:

```text
Terraform
Go
AWS CLI
AWS credentials
```

You can verify them:

```bash
terraform version
```

```bash
go version
```

```bash
aws sts get-caller-identity
```

If the last command successfully returns your AWS identity, Terraform and Terratest can use your configured AWS credentials.

Current Terratest documentation states that the current project requires Go 1.26 or later. For a stable learning setup, pinning a released Terratest version is preferable to blindly depending on a moving latest version.

---

# 18. Create the Project

Create the following structure:

```text
terraform-terratest-demo/
│
├── main.tf
├── variables.tf
├── outputs.tf
│
└── test/
    └── s3_bucket_test.go
```

Create it:

```bash
mkdir -p terraform-terratest-demo/test
cd terraform-terratest-demo
```

---

# 19. Create main.tf

Create:

```bash
cat > main.tf <<'EOF'
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_s3_bucket" "app_bucket" {
  bucket        = var.bucket_name
  force_destroy = true

  tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    Purpose     = "Application Artifacts"
  }
}

resource "aws_s3_bucket_versioning" "app_bucket" {
  bucket = aws_s3_bucket.app_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "app_bucket" {
  bucket = aws_s3_bucket.app_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "app_bucket" {
  bucket = aws_s3_bucket.app_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
EOF
```

---

# 20. Create variables.tf

Create:

```bash
cat > variables.tf <<'EOF'
variable "aws_region" {
  type    = string
  default = "us-east-1"
}

variable "bucket_name" {
  type = string
}

variable "environment" {
  type    = string
  default = "test"
}
EOF
```

---

# 21. Create outputs.tf

Create:

```bash
cat > outputs.tf <<'EOF'
output "bucket_name" {
  value = aws_s3_bucket.app_bucket.bucket
}

output "bucket_arn" {
  value = aws_s3_bucket.app_bucket.arn
}
EOF
```

---

# 22. Test the Terraform Code Manually First

Before introducing Terratest, make sure the Terraform code itself works.

Run:

```bash
terraform init
```

Then:

```bash
terraform validate
```

Then:

```bash
terraform plan \
  -var="bucket_name=terraform-terratest-demo-123456789"
```

Then:

```bash
terraform apply \
  -var="bucket_name=terraform-terratest-demo-123456789" \
  -auto-approve
```

Check the bucket from AWS:

```bash
aws s3 ls | grep terraform-terratest-demo
```

Check Terraform output:

```bash
terraform output
```

When finished with the manual test:

```bash
terraform destroy \
  -var="bucket_name=terraform-terratest-demo-123456789" \
  -auto-approve
```

This confirms that our Terraform configuration works before we automate the testing.

---

# 23. Initialize the Go Test Project

Move into the test directory:

```bash
cd test
```

Initialize a Go module:

```bash
go mod init terraform-terratest-demo
```

Install Terratest:

```bash
go get github.com/gruntwork-io/terratest@v1.0.1
```

Install the testing dependency:

```bash
go get github.com/stretchr/testify
```

Then:

```bash
go mod tidy
```

For a real enterprise repository, pin the versions of your testing dependencies rather than relying on unpinned `latest` dependencies.

---

# 24. Create the Terratest Test

Create:

```bash
cat > s3_bucket_test.go <<'EOF'
package test

import (
	"context"
	"fmt"
	"os"
	"testing"
	"time"

	"github.com/aws/aws-sdk-go-v2/service/s3/types"
	"github.com/gruntwork-io/terratest/modules/aws"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestS3Bucket(t *testing.T) {

	region := os.Getenv("AWS_REGION")

	if region == "" {
		region = "us-east-1"
	}

	bucketName := fmt.Sprintf(
		"terraform-terratest-demo-%d",
		time.Now().UnixNano(),
	)

	terraformOptions := &terraform.Options{
		TerraformDir: "../",
		Vars: map[string]interface{}{
			"aws_region":  region,
			"bucket_name": bucketName,
			"environment": "test",
		},
	}

	// Always destroy the infrastructure after the test.
	defer terraform.Destroy(t, terraformOptions)

	// Terraform init + apply.
	terraform.InitAndApply(t, terraformOptions)

	ctx := context.Background()

	// Test 1: Bucket should exist.
	aws.AssertS3BucketExistsContext(
		t,
		ctx,
		region,
		bucketName,
	)

	// Test 2: Versioning should be enabled.
	aws.AssertS3BucketVersioningExistsContext(
		t,
		ctx,
		region,
		bucketName,
	)

	// Test 3: Encryption should be enabled.
	aws.AssertS3BucketServerSideEncryptionContext(
		t,
		ctx,
		region,
		bucketName,
		types.ServerSideEncryptionAes256,
	)

	// Test 4: Required tags should exist.
	tags := aws.GetS3BucketTagsContext(
		t,
		ctx,
		region,
		bucketName,
	)

	assert.Equal(t, "test", tags["Environment"])
	assert.Equal(t, "Terraform", tags["ManagedBy"])
	assert.Equal(t, "Application Artifacts", tags["Purpose"])
}
EOF
```

Terratest's AWS module provides helpers for checking S3 bucket existence, versioning, encryption and tags, which makes this type of infrastructure acceptance test quite small.

---

# 25. Run the Test

From:

```text
terraform-terratest-demo/test
```

run:

```bash
go test -v -timeout 30m
```

You should see output similar to:

```text
=== RUN   TestS3Bucket

Initializing Terraform...

Applying Terraform...

S3 bucket created

Checking bucket existence...
PASS

Checking versioning...
PASS

Checking encryption...
PASS

Checking tags...
PASS

Destroying Terraform infrastructure...

Destroy complete

--- PASS: TestS3Bucket

PASS
ok
```

The exact output will vary.

---

# 26. What Just Happened?

This is the most important part of the demo.

We wrote:

```go
terraform.InitAndApply(t, terraformOptions)
```

Terratest executed Terraform.

Conceptually:

```text
Go Test
   |
   v
Terratest
   |
   v
terraform init
   |
   v
terraform apply
   |
   v
AWS
```

Then:

```go
aws.AssertS3BucketExistsContext(...)
```

Terratest contacted AWS and verified that the bucket actually exists.

Then:

```go
aws.AssertS3BucketVersioningExistsContext(...)
```

Terratest checked the real AWS bucket configuration.

Then:

```go
aws.AssertS3BucketServerSideEncryptionContext(...)
```

Terratest verified the encryption configuration.

Then:

```go
aws.GetS3BucketTagsContext(...)
```

Terratest retrieved the actual AWS tags.

Finally:

```go
defer terraform.Destroy(...)
```

destroyed the temporary infrastructure.

So the test is not simply checking Terraform text.

It is checking:

```text
Terraform Code
      |
      v
Actual AWS Infrastructure
      |
      v
Actual AWS Configuration
      |
      v
Assertions
```

That is the key value of Terratest.

---

# 27. Let's Intentionally Break the Terraform Code

Now we can demonstrate why testing is useful.

Open:

```text
main.tf
```

Find:

```hcl
resource "aws_s3_bucket_versioning" "app_bucket" {
  bucket = aws_s3_bucket.app_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

Change:

```hcl
status = "Enabled"
```

to:

```hcl
status = "Suspended"
```

Now run:

```bash
go test -v -timeout 30m
```

The test should fail at the versioning assertion.

Conceptually:

```text
Terraform creates bucket
        |
        v
Terratest checks bucket
        |
        v
Versioning = Suspended
        |
        v
Expected = Enabled
        |
        v
TEST FAILED
```

That is exactly what we want.

The test catches a production problem before the Terraform module is trusted.

---

# 28. Another Simple Failure Example

Change:

```hcl
Environment = var.environment
```

to:

```hcl
Environment = "wrong-value"
```

The Terraform deployment can still succeed.

But the Terratest assertion:

```go
assert.Equal(t, "test", tags["Environment"])
```

will fail.

This is a very good example to explain during an interview:

> Terraform apply succeeding does not necessarily mean our infrastructure meets the application's or organization's requirements.

---

# 29. Why This Is a Production-Style Test

The example is intentionally small, but the pattern is realistic.

In a real enterprise, the same approach can be used for:

```text
S3
IAM
VPC
EC2
RDS
EKS
ALB
Route53
Security Groups
CloudWatch
SNS
SQS
```

For example, imagine an enterprise Terraform VPC module.

The test could verify:

```text
VPC exists
       |
       +--> expected CIDR
       |
       +--> expected subnets
       |
       +--> private route tables
       |
       +--> NAT gateways
       |
       +--> expected tags
```

For an ALB:

```text
ALB exists
       |
       +--> listener exists
       |
       +--> HTTPS enabled
       |
       +--> target group exists
       |
       +--> health check works
```

For an EKS cluster:

```text
Cluster exists
       |
       +--> node groups exist
       |
       +--> expected Kubernetes resources
       |
       +--> application endpoint works
```

---

# 30. How Terratest Fits Into CI/CD

In an enterprise environment, we normally do not want developers manually running Terratest on their laptop.

We can run it in CI/CD.

A simplified pipeline could be:

```text
Developer
    |
    v
Git Push
    |
    v
CI/CD Pipeline
    |
    +--> terraform fmt
    |
    +--> terraform validate
    |
    +--> security checks
    |
    +--> terraform plan
    |
    +--> Terratest
    |
    v
Approval
    |
    v
Production Deployment
```

The Terratest stage may use a dedicated AWS test account.

For example:

```text
AWS Organization
        |
        +----------------+
        |                |
        v                v
 Test Account       Production Account
        |
        v
 Terratest
```

This is safer than running destructive integration tests against production infrastructure.

---

# 31. Why Dedicated Test Accounts Are Important

Suppose a test accidentally executes:

```text
terraform destroy
```

against production.

That is obviously dangerous.

Therefore, integration testing should ideally happen in:

```text
Development Account
Test Account
Sandbox Account
Ephemeral Environment
```

rather than directly against production.

A common enterprise strategy is:

```text
Pull Request
     |
     v
Terraform Tests
     |
     v
Temporary Test Environment
     |
     v
Tests
     |
     v
Destroy
```

---

# 32. Important Terratest Best Practice

One of the biggest mistakes is running expensive infrastructure for every tiny change.

For example:

```text
Every Pull Request
      |
      v
Create EKS cluster
      |
      v
Run test
      |
      v
Destroy EKS
```

This can become expensive and slow.

Therefore, teams usually choose carefully what belongs in:

```text
Fast tests
```

and what belongs in:

```text
Integration tests
```

For example:

```text
Every PR

terraform fmt
terraform validate
linting
security checks
plan
small Terraform tests
```

Then:

```text
Integration pipeline

Terratest
real AWS resources
HTTP testing
EKS testing
end-to-end validation
```

---

# 33. Terratest Test Isolation

Another important concept is test isolation.

Imagine two tests run simultaneously:

```text
Test A -> bucket-name-test
Test B -> bucket-name-test
```

They may collide.

Therefore, test infrastructure should normally use unique names.

In our example we used:

```go
time.Now().UnixNano()
```

to generate a unique bucket name.

In larger projects, teams commonly use dedicated random-name helpers and temporary directories.

The principle is:

> Each test should be isolated from other tests.

---

# 34. Idempotency

Terraform is supposed to be declarative and idempotent.

Our tests should also consider idempotency.

For example:

```text
terraform apply
      |
      v
Infrastructure created
      |
      v
terraform apply again
      |
      v
No unexpected changes
```

A mature Terraform test strategy can verify that repeated operations behave as expected.

This is especially important for reusable modules.

---

# 35. Testing Terraform Modules

This is a very common interview topic.

Suppose we have:

```text
modules/
└── s3/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

Multiple application teams use this module.

The test should verify the module contract.

For example:

```text
Input:
environment = prod

Expected:
Environment tag = prod
Versioning = Enabled
Encryption = Enabled
```

The idea is:

```text
Module Contract
      |
      v
Automated Tests
      |
      v
Any Change
      |
      v
Tests Run Again
```

This helps prevent breaking changes to shared infrastructure modules.

---

# 36. What Should Not Be Tested With Terratest?

Not everything needs Terratest.

For example:

```bash
terraform fmt
terraform validate
```

are cheap and should be run frequently.

You do not need to create an AWS environment just to check Terraform syntax.

Similarly, policy tools can be used for things such as:

```text
No public S3 buckets
No unrestricted SSH
Required tags
Encryption requirements
Approved regions
```

The best strategy is to use the right tool for the right problem.

---

# 37. A Practical Terraform Testing Strategy

A simple enterprise model is:

```text
Layer 1
Terraform formatting

Layer 2
Terraform validate

Layer 3
Linting / security / policy checks

Layer 4
Terraform plan

Layer 5
Native Terraform tests

Layer 6
Terratest integration tests

Layer 7
Application / end-to-end tests
```

This gives fast feedback first and expensive testing later.

---

# 38. Interview Question: What is Terratest?

Good answer:

> Terratest is a Go-based testing framework from Gruntwork that helps us test Infrastructure as Code. It can execute Terraform, deploy real infrastructure, query cloud APIs, and validate that the infrastructure behaves as expected. After the test, it can destroy the temporary infrastructure.

---

# 39. Interview Question: Why Use Terratest Instead of terraform validate?

Good answer:

> `terraform validate` checks whether the Terraform configuration is structurally valid. It does not verify that the deployed infrastructure actually behaves correctly. Terratest can deploy the infrastructure and then validate the real infrastructure using AWS APIs, HTTP requests, SSH, Kubernetes APIs, and other checks.

---

# 40. Interview Question: What Is the Difference Between terraform test and Terratest?

Good answer:

> Terraform has a native testing framework using `.tftest.hcl` files and the `terraform test` command. Terratest is an external Go-based framework. I would use native Terraform tests for module logic and configuration behavior, while Terratest is useful when I need programmable integration or end-to-end tests against real infrastructure.

---

# 41. Interview Question: Does Terratest Create Real Infrastructure?

Good answer:

> It can. One of the main strengths of Terratest is that it can run Terraform against real infrastructure and then validate that infrastructure. Because the infrastructure can be real, we need proper cleanup and preferably a dedicated test or sandbox account.

---

# 42. Interview Question: How Does Terratest Clean Up Resources?

Good answer:

> We normally register Terraform destroy as cleanup using Go's `defer`, so the destroy operation is attempted when the test exits. This is important because Terratest may create real cloud resources.

Example:

```go
defer terraform.Destroy(t, terraformOptions)
```

---

# 43. Interview Question: How Would You Test an AWS VPC Module?

Good answer:

> I would create a temporary VPC using the Terraform module and then use Terratest to verify the actual infrastructure. I would check the VPC CIDR, subnet count, subnet CIDRs, route tables, NAT gateways, tags, and other important properties. After the assertions complete, I would destroy the test infrastructure.

---

# 44. Interview Question: How Would You Test an ALB?

Good answer:

> First I would use Terraform to create the ALB and the target infrastructure. Then Terratest could retrieve the ALB DNS name and make an HTTP or HTTPS request. I could assert that the expected HTTP status is returned and that the application is reachable. This validates not only Terraform configuration but also the actual infrastructure and application path.

---

# 45. Interview Question: How Would You Test EKS Terraform Code?

Good answer:

> I would first verify the cluster and node groups using AWS APIs. Then I could configure access to the cluster, verify Kubernetes resources, and potentially test an application endpoint. For a full end-to-end test, I could create an ephemeral EKS environment, deploy an application, perform an HTTP request, verify the expected response, and then destroy the environment.

---

# 46. Interview Question: How Do You Avoid Terraform Test Costs?

Good answer:

> I would separate inexpensive tests from expensive integration tests. Formatting, validation, linting, policy checks, and plan-based tests can run frequently. Real infrastructure tests should run in a dedicated test account and should create the smallest possible environment. Cleanup must always be automated.

---

# 47. Interview Question: What Happens If the Terratest Assertion Fails?

Example:

```go
assert.Equal(t, "test", tags["Environment"])
```

Suppose AWS returns:

```text
Environment = production
```

instead of:

```text
Environment = test
```

The test fails.

But cleanup should still happen because we registered:

```go
defer terraform.Destroy(...)
```

The lifecycle becomes:

```text
Apply
  |
  v
Assertion fails
  |
  v
Test marked FAILED
  |
  v
Destroy
```

This is exactly what we want from an automated integration test.

---

# 48. The Most Important Terratest Concepts to Remember

For an interview, remember these concepts:

```text
Go
 |
 v
Terratest
 |
 +--> Terraform
 |
 +--> AWS APIs
 |
 +--> Kubernetes
 |
 +--> HTTP
 |
 +--> SSH
```

And remember the basic lifecycle:

```text
Init
 |
 v
Apply
 |
 v
Verify
 |
 v
Destroy
```

The most important Terratest functions you will commonly see are:

```go
terraform.InitAndApply()
terraform.Output()
terraform.Destroy()
```

along with assertions and cloud-provider helpers.

---

# 49. What Would a Production CI/CD Implementation Look Like?

A simplified implementation could be:

```text
Developer opens Pull Request
              |
              v
        CI/CD Pipeline
              |
      +-------+-------+
      |               |
      v               v
terraform fmt     terraform validate
      |               |
      +-------+-------+
              |
              v
       Security / Policy
              |
              v
        terraform plan
              |
              v
       Terraform Tests
              |
              v
         Terratest
              |
              v
      Temporary AWS Env
              |
              v
          Assertions
              |
              v
           Destroy
              |
              v
        Manual Approval
              |
              v
       Production Apply
```

This is a much stronger answer than simply saying:

> "We use Terratest."

It shows that you understand where testing belongs in the Infrastructure as Code lifecycle.

---

# 50. Native Terraform Testing Example

For completeness, here is a very small example of what native Terraform testing can look like.

Create:

```text
tests/s3.tftest.hcl
```

Example:

```hcl
run "s3_bucket_test" {

  command = apply

  assert {
    condition     = aws_s3_bucket_versioning.app_bucket.versioning_configuration[0].status == "Enabled"
    error_message = "S3 bucket versioning must be enabled."
  }

  assert {
    condition     = aws_s3_bucket.app_bucket.tags["ManagedBy"] == "Terraform"
    error_message = "ManagedBy tag must be Terraform."
  }
}
```

Run:

```bash
terraform test
```

The important concept is that Terraform itself executes the testing workflow and assertions.

Terraform test files use `.tftest.hcl` or `.tftest.json`, with `tests` being the conventional directory.

---

# 51. When Would I Choose Each Option?

A simple decision table:

| Requirement | Recommended Approach |
|---|---|
| Terraform syntax | `terraform validate` |
| Formatting | `terraform fmt` |
| Static Terraform quality | Linting |
| Security/policy | Policy/security tools |
| Test Terraform module behavior | `terraform test` |
| Test without real infrastructure | Native Terraform testing + mocking |
| Test real AWS infrastructure | Terratest |
| Test HTTP endpoint | Terratest |
| Test Kubernetes behavior | Terratest |
| Full infrastructure + application validation | Terratest |

---

# 52. Simple Mental Model

Remember this:

```text
terraform validate
        |
        v
"Is my Terraform configuration valid?"

terraform test
        |
        v
"Does my Terraform configuration behave as expected?"

Terratest
        |
        v
"Does the infrastructure I created actually work?"
```

This is not an absolute separation because the capabilities overlap, but it is a very useful mental model for interviews.

---

# 53. Final Interview Answer

If the interviewer asks:

> "How do you test Terraform code?"

A strong answer can be:

> "I use multiple layers of testing. First, I run terraform fmt and terraform validate for basic code quality and configuration validation. I use plan and policy or security checks to detect unwanted infrastructure changes and violations. For Terraform module behavior, I can use the native terraform test framework. For integration testing, I can use Terratest with Go. Terratest can execute Terraform, create temporary infrastructure in a dedicated test environment, verify the real resources using cloud APIs or application-level checks, and then destroy the test infrastructure. For example, for an S3 Terraform module I could create a temporary bucket, verify that it exists, check versioning, encryption and required tags, and then automatically destroy it."

That is a concise but technically strong interview answer.

---

# 54. One Sentence to Remember

The easiest sentence to remember is:

> Terraform creates the infrastructure, and Terratest verifies that the infrastructure actually behaves the way we expect.

---

# 55. Suggested Video Flow

For a simple YouTube educational video, the flow can be:

```text
1. Why Terraform testing is required
2. Different types of Terraform testing
3. terraform validate vs terraform test
4. What is Terratest?
5. Terratest architecture
6. Terratest lifecycle
7. Production-style S3 example
8. Write Terraform code
9. Write Go test
10. Run the test
11. Break the Terraform code intentionally
12. Watch the test fail
13. Explain CI/CD integration
14. Interview questions
```

This keeps the demonstration simple while still covering the concepts an interviewer is likely to expect.

# 56. Key Takeaways

```text
Terraform testing is about validating infrastructure behavior.

terraform validate checks configuration validity.

terraform test provides native Terraform testing.

Terratest is a Go-based infrastructure testing framework.

Terratest can test real cloud infrastructure.

Integration tests should normally run in isolated environments.

Temporary infrastructure must be cleaned up automatically.

Do not use expensive end-to-end tests for every small change.

Use multiple layers of testing in enterprise CI/CD.

The objective is not to test every Terraform line.

The objective is to prove that the infrastructure behaves correctly.
```
