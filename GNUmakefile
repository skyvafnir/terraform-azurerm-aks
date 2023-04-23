SHELL := /bin/bash

.PHONY: pre-commit
GO_DEPS_VOLUME_NAME ?= azure-tf-go-deps
TF_CACHE_VOLUME_NAME ?= azure-tf-tf-cache

-include $(shell curl -sSL "https://raw.githubusercontent.com/Azure/tfmod-scaffold/main/scripts/install.sh" | bash -s > /dev/null ; echo tfmod-scaffold/GNUmakefile)

init:
	@sh "$(CURDIR)/scripts/init.sh"

cleanup: docker.clean-volumes
	@sh "$(CURDIR)/scripts/cleanup.sh"


_docker.ensure-volumes:
	echo "Ensuring GOLANG deps volume exists"
	@docker volume create $(GO_DEPS_VOLUME_NAME)
	echo "Ensuring Terraform cache volume exists"
	@docker volume create $(TF_CACHE_VOLUME_NAME)

docker.clean-volumes:
	@echo "Cleaning up golang dependency and terraform cache volumes"
	@docker volume rm  $(GO_DEPS_VOLUME_NAME)
	@docker volume rm  $(TF_CACHE_VOLUME_NAME)

docker.shell:  # For diagnosis and debugging
	@docker run -it --rm -v $(GO_DEPS_VOLUME_NAME):/go/pkg/mod -v $(shell pwd):/src -w /src mcr.microsoft.com/azterraform:latest bash

docker.pre-commit: _docker.ensure-volumes
	@docker run --rm \
      -e TF_PLUGIN_CACHE_DIR="/tf_cache" \
      -v $(TF_CACHE_VOLUME_NAME):/tf_cache \
      -v $(GO_DEPS_VOLUME_NAME):/go/pkg/mod \
      -v $(shell pwd):/src \
      -w /src mcr.microsoft.com/azterraform:latest \
      make pre-commit

docker.pre-check: _docker.ensure-volumes
	@docker run --rm \
      -v $(TF_CACHE_VOLUME_NAME):/tf_cache \
      -v $(GO_DEPS_VOLUME_NAME):/go/pkg/mod \
      -v $(shell pwd):/src \
      -w /src mcr.microsoft.com/azterraform:latest \
      make pr-check

docker.full-check: docker.pre-commit docker.pre-check