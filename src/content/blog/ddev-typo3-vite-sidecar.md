---
title: "DDEV, TYPO3 and Vite: Easy integration with ddev-vite-sidecar"
pubDate: 2024-11-15
summary: How to develop for TYPO3 using Vite for hot reloading and bundling
author: Patrick Dehne
featureImage:
  src: /img/blog/2024/08/camera-and-laptop.jpeg
  alt: "Microsoft image creator: camera and laptop"
  credit: "Microsoft image creator: camera and laptop"
categories:
  - Guides
---

[Vite](https://vite.dev/) is a popular tool for frontend development. The [ddev-vite-sidecar addon](https://github.com/s2b/ddev-vite-sidecar) together with the [Vite AssetCollector for TYPO3](https://github.com/s2b/vite-asset-collector) and the [TYPO3 Vite Plugin](https://github.com/s2b/vite-plugin-typo3) can be used to seamlessly integrate Vite with DDEV. Together they provide a modern and efficient environment for rapid TYPO3 frontend development.

Let's first setup the DDEV environment for a basic TYPO3 project:

```bash
mkdir t3example
cd t3example
ddev config --project-type=typo3 --docroot=public --php-version 8.3 --timezone Europe/Berlin --web-environment="TYPO3_CONTEXT=Development" --webserver-type apache-fpm --router-http-port=8080 --router-https-port=8443
```

The ddev-vite-sidecar addon will host the Vite development server. In this case we use npm for running Vite. Please check the [addon documentation](https://github.com/s2b/ddev-vite-sidecar?tab=readme-ov-file#get-started) for other supported package managers.

```bash
VITE_PACKAGE_MANAGER=npm ddev add-on get s2b/ddev-vite-sidecar
```

The ddev-vite-sidecar addon needs to use the same HTTPS port as TYPO3 itself. To configure this we add the HTTPS port to the VITE_SERVER_URI environment variable in the addon configuration file `.ddev/config.vite.yaml`. For now we also need to remove the `#ddev-generated` comment or DDEV will overwrite the file on configuration changes.

After our changes it should look similiar to this:

```
additional_hostnames:
  - vite.t3-test
web_environment:
  - VITE_SERVER_URI=https://vite.t3-test.ddev.site:8443
  - VITE_PACKAGE_MANAGER=npm
```

Now we can start the DDEV environment:

```bash
ddev start
```

Next we need to install the TYPO3 distribution itself. To integrate Vite with TYPO3 we use the [Vite AssetCollector for TYPO3](https://github.com/s2b/vite-asset-collector) plugin. The Vite AssetCollect provides ViewHelpers to TYPO3, to embed assets from Vite into TYPO3.

```bash
ddev composer create --no-install "typo3/cms-base-distribution:^13.4"
ddev composer req "praetorius/vite-asset-collector"
```

Finally let's run the TYPO setup and restart DDEV:

```bash
ddev typo3 setup --server-type=apache --driver=mysqli --host=db --port=3306 --dbname=db --username=db --password=db --project-name="T3 Test" --admin-username=admin --admin-user-password=admin

ddev restart
```

The output of `ddev restart` will show the URL where our TYPO3 installation can be reached, e.g. `https://t3test.ddev.site:8443`.

To configure Vite we create a file named `vite.config.js` in the root folder of our project.

