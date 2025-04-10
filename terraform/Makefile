DEV_VARS := -var-file=dev.tfvars
PROD_VARS := -var-file=prod.tfvars
CORE_TARGETS := -target=azurerm_resource_group.this -target=azurerm_virtual_network.this -target=azurerm_subnet.public -target=azurerm_subnet.private -target=azurerm_databricks_workspace.this
STORAGE_TARGETS := -target=azurerm_storage_account.adls -target=azurerm_storage_data_lake_gen2_filesystem.this

TF_ORG := engenharia-academy
TF_WORKSPACE := databricks-platform

.PHONY: login
login:
	terraform login

.PHONY: init
init: login
	terraform init

.PHONY: dev-plan-core
dev-plan-core: init
	terraform plan $(DEV_VARS) $(CORE_TARGETS)

.PHONY: dev-apply-core
dev-apply-core: dev-plan-core
	terraform apply $(DEV_VARS) $(CORE_TARGETS)

.PHONY: dev-plan-storage
dev-plan-storage: dev-apply-core
	terraform plan $(DEV_VARS) $(STORAGE_TARGETS)

.PHONY: dev-apply-storage
dev-apply-storage: dev-plan-storage
	terraform apply $(DEV_VARS) $(STORAGE_TARGETS)

.PHONY: dev-plan-databricks
dev-plan-databricks: dev-apply-storage
	terraform plan $(DEV_VARS)

.PHONY: dev-apply-databricks
dev-apply-databricks: dev-plan-databricks
	@echo "Getting Databricks workspace URL..."
	./scripts/get_workspace_url.sh
	terraform apply $(DEV_VARS)

.PHONY: dev-deploy
dev-deploy: dev-apply-databricks
	@echo "Development environment deployment complete"

.PHONY: prod-plan-core
prod-plan-core: init
	terraform plan $(PROD_VARS) $(CORE_TARGETS)

.PHONY: prod-apply-core
prod-apply-core: prod-plan-core
	terraform apply $(PROD_VARS) $(CORE_TARGETS)

.PHONY: prod-plan-storage
prod-plan-storage: prod-apply-core
	terraform plan $(PROD_VARS) $(STORAGE_TARGETS)

.PHONY: prod-apply-storage
prod-apply-storage: prod-plan-storage
	terraform apply $(PROD_VARS) $(STORAGE_TARGETS)

.PHONY: prod-plan-databricks
prod-plan-databricks: prod-apply-storage
	terraform plan $(PROD_VARS)

.PHONY: prod-apply-databricks
prod-apply-databricks: prod-plan-databricks
	terraform apply $(PROD_VARS)

.PHONY: prod-deploy
prod-deploy: prod-apply-databricks
	@echo "Production environment deployment complete"

.PHONY: dev-destroy
dev-destroy:
	terraform destroy $(DEV_VARS)

.PHONY: prod-destroy
prod-destroy:
	terraform destroy $(PROD_VARS)

.PHONY: local-mode
local-mode:
	@echo "Setting up for local execution..."
	rm -f .terraform/terraform.tfstate
	rm -rf .terraform/providers

.PHONY: set-azure-credentials
set-azure-credentials:
	@echo "Please set the following environment variables before running this command:"
	@echo "export ARM_CLIENT_ID=\"your-client-id\""
	@echo "export ARM_CLIENT_SECRET=\"your-client-secret\""
	@echo "export ARM_TENANT_ID=\"your-tenant-id\""
	@echo "export ARM_SUBSCRIPTION_ID=\"your-subscription-id\""

.PHONY: local-init
local-init: local-mode set-azure-credentials
	terraform init

.PHONY: local-dev-deploy-phase1
local-dev-deploy-phase1: local-mode
	@echo "Phase 1: Deploying Azure infrastructure (without Databricks resources)..."
	terraform init
	terraform plan $(DEV_VARS) -target=azurerm_resource_group.this -target=azurerm_virtual_network.this -target=azurerm_subnet.public -target=azurerm_subnet.private -target=azurerm_network_security_group.databricks -target=azurerm_subnet_network_security_group_association.public -target=azurerm_subnet_network_security_group_association.private -target=azurerm_databricks_workspace.this -target=azurerm_key_vault.this -target=azurerm_storage_account.adls -target=azurerm_storage_data_lake_gen2_filesystem.this
	@echo "Review the plan above. Type 'yes' to apply or Ctrl+C to cancel."
	terraform apply $(DEV_VARS) -target=azurerm_resource_group.this -target=azurerm_virtual_network.this -target=azurerm_subnet.public -target=azurerm_subnet.private -target=azurerm_network_security_group.databricks -target=azurerm_subnet_network_security_group_association.public -target=azurerm_subnet_network_security_group_association.private -target=azurerm_databricks_workspace.this -target=azurerm_key_vault.this -target=azurerm_storage_account.adls -target=azurerm_storage_data_lake_gen2_filesystem.this
	@echo "Getting Databricks workspace URL..."
	./scripts/get_workspace_url.sh

.PHONY: local-dev-deploy-phase2
local-dev-deploy-phase2: local-mode
	@echo "Phase 2: Deploying Databricks resources..."
	@echo "Ensuring workspace URL is configured..."
	if [ ! -f workspace.auto.tfvars ]; then \
		./scripts/get_workspace_url.sh; \
	fi
	terraform init
	terraform plan $(DEV_VARS)
	@echo "Review the plan above. Type 'yes' to apply or Ctrl+C to cancel."
	terraform apply $(DEV_VARS)

.PHONY: local-dev-deploy
local-dev-deploy: local-mode
	@echo "Running two-phase deployment in local mode with credentials from credentials.auto.tfvars..."
	@echo "Phase 1: First deploying Azure infrastructure..."
	make local-dev-deploy-phase1
	@echo "Phase 2: Now deploying Databricks resources..."
	make local-dev-deploy-phase2

.PHONY: local-prod-deploy
local-prod-deploy: local-prod-deploy-phase1 local-prod-deploy-phase2
	@echo "Production environment deployment complete"

.PHONY: local-prod-deploy-phase1
local-prod-deploy-phase1: local-mode
	@echo "Phase 1: Deploying Azure infrastructure for PRODUCTION (without Databricks resources)..."
	terraform init -force-copy
	terraform plan $(PROD_VARS)
	@echo "Review the plan above. Type 'yes' to apply or Ctrl+C to cancel."
	terraform apply $(PROD_VARS) -target=azurerm_resource_group.this -target=azurerm_virtual_network.this -target=azurerm_subnet.public -target=azurerm_subnet.private -target=azurerm_network_security_group.databricks -target=azurerm_subnet_network_security_group_association.public -target=azurerm_subnet_network_security_group_association.private -target=azurerm_databricks_workspace.this -target=azurerm_key_vault.this -target=azurerm_storage_account.adls -target=azurerm_storage_data_lake_gen2_filesystem.this
	@echo "Getting Databricks workspace URL..."
	./scripts/get_workspace_url.sh

.PHONY: local-prod-deploy-phase2
local-prod-deploy-phase2: local-mode
	@echo "Phase 2: Deploying Databricks resources for PRODUCTION..."
	@echo "Ensuring workspace URL is configured..."
	if [ ! -f workspace.auto.tfvars ]; then \
		./scripts/get_workspace_url.sh; \
	fi
	terraform init
	terraform plan $(PROD_VARS)
	@echo "Review the plan above. Type 'yes' to apply or Ctrl+C to cancel."
	terraform apply $(PROD_VARS)

.PHONY: cloud-mode
cloud-mode:
	@echo "Setting up for Terraform Cloud execution..."
	@echo "Using organization: $(TF_ORG) and workspace: $(TF_WORKSPACE)"
	rm -f backend.tf
	@echo 'terraform {\n  cloud {\n    organization = "$(TF_ORG)"\n    \n    workspaces {\n      name = "$(TF_WORKSPACE)"\n    }\n  }\n}' > main.tf

.PHONY: cloud-init
cloud-init: cloud-mode login
	terraform init -ignore-remote-version

.PHONY: cloud-dev-deploy
cloud-dev-deploy:
	@echo "For VCS-connected Terraform Cloud workspaces, the recommended workflow is:"
	@echo "1. Make changes to your Terraform configuration"
	@echo "2. Push those changes to your version control system"
	@echo "3. Terraform Cloud automatically detects changes and triggers a plan"
	@echo "4. Review and apply the plan in the Terraform Cloud UI"
	@echo ""
	@echo "Your Terraform Cloud workspace URL:"
	@echo "https://app.terraform.io/app/$(TF_ORG)/workspaces/$(TF_WORKSPACE)"
	@echo ""
	@echo "Ensure your Azure credentials are configured as variables in Terraform Cloud:"
	@echo "- client_id"
	@echo "- client_secret"
	@echo "- tenant_id"
	@echo "- subscription_id"

.PHONY: cloud-prod-deploy
cloud-prod-deploy:
	@echo "For VCS-connected Terraform Cloud workspaces, the recommended workflow is:"
	@echo "1. Make changes to your Terraform configuration"
	@echo "2. Push those changes to your version control system"
	@echo "3. Terraform Cloud automatically detects changes and triggers a plan"
	@echo "4. Review and apply the plan in the Terraform Cloud UI"
	@echo ""
	@echo "Your Terraform Cloud workspace URL:"
	@echo "https://app.terraform.io/app/$(TF_ORG)/workspaces/$(TF_WORKSPACE)"
	@echo ""
	@echo "Ensure your Azure credentials are configured as variables in Terraform Cloud:"
	@echo "- client_id"
	@echo "- client_secret"
	@echo "- tenant_id"
	@echo "- subscription_id"

.PHONY: cloud-setup-guide
cloud-setup-guide:
	@echo "Terraform Cloud Setup Guide for VCS-connected workspace"
	@echo "-----------------------------------------------------"
	@echo ""
	@echo "1. Go to: https://app.terraform.io/app/$(TF_ORG)/workspaces/$(TF_WORKSPACE)/variables"
	@echo ""
	@echo "2. Add the following Terraform variables (mark as sensitive):"
	@echo "   - client_id       = \"$(ARM_CLIENT_ID)\""
	@echo "   - client_secret   = \"$(ARM_CLIENT_SECRET)\""
	@echo "   - tenant_id       = \"$(ARM_TENANT_ID)\""
	@echo "   - subscription_id = \"$(ARM_SUBSCRIPTION_ID)\""
	@echo ""
	@echo "3. Add the following Environment variables (mark as sensitive):"
	@echo "   - ARM_CLIENT_ID       = \"$(ARM_CLIENT_ID)\""
	@echo "   - ARM_CLIENT_SECRET   = \"$(ARM_CLIENT_SECRET)\""
	@echo "   - ARM_TENANT_ID       = \"$(ARM_TENANT_ID)\""
	@echo "   - ARM_SUBSCRIPTION_ID = \"$(ARM_SUBSCRIPTION_ID)\""
	@echo ""
	@echo "4. Push your changes to GitHub to trigger a plan"
	@echo ""
	@echo "5. Review and apply the plan in the Terraform Cloud UI"

.PHONY: azure-backend
azure-backend:
	./azure-backend.sh


.PHONY: help
help:
	@echo "Available targets:"
	@echo "  init              - Initialize Terraform"
	@echo "  login             - Login to Terraform Cloud"
	@echo "  dev-deploy        - Deploy complete development environment"
	@echo "  local-dev-deploy  - Deploy development environment in local mode"
	@echo "  local-prod-deploy - Deploy production environment in local mode"
	@echo "  cloud-dev-deploy  - Deploy development environment in Terraform Cloud"
	@echo "  cloud-prod-deploy - Deploy production environment in Terraform Cloud"
	@echo "  prod-deploy       - Deploy complete production environment"
	@echo "  dev-destroy       - Destroy development environment"
	@echo "  prod-destroy      - Destroy production environment"
	@echo "  azure-backend     - Set up Azure Storage backend"
	@echo "  local-mode        - Set Terraform to run in local mode"
	@echo "  cloud-mode        - Set Terraform to run in Terraform Cloud mode"
	@echo ""
	@echo "Individual phase targets are also available:"
	@echo "  dev-plan-core, dev-apply-core, dev-plan-storage, dev-apply-storage, etc."
