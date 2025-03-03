[![PR Build](https://github.com/HewlettPackard/roven/actions/workflows/hybrid-pr-build.yaml/badge.svg)](https://github.com/HewlettPackard/roven/actions/workflows/hybrid-pr-build.yaml)

# Hybrid Node Attestor
The `hybrid` node attestor plugin for SPIRE is an external plugin, that combines the power of any built-in plugin supported by spire. With this approach you can use any combination of the built-in plugins in order to attest the node. For example, you can mix the k8s_psat and the aws_iid plugins to attest that the agent node is running on an AWS EKS or an EC2 instance with a self managed k8s cluster.

## SpiffeID
The hybrid plugin will always return the SpiffeID generated by the first plugin of the list supplied to the server. The order of the plugins supplied for both server and agent does not matter.

## Basic deployment
The hybrid plugin works as any external plugin would. It is designed to work as an external plugin, acting as a plugin aggregator.
To deploy it, first build and update the configuration for both SPIRE Server and Agent passing the list of built-in plugins that will be used in combination ir order to attest the node.

## Building
Start by building the binaries

`make build`

## Configuring
In order to use the hybrid plugin with the SPIRE instance, add in the SPIRE configuration file as an external plugin, informing the name, `plugin_cmd` and in the section plugins, inside `plugin_data`, add the built-in plugins configuration. Here's an example:

 **Server**
 ```
    plugins{
      NodeAttestor "hybrid" {
        plugin_cmd = "path/to/builded/file"
        plugin_data {
          plugins {
            first_selected_plugin {
                [plugin_config]
            }
            second_selected_plugin {
                [plugin_config]
            }
            [...]
          }
        }
      }
    }
```

**Agent**
 ```
    plugins {
      NodeAttestor "hybrid" {
        plugin_cmd = "path/to/builded/file"
        plugin_data {
          plugins {
            first_selected_plugin {
                [plugin_config]
            }
            second_selected_plugin {
                [plugin_config]
            }
          }
        }
      }
    }
 ```

### Deploying the hybrid using aws_iid and k8s_psat node attestors
To combine those two plugins, you have to be running the agent and server in an kubernetes instance inside aws.

Start off by creating an ec2 instance and deploying a kind kubernetes cluster or creating an eks cluster.

After that, run the following make command while in hybrid root folder:

`make build`

Then, to construct the docker images and push to any repo desired, set the environment variable DOCKER_HUB that should point to the docker image registry and it's used by the MAKEFILE and run the following command:

`make docker`

With the hybrid node attestor built and the docker image constructed, you now have to deploy it in the running kubernetes cluster in the aws. 
Configure kubectl to point to the aws cluster.
Change the credentials in the server.yaml and agent.yaml that are in the hybrid/dev/kubernetes and run the following command:

`make deploy-agent-server-eks`

You should now be able to see the running agent/server node attestor. First set the corresponding environment variables with their required values ($AWS_SECRET_ACCESS_KEY, $AWS_ACCESS_KEY_ID, $AWS_ASSUME_ROLE, $AWS_ACCOUNT_ID)

