# Changelog
    
All notable changes to the project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]


## [0.2.2] - 2025/11/29 18:25:03

### Added

- allowed-domains.txt example


## [0.2.1] - 2025/11/29 18:23:38

### Changed

- init-firewall.sh no longer crashes out when it can't find an ip address, it will print which ip-addresses failed to load so the user is aware of which domains could not be added to the allowed list.
- type in the comments of the docker file
- readme.md with info on how to add more domains to the whitelist.


## [0.2.0] - 2025/11/06 18:52:10

### Added

- Added support for passing an allowed-domains.txt file for exposing ip addresses that the container can use.


## [0.1.0] - 2025/11/05 15:47:25

### Added

- initial version of the open-code dev container


