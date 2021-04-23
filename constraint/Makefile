all: test

# Get the currently used golang install path (in GOPATH/bin, unless GOBIN is set)
ifeq (,$(shell go env GOBIN))
GOBIN=$(shell go env GOPATH)/bin
else
GOBIN=$(shell go env GOBIN)
endif

all: lint test

# Run tests
native-test: generate manifests
	go test ./pkg/... -coverprofile cover.out

# Docker internal test
docker-internal-test:
	make native-test
# The remote driver has some unimplemented functions
# which result in a panic during testing.
#	./tests/test_remote_driver.sh

# Hook to run docker tests
test: generate
	docker build . -t constraint-test && docker run -t constraint-test

# Install CRDs into a cluster
install: manifests
	kubectl apply -f config/crds

# Generate manifests e.g. CRD, RBAC etc.
manifests: __controller-gen
	$(CONTROLLER_GEN) crd:crdVersions="v1beta1",allowDangerousTypes="true" paths="./pkg/..." output:crd:artifacts:config=config/crds
	kustomize build config -o deploy/crds.yaml

lint:
	golangci-lint -v run ./... --timeout 5m

TOOLING_IMAGE := gatekeeper-tooling
__tooling-image:
	docker build . \
		--tag $(TOOLING_IMAGE) \
		--file buildutils/Dockerfile

__controller-gen: __tooling-image
# __controller-gen:
CONTROLLER_GEN=docker run -v $(shell pwd):/constraint $(TOOLING_IMAGE) controller-gen

# Generate code
generate: __controller-gen
	$(CONTROLLER_GEN) object:headerFile=./hack/boilerplate.go.txt paths="./pkg/..."
