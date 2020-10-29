SHELL := /bin/bash

create-cluster:
	@echo "Remember to delete the cluster via 'make delete-cluster' after you're done with the tutorial"
	eksctl create cluster \
		--name telegraf-cluster \
		--region us-east-1 \
		--version 1.18 \
		--config-file cluster.yaml

delete-cluster:
	eksctl delete cluster \
		--name telegraf-cluster