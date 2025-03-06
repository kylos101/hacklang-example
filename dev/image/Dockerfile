FROM gitpod/workspace-base:2024-12-11-07-51-54
# Install useful debugging utilities
RUN sudo install-packages \
  bmon \
  dnsutils \
  expect \
  gdb \
  graphviz \
  iptraf-ng \
  net-tools \
  netcat \
  strace \
  tcpdump \
  tree
# Install Node
ENV NODE_VERSION=20.18.1
RUN curl -L https://raw.githubusercontent.com/tj/n/master/bin/n | sudo bash -s install ${NODE_VERSION}
ENV GP_VSCODE_NODE=/usr/local/bin/node
### Install gcloud (see https://cloud.google.com/sdk/docs/install#deb) ###
RUN set -ex \
  && echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list \
  && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - \
  && sudo install-packages google-cloud-cli google-cloud-sdk-gke-gcloud-auth-plugin google-cloud-cli-gke-gcloud-auth-plugin kubectl
### Install Composer 2.2.23 ###
ENV COMPOSER_DIR=/opt/composer
COPY --from=composer/composer:2.2.23-bin /composer ${COMPOSER_DIR}/composer.phar
### Install brew ###
# Install deps
RUN set -ex && sudo install-packages \
  build-essential \
  curl \
  file \
  git \
  procps
# Run installer
RUN set -ex \
  && mkdir ~/.cache \
  && NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
ENV PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin/:$PATH"
ENV MANPATH="${MANPATH:-}:/home/linuxbrew/.linuxbrew/share/man"
ENV INFOPATH="${INFOPATH:-}:/home/linuxbrew/.linuxbrew/share/info"
ENV HOMEBREW_NO_AUTO_UPDATE=1
# Install tools from bundle
COPY Brewfile* ./
RUN set -ex \
  && brew bundle install \
  && rm -rf Brewfile* \
  && rm -rf $(brew --cache)
# Setup Python
# From https://github.com/gitpod-io/workspace-images/blob/main/chunks/lang-python/Dockerfile
USER gitpod
ENV PYTHON_VERSION=3.12.1
RUN sudo install-packages python3-pip
# Recommended build environment from https://github.com/pyenv/pyenv/wiki#suggested-build-environment
RUN sudo install-packages make build-essential libssl-dev zlib1g-dev \
  libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
  libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
ENV PATH=$HOME/.pyenv/bin:$HOME/.pyenv/shims:$PATH
RUN curl -fsSL https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
RUN { echo; \
  echo 'eval "$(pyenv init -)"'; \
  echo 'eval "$(pyenv virtualenv-init -)"'; } >> /home/gitpod/.bashrc.d/60-python
RUN pyenv update
RUN pyenv install ${PYTHON_VERSION}
RUN pyenv global ${PYTHON_VERSION}
RUN python3 -m pip install --no-cache-dir --upgrade pip
RUN python3 -m pip install --no-cache-dir --upgrade \
  setuptools wheel virtualenv pipenv pylint rope flake8 \
  mypy autopep8 pep8 pylama pydocstyle bandit notebook \
  twine
RUN sudo rm -rf /tmp/*
ENV PIP_USER=no
ENV PIPENV_VENV_IN_PROJECT=true
ENV PYTHONUSERBASE=/workspace/.pip-modules
ENV PATH=$PYTHONUSERBASE/bin:$PATH
USER root
RUN mkdir -p $PYTHONUSERBASE && chown gitpod $PYTHONUSERBASE
# Make directory for Rollbar
RUN mkdir -p /var/log/rollbar/events
RUN chmod 777 /var/log/rollbar/events
USER gitpod
# https://github.com/gitpod-io/workspace-images/blob/eb527a231d9b35782b623cfd6fae472796655065/chunks/lang-java/Dockerfile#L11-L28
# https://github.com/gitpod-io/workspace-images/blob/eb527a231d9b35782b623cfd6fae472796655065/chunks/lang-java/chunk.yaml#L7
ENV JAVA_VERSION=17.0.13.fx-zulu
RUN curl -fsSL "https://get.sdkman.io" | bash \
  && bash -c ". /home/gitpod/.sdkman/bin/sdkman-init.sh \
  && sed -i 's/sdkman_selfupdate_enable=true/sdkman_selfupdate_enable=false/g' /home/gitpod/.sdkman/etc/config \
  && sed -i 's/sdkman_selfupdate_feature=true/sdkman_selfupdate_feature=false/g' /home/gitpod/.sdkman/etc/config \
  && sdk install java ${JAVA_VERSION} \
  && sdk install java 17.0.13.fx-zulu \
  && sdk default java ${JAVA_VERSION} \
  && sdk install gradle \
  # 4.x requires Java 17+ https://maven.apache.org/docs/history.html
  && sdk install maven 3.9.9 \
  && sdk flush archives \
  && sdk flush temp \
  && mkdir /home/gitpod/.m2 \
  && printf '<settings>\n  <localRepository>/workspace/m2-repository/</localRepository>\n</settings>\n' > /home/gitpod/.m2/settings.xml \
  && echo 'export SDKMAN_DIR=\"/home/gitpod/.sdkman\"' >> /home/gitpod/.bashrc.d/99-java \
  && echo '[[ -s \"/home/gitpod/.sdkman/bin/sdkman-init.sh\" ]] && source \"/home/gitpod/.sdkman/bin/sdkman-init.sh\"' >> /home/gitpod/.bashrc.d/99-java"
# above, we are adding the sdkman init to .bashrc (executing sdkman-init.sh does that), because one is executed on interactive shells, the other for non-interactive shells (e.g. plugin-host)
ENV GRADLE_USER_HOME=/workspace/.gradle/
# Add the mysql-client@8.4 keg to the path
ENV PATH="/home/linuxbrew/.linuxbrew/opt/mysql-client@8.4/bin:$PATH"
### Git configuration ###
# COPY tooling/devenvs/image/default.gitconfig /etc/gitconfig
# COPY --chown=gitpod:gitpod tooling/devenvs/image/default.gitconfig /home/gitpod/.gitconfig
### Repo Setup ###
# ENV PATH="/workspace/web/tooling/devenvs/bin:$PATH"
# COPY tooling/containerization/qhhvm-cli.sh /usr/local/bin/qhhvm-cli
# The version of /usr/bin/env (coreutils) installed in Ubuntu 18.04 does not support the -S flag,
# so we can't do `#!/usr/bin/env -S yarn tsx` in TS scripts. Instead, we install tsx globally.
RUN yarn global add tsx
RUN curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o /home/gitpod/.bashrc.d/.git-completion.bash
# COPY --chown=gitpod:gitpod tooling/devenvs/image/bash_aliases.sh /home/gitpod/.bashrc.d/
# COPY --chown=gitpod:gitpod tooling/devenvs/image/.my.cnf /home/gitpod/
# Install github CLI for downloading hack transpiler binary
RUN sudo mkdir -p -m 755 /etc/apt/keyrings \
  && wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo install-packages gh