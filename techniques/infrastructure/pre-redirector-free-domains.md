---
description: Using azure services to front your redirectors for additional domains.
---

# Pre-redirector (free domains!)



Pre-redirectors offer basic reverse proxy functionality in front of your HTTPS redirector, which ultimately handles the in depth processing of validating requests. Typically, services which provide high quality domain names are used for pre-redirectors, both to add legitimacy to your traffic & to increase the total domain endpoints you can use.

### Azure CDN - \*.azureedge.net

* From the Azure Portal, access "Templates"
  * Example ARM template to create 3 Azure Front Door endpoints simultaneously included below
* Use template to create an azure CDN endpoint with the desired name, this will apply required configurations such as disabling caching
* Choose three CDN endpoint names, which will be used in the format \<endpoint>.azureedge.net

Deployment will take a while, up to two hours at peak times

***

### Azure Front Door - \*.azurefd.net

* From the Azure Portal, access "Templates"
  * Example ARM template to create 3 Azure Front Door endpoints simultaneously included below
* Use Azure Front Door Template
* Choose `Deploy`
* Fill out your Project name for resource tagging
* Origin with the domain name of your redirector
* Choose three FD endpoint names, which will be used in the format \<endpoint>.azurefd.net

### Azure Websites - \*.azurewebsites.com

{% hint style="danger" %}
This uses IIS based App Service instances, which are more expensive than their Linux counterparts.
{% endhint %}

* From the Azure Portal, access "Templates"
  * Example ARM template to create 3 Azure Front Door endpoints simultaneously included below
* Use Azure Websites template
* Choose "Deploy"
* Fill out your Project name for resource tagging
* Origin with the domain name of your redirector
* Choose three endpoint names, which will be used in the format \<endpoint>.azurewebsites.com
* Once deployed, access the management pane of the app at \<endpoint>.**scm**.azurewebsites.com
* Select `Debug Console` > `Powershell` > `Home Icon`
* Browse to `site` folder and upload the following as `applicationHost.xdt`

```
<!--?xml version="1.0"?-->
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <system.webServer>
    <proxy xdt:Transform="InsertIfMissing" enabled="true" preserveHostHeader="false" reverseRewriteHostInResponseHeaders="false" />
  </system.webServer>
</configuration>
```

* Browse into `wwwroot` and upload the following as `web.config` - Make sure to edit the site to point to your redirector!

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
	<system.webServer>
		<httpErrors errorMode="Detailed" />
		<rewrite>
			<rules>
				<rule name="ForceSSL" stopProcessing="true">
					<match url="(.*)" />
					<conditions>
						<add input="{HTTPS}" pattern="^OFF$" ignoreCase="true" />
					</conditions>
					<action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
				</rule>
				<rule name="Proxy" stopProcessing="true">
					<match url="(.*)" />
					<action type="Rewrite" url="https://REDIRECTOR.COM/{R:1}" />
				</rule>
			</rules>
		</rewrite>
	</system.webServer>
</configuration>
```

* Restart the web service from the Azure portal



## ARM templates

### Azure CDN - \*.azureedge.net

ARM template to create three CDN endpoints in a single CDN profile

```json
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "CDN_group_name": {
      "defaultValue": "",
      "type": "String"
    },
    "Project": {
      "defaultValue": "",
      "type": "String"
    },
    "origin_hostname": {
      "defaultValue": "",
      "type": "String"
    },
    "CDNEndpoint1": {
      "defaultValue": "",
      "type": "String"
    },
    "CDNEndpoint2": {
      "defaultValue": "",
      "type": "String"
    },
    "CDNEndpoint3": {
      "defaultValue": "",
      "type": "String"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Cdn/profiles",
      "apiVersion": "2020-09-01",
      "name": "[parameters('CDN_group_name')]",
      "location": "Global",
      "tags": {
        "Project": "[parameters('Project')]"
      },
      "sku": {
        "name": "Standard_Verizon"
      },
      "kind": "cdn",
      "properties": {}
    },
    {
      "type": "Microsoft.Cdn/profiles/endpoints",
      "apiVersion": "2020-09-01",
      "name": "[format('{0}/{1}', parameters('CDN_group_name'), parameters('CDNEndpoint1'))]",
      "location": "Global",
      "dependsOn": [
        "[resourceId('Microsoft.Cdn/profiles', parameters('CDN_group_name'))]"
      ],
      "tags": {
        "Project": "[parameters('Project')]"
      },
      "properties": {
        "originHostHeader": "[parameters('origin_hostname')]",
        "contentTypesToCompress": [
          "text/plain",
          "text/html",
          "text/css",
          "text/javascript",
          "application/x-javascript",
          "application/javascript",
          "application/json",
          "application/xml"
        ],
        "isCompressionEnabled": true,
        "isHttpAllowed": true,
        "isHttpsAllowed": true,
        "queryStringCachingBehavior": "BypassCaching",
        "optimizationType": "GeneralWebDelivery",
        "origins": [
          {
            "name": "CDN-endpoint1",
            "properties": {
              "hostName": "[parameters('origin_hostname')]",
              "enabled": true
            }
          }
        ],
        "originGroups": [],
        "geoFilters": [],
        "deliveryPolicy": {
          "rules": [
            {
              "order": 0,
              "conditions": [],
              "actions": [
                {
                  "name": "CacheExpiration",
                  "parameters": {
                    "cacheBehavior": "BypassCache",
                    "cacheType": "All",
                    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleCacheExpirationActionParameters"
                  }
                }
              ]
            }
          ]
        }
      }
    },
    {
      "type": "Microsoft.Cdn/profiles/endpoints",
      "apiVersion": "2020-09-01",
      "name": "[format('{0}/{1}', parameters('CDN_group_name'), parameters('CDNEndpoint2'))]",
      "location": "Global",
      "dependsOn": [
        "[resourceId('Microsoft.Cdn/profiles', parameters('CDN_group_name'))]"
      ],
      "tags": {
        "Project": "[parameters('Project')]"
      },
      "properties": {
        "originHostHeader": "[parameters('origin_hostname')]",
        "contentTypesToCompress": [
          "text/plain",
          "text/html",
          "text/css",
          "text/javascript",
          "application/x-javascript",
          "application/javascript",
          "application/json",
          "application/xml"
        ],
        "isCompressionEnabled": true,
        "isHttpAllowed": true,
        "isHttpsAllowed": true,
        "queryStringCachingBehavior": "BypassCaching",
        "optimizationType": "GeneralWebDelivery",
        "origins": [
          {
            "name": "CDN-endpoint1",
            "properties": {
              "hostName": "[parameters('origin_hostname')]",
              "enabled": true
            }
          }
        ],
        "originGroups": [],
        "geoFilters": [],
        "deliveryPolicy": {
          "rules": [
            {
              "order": 0,
              "conditions": [],
              "actions": [
                {
                  "name": "CacheExpiration",
                  "parameters": {
                    "cacheBehavior": "BypassCache",
                    "cacheType": "All",
                    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleCacheExpirationActionParameters"
                  }
                }
              ]
            }
          ]
        }
      }
    },
    {
      "type": "Microsoft.Cdn/profiles/endpoints",
      "apiVersion": "2020-09-01",
      "name": "[format('{0}/{1}', parameters('CDN_group_name'), parameters('CDNEndpoint3'))]",
      "location": "Global",
      "dependsOn": [
        "[resourceId('Microsoft.Cdn/profiles', parameters('CDN_group_name'))]"
      ],
      "tags": {
        "Project": "[parameters('Project')]"
      },
      "properties": {
        "originHostHeader": "[parameters('origin_hostname')]",
        "contentTypesToCompress": [
          "text/plain",
          "text/html",
          "text/css",
          "text/javascript",
          "application/x-javascript",
          "application/javascript",
          "application/json",
          "application/xml"
        ],
        "isCompressionEnabled": true,
        "isHttpAllowed": true,
        "isHttpsAllowed": true,
        "queryStringCachingBehavior": "BypassCaching",
        "optimizationType": "GeneralWebDelivery",
        "origins": [
          {
            "name": "CDN-endpoint1",
            "properties": {
              "hostName": "[parameters('origin_hostname')]",
              "enabled": true
            }
          }
        ],
        "originGroups": [],
        "geoFilters": [],
        "deliveryPolicy": {
          "rules": [
            {
              "order": 0,
              "conditions": [],
              "actions": [
                {
                  "name": "CacheExpiration",
                  "parameters": {
                    "cacheBehavior": "BypassCache",
                    "cacheType": "All",
                    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleCacheExpirationActionParameters"
                  }
                }
              ]
            }
          ]
        }
      }
    }
  ]
}
```

### Azure Front Door - \*.azurefd.net

ARM template to create three Az Front Door endpoints.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "ProjectTag": {
      "defaultValue": "ProjectName",
      "type": "String"
    },
    "BackendHostname": {
      "defaultValue": "google.com",
      "type": "String"
    },
    "Redir1_Name": {
        "defaultValue": "",
        "type": "String"
      },
      "Redir2_Name": {
        "defaultValue": "",
        "type": "String"
      },
      "Redir3_Name": {
        "defaultValue": "",
        "type": "String"
      }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Network/frontdoors",
      "apiVersion": "2020-05-01",
      "name": "[parameters('Redir1_Name')]",
      "location": "Global",
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "routingRules": [
          {
            "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), '/RoutingRules/RoutingRule')]",
            "name": "RoutingRule",
            "properties": {
              "routeConfiguration": {
                "forwardingProtocol": "HttpsOnly",
                "backendPool": {
                  "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), '/backendPools/Backend')]"
                },
                "@odata.type": "#Microsoft.Azure.FrontDoor.Models.FrontdoorForwardingConfiguration"
              },
              "resourceState": "Enabled",
              "frontendEndpoints": [
                {
                  "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), concat('/frontendEndpoints/', parameters('Redir1_Name'), '-azurefd-net'))]"
                }
              ],
              "acceptedProtocols": [
                "Http",
                "Https"
              ],
              "patternsToMatch": [
                "/*"
              ],
              "enabledState": "Enabled"
            }
          }
        ],
        "resourceState": "Enabled",
        "loadBalancingSettings": [
          {
            "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), '/LoadBalancingSettings/loadBalancingSettings-1636899435999')]",
            "name": "loadBalancingSettings-1636899435999",
            "properties": {
              "resourceState": "Enabled",
              "sampleSize": 4,
              "successfulSamplesRequired": 2,
              "additionalLatencyMilliseconds": 0
            }
          }
        ],
        "healthProbeSettings": [
          {
            "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), '/HealthProbeSettings/healthProbeSettings-1636899435999')]",
            "name": "healthProbeSettings-1636899435999",
            "properties": {
              "resourceState": "Enabled",
              "path": "/",
              "protocol": "Https",
              "intervalInSeconds": 30,
              "enabledState": "Disabled",
              "healthProbeMethod": "Head"
            }
          }
        ],
        "backendPools": [
          {
            "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), '/BackendPools/Backend')]",
            "name": "Backend",
            "properties": {
              "backends": [
                {
                  "address": "[parameters('BackendHostname')]",
                  "httpPort": 80,
                  "httpsPort": 443,
                  "priority": 1,
                  "weight": 50,
                  "backendHostHeader": "[parameters('BackendHostname')]",
                  "enabledState": "Enabled"
                }
              ],
              "resourceState": "Enabled",
              "loadBalancingSettings": {
                "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), '/loadBalancingSettings/loadBalancingSettings-1636899435999')]"
              },
              "healthProbeSettings": {
                "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), '/healthProbeSettings/healthProbeSettings-1636899435999')]"
              }
            }
          }
        ],
        "frontendEndpoints": [
          {
            "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir1_Name')), concat('/FrontendEndpoints/', parameters('Redir1_Name'), '-azurefd-net'))]",
            "name": "[concat(parameters('Redir1_Name'), '-azurefd-net')]",
            "properties": {
              "resourceState": "Enabled",
              "hostName": "[concat(parameters('Redir1_Name'), '.azurefd.net')]",
              "sessionAffinityEnabledState": "Disabled",
              "sessionAffinityTtlSeconds": 0
            }
          }
        ],
        "backendPoolsSettings": {
          "enforceCertificateNameCheck": "Enabled",
          "sendRecvTimeoutSeconds": 30
        },
        "enabledState": "Enabled",
        "friendlyName": "[parameters('Redir1_Name')]"
      }
    },
    {
        "type": "Microsoft.Network/frontdoors",
        "apiVersion": "2020-05-01",
        "name": "[parameters('Redir2_Name')]",
        "location": "Global",
        "tags": {
          "Project": "[parameters('ProjectTag')]"
        },
        "properties": {
          "routingRules": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), '/RoutingRules/RoutingRule')]",
              "name": "RoutingRule",
              "properties": {
                "routeConfiguration": {
                  "forwardingProtocol": "HttpsOnly",
                  "backendPool": {
                    "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), '/backendPools/Backend')]"
                  },
                  "@odata.type": "#Microsoft.Azure.FrontDoor.Models.FrontdoorForwardingConfiguration"
                },
                "resourceState": "Enabled",
                "frontendEndpoints": [
                  {
                    "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), concat('/frontendEndpoints/', parameters('Redir2_Name'), '-azurefd-net'))]"
                  }
                ],
                "acceptedProtocols": [
                  "Http",
                  "Https"
                ],
                "patternsToMatch": [
                  "/*"
                ],
                "enabledState": "Enabled"
              }
            }
          ],
          "resourceState": "Enabled",
          "loadBalancingSettings": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), '/LoadBalancingSettings/loadBalancingSettings-1636899435999')]",
              "name": "loadBalancingSettings-1636899435999",
              "properties": {
                "resourceState": "Enabled",
                "sampleSize": 4,
                "successfulSamplesRequired": 2,
                "additionalLatencyMilliseconds": 0
              }
            }
          ],
          "healthProbeSettings": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), '/HealthProbeSettings/healthProbeSettings-1636899435999')]",
              "name": "healthProbeSettings-1636899435999",
              "properties": {
                "resourceState": "Enabled",
                "path": "/",
                "protocol": "Https",
                "intervalInSeconds": 30,
                "enabledState": "Disabled",
                "healthProbeMethod": "Head"
              }
            }
          ],
          "backendPools": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), '/BackendPools/Backend')]",
              "name": "Backend",
              "properties": {
                "backends": [
                  {
                    "address": "[parameters('BackendHostname')]",
                    "httpPort": 80,
                    "httpsPort": 443,
                    "priority": 1,
                    "weight": 50,
                    "backendHostHeader": "[parameters('BackendHostname')]",
                    "enabledState": "Enabled"
                  }
                ],
                "resourceState": "Enabled",
                "loadBalancingSettings": {
                  "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), '/loadBalancingSettings/loadBalancingSettings-1636899435999')]"
                },
                "healthProbeSettings": {
                  "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), '/healthProbeSettings/healthProbeSettings-1636899435999')]"
                }
              }
            }
          ],
          "frontendEndpoints": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir2_Name')), concat('/FrontendEndpoints/', parameters('Redir2_Name'), '-azurefd-net'))]",
              "name": "[concat(parameters('Redir2_Name'), '-azurefd-net')]",
              "properties": {
                "resourceState": "Enabled",
                "hostName": "[concat(parameters('Redir2_Name'), '.azurefd.net')]",
                "sessionAffinityEnabledState": "Disabled",
                "sessionAffinityTtlSeconds": 0
              }
            }
          ],
          "backendPoolsSettings": {
            "enforceCertificateNameCheck": "Enabled",
            "sendRecvTimeoutSeconds": 30
          },
          "enabledState": "Enabled",
          "friendlyName": "[parameters('Redir2_Name')]"
        }
      },
      {
        "type": "Microsoft.Network/frontdoors",
        "apiVersion": "2020-05-01",
        "name": "[parameters('Redir3_Name')]",
        "location": "Global",
        "tags": {
          "Project": "[parameters('ProjectTag')]"
        },
        "properties": {
          "routingRules": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), '/RoutingRules/RoutingRule')]",
              "name": "RoutingRule",
              "properties": {
                "routeConfiguration": {
                  "forwardingProtocol": "HttpsOnly",
                  "backendPool": {
                    "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), '/backendPools/Backend')]"
                  },
                  "@odata.type": "#Microsoft.Azure.FrontDoor.Models.FrontdoorForwardingConfiguration"
                },
                "resourceState": "Enabled",
                "frontendEndpoints": [
                  {
                    "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), concat('/frontendEndpoints/', parameters('Redir3_Name'), '-azurefd-net'))]"
                  }
                ],
                "acceptedProtocols": [
                  "Http",
                  "Https"
                ],
                "patternsToMatch": [
                  "/*"
                ],
                "enabledState": "Enabled"
              }
            }
          ],
          "resourceState": "Enabled",
          "loadBalancingSettings": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), '/LoadBalancingSettings/loadBalancingSettings-1636899435999')]",
              "name": "loadBalancingSettings-1636899435999",
              "properties": {
                "resourceState": "Enabled",
                "sampleSize": 4,
                "successfulSamplesRequired": 2,
                "additionalLatencyMilliseconds": 0
              }
            }
          ],
          "healthProbeSettings": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), '/HealthProbeSettings/healthProbeSettings-1636899435999')]",
              "name": "healthProbeSettings-1636899435999",
              "properties": {
                "resourceState": "Enabled",
                "path": "/",
                "protocol": "Https",
                "intervalInSeconds": 30,
                "enabledState": "Disabled",
                "healthProbeMethod": "Head"
              }
            }
          ],
          "backendPools": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), '/BackendPools/Backend')]",
              "name": "Backend",
              "properties": {
                "backends": [
                  {
                    "address": "[parameters('BackendHostname')]",
                    "httpPort": 80,
                    "httpsPort": 443,
                    "priority": 1,
                    "weight": 50,
                    "backendHostHeader": "[parameters('BackendHostname')]",
                    "enabledState": "Enabled"
                  }
                ],
                "resourceState": "Enabled",
                "loadBalancingSettings": {
                  "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), '/loadBalancingSettings/loadBalancingSettings-1636899435999')]"
                },
                "healthProbeSettings": {
                  "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), '/healthProbeSettings/healthProbeSettings-1636899435999')]"
                }
              }
            }
          ],
          "frontendEndpoints": [
            {
              "id": "[concat(resourceId('Microsoft.Network/frontdoors', parameters('Redir3_Name')), concat('/FrontendEndpoints/', parameters('Redir3_Name'), '-azurefd-net'))]",
              "name": "[concat(parameters('Redir3_Name'), '-azurefd-net')]",
              "properties": {
                "resourceState": "Enabled",
                "hostName": "[concat(parameters('Redir3_Name'), '.azurefd.net')]",
                "sessionAffinityEnabledState": "Disabled",
                "sessionAffinityTtlSeconds": 0
              }
            }
          ],
          "backendPoolsSettings": {
            "enforceCertificateNameCheck": "Enabled",
            "sendRecvTimeoutSeconds": 30
          },
          "enabledState": "Enabled",
          "friendlyName": "[parameters('Redir3_Name')]"
        }
      }
  ]
}
```

### Azure Websites - \*.azurewebsites.com

ARM template to create three Az App Service websites.

{% hint style="danger" %}
This uses IIS based App Service instances, which are more expensive than their Linux counterparts.
{% endhint %}

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "AppSvcGroupName": {
      "defaultValue": "ProjectNameRedirs",
      "type": "String"
    },
    "ProjectTag": {
      "defaultValue": "ProjectName",
      "type": "String"
    },
    "Redir1_Name": {
      "defaultValue": "",
      "type": "String"
    },
    "Redir2_Name": {
      "defaultValue": "",
      "type": "String"
    },
    "Redir3_Name": {
      "defaultValue": "",
      "type": "String"
    }
  },
    "variables": {},
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2021-02-01",
      "name": "[parameters('AppSvcGroupName')]",
      "location": "West Europe",
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "sku": {
        "name": "F1",
        "tier": "Free",
        "size": "F1",
        "family": "F",
        "capacity": 0
      },
      "kind": "app",
      "properties": {
        "perSiteScaling": false,
        "elasticScaleEnabled": false,
        "maximumElasticWorkerCount": 1,
        "isSpot": false,
        "reserved": false,
        "isXenon": false,
        "hyperV": false,
        "targetWorkerCount": 0,
        "targetWorkerSizeId": 0,
        "zoneRedundant": false
      }
    },
    {
      
      "type": "Microsoft.Web/sites",
      "apiVersion": "2021-02-01",
      "name": "[parameters('Redir1_Name')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('AppSvcGroupName'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "kind": "app",
      "properties": {
        "enabled": true,
        "hostNameSslStates": [
          {
            "name": "[concat(parameters('Redir1_Name'), '.azurewebsites.net')]",
            "sslState": "Disabled",
            "hostType": "Standard"
          },
          {
            "name": "[concat(parameters('Redir1_Name'), '.scm.azurewebsites.net')]",
            "sslState": "Disabled",
            "hostType": "Repository"
          }
        ],
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', parameters('AppSvcGroupName'))]",
        "reserved": false,
        "isXenon": false,
        "hyperV": false,
        "siteConfig": {
          "numberOfWorkers": 1,
          "acrUseManagedIdentityCreds": false,
          "alwaysOn": false,
          "http20Enabled": false,
          "functionAppScaleLimit": 0,
          "minimumElasticInstanceCount": 1
        },
        "scmSiteAlsoStopped": false,
        "clientAffinityEnabled": true,
        "clientCertEnabled": false,
        "clientCertMode": "Required",
        "hostNamesDisabled": false,
        "customDomainVerificationId": "20F69414BAA93D84D0C2B10CA560650EC6C421E0B86499A66DB0FEAB468F1BB1",
        "containerSize": 0,
        "dailyMemoryTimeQuota": 0,
        "httpsOnly": false,
        "redundancyMode": "None",
        "storageAccountRequired": false,
        "keyVaultReferenceIdentity": "SystemAssigned"
      }
    },
    {
      
      "type": "Microsoft.Web/sites/basicPublishingCredentialsPolicies",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir1_Name'), '/ftp')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir1_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "allow": true
      }
    },
    {
      
      "type": "Microsoft.Web/sites/basicPublishingCredentialsPolicies",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir1_Name'), '/scm')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir1_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "allow": true
      }
    },
    {
      
      "type": "Microsoft.Web/sites/config",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir1_Name'), '/web')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir1_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "numberOfWorkers": 1,
        "defaultDocuments": [
          "Default.htm",
          "Default.html",
          "Default.asp",
          "index.htm",
          "index.html",
          "iisstart.htm",
          "default.aspx",
          "index.php",
          "hostingstart.html"
        ],
        "netFrameworkVersion": "v4.0",
        "requestTracingEnabled": false,
        "remoteDebuggingEnabled": false,
        "remoteDebuggingVersion": "VS2019",
        "httpLoggingEnabled": false,
        "acrUseManagedIdentityCreds": false,
        "logsDirectorySizeLimit": 35,
        "detailedErrorLoggingEnabled": false,
        "publishingUsername": "$dev-test123",
        "scmType": "None",
        "use32BitWorkerProcess": true,
        "webSocketsEnabled": false,
        "alwaysOn": false,
        "managedPipelineMode": "Integrated",
        "virtualApplications": [
          {
            "virtualPath": "/",
            "physicalPath": "site\\wwwroot",
            "preloadEnabled": false
          }
        ],
        "loadBalancing": "LeastRequests",
        "experiments": {
          "rampUpRules": []
        },
        "autoHealEnabled": false,
        "vnetRouteAllEnabled": false,
        "vnetPrivatePortsCount": 0,
        "localMySqlEnabled": false,
        "ipSecurityRestrictions": [
          {
            "ipAddress": "Any",
            "action": "Allow",
            "priority": 1,
            "name": "Allow all",
            "description": "Allow all access"
          }
        ],
        "scmIpSecurityRestrictions": [
          {
            "ipAddress": "Any",
            "action": "Allow",
            "priority": 1,
            "name": "Allow all",
            "description": "Allow all access"
          }
        ],
        "scmIpSecurityRestrictionsUseMain": false,
        "http20Enabled": false,
        "minTlsVersion": "1.2",
        "scmMinTlsVersion": "1.0",
        "ftpsState": "AllAllowed",
        "preWarmedInstanceCount": 0,
        "functionAppScaleLimit": 0,
        "functionsRuntimeScaleMonitoringEnabled": false,
        "minimumElasticInstanceCount": 1,
        "azureStorageAccounts": {}
      }
    },
    {
      
      "type": "Microsoft.Web/sites/hostNameBindings",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir1_Name'), '/', parameters('Redir1_Name'), '.azurewebsites.net')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir1_Name'))]"
      ],
      "properties": {
        "siteName": "[parameters('Redir1_Name')]",
        "hostNameType": "Verified"
      }
    },
    {
      
      "type": "Microsoft.Web/sites",
      "apiVersion": "2021-02-01",
      "name": "[parameters('Redir2_Name')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('AppSvcGroupName'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "kind": "app",
      "properties": {
        "enabled": true,
        "hostNameSslStates": [
          {
            "name": "[concat(parameters('Redir2_Name'), '.azurewebsites.net')]",
            "sslState": "Disabled",
            "hostType": "Standard"
          },
          {
            "name": "[concat(parameters('Redir2_Name'), '.scm.azurewebsites.net')]",
            "sslState": "Disabled",
            "hostType": "Repository"
          }
        ],
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', parameters('AppSvcGroupName'))]",
        "reserved": false,
        "isXenon": false,
        "hyperV": false,
        "siteConfig": {
          "numberOfWorkers": 1,
          "acrUseManagedIdentityCreds": false,
          "alwaysOn": false,
          "http20Enabled": false,
          "functionAppScaleLimit": 0,
          "minimumElasticInstanceCount": 1
        },
        "scmSiteAlsoStopped": false,
        "clientAffinityEnabled": true,
        "clientCertEnabled": false,
        "clientCertMode": "Required",
        "hostNamesDisabled": false,
        "customDomainVerificationId": "20F69414BAA93D84D0C2B10CA560650EC6C421E0B86499A66DB0FEAB468F1BB1",
        "containerSize": 0,
        "dailyMemoryTimeQuota": 0,
        "httpsOnly": false,
        "redundancyMode": "None",
        "storageAccountRequired": false,
        "keyVaultReferenceIdentity": "SystemAssigned"
      }
    },
    {
      
      "type": "Microsoft.Web/sites/basicPublishingCredentialsPolicies",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir2_Name'), '/ftp')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir2_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "allow": true
      }
    },
    {
      
      "type": "Microsoft.Web/sites/basicPublishingCredentialsPolicies",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir2_Name'), '/scm')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir2_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "allow": true
      }
    },
    {
      
      "type": "Microsoft.Web/sites/config",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir2_Name'), '/web')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir2_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "numberOfWorkers": 1,
        "defaultDocuments": [
          "Default.htm",
          "Default.html",
          "Default.asp",
          "index.htm",
          "index.html",
          "iisstart.htm",
          "default.aspx",
          "index.php",
          "hostingstart.html"
        ],
        "netFrameworkVersion": "v4.0",
        "requestTracingEnabled": false,
        "remoteDebuggingEnabled": false,
        "remoteDebuggingVersion": "VS2019",
        "httpLoggingEnabled": false,
        "acrUseManagedIdentityCreds": false,
        "logsDirectorySizeLimit": 35,
        "detailedErrorLoggingEnabled": false,
        "publishingUsername": "$dev-test123",
        "scmType": "None",
        "use32BitWorkerProcess": true,
        "webSocketsEnabled": false,
        "alwaysOn": false,
        "managedPipelineMode": "Integrated",
        "virtualApplications": [
          {
            "virtualPath": "/",
            "physicalPath": "site\\wwwroot",
            "preloadEnabled": false
          }
        ],
        "loadBalancing": "LeastRequests",
        "experiments": {
          "rampUpRules": []
        },
        "autoHealEnabled": false,
        "vnetRouteAllEnabled": false,
        "vnetPrivatePortsCount": 0,
        "localMySqlEnabled": false,
        "ipSecurityRestrictions": [
          {
            "ipAddress": "Any",
            "action": "Allow",
            "priority": 1,
            "name": "Allow all",
            "description": "Allow all access"
          }
        ],
        "scmIpSecurityRestrictions": [
          {
            "ipAddress": "Any",
            "action": "Allow",
            "priority": 1,
            "name": "Allow all",
            "description": "Allow all access"
          }
        ],
        "scmIpSecurityRestrictionsUseMain": false,
        "http20Enabled": false,
        "minTlsVersion": "1.2",
        "scmMinTlsVersion": "1.0",
        "ftpsState": "AllAllowed",
        "preWarmedInstanceCount": 0,
        "functionAppScaleLimit": 0,
        "functionsRuntimeScaleMonitoringEnabled": false,
        "minimumElasticInstanceCount": 1,
        "azureStorageAccounts": {}
      }
    },
    {
      
      "type": "Microsoft.Web/sites/hostNameBindings",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir2_Name'), '/', parameters('Redir2_Name'), '.azurewebsites.net')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir2_Name'))]"
      ],
      "properties": {
        "siteName": "[parameters('Redir2_Name')]",
        "hostNameType": "Verified"
      }
    },
    {
      
      "type": "Microsoft.Web/sites",
      "apiVersion": "2021-02-01",
      "name": "[parameters('Redir3_Name')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('AppSvcGroupName'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "kind": "app",
      "properties": {
        "enabled": true,
        "hostNameSslStates": [
          {
            "name": "[concat(parameters('Redir3_Name'), '.azurewebsites.net')]",
            "sslState": "Disabled",
            "hostType": "Standard"
          },
          {
            "name": "[concat(parameters('Redir3_Name'), '.scm.azurewebsites.net')]",
            "sslState": "Disabled",
            "hostType": "Repository"
          }
        ],
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', parameters('AppSvcGroupName'))]",
        "reserved": false,
        "isXenon": false,
        "hyperV": false,
        "siteConfig": {
          "numberOfWorkers": 1,
          "acrUseManagedIdentityCreds": false,
          "alwaysOn": false,
          "http20Enabled": false,
          "functionAppScaleLimit": 0,
          "minimumElasticInstanceCount": 1
        },
        "scmSiteAlsoStopped": false,
        "clientAffinityEnabled": true,
        "clientCertEnabled": false,
        "clientCertMode": "Required",
        "hostNamesDisabled": false,
        "customDomainVerificationId": "20F69414BAA93D84D0C2B10CA560650EC6C421E0B86499A66DB0FEAB468F1BB1",
        "containerSize": 0,
        "dailyMemoryTimeQuota": 0,
        "httpsOnly": false,
        "redundancyMode": "None",
        "storageAccountRequired": false,
        "keyVaultReferenceIdentity": "SystemAssigned"
      }
    },
    {
      
      "type": "Microsoft.Web/sites/basicPublishingCredentialsPolicies",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir3_Name'), '/ftp')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir3_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "allow": true
      }
    },
    {
      
      "type": "Microsoft.Web/sites/basicPublishingCredentialsPolicies",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir3_Name'), '/scm')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir3_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "allow": true
      }
    },
    {
      
      "type": "Microsoft.Web/sites/config",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir3_Name'), '/web')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir3_Name'))]"
      ],
      "tags": {
        "Project": "[parameters('ProjectTag')]"
      },
      "properties": {
        "numberOfWorkers": 1,
        "defaultDocuments": [
          "Default.htm",
          "Default.html",
          "Default.asp",
          "index.htm",
          "index.html",
          "iisstart.htm",
          "default.aspx",
          "index.php",
          "hostingstart.html"
        ],
        "netFrameworkVersion": "v4.0",
        "requestTracingEnabled": false,
        "remoteDebuggingEnabled": false,
        "remoteDebuggingVersion": "VS2019",
        "httpLoggingEnabled": false,
        "acrUseManagedIdentityCreds": false,
        "logsDirectorySizeLimit": 35,
        "detailedErrorLoggingEnabled": false,
        "publishingUsername": "$dev-test123",
        "scmType": "None",
        "use32BitWorkerProcess": true,
        "webSocketsEnabled": false,
        "alwaysOn": false,
        "managedPipelineMode": "Integrated",
        "virtualApplications": [
          {
            "virtualPath": "/",
            "physicalPath": "site\\wwwroot",
            "preloadEnabled": false
          }
        ],
        "loadBalancing": "LeastRequests",
        "experiments": {
          "rampUpRules": []
        },
        "autoHealEnabled": false,
        "vnetRouteAllEnabled": false,
        "vnetPrivatePortsCount": 0,
        "localMySqlEnabled": false,
        "ipSecurityRestrictions": [
          {
            "ipAddress": "Any",
            "action": "Allow",
            "priority": 1,
            "name": "Allow all",
            "description": "Allow all access"
          }
        ],
        "scmIpSecurityRestrictions": [
          {
            "ipAddress": "Any",
            "action": "Allow",
            "priority": 1,
            "name": "Allow all",
            "description": "Allow all access"
          }
        ],
        "scmIpSecurityRestrictionsUseMain": false,
        "http20Enabled": false,
        "minTlsVersion": "1.2",
        "scmMinTlsVersion": "1.0",
        "ftpsState": "AllAllowed",
        "preWarmedInstanceCount": 0,
        "functionAppScaleLimit": 0,
        "functionsRuntimeScaleMonitoringEnabled": false,
        "minimumElasticInstanceCount": 1,
        "azureStorageAccounts": {}
      }
    },
    {
      
      "type": "Microsoft.Web/sites/hostNameBindings",
      "apiVersion": "2021-02-01",
      "name": "[concat(parameters('Redir3_Name'), '/', parameters('Redir3_Name'), '.azurewebsites.net')]",
      "location": "West Europe",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', parameters('Redir3_Name'))]"
      ],
      "properties": {
        "siteName": "[parameters('Redir3_Name')]",
        "hostNameType": "Verified"
      }
    }
  ]
}
```

