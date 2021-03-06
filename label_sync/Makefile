# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

all: build test


LABEL_SYNC_VERSION       ?= 0.1

# These are the usual GKE variables.
PROJECT       ?= k8s-prow
BUILD_PROJECT ?= k8s-prow-builds
ZONE          ?= us-central1-f
CLUSTER       ?= prow

# Build and push specific variables.
REGISTRY ?= gcr.io
PUSH     ?= gcloud docker -- push

DOCKER_LABELS=--label io.k8s.prow.git-describe="$(shell git describe --tags --always --dirty)"

get-cluster-credentials:
	gcloud container clusters get-credentials "$(CLUSTER)" --project="$(PROJECT)" --zone="$(ZONE)"

get-build-cluster-credentials:
	gcloud container clusters get-credentials "$(CLUSTER)" --project="$(BUILD_PROJECT)" --zone="$(ZONE)"

build:
	go install ./...

test:
	go test -race -cover $$(go list ./... | grep -v "\/vendor\/")

.PHONY: build test get-cluster-credentials

compile-static:
	CGO_ENABLED=0 go build -o label_sync k8s.io/test-infra/label_sync

label_sync-image: compile-static
	docker build -t "$(REGISTRY)/$(PROJECT)/label_sync:$(LABEL_SYNC_VERSION)" $(DOCKER_LABELS) .
	$(PUSH) "$(REGISTRY)/$(PROJECT)/label_sync:$(LABEL_SYNC_VERSION)"

label_sync-deployment: get-cluster-credentials
	kubectl apply -f cluster/label_sync_deployment.yaml

label_sync-cronjob: get-cluster-credentials
	kubectl apply -f cluster/label_sync_cron_job.yaml

.PHONY: label_sync-image label_sync-deployment label_sync-cronjob
