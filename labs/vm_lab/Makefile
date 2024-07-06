DOCKER := docker
BUILD_DATE:=$(shell date --rfc-3339=seconds)
GIT_SHA:=$(shell git rev-parse HEAD)

REGISTRY?=docker.io/dozman99
IMAGE_NAME?=$(REGISTRY)/lab-vm-os
TAG?=$(shell git rev-parse --short HEAD)

OS_VERSION?=22.04 # RELEASE follows the kubernetes release

build: $(addprefix build-,$(OS_VERSION))
build-%:
	$(DOCKER) build -t $(IMAGE_NAME):$*  \
		--build-arg OS_VERSION=$* \
		--build-arg OS_VERSION=$(OS_VERSION) \
		--label "org.opencontainers.image.created"="$(BUILD_DATE)" \
		--label "org.opencontainers.image.authors"="Liquid Metal Authors" \
		--label "org.opencontainers.image.url"="https://github.com/weaveworks-liquidmetal/image-builder" \
		--label "org.opencontainers.image.source"="https://github.com/weaveworks-liquidmetal/image-builder/tree/main/capmvm/kubernetes" \
		--label "org.opencontainers.image.revision"="$(GIT_SHA)" \
		--label "org.opencontainers.image.vendor"="Weaveworks" \
		--label "org.opencontainers.image.licenses"="MPL-2.0" \
		--label "org.opencontainers.image.title"="Cluster API Provider Microvm Kubernetes v$*" \
		--label "org.opencontainers.image.description"="A image to use for a microvm volume when creating a cluster using Cluster API Provider Microvm" \
		--label "works.weave.liquidmetal.kubernetes"="$*" \
		--label "works.weave.liquidmetal.image_version"="$(OS_VERSION)" \
		.


push: $(addprefix push-,$(OS_VERSION))
push-%:
	$(DOCKER) push $(IMAGE_NAME):$*
