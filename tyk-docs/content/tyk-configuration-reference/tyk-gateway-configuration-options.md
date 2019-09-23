   ---
date: 2017-03-27T12:51:44+01:00
title: Tyk Gateway Configuration Options
menu:
  main:
    parent: "Tyk Configuration Reference"
weight: 1 
---

## <a name="configuration"></a> Tyk Gateway Configuration


The Tyk Gateway server is configured primarily via the `tyk.conf` file, this file resides in `/opt/tyk-gateway` on most systems, but can also live anywhere and be directly targeted with the `--conf` flag.

### Environment Variables

Environment variables can be used to override the settings defined in the configuration file. See [Environment Variables](docs/tyk-configuration-reference/environment-variables/) for details. Where an environment variable is specified, its value will take precedence over the value in the configuration file.

### <a name="linter"></a> tyk lint
In **v2.4** we have added a new `tyk lint ` command which will validate your `tyk.conf` file and validate it for syntax correctness, misspelled attribute names or format of values. The Syntax can be:

`tyk lint` or `tyk --conf=path lint`

If `--conf` is not used, the first of the following paths to exist is used:

`./tyk.conf`
`/etc/tyk/tyk.conf`

### <a name="listen_port"></a> listen_port

Setting this value will change the port that Tyk listens on, by default Tyk will try to listen on port 8080.

### <a name="node_secret"></a> node_secret

The shared secret between the Gateway and the Dashboard to ensure that API Definition downloads, heartbeat and Policy loads are from a valid source.

### <a name="secret"></a> secret

This should be changed as soon as Tyk is installed on the system. This value is used in every interaction with the Tyk REST API, it should be passed along as the `X-Tyk-Authorization` header in any requests made. Tyk assumes that you are sensible enough not to expose the management endpoints to the public and to keep this configuration value to yourself.

### <a name="template_path"></a> template_path

This is the path to the Tyk templates, as of the current version there is only one template `error.json` which determines the output of any errors that are returned by Tyk to any clients attempting to connect (with the exception of the OAuth flow, which has hard-coded templates).

### <a name="app_path"></a> app_path

If Tyk is being used in its standard configuration (CE Mode), then API definitions are stored in the apps folder (by default in `/opt/tyk-gateway/apps`). This file is scanned for files that ending in `.json` extension and interpreted at startup or reload. See the APIs section of the [Tyk Gateway API Swagger/OpenAPI documentation](/docs/tyk-gateway-api/) for more details.

### <a name="hash_keys"></a> hash_keys

Set this value to `true` to enable key hashing, this will start hashing all keys that are generated by Tyk in Redis. Enabling this means that keys that are created are only exposed once, during creation. If the key is lost or misplaced after this it will not be retrievable (however its hash can still be deleted, if known).

If this is set to `true`, the same value should be enabled in the Dashboard configuration so that the UI can react appropriately.

### <a name="enable_hashed_keys_listing"></a>enable_hashed_keys_listing

Set to `false` by default, set this to `true` to enable the retrieval all (or per API) key hash listings.

### <a name="allow_master_keys"></a> allow_master_keys

If this value is set to `true`, session objects (key definitions) that do not have explicit access rights set will be allowed by Tyk. This means that keys that are created have access to ALL APIs, which in many cases is unwanted behaviour unless you are sure about what you are doing.

### <a name="min_key_length"></a>min_key_length

This allows you to set a minimum key length for Authorisation key requests. Any request containing less than the minimum length will automatically rejected. The default setting is `3`. 

### <a name="use_db_app_configs"></a> use_db_app_configs

If you are a Tyk Pro user, this option will enable polling the Dashboard service for API definitions. On startup Tyk will attempt to connect and download any relevant application configurations from from a Dashboard instance. The documents are exactly the same as the JSON configuration on disk with the exception of a BSON ID supplied by the service.

### <a name="db_app_conf_options"></a> db_app_conf_options

This section defines API loading and shard options, enable these settings to only selectively load API definitions on a node from a Dashboard service.

#### <a name="db_app_conf_options-connection_string"></a> db_app_conf_options.connection_string

Set the URL to your Dashboard instance (or a load balanced instance), the URL should take the form of: `http://dashboard_host:port`

> **Note**: the same will need to be done for Policy loading, see the policies section below.


#### <a name="db_app_conf_options-node_is_segmented"></a> db_app_conf_options.node_is_segmented

Set to `true` to enable filtering (sharding) of APIs.

> **Note**: If you set to `true` for multiple gateway nodes, you should ensure that `management_node` is set to `false`. This is to ensure visibility for the management node across all APIs. 

#### <a name="db_app_conf_options-tags"></a> db_app_conf_options.tags

The tags to use when filtering (sharding) Tyk Gateway nodes, tags are processed as OR operations. If you include a non-filter tag (e.g. an identifier such as `node-id-1`, this will become available to your analytics Dashboard).

### <a name="storage"></a> storage

This section of the configuration is to hold the Redis configuration.

#### <a name="storage-type"></a> storage.type

This should be set to `"redis"` (lowercase), the previous backup and testing option of `csv` is deprecated.

#### <a name="storage-host"></a> storage.host

The Redis host, by default this is set to `localhost`, but for production this should be set to a cluster.

#### <a name="storage-port"></a> storage.port

The Redis instance port.

#### <a name="storage-password"></a> storage.password

If your Redis instance has a password set for access, you can tell Tyk about it here.

#### <a name="storage-timeout"></a> storage.timeout

Set cutom timeout for Redis network operations. Default value 5 seconds.

#### <a name="storage-optimisation_max_idle"></a> storage.optimisation_max_idle

Set the number of maximum idle connections in the Redis connection pool, defaults to 100, set to higher if expecting more traffic.

#### <a name="storage-use_ssl"></a> storage.use_ssl

Enable SSL/TLS connection between Tyk Gateway &amp; Redis.

#### <a name="enable_analytics"></a> enable_analytics

Tyk is capable of recording every hit to your API into a database with various filtering parameters, set this value to `true` and fill in the sub-section below to enable logging.

> **Note**: For performance reasons, Tyk will store traffic data to Redis initially and then purge the data from Redis to  MongoDB or other, [data stores](/docs/analytics-and-reporting/other-data-stores/), on a regular basis as determined by the `purge_delay` setting in your Tyk Pump configuration.

### <a name="analytics_config"></a> analytics_config

This section defines options on what analytics data to store.

#### <a name="analytics_config-enable_detailed_recording"></a> analytics_config.enable_detailed_recording

Set this value to `true` to have Tyk store the inbound request and outbound response data in HTTP Wire format as part of the Analytics data. Please note, this will greatly increase your analytics DB size and can cause performance degradation on analytics processing by the Dashboard. This setting can be overridden with an organisation flag.

Setting `enforce_org_data_detail_logging` in the `tyk.conf` will enforce it (quotas must also be enforced for this to work), then setting `enable_detail_recording` in the org session object will enable or disable the logging method on a per-organisation basis. This can be useful for debugging live APIs.

#### <a name="analytics_config-enable_geo_ip-enable_geo_ip_db_path"></a> analytics_config.enable_geo_ip and analytics_config.geo_ip_db_path

##### <a name="enable_geo_ip"></a> enable_geo_ip

As of Tyk API Gateway 2.0, Tyk can store GeoIP information based on MaxMind DB's, to enable GeoIP tracking on inbound request analytics, set this value to `true` and assign a DB using the `geo_ip_db_path` setting.

Please make sure you have also enabled analytics storing by setting [`enable_analytics`](/docs/tyk-configuration-reference/tyk-gateway-configuration-options/#a-name-enable-analytics-a-enable-analytics) in the Gateway.

##### <a name="enable_geo_ip_db_path"></a> geo_ip_db_path

Set this value to the absolute path of your MaxMind GeoIP Database file, e.g.: `./GeoLite2-City.mmdb`. The analytics GeoIP DB can be replaced on disk, it will cleanly auto-reload every hour. 

Don't forget to mount it in case you use containers.

#### <a name="analytics_config-ignored_ips"></a> analytics_config.ignored_ips

Adding IP addresses to this list will cause Tyk to ignore these IPs in the analytics data, these IP addresses will not produce an analytics log record. This is useful for health checks and other samplers that might skew usage data. The IP addresses must be provided as a JSON array, with the values being single IPs. CIDR values are not supported.This is useful for health checks and other samplers that might skew usage data.

#### <a name="analytics_config-normalise_urls"></a> analytics_config.normalise_urls

This section describes methods that enable you to normalise inbound URLs in your analytics so as to have more meaningful per-path data.

#### <a name="analytics_config-normalise_urls-enabled"></a> analytics_config.normalise_urls.enabled

Set this to `true` to enable normalisation.

#### <a name="analytics_config-normalise_urls-normalise_uuids"></a> analytics_config.normalise_urls.normalise_uuids

Set this to `true` to have Tyk automatically clean up UUIDs, it will match the following styles:

* `/15873a748894492162c402d67e92283b/search`
* `/CA761232-ED42-11CE-BACD-00AA0057B223/search`
* `/ca761232-ed42-11ce-BAcd-00aa0057b223/search`
* `/ca761232-ed42-11ce-BAcd-00aa0057b223/search`

Each UUID will be replaced with a placeholder `{uuid}`

#### <a name="analytics_config-normalise_urls-normalise_numbers"></a> analytics_config.normalise_urls.normalise_numbers

Set this to `true` to have Tyk automatically match for numeric ID's, it will match with a preceding slash so as not to capture actual numbers:

* `/widgets/123456/getParams will become /widgets/{id}/getParams`

#### <a name="analytics_config-normalise_urls-custom_patterns"></a> analytics_config.normalise_urls.custom_patterns

This is a list of custom patterns you can add, these must be valid regex strings, Tyk will replace these values with a `{var}` placeholder.

#### <a name="analytics_config.storage_expiration_time"></a>analytics_config.storage_expiration_time

You can set a time (in seconds) to configure how long analytics are kept if they are not processed. The default is 60 seconds. This is used to prevent the potential infinite growth of Redis analytics storage.

> **NOTE:** This option is available from v2.5.2 onwards.

### <a name="policies"></a> policies

The policies section allows you to define where Tyk can find its policy templates. Policy templates are similar to key definitions in that they allow you to set quotas, access rights and rate limits for keys.

Policies are loaded when Tyk starts and if changed require a hot-reload so they are loaded into memory.

A policy can be defined in a file (stand alone mode) or from the same database as the Dashboard.

#### <a name="policies-policy_source"></a> policies.policy_source


Set this value to `file` to look on the file system for a definition file, set to service to use the Dashboard `service`. This value must be set. Note the option for `mongo` has now been removed.

#### <a name="policies-policy_connection_string"></a> policies.policy_connection_string

This option is required if you desire to use `policies.policy_source` as `service`. Set this to the URL of your dashboard installation, the URL should take the form of: `http://dashboard_host:port`.

#### <a name="policies-policy_record_name"></a> policies.policy_record_name

This option is required if you desire to use `policies.policy_source` as `file`.
Specifies the path of a JSON file containing the available policies.

### <a name="health_check"></a> health_check

This section enables the configuration of the health-check API endpoint and the size of the sample data cache (in seconds).

#### <a name="health_check-enable_health_checks"></a> health_check.enable_health_checks

Setting this value to `true` will enable the health-check endpoint on `/Tyk/health`.

#### <a name="health_check_endpoint_name"></a>health_check_endpoint_name

From v2.7.5 you can now rename the `/hello`  endpoint by using this option. 

#### <a name="health_check-health_check_value_timeouts"></a> health_check.health_check_value_timeouts

This setting defaults to `60`, this is the time window that Tyk will use to sample health-check data. Increase this value for more accurate data (larger sample period), and decrease for less accurate. The reason this value is configurable is because sample data takes up space in your Redis DB to store the data to calculate samples, on high-availability systems this may not be desirable and smaller values may be preferred.

### <a name="http_server_options"></a> http_server_options

Set these options to hard-code values into the way the HTTP server behaves.

```
"http_server_options": {
  "enable_http2": true,
  "override_defaults": false,
  "use_ssl": false,
  "enable_websockets": false,
  "flush_interval": 1,
  "certificates": [
    {
      "domain_name": "ssl.domain.com",
      "cert_file": "./certs/ssl.domain.com.cert",
      "key_file": "./certs/ssl.domain.com.cert.key"
    },
    {
      "domain_name": "cname.domain.com",
      "cert_file": "./certs/cert2/cname.domain.com.cert.cert",
      "key_file": "./certs/cert2/cname.domain.com.key"
    }
  ],
  "ssl_insecure_skip_verify": false
},
```

#### <a name="enable_http2"></a> enable_http2

This defaults to true for HTTP/2 connections.

#### <a name="http_server_options-use_ssl"></a> http_server_options.use_ssl

Set to `true` to enable SSL connections.

#### <a name="http_server_options-certificates"></a> http_server_options.certificates

A list of certificates and domains to match against. Please see the SSL section for more detail for this feature.

#### <a name="http_server_options-ssl_certificates"></a> http_server_options.ssl_certificates

Added in 2.4, as altertnative to `http_server_options.certificates`, which supports our [Certificate API](/docs/basic-config-and-security/security/tls-and-ssl/mutual-tls/#a-name-certificates-management-a-certificate-management) format. It should be a list of certificate IDs returned by Certificate API, or paths to certificate in PEM format (including private key). 

#### <a name="http_server_options-skip_url_cleaning"></a> http_server_options.skip_url_cleaning

Setting this option to true will allow the use of a double slash in url path, and can be useful if you need to pass raw URLs to your API endpoints. For example: `http://myapi.com/get/http://example.com`.

#### <a name="http_server_options-skip_target_path_escaping"></a> http_server_options.skip_target_path_escaping

Setting this option to true will disable automatic character escaping, allowing to path original url data to the upstream.

#### <a name="http_server_options-flush_interval"></a> http_server_options.flush_interval

Set this to the number of seconds that Tyk should use to flush content from the proxied upstream connection to the open downstream connection. This is usually required for HTTP Streams.

For more resilient connection management, it is suggested to use the `close_connections` option below.

`flush_interval` is set in **milliseconds**.

#### <a name="http_server_options-enable_websockets"></a> http_server_options.enable_websockets

As of v2.2, Tyk supports transparent websocket connection upgrades, to enable this feature, ensure that this value is set to `true`.

#### <a name="http_server_options-ssl_insecure_skip_verify"></a>http_server_options.ssl_insecure_skip_verify

Allows usage of self-signed certificates when connecting to the Gateway.

#### <a name="security.pinned_public_keys"></a> security.pinned_public_keys

Use this option to map pinned public keys. You need to use the following format:

```
{
  "example.com": "<key-id>",
  "foo.com": "/path/to/pub.pem",
  "*.wild.com": "<key-id>,<key-id-2>"
}
```

For `key-id` you should set the ID returned after you upload the public key using the Certificate API. Additionally, you can just set path to public key, located on your server. You can specify multiple public keys by separating their IDs by a comma. Upstream certificates now also have wildcard domain support

Note that only public keys in PEM format are supported.

> **NOTE:** This option is available from v2.6.0 onwards.

#### <a name="close_connections"></a> close_connections

Set this value to `true` to force Tyk to close the connection with the client, otherwise the connections will remain open for as long as your OS keeps TCP connections open. This can cause a file-handler limit to be exceeded. Setting to `false` can have performance benefits as the connection can be reused.

Prior to v2.6 this setting controlled the behaviour of both the client/Tyk connection and Tyk/server connection. Since v2.6 it is just for the client/Tyk connection, with Tyk/server being controlled by <a href='#proxy_close_connections'>proxy_close_connections</a>

> **NOTE:** This option is available from v2.3.5 onwards.

#### <a name="proxy_close_connections"></a> proxy_close_connections

Set this value to `true` to force Tyk to close the connection with the server, otherwise the connections will remain open for as long as your OS keeps TCP connections open. This can cause a file-handler limit to be exceeded. Setting to `false` can have performance benefits as the connection can be reused.

> **NOTE:** This option is available from v2.6 onwards.

### <a name="monitor"></a> monitor

The monitor section is useful if you wish to enforce a global trigger limit on organisation and user quotas. This feature will trigger a webhook event to fire when specific triggers are reached. Triggers can be global (set in the node), by organisation (set in the organisation session object) or by key (set in the key session object).

While Organisation-level and Key-level triggers can be tiered (e.g. trigger at 10%, trigger at 20%, trigger at 80%), in the node-level configuration only a global value can be set. If a global value and specific trigger level are the same the trigger will only fire once:

```
"monitor": {
  "enable_trigger_monitors": true,
  "configuration": {
    "method": "POST",
    "target_path": "http://domain.com/notify/quota-trigger",
    "template_path": "templates/monitor_template.json",
    "header_map": {
      "some-secret": "89787855"
    },
    "event_timeout": 10
  },
  "global_trigger_limit": 80.0,
  "monitor_user_keys": false,
  "monitor_org_keys": true
},
```
#### <a name="monitor-enable_trigger_monitors"></a> monitor.enable_trigger_monitors

Set this to `true` to have monitors enabled in your configuration for the node.

#### <a name="monitor-configuration-method"></a> monitor.configuration.method

The method to use for the webhook.

#### <a name="monitor-configuration-target_path"></a> monitor.configuration.target_path

The target path on which to send the request.

#### <a name="monitor-configuration-template_path"></a> monitor.configuration.template_path

The template to load in order to format the request.

#### <a name="monitor-configuration-header_map"></a> monitor.configuration.header_map

Headers to set when firing the webhook.

#### <a name="monitor-configuration-event_timeout"></a> monitor.configuration.event_timeout

The cool-down for the event so it does not trigger again (in seconds).

#### <a name="monitor-global_trigger_limit"></a> monitor.global_trigger_limit

The trigger limit, as a percentage of the quota that must be reached in order to trigger the event, any time the quota percentage is increased the event will trigger.

#### <a name="monitor-monitor_user_keys"></a> monitor.monitor_user_keys

Apply the monitoring subsystem to user keys.

#### <a name="monitor-monitor_org_keys"></a> monitor.monitor_org_keys

Apply the monitoring subsystem to organisation keys.

### <a name="uptime_tests"></a> uptime_tests

Tyk nodes can provide uptime awareness, uptime testing and analytics for your underlying APIs uptime and availability.

Tyk can also notify you when a service goes down.

#### <a name="uptime_tests-disable"></a> uptime_tests.disable

To disable uptime tests on this node, switch this value to `true`.

#### <a name="uptime_tests-config"></a> uptime_tests.config

The configuration section for the uptime tests on this node.

#### <a name="uptime_tests-config-failure_trigger_sample_size"></a> uptime_tests.config.failure_trigger_sample_size

Set this value to `true` to have the node capture and record analytics data regarding the uptime tests.

The sample size to trigger a `HostUp` or `HostDown` event, e.g. a setting of `3` will require at least three failures to occur before the uptime test is triggered.

#### <a name="uptime_tests-config-time_wait"></a> uptime_tests.config.time_wait

The amount of seconds between tests runs, all tests will run simultaneously, this value will set the periodicity between those tests. e.g. A value of `60` will run all uptime tests every 60 seconds.

#### <a name="uptime_tests-config-checker_pool_size"></a> uptime_tests.config.checker_pool_size

The goroutine pool size to keep idle for uptime tests, if you have many uptime tests running at a high periodicity, then make this value higher.

### <a name="local_session_cache"></a> local_session_cache

Tyk can cache some data locally, this can speed up lookup times on a single node and lower the number of connections and operations being done on Redis, it will however introduce a slight delay when updating or modifying keys as the cache must expire.

This does not affect rate limiting.

#### <a name="local_session_cache-disable_cached_session_state"></a> local_session_cache.disable_cached_session_state

By default sessions are set to cache, set this to true to stop Tyk from caching keys locally on the node.

### <a name="allow_explicit_policy_id"></a> allow_explicit_policy_id

As of v2.1, by default, in a Pro installation, Tyk will load Policy IDs and use the internal object-ID as the ID of the policy. This is not portable in cases where the data needs to be moved from installation to installation.

If you set this value to `true`, then the `id` parameter in a stored policy (or imported policy using the REST API of the Dashboard), will be used instead of the internal ID.

This option should only be used when transporting an installation to a new database.

### <a name="oauth_token_expire"></a> oauth_token_expire

Change the expiry time of OAuth token (in seconds).

### <a name="oauth_refresh_token_expire"></a> oauth_refresh_token_expire

Change the expiry time of refresh token, by default 1 hour (in seconds).

### <a name="oauth_token_expired_retain_period"></a>oauth_token_expired_retain_period

Specifies how long expired tokens are stored in Redis. The value is in seconds and the default is `0`. Using the default means expired tokens are never removed from Redis.

> **NOTE:** This option is available from v2.6.0 onwards.

### <a name="control_api_hostname"></a> control_api_hostname

The hostname to bind the REST API to.

### <a name="control-api"></a>control_api_port

This allows you to run the Gateway Control API on separate port, and protect it behind a firewall if needed.
Please make sure you follow these [instructions](/docs/planning-for-production/#change-your-control-port) when setting the control port.

> **Note:** This option is available from v2.4 onwards.

### <a name="enable_api_segregation"></a> enable_api_segregation

For additional security it is possible to have Tyk put its REST API on a separate hostname, this means that all calls to the `/api` functions must go via this hostname for this node otherwise they will 404. This is useful if you do not wish to expose the Tyk API to proxy API users.

> **NOTE**: This has been deprecated. Enter a value for `control_api_hostname` instead.

### <a name="hostname"></a> hostname

The hostname to bind the node to. If set, all API traffic must go via this host name, otherwise it will raise a 404.

### <a name="enable_custom_domains"></a> enable_custom_domains

Set this value to `true` to enable this node to bind APIs to custom domains set in the API definition.

### <a name="proxy_enable_http2"></a> proxy_enable_http2

This defaults to `true` for HTTP/2 upstream connections.


### <a name="enable_jsvm"></a> enable_jsvm

By default we have now disabled the JavaScript middleware system to ensure higher performance on nodes. If you are using the JSVM (custom middleware, or virtual endpoints), then enable this setting.

### <a name="disable_virtual_path_blobs"></a> disable_virtual_path_blobs

If you do not wish for virtual path JavaScript code that is loaded from the dashboard to run on virtual endpoints in the node, set this value to `true` and the code will not be loaded into the VM when the API definition initialises. This is useful for systems where you want to avoid having third-party code run.

### <a name="experimental_process_org_off_thread"></a> experimental_process_org_off_thread

Setting this option to `true` will cause the organisation quota checker to handle quota limits off the main request pipeline, this can provide a speed boost in some situations. enabling this will cause quotas to be enforced by a "lock", in order to re-access the API once the quota has reset, the lock must be broken by sending one failed request, this will cause the sentinel to be disabled and allow traffic to pass normally again.

### <a name="enable_sentinel_rate_limiter"></a> enable_sentinel_rate_limiter

To enable, set to `true`. The sentinel-based rate limiter provides a smoother performance curve as rate-limit calculations happen off-thread, but a stricter time-out based cool-down for clients, i.e. when a throttling action is triggered, they are required to cool-down for the period of the rate limit.

Disabling the sentinel based limiter will make rate-limit calculations happen on-thread and therefore offers staggered cool-down and a smoother rate-limit experience for the client. I.e. the client can slow their connection throughput to regain entry into their rate limit, this is more of a "throttle" than a "block". The standard rate limiter offers similar performance as the sentinel-based limiter. This is disabled by default.

### <a name="enable_non_transactional_rate_limiter"></a> enable_non_transactional_rate_limiter

This will be enabled by default. This is a new improved rate limiter that offers a significant improvement in performance by not using transactions on Redis rate-limit buckets. To go back to the old rate limiter, simply set this value to `false`.

### <a name="enforce_org_data_detail_logging"></a> enforce_org_data_detail_logging

Set this value to true to make it possible to have detail logging occur on a per-organisation basis, org quotas must also be enabled for this to work. Setting `enable_detail_recording` in the org session object will enable or disable the logging method on the fly.

### <a name="enforce_org_quotas"></a> enforce_org_quotas

Set this value to `true` to have the Tyk node enforce Organisation quotas on the APIs it is managing.

### <a name="enforce_org_data_age"></a> enforce_org_data_age

Set to `true` to have Tyk enforce data age on analytics data sent to the Mongo DB purger. This must be accompanied by an additional parameter to an organisation session object called `data_expires` which is larger than `0`, this will define in seconds when analytics records for the inbound request will expire.

### <a name="use_sentry"></a> use_sentry

Set this to `true` to enable the sentry logger, you must specify a sentry DSN under `sentry_code`.

### <a name="sentry_code"></a> sentry_code

The Sentry-assigned DSN (a kind of URL endpoint) that Tyk can send log data to.

### <a name="suppress_redis_signal_reload"></a> suppress_redis_signal_reload

Tyk v1.6 will auto-reload when a change is detected when using the Dashboard, this used to be done by the host-manager but is now directly integrated into the Gateway. For backwards compatibility, this behaviour can be suppressed by setting this value to `true`.

### <a name="optimisations_use_async_session_write"></a> optimisations_use_async_session_write

Set this value to `true` to have Tyk manage session data using a goroutine, this is quite safe and can significantly boost performance in HA environments where Tyk is installed on a machine with multiple cores.

### <a name="disable_dashboard_zeroconf"></a> disable_dashboard_zeroconf

Disable the capability of the Gateway to "autodiscover" the Dashboard through heartbeat messages via Redis. The goal of zeroconf is auto-discovery, so you do not have to specify the Tyk Dashboard address in `tyk.conf`. In some specific cases, for example, when the Dashboard is bound to a public domain, not accessible inside an internal network, or similar, `disable_dashboard_zeroconf` can be set to true, in favour of directly specifying a Tyk Dashboard address.

### <a name="auth_override"></a> auth_override

Is used as part of the RPC / Hybrid back-end configuration when using Tyk Enterprise and isn't used anywhere else.

### <a name="use_redis_log"></a> use_redis_log

Enables the real-time Gateway log view in the Dashboard.


### <a name="management_node"></a> management_node

If set to `true`, distributed rate limiter will be disabled for this node, and it will be excluded from rate limit calculation.

> **Note**: If you set `db_app_conf_options.node_is_segmented` to `true` for multiple gateway nodes, you should ensure that `management_node` is set to `false`. This is to ensure visibility for the management node across all APIs. 

> This option is available from v2.3.4 and onwards.

### <a name="proxy_ssl_insecure_skip_verify"></a> proxy_ssl_insecure_skip_verify

This boolean option allows you to skip SSL checking for upstream APIs with self-signed certificates. The default setting is false. 

> **NOTE:** This option is available from v2.3.5 onwards.

### <a name="proxy_default"></a>proxy_default_timeout

This can specify a default timeout in seconds for upstream API requests.

> **NOTE:** This option is available from v2.3.8 onwards.

### <a name="log-level"></a>log_level
You can now set a logging level via the `tyk.conf` file. The following levels can be set:
* debug
* info
* warn
* error
If unset or left empty, it will default to `info`.

> **NOTE:** This option is available from v2.4 onwards.

### <a name="enable-key-logging"></a>enable_key_logging
By default all key ids in logs are hidden. Turn it on if you want to see them for debugging reasons.

> **NOTE:** This option is available from v2.5 onwards.

### <a name="max_conn_time"></a>max_conn_time

This setting forces a DNS cache flush (in seconds). The default setting is `0`.

> **NOTE:** This option is available from v2.5.2 onwards.

### <a name="proxy_ssl_min_version"></a>proxy_ssl_min_version

You use this setting to have Tyk only accept connections from TLS V1.0, 1.1 and 1.2 respectively.

You need to use the following values for this setting:

| TLS Version   | Value to Use   |
|---------------|----------------|
|      1.0      |      769       |
|      1.1      |      770       |
|      1.2      |      771       |

### <a name="proxy_ssl_ciphers"></a>proxy_ssl_ciphers

This allows you to add ssl ciphers which takes an array of strings as its value.

Each string must be one of the allowed cipher suites as defined at https://golang.org/pkg/crypto/tls/#pkg-constants

### <a name="proxy_ssl_disable_renegotiation"></a>proxy_ssl_disable_renegotiation

From v2.7.2, TLS renegotiation is now enabled by default. You can disable it by setting `proxy_ssl_disable_renegotiation` to `false`.

### <a name="disable_regexp_cache"></a>disable_regexp_cache

If set to `true` this allows you to disable the regular expression cache. The default setting is `false`.

> **NOTE:** This option is available from v2.7.0 onwards.

### <a name="regexp_cache_expire"></a>regexp_cache_expire

If you set `disable_regexp_cache` to false, you can use this setting to limit how long the regular expression cache is kept for in seconds. The default is `60` seconds. This must be a positive value. If you set to `0` this sets it uses the default value.

> **NOTE:** This option is available from v2.7.0 onwards.
