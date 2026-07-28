---
trigger: always_on
description: The *agent* is the part of Fabric8 that is responsible for applying the profiles to containers. The *agent* can be run into any container and its role is to retrieve profile information from the registry and apply them locally.
---

## Agent

The *agent* is the part of Fabric8 that is responsible for applying the profiles to containers. The *agent* can be run into any container and its role is to retrieve profile information from the registry and apply them locally.
In more details, the *agent* will retrieve the profiles and versions that are assigned to the container on which its running, reconfigure the container, calculate what needs to be installed, removed or updated on the container and finally perform those task.

In reality there are two modules involved in the process:

* **fabric-configadmin** The configuration admin bridge. Translates the registry information into configuration.
* **fabric-agent** The deployment agent. Reads the translated configuration and provisions the container accordingly.

Often the term *agent* refers to just the deployment agent (fabric-agent module), but this section of the documentation will cover both and provide detailed information about the role of each one of them.

### The configuration admin bridge (fabric-configadmin)

The configuration admin bridge is responsible for bridging the zookeeper registry with the configuration admin service.
Once it connects to the zookeeper registry it will determine what version is assigned to the container, retrieve the version of profiles that are assigned to the container and translate them into configuration, which is then applied locally.

Note, that the profile contain two types of information:

* **Configuration information**
  * **System configuration**
  * **OSGi configuration**
* **Provisioning information**
  * **Bundles**
  * **Features**
  * **Fabs**

The configuration admin bridge will read all of them and create the OSGi configuration that represents them.
All the provisioning & system information will go under the **io.fabric8.agent** pid. For containers that are assigned multiple profiles, or just use the profile hierarchy the overlay view of the profiles will be added to io.fabric8.agent *(There is a single pid even if we have multiple profiles)*.

All the OSGi configuration will just use the pid and key value pairs that have been added to the profile.

The bridge will also watch the registry for changes, so any change in the profiles that are assigned to the container will be *tracked* and will immediately be applied to the local OSGi configuration of the container.

### The deployment agent (fabric-agent)

The deployment agent is listening for local configuration changes on the io.fabric8.agent pid. Any change in that configuration will trigger the *deployment agent*.
Once the *deployment agent* is triggered, it will read the whole **io.fabric8.agent** pid and calculate the bundles that the container should have installed.
If the profiles assigned to the container also contain features or fabs, the deployment agent will translate them to bundles in order to populate a full list of bundles that the container should use.
The next action of the *deployment agent* is to compare that list of bundles, with the list of bundles that are currently installed, in order to identify:

* **Bundles to Uninstall**
* **Bundles to Install**
* **Bundles to Update**

Finally, it downloads all the missing parts and perform the installation / uninstallation of bundles.

### How the deployment agent downloads artifacts

The *deployment agent* is capable of downloading artifacts from two different types of maven repositories:

* **Registered Fabric Maven Proxies** More information on [fabric maven proxies](fabric-maven-proxy.html).
* **Configured Maven repositories** Any maven repository that is configured on the profile.

Priority is always give to the [fabric maven proxies](fabric-maven-proxy.html). If more than one is registered in the cluster they will be used in order from the oldest to the newest.

If the target artifact is not found, then the configured maven repositories will be used. The list of repositories can be retrieved from the *org.ops4j.pax.url.mvn.repositories* property of the **io.fabric8.agent** pid.

To change that list of repositories for a specific profile you can simply, you the [fabric:profile-edit](commands/fabric-profile-edit.html) command:

      fabric:profile-edit --pid io.fabric8.agent/org.ops4j.pax.url.mvn.repositories=http://repositorymanager.mylocalnetwork.net default

It is suggested to keep this configuration in one profile and have the rest of profiles inherit from it. The default profile, which is the root of all of the default profiles, seems the ideal place.

### Configuring the download thread pool

By default the *deployment agent* uses a thread pool for concurrent download of needed artifcats. The default size of the pool is 2 which you can reconfigure by configuring the option with key `io.fabric8.agent.download.threads` in the `etc\custom.properties` file. 

For example to use 5 threads:

    io.fabric8.agent.download.threads=5


### No container restarts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jboss-fuse/fabric8](https://github.com/jboss-fuse/fabric8) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
