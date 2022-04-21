# Docker Keycloak with 2 Keycloak Containers, Postgres and Reverse Proxy

This repository provides the docker-compose to build Keycloak with JDBC Ping between 2 containers, postgres, and a reverse proxy via nginx.  This walkthrough assumes you have already installed both docker and the latest version of docker compose.  

The postgres container provides a volume so that the data is persistent across container restarts/stops/etc.  

If you plan to use SSL, you will need to place your .crt and .key or .pem file in the certs folder located in the same directory.  The nginx conf file is already set to redirect via ssl to the containers that run keycloak.

While its not strictly necessary, the docker compose file creates the network and exposes only the services required inside the network so that postgres is not exposed externally.  Since all the containers will run inside this network, the only ports open are those for nginx.  If you would like to expose these ports externally, simply make the required edits to the docker-compose file.  nginx will load-balance between the 2 keycloak containers

Unless you make explicit edits to the docker-compose, all default passwords will be used on creation.  

**Easy Instructions**

1. Clone the repository and enter the directory
2. If using SSL, be sure to place your certicates and keys in the certs folder
2a. If you just want to use http, change the line from 443 ssl to 80 and remove the cert locations.
3. update the nginx conf with the names of certificates
4. Review the docker-compose for any additional changes you'd like to make.  NOTE: **Update your location for nginx conf and certs!**
5. Run ```docker-compose up -d``` to bring up the environment

If you installed with ssl, you can find your environment at https://localhost or whatever port you set this as.

After logging in, you can login with admin/admin, be sure to change this password after logging in.

That's it, you're now ready to go!  Good luck!

**Backup/Restore **

We can use the native tools inside the keycloak container to export our realm with client secrets and use the same for importing:

EXPORT TEMPLATE:
[podman | docker] exec -it <pod_name> opt/jboss/keycloak/bin/standalone.sh
	-Djboss.socket.binding.port-offset=<interger_value> Docker recommend  an offset of 100 at least
	-Dkeycloak.migration.action=[export | import]
	-Dkeycloak.migration.provider=[singleFile | dir]
	-Dkeycloak.migration.dir=<DIR TO EXPORT TO> Use only iff .migration.provider=dir
	-Dkeycloak.migration.realmName=<REALM_NAME_TO_EXPORT>
	-Dkeycloak.migration.usersExportStrategy=[DIFFERENT_FILES | SKIP | REALM_FILE | SAME_FILE]
	-Dkeycloak.migration.usersPerFile=<integer_value> Use only iff .usersExportStrategy=DIFFERENT_FILES
	-Dkeycloak.migration.file=<FILE TO EXPORT TO>



IMPORT TEMPLATE:
[podman | docker] exec -it <container_name> <PATH_TO_KEYCLOAK_IN_THE_POD>/bin/standalone.sh
	 -Djboss.socket.binding.port-offset=100
	 -Dkeycloak.migration.action=import
	 -Dkeycloak.migration.provider=singleFile
	 -Dkeycloak.migration.realmName=quarkus
	 -Dkeycloak.migration.usersExportStrategy=REALM_FILE
	 -Dkeycloak.migration.file=<FILE_TO_IMPORT>
	 -Dkeycloak.profile.feature.upload_scripts=enabled
	 -Dkeycloak.profile.feature.scripts=enabled
	 -Dkeycloak.migration.strategy=[OVERWRITE_EXISTING | IGNORE_EXISTING]
