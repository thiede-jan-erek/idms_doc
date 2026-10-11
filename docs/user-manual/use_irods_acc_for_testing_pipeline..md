# Use iRODS acceptance for testing pipelines
If you want to test a pipeline in iDMS, it is highly recommended to test in the acceptance environment. This keeps the production environment nice and tidy. In iDMS (biorods.rivm.nl) you can logout of production and choose the **acceptance** environment to login.

It might be that you need to (re)create a project with (test) data. For uploading data to your project you will need a metadata schema. You can ask BIR to create these.

# Switch from irods production to irods acceptance in the Scientific Workplace
For approaching acceptance from the scientific workplace (wetenschappelijke werkplek), you can follow the steps below.
 
You need to replace **irods_host**, **irods_zone_name** and the **irods_ssl_\*** paths in the irods config file `~/.irods/irods_environment.json`.

| attribute | value production| value acceptance | remark |
| --------- | --------------- | ---------------- | ------ |
| irods_host | rivm-bioir-l04p.rivm.ssc-campus.nl | rivm-bioir-l05a.rivm.ssc-campus.nl | iRODS producer |
| irods_zone_name | rivmZone | rivmZone_acc_01 | |
| irods_ssl_ path | /etc/irods/ssl | /etc/irods/ssl/acc | irods_ssl_certificate_chain_file,<br> irods_ssl_dh_params_file and<br> irods_ssl_ca_certificate_file|

example:

```
$ ils
/rivmZone/home/verhager:
  3a_env_management.lecture.html
  README.md
...

$ cat .irods/irods_environment.json
{
    "irods_authentication_scheme": "pam_password",
    "irods_client_server_negotiation": "request_server_negotiation",
    "irods_default_resource": "storageResc",
    "irods_encryption_algorithm": "AES-256-CBC",
    "irods_encryption_key_size": 32,
    "irods_encryption_num_hash_rounds": 16,
    "irods_encryption_salt_size": 8,
    "irods_host": "rivm-bioir-l04p.rivm.ssc-campus.nl",
    "irods_port": 1247,
    "irods_ssl_ca_certificate_file": "/etc/irods/ssl/irods.crt",
    "irods_ssl_certificate_chain_file": "/etc/irods/ssl/irods.crt",
    "irods_ssl_dh_params_file": "/etc/irods/ssl/dhparams.pem",
    "irods_ssl_verify_server": "cert",
    "irods_user_name": "verhager",
    "irods_zone_name": "rivmZone"
}

$ cp ~/.irods/irods_environment.json_acc ~/.irods/irods_environment.json

$ ils
[-]  /repos/irods/plugins/auth/native/libnative.cpp:342:irods::error native_auth_client_response(irods::plugin_context &, rcComm_t *) :  status [CAT_INVALID_AUTHENTICATION]  errno [] -- message [Call to rcAuthResponseFailed.]

failed with error -826000 CAT_INVALID_AUTHENTICATION 

$ iinit
Enter your current PAM password:

$ ils
/rivmZone_acc_01/home/verhager:
  1
  10
  11
  12
  13
  14
  15
  16
  17
  18
...

$ cat ~/.irods/irods_environment.json
{
    "irods_authentication_scheme": "pam_password",
    "irods_client_server_negotiation": "request_server_negotiation",
    "irods_encryption_algorithm": "AES-256-CBC",
    "irods_encryption_key_size": 32,
    "irods_encryption_num_hash_rounds": 16,
    "irods_encryption_salt_size": 8,
    "irods_host": "rivm-bioir-l05a.rivm.ssc-campus.nl",
    "irods_port": 1247,
    "irods_ssl_ca_certificate_file": "/etc/irods/ssl/irods.crt",
    "irods_ssl_certificate_chain_file": "/etc/irods/ssl/irods.crt",
    "irods_ssl_dh_params_file": "/etc/irods/ssl/dhparams.pem",
    "irods_ssl_verify_server": "cert",
    "irods_user_name": "verhager",
    "irods_zone_name": "rivmZone_acc_01"
}
```
