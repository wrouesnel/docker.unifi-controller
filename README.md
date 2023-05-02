# Execution Requirements
The container is designed to work with a read-only filesystem.

It requires `/data` to be mounted read-write, and `/run` and `/tmp` to be
writeable but not persistent (i.e. `--tmpfs /run:exec,suid` in `docker run`).

## Deployment

The Unifi Controller depends on Layer 2 connectivity in order to detect Unifi
devices on the local network. For deployment, you need to create a `macvlan`
network on your docker host so provide direct connectivity (or run the container
as `--net=host` but this is much less preferable).

It is possible to manage SSL certificates in a container, but its a lot easier
to use an nginx proxy (and incorporate LetsEncrypt into your setup for
security):

### Example Configuration Stanzas
Deploying the unifi controller via Ansible:
```yaml
  - name: ensure internal private docker network exists
    docker_network:
      name: private
      force: true
      state: present
      driver: bridge
      ipam_driver: default
      ipam_options:
        subnet: 172.20.0.1/24

  - name: ensure docker public network exists
    docker_network:
      name: public
      force: true
      state: present
      driver: macvlan
      driver_options:
        # Adjust as necessary obviously.
        parent: eth0
      ipam_driver: default
      ipam_options:
        subnet: 192.168.1.0/24
        # Assuming your router is probably on .1, docker will only assign .16-.30
        iprange: 192.168.1.16/28

  - name: deploying unifi controller container
    docker_container:
      name: unifi-controller
      image: ghcr.io/wrouesnel/unifi-controller
      state: started
      pull: true
      purge_networks: yes
      networks:
      # The public network will be reachable on your network - but due to how
      # macvlan works the host will not be reachable.
      - name: public
        ipv4_address: 192.168.1.17
      # So we have the private network, which is how nginx talks to the private
      # interface.
      - name: private
        ipv4_address: 172.20.0.10
      volumes:
      - "{{ unifi_controller.data_dir }}:/data"
      - "{{ ssl_certificates.cert_root }}:{{ ssl_certificates.cert_root }}:ro"
      tmpfs:
      - /run:exec,suid
      - /tmp:exec,suid
```
The nginx serer stanza:
```
server {
	listen 443;
	listen [::]:443;
	
	server_name unifi unifi.mydomain.example.com;
	
	ssl on;
	# Lego is scripted elsewhere to manage certificates.
	ssl_certificate /etc/lego/certificates/mydomain.example.com.crt;
	ssl_certificate_key /etc/lego/certificates/mydomain.example.com.key;
	ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    
    client_max_body_size 768m;
    
    location / {
        proxy_pass https://172.20.0.10:8443;
        
        # Note: this is intentional since the unifi controller always generates
        # its own SSL certificate, but we are running it locally.
        proxy_ssl_verify off;
        proxy_cache off;
        proxy_store off;
        
        proxy_set_header Host            $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto http;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_buffering off;
        proxy_redirect off;
        proxy_http_version 1.1;
        
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
```
