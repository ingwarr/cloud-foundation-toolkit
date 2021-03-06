# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
ARG RUBY_VERSION
FROM ruby:$RUBY_VERSION-alpine

# Required to download and install Terraform
ARG TERRAFORM_VERSION
ENV TERRAFORM_VERSION ${TERRAFORM_VERSION}

# Required to download and install terraform-docs
ARG TERRAFORM_DOCS_VERSION
ENV TERRAFORM_DOCS_VERSION ${TERRAFORM_DOCS_VERSION}

# Required to download and install the Terraform gsuite provider
ARG GSUITE_PROVIDER_VERSION
ENV GSUITE_PROVIDER_VERSION ${GSUITE_PROVIDER_VERSION}

# Required to download and install Google Cloud SDK
# Google Cloud SDK is located at /usr/local/google-cloud-sdk
ARG CLOUD_SDK_VERSION
ENV CLOUD_SDK_VERSION ${CLOUD_SDK_VERSION}

# $WORKSPACE is intended for assets that persist across multiple build steps in a pipeline.
# It's also where the project git repository is located.
# https://cloud.google.com/cloud-build/docs/build-config
ENV WORKSPACE="/workspace"
ENV PATH /usr/local/google-cloud-sdk/bin:$PATH

# bash is preferred over /bin/sh
RUN apk add --no-cache bash

# All package-level dependencies are handled by install_dependencies.sh
ADD ./build/install_dependencies.sh /build/
RUN /build/install_dependencies.sh

ADD ./build/install_cloud_sdk.sh /build/
RUN /build/install_cloud_sdk.sh ${CLOUD_SDK_VERSION}

ADD ./build/install_gsuite_terraform_provider.sh /build/
RUN /build/install_gsuite_terraform_provider.sh ${GSUITE_PROVIDER_VERSION}

ADD ./build/install_terraform.sh /build/
RUN /build/install_terraform.sh ${TERRAFORM_VERSION}

ADD ./build/install_shellcheck.sh /build/
RUN /build/install_shellcheck.sh

ADD ./build/install_hadolint.sh /build/
RUN /build/install_hadolint.sh

ADD ./build/install_terraform_docs.sh /build/
RUN /build/install_terraform_docs.sh ${TERRAFORM_DOCS_VERSION}

ADD ./build/install_kubectl.sh /build/
RUN /build/install_kubectl.sh

WORKDIR /opt/kitchen
ADD ./build/data/Gemfile .
ADD ./build/data/Gemfile.lock .
ADD ./build/data/requirements.txt .
RUN bundle install && pip3 install -r requirements.txt

COPY ./build/install_verify_boilerplate.sh /build/
COPY ./build/verify_boilerplate/ /build/verify_boilerplate/
RUN /build/install_verify_boilerplate.sh

# NOTE: Most developers are expected to source this file when entering the
# container, then run init_credentials
ADD ./build/scripts/task_helper_functions.sh /usr/local/bin/
# Provide task helper functions with wrapper scripts for Terraform module lint
# checks
ADD ./build/scripts/task_wrapper_scripts/* /usr/local/bin/
# Execute a command with SA credentials configured.
ADD ./build/scripts/execute_with_credentials.sh /usr/local/bin/
# Execute the module's integration tests and cleanup.
ADD ./build/scripts/test_integration.sh /usr/local/bin/
# Execute the module's lint tests and cleanup.
ADD ./build/scripts/test_lint.sh /usr/local/bin/

# Intended to help developers iterate quickly
ADD ./build/home/bash_history /root/.bash_history
ADD ./build/home/bashrc /root/.bashrc

WORKDIR $WORKSPACE
RUN terraform --version && \
    terraform-docs --version && \
    gcloud --version && \
    ruby --version && \
    bundle --version && \
    kubectl version --client=true

# Cleanup intermediate build artifacts
RUN rm -rf /build
CMD ["/bin/bash"]
