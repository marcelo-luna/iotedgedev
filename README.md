# Azure IoT Edge Dev Tool

[![PyPI version](https://badge.fury.io/py/iotedgedev.svg)](https://badge.fury.io/py/iotedgedev)
[![Build Status](https://dev.azure.com/Azure-IoT-DDE-EdgeExperience/IoTEdgeDev/_apis/build/status/Azure.iotedgedev?branchName=main)](https://dev.azure.com/Azure-IoT-DDE-EdgeExperience/IoTEdgeDev/_build/latest?definitionId=35&branchName=main)

The **IoT Edge Dev Tool** greatly simplifies [Azure IoT Edge](https://azure.microsoft.com/en-us/services/iot-edge/) development down to simple commands driven by environment variables.

- It gets you started with IoT Edge development with the [IoT Edge Dev Container](https://hub.docker.com/r/microsoft/iotedgedev/) and IoT Edge solution scaffolding that contains a default module and all the required configuration files.
- It speeds up your inner-loop dev (dev, debug, test) by reducing multi-step build & deploy processes into one-line CLI commands as well as drives your outer-loop CI/CD pipeline. _You can use all the same commands in both stages of your development life-cycle._

## Overview

For the absolute fastest way to get started with IoT Edge Dev, please see the [Quickstart](https://github.com/Azure/iotedgedev/wiki/quickstart) section below.

For a more detailed overview of IoT Edge Dev Tool including setup, commands and troubleshooting, please see the [Wiki](https://github.com/Azure/iotedgedev/wiki).

## Data/Telemetry

This project collects usage data and sends it to Microsoft to help improve our products and services. Read our [privacy statement](http://go.microsoft.com/fwlink/?LinkId=521839) to learn more.
If you don’t wish to send usage data to Microsoft, you can change your telemetry settings by updating `collect_telemetry` to `no` in `~/.iotedgedev/settings.ini`.

## Contributing

This project welcomes contributions and suggestions. Please refer to the [Contributing file](CONTRIBUTING.md) for details on contributing changes.

Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to,
and actually do, grant us the rights to use your contribution. For details, visit
<https://cla.microsoft.com>.

When you submit a pull request, a CLA-bot will automatically determine whether you need
to provide a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the
instructions provided by the bot. You will only need to do this once across all repositories using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/)
or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Support

The team monitors the issue section on regular basis and will try to assist with troubleshooting or questions related IoT Edge tools on a best effort basis.

A few tips before opening an issue. Try to generalize the problem as much as possible. Examples include:

- Removing 3rd party components
- Reproduce the issue with provided deployment manifest used
- Specify whether issue is reproducible on physical device or simulated device or both
Also, Consider consulting on the [docker docs channel](https://github.com/docker/docker.github.io) for general docker questions.

## Quickstart

This quickstart will run a container, create a solution, setup Azure resources, build and deploy modules to your device, setup and start the IoT Edge simulator, monitor messages flowing into IoT Hub, and finally deploy to the IoT Edge runtime.

At the end of this quickstart, you should have the following:

- Development environment.
- IoT Hub deployed in your Azure subscription.
- A directory with the [IoT edge deployment manifest](https://docs.microsoft.com/en-us/azure/iot-edge/module-composition?view=iotedge-2020-11), [.env file](.env.tmp) with the environment variables needed for the `iotedgedev` tool, and the source file of a custom IoT edge module (`filtermodule`).
- The docker image of the `filtermodule`.
- Starting the IoT Edge simulator, which runs the containers defined in the device manifest.
- Monitoring the messages sent to the IoT Hub.
- Cleaning up the docker containers and images.

### Pre-requisites

- Docker (see [Install Docker docs](./docs/Install-Docker.md) for details).

### Quickstart steps

1. Setup development environment

    There are three options to setup your development environment:

    - Setup the development environment manually. Please follow the [Manual Development Machine Setup Wiki](https://github.com/Azure/iotedgedev/wiki/manual-dev-machine-setup).
    - Starting the devcontainer in VS Code (see [Developing inside a Container](https://code.visualstudio.com/docs/remote/containers) for steps on how to do so).
    - Starting the devcontainer with Docker. Please follow the [Run the IoT Edge Dev Container with Docker docs](./docs/Run-Devcontainer-Docker.md).
  
2. Create empty directory for the IoT Edge solution resources.

   ```sh
   mkdir -p iotedge
   cd iotedge
   ```

3. Initialize IoT Edge solution and setup Azure resources.

    ```sh
    iotedgedev init
    ```

    > `iotedgedev init` will create a solution and setup your Azure IoT Hub in a single command. The solution comes with a default C# module named `filtermodule`.

    <details>
    <summary>More information</summary>

    1. You will see structure of current folder like below:

    ```tree
        │  .env
        │  .gitignore
        │  deployment.debug.template.json
        │  deployment.template.json
        │
        ├─.vscode
        │      launch.json
        │
        └─modules
            └─filtermodule
                │  .gitignore
                │  Dockerfile.amd64
                │  Dockerfile.amd64.debug
                │  Dockerfile.arm32v7
                │  Dockerfile.windows-amd64
                │  filtermodule.csproj
                │  module.json
                │  Program.cs
    ```

    1. Open `.env` file, you will see the `IOTHUB_CONNECTION_STRING` and `DEVICE_CONNECTION_STRING` environment variables filled correctly.
    2. Open `deployment.template.json` file.
        1. You will see below section in the modules section:

        ```json
        "filtermodule": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
                "image": "${MODULES.filtermodule}",
                "createOptions": {}
            }
        }
        ```

        1. Two default routes are added:

        ```json
        "routes": {
            "sensorTofiltermodule": "FROM /messages/modules/tempSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/filtermodule/inputs/input1\")",
            "filtermoduleToIoTHub": "FROM /messages/modules/filtermodule/outputs/* INTO $upstream"
        }
        ```

    3. You will see privacy statement like below:

        ```txt
        Welcome to iotedgedev!
        -------------------------
        Telemetry
        ---------
        The iotedgedev collects usage data in order to improve your experience.
        The data is anonymous and does not include commandline argument values.
        The data is collected by Microsoft.
        
        You can change your telemetry settings by updating 'collect_telemetry' to 'no' in ~/.iotedgedev/setting.ini
        ```

    </details>

4. Build IoT Edge module images.

    ```sh
    iotedgedev build
    ```

    > This step will build user modules in deployment.template.json targeting amd64 platform.

    <details>
    <summary>More information</summary>

    1. You will see a "BUILD COMPLETE" for each module and no error messages in the terminal output.
    2. Open `config/deployment.amd64.json` file, you will see the module image placeholders expanded correctly.
    3. Run `sudo docker image ls`, you will see the module images you just built.

    </details>

5. Setup and start the IoT Edge Simulator to run the solution

    ```sh
    iotedgedev start --setup --file config/deployment.amd64.json
    ```

    > The [IoT Edge Hub Simulator](https://github.com/Azure/iotedgehubdev) starts the containers defined in the IoT edge device manifest in your local machine.

    <details>
    <summary>More information</summary>

    1. You will see an "IoT Edge Simulator has been started in solution mode." message at the end of the terminal output
    2. Run `sudo docker ps`, you will see your modules running as well as an edgeHubDev container

    </details>

6. Monitor messages sent from IoT Edge Simulator to IoT Hub.

    ```sh
    iotedgedev monitor
    ```

    <details>
    <summary>More information</summary>

    1. You will see your expected messages sending to IoT Hub
    2. Stopping the monitor doesn't stop the simulator. It will continue running until it is explicitely stopped using `iotedgedev stop` and at that time all containers used by the simulator will be cleaned up.

    </details>

7. Stop docker containers and remove images.

    ```sh
    # stop containers started by the simulator
    iotedgedev stop
    # remove stopped containers
    iotedgedev docker clean -c
    # remove docker images
    iotedgedev docker clean -
    ```

## Additional Resources

Please refer to the [Wiki](https://github.com/Azure/iotedgedev/wiki) for details on setup, usage, and troubleshooting.
