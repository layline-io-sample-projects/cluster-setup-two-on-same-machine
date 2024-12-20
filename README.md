# Sample configuration of a Reactive Cluster running with two Reactive Engines on the same machine

In this repo we demonstrate an example on how to set up a Reactive Cluster with two nodes.
This demo requires access to a database. The example is configured to access a mysql database.

## Prerequisites
In order to achieve the multi-node setup in this example we require the following:

1. The repo only contains the config files stored for the user under which layline.io has been installed. The actual runnables are not part of this repo.
2. To obtain a version of layline.io please visit https://layline.io and download the version matching your hardware achitecture prior to applying this setup to your installation.
3. In order to setup a running multi-node cluster and a configuration server we have the following directories:
   - .layline/config-server/config -> This contains the `application.conf` file for the configuration server
   - .layline/reactive-engine/config -> This contains the `application.conf` file for the FIRST reactive engine
   - .layline/reactive-engine-2/config -> This contains the `application.conf` file for the SECOND reactive engine
   The secret to establishing the cluster is in the settings of each `application.conf` file of the reactive engines.
   We are describing them below, but you should check out the files in the repo to understand them in detail.

Please note, that all relevant cluster settings can be reviewed at the [pekko docoumentation website](https://pekko.apache.org/docs/pekko/1.1/general/configuration.html) for furthe reference.

## Important files in the repo

### Configuration Server
Check the file `.layline/config-server/config/application.conf`.

This file, `application.conf`, is a configuration file for the Layline config server and Pekko persistence settings. Here's a breakdown of its contents:

- **Layline Config Server**:
  - **REST server** binding hostname is set to `0.0.0.0`, allowing it to listen on all network interfaces.

- **Pekko Persistence**:
  - **Journal Plugin**: Uses `jdbc-journal`.
  - **Snapshot Store Plugin**: Uses `jdbc-snapshot-store`.

- **Pekko Persistence JDBC**:
  - **Shared Databases** configuration for `h2file`:
    - Database profile: `slick.jdbc.H2Profile$`.
    - Database URL: Specifies the H2 database file and initialization script.
    - User and password for the database: `root`.
    - JDBC driver: `org.h2.Driver`.
    - Connection pool settings: Number of threads, max, and min connections.

- **JDBC Journal, Read Journal, and Snapshot Store**:
  - All use the shared database configured as `h2file`.

You can view the file [here](https://github.com/layline-io-sample-projects/cluster-setup-two-on-same-machine/blob/9ba241df0085f113a656c61b4487f26d33295668/.layline/config-server/config/application.conf).

### Reactive Engine #1

This file, [application.conf](https://github.com/layline-io-sample-projects/cluster-setup-two-on-same-machine/blob/9ba241df0085f113a656c61b4487f26d33295668/.layline/reactive-engine/config/application.conf), is a configuration file for the Layline reactive engine and Pekko settings. Here's a breakdown of its contents:

- **Layline Reactive Engine**:
  - **REST server**:
    - Canonical hostname: `ubuntu.orb.local` -> In our example, this is how the reactive engine can be reached. Replace with the canonical name of your setup, or remove.
    - Bind port: `5842` -> The port under which this node can be reached
    - Bind hostname: `0.0.0.0` -> Configured to be reached by all networks

- **Pekko**:
  - **Actor provider**: Cluster
  - **Management HTTP**:
    - Bind port: `8558`
  - **Remote**:
    - Lifecycle events logging: Off
    - Transport: TCP
    - Canonical hostname: `127.0.0.1` 
    - Canonical port: `5843` -> Unique canonical port under which this node can be reached in the cluster
    - Bind hostname: `0.0.0.0` -> Allow all network access
    - Bind port: `5843` -> Unique bind port under which this node can be reached in the cluster. This must be different for each running node.
  - **Cluster**:
    - Minimum number of members: `2` -> This is an example with two nodes in one cluster. The cluster can only be established when two nodes are running initially.
    - Seed nodes: `127.0.0.1:5843` and `127.0.0.1:6843` -> These are the two nodes which are part of the cluster.
    - Split-brain resolver: Keep majority strategy

- **Pekko Persistence**:
  - **Journal plugin**: `jdbc-journal`
  - **Snapshot store plugin**: `jdbc-snapshot-store`

- **Pekko Persistence JDBC**:
  - **Shared databases**:
    - **MySQL**:
      - Host: `localhost`
      - Port: `3306`
      - URL: JDBC connection URL -> Set your mysql connection string
      - User: `root`
      - Password: Empty
      - Driver: `com.mysql.cj.jdbc.Driver`
      - Connection pool settings
    - **H2 file**: -> This setting is the default and not used in the example. In a multi-node cluster you must use a shareable database like mysql, mongo, cassandra, or other.
      
- **JDBC Journal, Read Journal, and Snapshot Store**:
  - All use the shared database configured as `mysql`.

### Reactive Engine #2

This file, [application.conf](https://github.com/layline-io-sample-projects/cluster-setup-two-on-same-machine/blob/9ba241df0085f113a656c61b4487f26d33295668/.layline/reactive-engine-2/config/application.conf), is a configuration file for the second Layline reactive engine. 
Here's a breakdown of its contents:

- **Layline Reactive Engine**:
  - **REST server**:
    - Canonical hostname: `ubuntu.orb.local` -> In our example, this is how the reactive engine can be reached. Replace with the canonical name of your setup, or remove.
    - Bind port: `6842` -> The port under which this node can be reached
    - Bind hostname: `0.0.0.0` -> Configured to be reached by all networks

- **Pekko**:
  - **Actor provider**: Cluster
  - **Management HTTP**:
    - Bind port: `9558`
  - **Remote**:
    - Lifecycle events logging: Off
    - Transport: TCP
    - Canonical hostname: `127.0.0.1`
    - Canonical port: `6843` -> Unique canonical port under which this node can be reached in the cluster
    - Bind hostname: `0.0.0.0` -> Allow all network access
    - Bind port: `6843`-> Unique bind port under which this node can be reached in the cluster. This must be different for each running node.
  - **Cluster**:
    - Minimum number of members: `2` -> This is an example with two nodes in one cluster. The cluster can only be established when two nodes are running initially.
    - Seed nodes: `127.0.0.1:5843` and `127.0.0.1:6843` -> These are the two nodes which are part of the cluster.
    - Split-brain resolver: Keep majority strategy

- **Pekko Persistence**:
  - **Journal plugin**: `jdbc-journal`
  - **Snapshot store plugin**: `jdbc-snapshot-store`

- **Pekko Persistence JDBC**:
  - **Shared databases**:
    - **MySQL**:
      - Host: `localhost`
      - Port: `3306`
      - URL: JDBC connection URL -> Set your mysql connection string
      - User: `root`
      - Password: Empty
      - Driver: `com.mysql.cj.jdbc.Driver`
      - Connection pool settings
    - **H2 file**: -> This setting is the default and not used in the example. In a multi-node cluster you must use a shareable database like mysql, mongo, cassandra, or other.

- **JDBC Journal, Read Journal, and Snapshot Store**:
  - All use the shared database configured as `mysql`.
 
## Starting the Reactive Cluster
(This assumes that upon installation of layline.io you have also installed the symbolic links to both `config-server` and `reactive-engine`. 
If this is not the case, then you need to invoke both the Configuration Server and Reactive Engine via their respective installation paths.)

Start the two Reactive Engines - which then form the Reactive Cluster - like so:

#### Starting the 1st Engine:

```bash
reactive-engine -J-Dlayline.configDirectory=/<username>/.layline/reactive-engine
```

This should give you the following output:

```bash
24-12-19 15:12:09.072 INFO  Layline                                            - [LAY-00050] ###################################################################
24-12-19 15:12:09.074 INFO  Layline                                            - [LAY-00050] #       Layline Reactive Engine 2.1.1 (2024-12-13 13:16:36)       #
24-12-19 15:12:09.074 INFO  Layline                                            - [LAY-00050] #                                                                 #
24-12-19 15:12:09.074 INFO  Layline                                            - [LAY-00050] #  Copyright (C) 2018-2024  layline.io GmbH <https://layline.io>  #
24-12-19 15:12:09.075 INFO  Layline                                            - [LAY-00050] ###################################################################
24-12-19 15:12:11.358 INFO  Layline                                            - [LAY-11000] trying to connect to the cluster
```

The engine is now waiting for the second engine to join the cluster.

#### Starting the 2nd Engine:

Open another terminal and enter the following:

```bash
reactive-engine -J-Dlayline.configDirectory=/<username>/.layline/reactive-engine-2
```

This should give you the following output:

```bash
24-12-19 15:11:57.466 INFO  Layline                                            - [LAY-00050] ###################################################################
24-12-19 15:11:57.468 INFO  Layline                                            - [LAY-00050] #       Layline Reactive Engine 2.1.1 (2024-12-13 13:16:36)       #
24-12-19 15:11:57.468 INFO  Layline                                            - [LAY-00050] #                                                                 #
24-12-19 15:11:57.469 INFO  Layline                                            - [LAY-00050] #  Copyright (C) 2018-2024  layline.io GmbH <https://layline.io>  #
24-12-19 15:11:57.469 INFO  Layline                                            - [LAY-00050] ###################################################################
24-12-19 15:11:59.640 INFO  Layline                                            - [LAY-11000] trying to connect to the cluster
24-12-19 15:12:17.963 INFO  Layline                                            - [LAY-11001] cluster successfully joined, member status is up
24-12-19 15:12:18.284 INFO  Layline                                            - [LAY-11600] starting the activation controller
24-12-19 15:12:18.327 INFO  Layline.SchedulerSlave                             - [LAY-12000] cluster node local scheduler slave started
24-12-19 15:12:18.367 INFO  Layline.Format                                     - [LAY-15000] creating the format controller
24-12-19 15:12:18.371 INFO  Layline.Resource                                   - [LAY-02650] starting the resource controller
24-12-19 15:12:18.373 INFO  Layline.Connection                                 - [LAY-17500] starting the connection controller
24-12-19 15:12:18.381 INFO  Layline.Service                                    - [LAY-20000] creating the service controller
24-12-19 15:12:18.383 INFO  Layline.Extension                                  - [LAY-45000] starting the extension controller
24-12-19 15:12:18.388 INFO  Layline.Source                                     - [LAY-25000] starting the source controller
24-12-19 15:12:18.396 INFO  Layline.Sink                                       - [LAY-30000] starting the sink controller
24-12-19 15:12:18.404 INFO  Layline.Workflow                                   - [LAY-04000] starting the workflow controller
24-12-19 15:12:18.707 INFO  Layline.SchedulerSlave                             - [LAY-12003] successfully registered at master on node pekko://layline@127.0.0.1:5843
24-12-19 15:12:18.730 INFO  Layline.ActivationController                       - [LAY-11611] new activation target 'DeploymentRoot' received
24-12-19 15:12:18.885 INFO  Layline                                            - [LAY-11603] received data of deployment 'DeploymentRoot' from the deployment storage
24-12-19 15:12:18.888 INFO  Layline.ActivationController                       - [LAY-11612] switching to activation 'DeploymentRoot'
24-12-19 15:12:18.994 INFO  Layline.RestServer                                 - [LAY-00360] rest server is listening on address http://0.0.0.0:6842
24-12-19 15:12:45.038 INFO  Layline.DeploymentStorage                          - [LAY-11500] deployment storage created
24-12-19 15:12:45.043 INFO  Layline.DeploymentStorage.Objects                  - [LAY-11502] starting the object database
24-12-19 15:12:45.077 INFO  Layline.AccessCoordinator.Resource                 - [LAY-11737] recovery of resources coordinator completed
24-12-19 15:12:45.080 INFO  Layline.AccessCoordinator.Source                   - [LAY-11703] recovery of sources coordinator completed
24-12-19 15:12:45.082 INFO  Layline.SnifferDirectory                           - [LAY-12403] recovery of the sniffer directory completed (0 storages used)
24-12-19 15:12:45.105 INFO  Layline.AiStorage                                  - [LAY-12505] recovery of the AI model database completed
24-12-19 15:12:45.106 INFO  Layline.SecurityStorage                            - [LAY-12248] recovery of the oauth storage completed (0 token(s) loaded)
24-12-19 15:12:45.114 INFO  Layline.DeploymentStorage.Objects                  - [LAY-11505] recovery of the object database completed
24-12-19 15:12:45.223 INFO  Layline.SecurityStorage                            - [LAY-12203] recovery of the security storage completed (1 RSA key(s) loaded)
24-12-19 15:12:45.225 INFO  Layline.UserStorage                                - [LAY-12303] recovery of the user storage completed (1 user(s) loaded)
24-12-19 15:12:45.621 INFO  Layline.DeploymentStorage.Deployments              - [LAY-11507] starting the deployment database
24-12-19 15:12:45.632 INFO  Layline.DeploymentStorage.Deployments              - [LAY-11510] recovery of the deployment database completed
```

You should now also observe that the engine in the terminal where you started the first engine should have joined the cluster. 
In case the engines cannot find each other due to network or configuration issues, they will be stuck at `trying to connect to the cluster`. 
In that case check your settings carefully.

## Starting the Configuration Server

Note: The Configuration Server can be started independently of the Reactive Cluster.

Open another terminal and enter the following:

```bash
config-server
```

This should give you the following output:

```bash
24-12-19 15:02:28.964 INFO  ConfigServer                         - [LAY-00050] ###################################################################
24-12-19 15:02:28.975 INFO  ConfigServer                         - [LAY-00050] #        Layline Config Server 2.1.1 (2024-12-13 13:16:36)        #
24-12-19 15:02:28.975 INFO  ConfigServer                         - [LAY-00050] #                                                                 #
24-12-19 15:02:28.975 INFO  ConfigServer                         - [LAY-00050] #  Copyright (C) 2018-2024  layline.io GmbH <https://layline.io>  #
24-12-19 15:02:28.975 INFO  ConfigServer                         - [LAY-00050] ###################################################################
24-12-19 15:02:30.310 INFO  ConfigServer.SecurityStorage         - [LAY-12200] starting the security storage
24-12-19 15:02:30.421 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 's3' ('io.layline.config.plugin.common.s3.PluginFactory')
24-12-19 15:02:30.424 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'ftp' ('io.layline.config.plugin.common.ftp.FtpPluginFactory')
24-12-19 15:02:30.426 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'processor_mapping' ('io.layline.config.plugin.processor.mapping.PluginFactory')
24-12-19 15:02:30.428 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'event_bridge' ('io.layline.config.plugin.common.event_bridge.PluginFactory')
24-12-19 15:02:30.430 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'smb' ('io.layline.config.plugin.common.smb.SmbPluginFactory')
24-12-19 15:02:30.430 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'google_cloud' ('io.layline.config.plugin.common.gc.PluginFactory')
24-12-19 15:02:30.435 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'webdav' ('io.layline.config.plugin.common.webdav.WebdavPluginFactory')
24-12-19 15:02:30.440 INFO  ConfigServer                         - [LAY-10016] starting the configuration server root actor
24-12-19 15:02:30.471 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'format_asn1' ('io.layline.config.plugin.format.asn1.PluginFactory')
24-12-19 15:02:30.475 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'format_generic' ('io.layline.config.plugin.format.generic.PluginFactory')
24-12-19 15:02:30.477 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'kinesis' ('io.layline.config.plugin.common.kinesis.PluginFactory')
24-12-19 15:02:30.479 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'sqs' ('io.layline.config.plugin.common.sqs.PluginFactory')
24-12-19 15:02:30.482 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'xml' ('io.layline.config.plugin.common.xml.PluginFactory')
24-12-19 15:02:30.484 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'hazelcast' ('io.layline.config.plugin.common.hazelcast.PluginFactory')
24-12-19 15:02:30.486 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'kafka' ('io.layline.config.plugin.common.kafka.KafkaPluginFactory')
24-12-19 15:02:30.488 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'sns' ('io.layline.config.plugin.common.sns.PluginFactory')
24-12-19 15:02:30.490 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'aws' ('io.layline.config.plugin.common.aws.PluginFactory')
24-12-19 15:02:30.492 INFO  ConfigServer.Plugins                 - [LAY-00321] successfully registered plugin 'soap' ('io.layline.config.plugin.common.soap.PluginFactory')
24-12-19 15:02:30.791 INFO  ConfigServer.RestServer              - [LAY-00360] rest server is listening on address http://0.0.0.0:5841
24-12-19 15:02:30.792 INFO  ConfigServer                         - [LAY-10018] layline configuration server is up and running ...
24-12-19 15:02:31.002 INFO  ConfigServer.SecurityStorage         - [LAY-12248] recovery of the oauth storage completed (0 token(s) loaded)
24-12-19 15:02:31.055 INFO  ConfigServer.SecurityStorage         - [LAY-12203] recovery of the security storage completed (1 RSA key(s) loaded)
24-12-19 15:02:31.075 INFO  ConfigServer.UserStorage             - [LAY-12303] recovery of the user storage completed (1 user(s) loaded)
```

Make sure that it outputs `rest server is listening on address http://0.0.0.0:5841` so that it excepts external connections and you can reach it via brrowser from outside the machine.

## Invoking the Configuration Server via browser

Open a browser and point it to `http://<ip-address / url>:5841`. In our example this is `http://ubuntu.orb.local:5841'.

This should give you the following output:

![image](https://github.com/user-attachments/assets/4e0e1dd4-eaf3-4801-97a9-58fdc6b654a8)

## Check that the cluster connection works and the cluster is running

#### Log in to the Configuration Server

Enter `admin/admin` for user name and password.

#### Add a cluster setting

Head to the Settings tab to add a cluster setting.
Follow the numbers in the following image to create the setting:

![2024-12-20 at 12 12 05](https://github.com/user-attachments/assets/c71574b2-01c7-481e-af8d-323bb9fc462d)

![2024-12-20 at 11 59 28](https://github.com/user-attachments/assets/69528253-276f-4087-b481-e23110f2f72b)

#### Check that the Reactive Cluster is reachable and runs

Now that we have created the cluster setting, let's check that we can reach it and it runs.
For this, go to the Operations tab and select the Cluster we just added from the drop-down menu:

![2024-12-20 at 12 13 57](https://github.com/user-attachments/assets/ddff9340-cecf-4f5c-9bfd-29d75a73b2bd)

If everything is configured correctly, a login dialogue should appead. At the bottomg of it you should see a green `Cluster available` hint.
If you don't see this, then the Cluster cannot be reached under the bootstrap node you configured.

Enter `admin/admin` as user name and password to log in to the cluster.

The scren should now display the following:

![2024-12-20 at 12 17 04](https://github.com/user-attachments/assets/8bb2f412-5cda-4ab0-8f6c-8d723841e4e2)

You should see the two Reactive Engines in the Cluster. If you select one of them you can see it's state and technical parameters.

---

That's it!

Should you run into any issues, feel free to contact our [support](mailto:support@layline.io).
We are more than happy to get your environment up and running.

Also check out out [website](https://layline.io) and [documentation](https://doc.layline.io).




