This will NOT work without first reading the [**Crowdsec Integration**](https://blog.foxxmd.dev/posts/migrating-to-traefik/#crowdsec-integration) section.

For the included configs the best approach is to start each crowdsec sec with empty bind mount folders *first* so that crowdsec generates all the default configs. Then copy in each included config to the populated folders.

Make sure that crowdsec_ingest `/etc/crowdsec/config.yaml` has `api.server.enable: false`