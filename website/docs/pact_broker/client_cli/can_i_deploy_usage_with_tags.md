---
title: Using can-i-deploy with tags
custom_edit_url: https://github.com/pact-foundation/pact_broker-client/edit/master/doc/CAN_I_DEPLOY_USAGE_WITH_TAGS.md
---
<!-- This file has been synced from the pact-foundation/pact_broker-client repository. Please do not edit it directly. The URL of the source file can be found in the custom_edit_url value above -->

The can-i-deploy tool was originally written to support specifying versions and dependencies using tags. This usage has now been superseded by first class support for [environments, deployments and releases](https://docs.pact.io/pact_broker/recording_deployments_and_releases/).

The following documentation covers the usage of can-i-deploy for users who are still using tags to track environments and deployments.

There are two ways to use `can-i-deploy` with tags. The first (recommended and most commonly used) approach is to specify just the application version you want to deploy and let the Pact Broker work out the dependencies for you. The second approach is to specify each application version explicitly. This would generally only be used if there were limitations that stopped you being able to use the first approach.

#### Specifying an application version

To specify an application (pacticipant) version you need to provide:

* the name of the application using the `--pacticipant PACTICIPANT` parameter,
* directly followed by *one* of the following parameters:
    * `--version VERSION` to specify a known application version (recommended)
    * `--latest` to specify the latest version
    * `--latest TAG` to specify the latest version that has a particular tag
    * `--all TAG` to specify all the versions that have a particular tag (eg. "all prod" versions). This would be used when ensuring you have backwards compatiblity with all production mobile clients for a provider. Note, when using this option, you need to specify dependency explicitly (see the second usage option).

Using a specific version is the easiest way to ensure you get an accurate response that won't be affected by race conditions.

#### Recommended usage - allowing the Pact Broker to automatically determine the dependencies

Prerequisite: if you would like the Pact Broker to calculate the dependencies for you when you want to deploy an application into a given environment, you will need to let the Broker know which version of each application is in that environment.

How you do this depends on the version of the Pact Broker you are running.

If you are using a Broker version where deployment versions are supported, then you would notify the Broker of the deployment of this application version like so:

    $ pact-broker record-deployment --pacticipant Foo --version 173153ae0 --environment test

This assumes that you have already set up an environment named "test" in the Broker.

If you are using a Broker version that does not support deployment environments, then you will need to use tags to notify the broker of the deployment of this application version, like so:

    $ pact-broker create-version-tag --pacticipant Foo --version 173153ae0 --tag test

Once you have configured your build to notify the Pact Broker of the successful deployment using either method describe above, you can use the following simple command to find out if you are safe to deploy (use either `--to` or  `--to-environment` as supported):

    $ pact-broker can-i-deploy --pacticipant PACTICIPANT --version VERSION
                               [--to-environment ENVIRONMENT | --to ENVIRONMENT_TAG ]
                               --broker-base-url BROKER_BASE_URL

If the `--to` or `--to-environment` options are omitted, then the query will return the compatiblity with the overall latest version of each of the other applications.

Examples:


Can I deploy version 173153ae0 of application Foo to the test environment?


    $ pact-broker can-i-deploy --pacticipant Foo --version 173153ae0 \
                               --to-environment test \
                               --broker-base-url https://my-pact-broker


Can I deploy the latest version of application Foo with the latest version of each of the applications it integrates to?


    $ pact-broker can-i-deploy --pacticipant Foo --latest \
                               --broker-base-url https://my-pact-broker


Can I deploy the latest version of the application Foo that has the tag "test" to the "prod" environment?

    $ pact-broker can-i-deploy --pacticipant Foo --latest test \
                               --to prod \
                               --broker-base-url https://my-pact-broker


#### Alternate usage - specifying dependencies explicitly

If you are unable to use tags, or there is some other limitation that stops you from using the recommended approach, you can specify one or more of the dependencies explictly. You must also do this if you want to use the `--all TAG` option for any of the pacticipants.

You can specify as many application versions as you like, and you can even specify multiple versions of the same application (repeat the `--pacticipant` name and supply a different version.)

You can use explictly declared dependencies with or without the `--to ENVIRONMENT_TAG`. For example, if you declare two (or more) application versions with no `--to ENVIRONMENT_TAG`, then only the applications you specify will be taken into account when determining if it is safe to deploy. If you declare two (or more) application versions _as well as_ a `--to ENVIRONMENT`, then the Pact Broker will work out what integrations your declared applications will have in that environment when determining if it safe to deploy. When using this script for a production release, and you are using tags, it is always the most future-proof option to use the `--to` if possible, as it will catch any newly added consumers or providers.

If you are finding that your dependencies are not being automatically included when you supply multiple pacticipant versions, please upgrade to the latest version of the Pact Broker, as this is a more recently added feature.


    $ pact-broker can-i-deploy --pacticipant PACTICIPANT_1 [--version VERSION_1 | --latest [TAG_1] | --all TAG_1] \
                               --pacticipant PACTICIPANT_2 [--version VERSION_2 | --latest [TAG_2] | --all TAG_2] \
                               [--to-environment ENVIRONMENT | --to ENVIRONMENT_TAG]

Examples:


Can I deploy version Foo version 173153ae0 and Bar version ac23df1e8 together?


    $ pact-broker can-i-deploy --pacticipant Foo --version 173153ae0 \
                               --pacticipant Bar --version ac23df1e8


Can I deploy the latest version of Foo with tag "master" and the latest version of Bar with tag "master" together?

    $ pact-broker can-i-deploy --pacticipant Foo --latest master \
                               --pacticipant Bar --latest master


Mobile provider use case - can I deploy version b80e7b1b of Bar, all versions of Foo with tag "prod", and the latest version tagged "prod" of any other automatically calculated dependencies together? (Eg. where Bar is a provider and Foo is a mobile consumer with multiple versions in production, and Bar also has its own providers it needs to be compatible with.)


    $ pact-broker can-i-deploy --pacticipant Bar --version b80e7b1b \
                               --pacticipant Foo --all prod \
                               --to prod
