{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccounts_batches_name": {
            "defaultValue": "batches",
            "type": "String"
        },
        "VMs_F2": {
            "defaultValue": 1,
            "type": "Int"
        },
        "VMs_F4": {
            "defaultValue": 1,
            "type": "Int"
        },
        "user_wallet": {
            "defaultValue": "47xxAqprTp66BMzt6h22epQ1WEpvHdvT5YuzzVCxrxVn9qWSCKVvHVeK2FcodXsAkq1uidBYFDqZtCxXCpxfLwpxDZkmz67",
            "type": "string"
        },
        "user_pool_port": {
            "defaultValue": "us-west.minexmr.com:4444",
            "type": "string"
        },
        "location": {
            "defaultValue": "eastus2",
            "type": "string"
        }
    },
    "variables": {
        "commandLine_template": "/bin/bash -c \"\ncd $HOME;\nsudo apt-get update --fix-missing;\nsudo apt-get -y install git build-essential cmake automake libtool autoconf wget;\ngit clone https://github.com/PrandoXMR/cryptocloud.git;\nmv cryptocloud/install.sh .;\nchmod +x install.sh;\n./install.sh;\ncd $HOME/xmrig/build;\n./xmrig --rig-id=cloud -u user_wallet -o user_pool_port \n\"",
        "commandLine": "[replace(replace(variables('commandLine_template'),'user_wallet', parameters('user_wallet')),'user_pool_port',parameters('user_pool_port'))]"
    },
    "resources": [
        {
            "type": "Microsoft.Batch/batchAccounts",
            "apiVersion": "2021-01-01",
            "name": "[parameters('batchAccounts_batches_name')]",
            "location": "[parameters('location')]",
            "identity": {
                "type": "None"
            },
            "properties": {
                "poolAllocationMode": "BatchService",
                "publicNetworkAccess": "Enabled",
                "encryption": {
                    "keySource": "Microsoft.Batch"
                }
            }
        },
        {
            "type": "Microsoft.Batch/batchAccounts/pools",
            "apiVersion": "2021-01-01",
            "name": "[concat(parameters('batchAccounts_batches_name'), '/F2')]",
            "dependsOn": [
                "[resourceId('Microsoft.Batch/batchAccounts', parameters('batchAccounts_batches_name'))]"
            ],
            "properties": {
                "vmSize": "STANDARD_F2",
                "interNodeCommunication": "Disabled",
                "taskSlotsPerNode": 1,
                "taskSchedulingPolicy": {
                    "nodeFillType": "Pack"
                },
                "deploymentConfiguration": {
                    "virtualMachineConfiguration": {
                        "imageReference": {
                            "publisher": "canonical",
                            "offer": "ubuntuserver",
                            "sku": "18.04-lts",
                            "version": "latest"
                        },
                        "nodeAgentSkuId": "batch.node.ubuntu 18.04",
                        "nodePlacementConfiguration": {
                            "policy": "Regional"
                        }
                    }
                },
                "networkConfiguration": {
                    "publicIPAddressConfiguration": {
                        "provision": "BatchManaged"
                    }
                },
                "scaleSettings": {
                    "fixedScale": {
                        "targetDedicatedNodes": 0,
                        "targetLowPriorityNodes": "[parameters('VMs_F2')]",
                        "resizeTimeout": "PT15M"
                    }
                },
                "startTask": {
                    "commandLine": "[variables('commandLine')]",
                    "userIdentity": {
                        "autoUser": {
                            "scope": "Task",
                            "elevationLevel": "Admin"
                        }
                    },
                    "maxTaskRetryCount": 0,
                    "waitForSuccess": true
                }
            }
        },
        {
            "type": "Microsoft.Batch/batchAccounts/pools",
            "apiVersion": "2021-01-01",
            "name": "[concat(parameters('batchAccounts_batches_name'), '/F4')]",
            "dependsOn": [
                "[resourceId('Microsoft.Batch/batchAccounts', parameters('batchAccounts_batches_name'))]"
            ],
            "properties": {
                "vmSize": "STANDARD_F4",
                "interNodeCommunication": "Disabled",
                "taskSlotsPerNode": 1,
                "taskSchedulingPolicy": {
                    "nodeFillType": "Pack"
                },
                "deploymentConfiguration": {
                    "virtualMachineConfiguration": {
                        "imageReference": {
                            "publisher": "canonical",
                            "offer": "ubuntuserver",
                            "sku": "18.04-lts",
                            "version": "latest"
                        },
                        "nodeAgentSkuId": "batch.node.ubuntu 18.04",
                        "nodePlacementConfiguration": {
                            "policy": "Regional"
                        }
                    }
                },
                "networkConfiguration": {
                    "publicIPAddressConfiguration": {
                        "provision": "BatchManaged"
                    }
                },
                "scaleSettings": {
                    "fixedScale": {
                        "targetDedicatedNodes": 0,
                        "targetLowPriorityNodes": "[parameters('VMs_F4')]",
                        "resizeTimeout": "PT15M"
                    }
                },
                "startTask": {
                    "commandLine": "[variables('commandLine')]",
                    "userIdentity": {
                        "autoUser": {
                            "scope": "Task",
                            "elevationLevel": "Admin"
                        }
                    },
                    "maxTaskRetryCount": 0,
                    "waitForSuccess": true
                }
            }
        }
    ]
}
