# DDEV-cypress <!-- omit in toc -->

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Steps](#steps)
  - [Configure `DISPLAY`](#configure-display)
    - [Windows 10](#windows-10)
      - [Running DDEV on Win10 (not WSL)](#running-ddev-on-win10-not-wsl)
  - [A note about the Cypress image](#a-note-about-the-cypress-image)
- [Commands](#commands)
  - [`cypress-open`](#cypress-open)
  - [`cypress-run`](#cypress-run)
- [Notes](#notes)
- [Troubleshooting](#troubleshooting)
  - ["Could not find a Cypress configuration file, exiting"](#could-not-find-a-cypress-configuration-file-exiting)
  - ["Unable to open X display."](#unable-to-open-x-display)
- [TODO](#todo)

## Introduction

[Cypress](https://www.cypress.io/) is a "complete end-to-end testing experience". It allows you to write Javascript test files that automate real browsers.  For more details, see the [Cypress Overview](https://docs.cypress.io/guides/overview/why-cypress) page.

This recipe integrates a Cypress docker image with your DDEV project.

## Requirements

- DDEV >= 1.19
- Windows or Linux

NOTE: This uses [cypress/include](https://hub.docker.com/r/cypress/included) which does not have arm64 images and therefore does **not** support M1 Macs.

## Steps

- Install service

  ```shell
  ddev get tyler36/ddev-cypress
  ddev restart
  ```

- Add a `./cypress.json` cypress configuration. Note: DDEV automatically sets the "BaseURL" via the image environmental variables. The `baseURL` setting below will be ignored.

```json
{
    "baseURL": "https://ddev-cypress-demo.ddev.site",
    "integrationFolder": "./tests/E2E",
}
```

- Run cypress via `cypress run` (headless) or `cypress open`.

### Configure `DISPLAY`

To correctly display the Cypress screen and browser output, you must configure a `DISPLAY` environment variable.

#### Windows 10

If you are running DDEV on Win10 or WSL2, you need to configure a display server on Win10.
You are free to use any X11-compatible server. A configuration-free solution is to install [GWSL via the Windows Store](https://www.microsoft.com/en-us/p/gwsl/9nl6kd1h33v3#activetab=pivot:overviewtab).

##### Running DDEV on Win10 (not WSL)

- Install [GWSL via the Windows Store](https://www.microsoft.com/en-us/p/gwsl/9nl6kd1h33v3#activetab=pivot:overviewtab)
- Get you "IPv4 Address" for your "Ethernet adapter" via networking panel or by typing `ipconfig` in a terminal. The address in the below example is `192.168.0.196`

```shell
❯ ipconfig

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 192.168.0.196
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.0.1
```

- In your project `./docker-compose.cypress.yaml`, add the IPv4 address and `:0` (EG. `192.168.0.196:0` ) to the display section under environment.

```yaml
    environment:
      - DISPLAY=192.168.0.196:0
```

### A note about the Cypress image

This recipe uses the latest `cypress/include` image which includes the following browsers:

- Chrome
- Firefox
- Electron

It is considered best practice to use a [specific image tag](https://github.com/cypress-io/cypress-docker-images#best-practice).

- If you require a specific browser, update `image` in your `./.ddev/docker-compose.cypress.yaml`.
- Available images and versions can be found on the [cypress-docker-images](https://github.com/cypress-io/cypress-docker-images) page.

## Commands

Cypress can run into 2 different modes: interactive and runner.
This recipe includes 2 alias commands to help you use Cypress.

### `cypress-open`

To open cypress in "interactive" mode, run the following command:

```shell
ddev cypress-open
```

This command also accepts arguments. Refer to the ["#cyress open" documentation](https://docs.cypress.io/guides/guides/command-line#cypress-open) for further details.

Example: To open Cypress in interactive mode, and specify a config file

```shell
ddev cypress-open --config cypress.json
```

### `cypress-run`

To run Cypress in "runner" mode, run the following command:

```shell
ddev cypress-run
```

This command also accepts arguments. Refer to [#cypress run](https://docs.cypress.io/guides/guides/command-line#cypress-run) page for a full list of available arguments.

Example: To run all Cypress tests, using Chrome in headless mode

```shell
ddev cypress-run --browser chrome
```

### `cypress-info`

Prints information about Cypress and the current environment.

```shell
ddev cypress-info
```

### `cypress-verify`

Verify that Cypress is installed correctly and is executable.

```shell
ddev cypress-verify
```

### `cypress-version`

Prints the installed Cypress binary version, the Cypress package version, the version of Electron used to build Cypress, and the bundled Node version.

```shell
ddev cypress-version
```

This command also accepts a signal command to check either the package, binary, browser or node component versions. Refer to [#cypress verify](https://docs.cypress.io/guides/guides/command-line#cypress-version) for more information.

Example: To print an individual component's version number

```shell
ddev cypress-version --component binary
```

### `ddev cypress-cache [command]`

Commands for managing the global Cypress cache[#cypress cache](https://docs.cypress.io/guides/guides/command-line#cypress-cache-command) for more information.


```shell
ddev cypress-cache path
```
Print the path to the Cypress cache folder.

```shell
ddev cypress-cache list 
```

Print all existing installed versions of Cypress.

```shell
ddev cypress-cache clear 
```
Clear the contents of the Cypress cache.

```shell
ddev cypress-cache prune 
```

Delete all installed Cypress versions from the cache except for the currently-installed version.

### Debugging commands

Any of the ddev-generated commands are modifiable to print debug information when the command is run.  You can also customize `ddev cypress-debug` to run any of the cli commands with debut information set.  Please note that the commands 
are different for Mac or Linux and Windows.

Mac or Linux - `DEBUG=cypress:* cypress open`
Windows
`set DEBUG=cypress:*`
`cypress open`

## Notes

- The dockerized Cypress *should* find any locally installed plugins in your project's `node_modules`; assuming they are install via npm or yarn.
- Some plugins may require additional settings, such as environmental variables. These can be passed through via command arguments.
- If you have additional hostnames and want to connect to any of these, you will need to explicitly list each as an additional line in the `external links:` stanza of the docker-composer.cypress.yaml file. 

## Troubleshooting

### "Could not find a Cypress configuration file, exiting"

Cypress expects a directory strutures containing the tests, plugins and support files.

- If the `./cypress` directory does not exist, it will scaffold out these directories, including a default `cypress.json` setting file and example tests when you first run `ddev cypress-open`.
- Make sure you have a `cypress.json` file in your project root, or use `--config [file]` argument to specify one.

### "Unable to open X display."

- This recipe forwards the Cypress GUI via an X11 / X410 server. Please ensure you have this working on your host system.
- For Windows 10 users, try [GWSL](https://opticos.github.io/gwsl/tutorials/manual.html) (via [Microsoft store](ms-windows-store://pdp/?productid=9NL6KD1H33V3)), or [VcXsrv](https://sourceforge.net/projects/vcxsrv/) (via [chocolatey](https://community.chocolatey.org/packages/vcxsrv#versionhistory))

## TODO

- [ ] Add arm64 / mac solution
- [ ] Add steps for intergrating into Github Actions

**Contributed by [@tyler36](https://github.com/tyler36)**
