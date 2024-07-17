#  Assumptions

- The deployment directory is: `/opt/deployment/`
- The applications, config, and related files are staged in the deployment directory:
```{.console .no-copy}
/opt/deployment/keycloak/
/opt/deployment/stigman/
/opt/deployment/mysql/
/opt/deployment/nginx/
/opt/deployment/docs/
```
-  STIG Manager version is: `{{ stigman.version }}`
-  keycloak version is: `{{ keycloak.version }}`
-  The server hostname is: `{{ server.hostname }}`
-  Realm name is: `{{ realm.name }}`
-  User is logged in as a user with root privileges on the system.
