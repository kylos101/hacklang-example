# USAGE https://github.com/Homebrew/homebrew-bundle
tap "homebrew/bundle"
tap "hashicorp/tap"
tap "derailed/k9s"
# Tools for the dev environment
brew "git"
brew "hub"
brew "k9s"
# Tools for DB development
tap "planetscale/homebrew-tap"
brew "pscale"
brew "mysql-client@8.4" # We run MySQL Server 8.0, but the client is backwards-compatible (https://dev.mysql.com/doc/mysql-compat-matrix/en/)
# utilities that our documentation expects you have
brew "findutils"
brew "shellcheck"
brew "shfmt"
# Runtimes
# required by ads tooling
brew "pipenv"
# required by this app's tooling
brew "circleci"
# If your Yarn isn't the right version, try `npm remove -g yarn && nodenv rehash` to delete any npm-installed one.
# Omit Node because we use Nodenv for that https://yarnpkg.com/en/docs/install#mac-stable
brew "yarn"
brew "nss"
brew "semgrep"
brew "jq"
if OS.linux?
  # Checking for linux is a poor proxy for if we're in a gitpod.
  # We don't need these deps outside of gitpods, and they take a long time to install.
  brew "hashicorp/tap/consul"
  brew "sops" # On laptops, this is installed in eng-setup
  brew "age"
  brew "less"
  brew "openssh"
  brew "vim"
end
if OS.mac?
  # for syncing with Fullstacks, and also used by Jest
  brew "watchman"
  # file handler defaults
  brew "duti"
  # In the container this is installed elsewhere
  brew "composer"
  # Node is installed elsewhere in the container
  brew "node-build"
end