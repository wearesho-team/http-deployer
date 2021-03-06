# Simple HTTP Deploy tool
It will update package version in docker-compose and run commands:
```bash
docker-compose -f custom.docker-compose.yml up -d
```
Previous docker compose file will be saved as `{previousDockerComposePath}_{previousImageVersion}`

## Configuration
Configuration by environment variables:
- **DEPLOYER_PORT** - port that Express listen
- **DEPLOYER_CONFIG_PATH** - path to JSON file with projects config

JSON config file should contain array, each element have to implement [ConfigInterface](src/ConfigInterface.ts)

## Requesting

#### Request
POST / 

Content-Type: application/json

```json
{
  "name": "image-name",
  "tag": "docker-registry.private.com/image-name:1.0.0"
}
```

#### Response
Content-Type: application/json

| Http Code 	| JSON Code 	| Message                                        	| What to do?                                                                                                                                                      	|
|-----------	|-----------	|------------------------------------------------	|------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| 200       	| 0         	| Successful                                     	| Drink beer                                                                                                                                                       	|
| 403       	| -1        	| Forbidden                                      	| Try to pass valid                                                                                                                                                	|
| 422       	| 101       	| Missing name or tag in body                    	| Put JSON body with name and tag fields                                                                                                                           	|
| 422       	| 102       	| {name} deploy does not exist                   	| Configure {name} deploy on server                                                                                                                                	|
| 501       	| 201       	| Compose file does not exist                    	| Fix path to docker-compose file in global configuration                                                                                                          	|
| 501       	| 202       	| Can not find image name in docker-compose file 	| Make sure that image specified in docker-compose file. Image name should be placed between `"` and have 3-digit version after `:` (example: `"image-name:1.0.0`) 	|
| 501       	| 203       	| Invalid {type} time: {invalidTimeFormat}       	| Make sure that time rules in configuration if specified in format `HH:MM:SS`                                                                                     	|
| 500       	| 301       	| BeforeDeploy Error                             	| Error in some BeforeDeploy script. See deploy logs for details and improve script                                                                                	|
| 500       	| 302       	| DockerDeploy Error                             	| Error while executing `docker-compose up -d`. See deploy logs for details                                                                                        	|
| 500       	| 303       	| AfterDeploy Error                              	| Error in some AfterDeploy script. See deploy logs for details and improve script                                                                                 	|
| 503       	| 403       	| Update not available                           	| Try run update considering time specified in time rules                                                                                        	                  |

**Notice: message can be changed in minor version. You should not parse it and depend your CI process on it.**

Example:
```json
{
  "code": 0,
  "message": "Successful"
}
```

#### Request
GET /status/:projectName

#### Response
Content-Type: application/json

| Http Code | JSON Code   | Message                     | What to do?                                             |
|---------- |------------ |---------------------------  |-------------------------------------------------------- |
| 200       | 0           | Successful                  | Order Pizza                                             |
| 501       | 201       	| Compose file does not exist | Fix path to docker-compose file in global configuration |
| 503       | any number  | Docker Error                | Check executing docker-compose ps on server             |    

**Notice: message can be changed in minor version. You should not parse it and depend your CI process on it.**

Example:
```json
{
  "code": 0,
  "message": "Successful",
  "containers": [
    {
      "name": "app-1",
      "state": "Up"
    },
    {
      "name": "app-2",
      "state": "Restarting"
    }
  ]
}
```

## Author
[Alexander <horat1us> Letnikow](mailto:reclamme@gmail.com)

## License
MIT
