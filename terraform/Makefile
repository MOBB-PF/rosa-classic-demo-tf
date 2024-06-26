######################
# Log into your AWS account before running this make file. See helpers section for make statements.
# Create .env file with your ROSA and helm repo token (if using seed.). This file will be ignored by git.
# format.
# TF_VAR_helm_token=<Helm Repository TOKEN>
# TF_VAR_token=<ROSA TOKEN>

include .env
export $(shell sed '/^\#/d; s/=.*//' .env)
TF_VAR_AWS_DEFAULT_REGION := $(shell aws configure get region)
TF_VAR_token := $(RHCS_TOKEN)

######################
# State variables. Set your state variables for your bucket and which cluster you are deploying to.
# TF_LOG                        # debug level
# WORKSPACE                     # path within state file directory.
# TF_BACKEND_BUCKET             # S3 bucket name.
# TF_BACKEND_DYNAMO=            # dynamo lock shouldnt need to change.
# TF_BACKEND_KEY=               # path within state file directory.
# VARS                          # vars file of the cluster you want to build.

TF_LOG=ERROR
WORKSPACE=nonprod-dev
TF_BACKEND_BUCKET=terraform-state-foster-ocm-nonprod
TF_BACKEND_DYNAMO=terraform-lock
TF_BACKEND_KEY=cluster-nonprod
VARS=clusters/cluster-nonprod-public.tfvars.json
# TF_BACKEND_KEY=cluster4-nonprod
# VARS=clusters/cluster4-nonprod-public.tfvars.json
######################
.EXPORT_ALL_VARIABLES:

# Run make init \ make plan \ make apply \ make destroy

init:
	terraform init -input=false -lock=false -no-color -reconfigure \
	-backend-config="dynamodb_table=$(TF_BACKEND_DYNAMO)" \
	-backend-config="bucket=$(TF_BACKEND_BUCKET)" \
	-backend-config="key=$(TF_BACKEND_KEY)"
	echo "Selecting Terraform workspace $(WORKSPACE)"
	terraform workspace new $(WORKSPACE) 2>/dev/null || true
	terraform workspace "select" $(WORKSPACE)
.PHONY: init

plan: format validate
	terraform plan -lock=false -var-file=$(VARS) -out=.terraform-plan
.PHONY: plan

apply:
	date
	terraform apply .terraform-plan
.PHONY: apply

destroy:
	terraform destroy -auto-approve -input=false  -var-file=$(VARS)
.PHONY: destroy

output:
	terraform output > tf-output-parameters
.PHONY: output

format:
	terraform fmt -check

validate:
	terraform validate
