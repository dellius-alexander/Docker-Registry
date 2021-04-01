# Deploy Private docker registry

---

## Native Basic Auth

<br/>

1. Create a secret file to be used to validate username & password.
    - This example uses native basic authentication using `htpasswd` to store the secrets. 
        - Install package `httpd-tools`, as `htpasswd` is part of this package
    - Create a `auth` directory for your secret.
        ```bash
        $  mkdir auth
        ```
    - Run the command:
        - htpasswd -Bbn \<username> "\<password>" > auth/htpasswd
        ```bash
        $ htpasswd -Bbn <username> "<password>" > auth/htpasswd
        $ cat auth/htpasswd
        dalexander:<Redacted Password>
        ```

2. Create a `certs` directory and copy the .crt and .key files from the CA into the certs directory. The following steps assume that the files are named domain.crt and domain.key.
    - Get a registered domain/URL like: `https://myregistry.domain.com/`
    - Your DNS, routing, and firewall settings allow access to the registry’s host on port 5000/443. This example uses port 5000.
    - You have already obtained a certificate from a certificate authority (CA)

3. Start the registry with basic authentication via docker-compose file.

    - registry.docker-compose.yaml

        ```yml
        version: '3.8'
        services: 
          registry:
            image: registry:2.7.1
            container_name: registry
            environment: 
                # Set Registry Environment
              REGISTRY_AUTH: "htpasswd"
              REGISTRY_AUTH_HTPASSWD_REALM: "Registry Realm"
              REGISTRY_AUTH_HTPASSWD_PATH: "/auth/htpasswd"
              REGISTRY_HTTP_TLS_CERTIFICATE: "/certs/_.dellius.app.crt"
              REGISTRY_HTTP_TLS_KEY: "/certs/_.dellius.app.key"
            volumes: 
              - ${PWD}/auth:/auth
              - ${PWD}/certs:/certs
              # persist registry data externally
              - ${PWD}/data:/var/lib/registry
            restart: on-failure
            expose: 
              - 5000
            ports: 
              - 5000:5000
            networks: 
              registry-network:
                ipv4_address: 172.27.0.3
        networks: 
          registry-network:
            driver: bridge
            ipam: 
              driver: default
              config: 
                - subnet: 172.27.0.0/28
        ```
    - Run the command to start your registry
        ```bash
          # Start docker registry
        $ docker-compose -f registry.docker-compose.yaml up -d && \
          # for logging purposes
          docker-compose -f registry.docker-compose.yaml logs -f
        ```
4. Try to pull an image from the registry, or push an image to the registry. These commands should fail.
    ```bash
    $ docker push myregistry.domain.com:5000/nginx:1.19.5
    The push refers to repository [myregistry.domain.com:5000/nginx]
    ea6033164031: Preparing 
    997bdb5b26cc: Preparing 
    f3ee98cb305c: Preparing 
    2111bafa5ce4: Preparing 
    87c8a1d8f54f: Preparing 
    no basic auth credentials

    ```
5. Login into the registry.

    ```bash
    $ docker login myregistry.domain.com:5000
    Username: dalexander
    Password: 
    WARNING! Your password will be stored unencrypted in /path/to/docker/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store
    
    Login Succeeded
    
    ```

**Thats it your are done...**