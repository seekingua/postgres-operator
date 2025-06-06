.PHONY: clean copy docker push tools test

BINARY ?= postgres-operator-e2e-tests-runner
BUILD_FLAGS ?= -v
CGO_ENABLED ?= 0
ifeq ($(RACE),1)
	BUILD_FLAGS += -race -a
    CGO_ENABLED=1
endif

LOCAL_BUILD_FLAGS ?= $(BUILD_FLAGS)
LDFLAGS ?= -X=main.version=$(VERSION)

IMAGE            ?= registry.opensource.zalan.do/acid/$(BINARY)
VERSION          ?= $(shell git describe --tags --always --dirty)
TAG              ?= $(VERSION)
GITHEAD          = $(shell git rev-parse --short HEAD)
GITURL           = $(shell git config --get remote.origin.url)
GITSTATU         = $(shell git status --porcelain || echo 'no changes')
TTYFLAGS         = $(shell test -t 0 && echo '-it')

ifndef GOPATH
	GOPATH := $(HOME)/go
endif

PATH := $(GOPATH)/bin:$(PATH)

default: tools

clean:
	rm -rf manifests
	rm -rf tls

copy: clean
	mkdir manifests
	cp -r ../manifests .
	mkdir tls

docker:
	docker build -t "$(IMAGE):$(TAG)" .

push: docker
	docker push "$(IMAGE):$(TAG)"

tools:
	# install pinned version of 'kind'
	# go install must run outside of a dir with a (module-based) Go project !
	# otherwise go install updates project's dependencies and/or behaves differently
	cd "/tmp" && GO111MODULE=on go install sigs.k8s.io/kind@v0.24.0

e2etest: tools copy clean
	./run.sh main

cleanup: clean
	./run.sh cleanup