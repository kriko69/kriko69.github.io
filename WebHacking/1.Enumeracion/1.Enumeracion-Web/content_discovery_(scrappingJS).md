# CONTENT DISCOVERY

Tecnicas para decubrir directorios y endpoints con web scrapping

## Table Obsidian

- [[#Table of contents|Table of contents]]
- [[#DICCIONARIO PARA ENCONTRAR VULNERABILIDADES WEB|DICCIONARIO PARA ENCONTRAR VULNERABILIDADES WEB]]
- [[#TIPS|TIPS]]
- [[#SCRAPPING JS|SCRAPPING JS]]
	- [[#DIRSCRAPER|DIRSCRAPER]]
	- [[#EXTRACT ENDPOINT FROM ANGULAR Y ANGULARJS|EXTRACT ENDPOINT FROM ANGULAR Y ANGULARJS]]
	- [[#JSEARCH|JSEARCH]]
	- [[#LINKFINDER|LINKFINDER]]
		- [[#collector.py|collector.py]]
	- [[#jsAlert.py & getScriptTagContent.py|jsAlert.py & getScriptTagContent.py]]
	- [[#getJSWords.py|getJSWords.py]]
	- [[#getSrc|getSrc]]
	- [[#availableForPurchase.py|availableForPurchase.py]]
	- [[#SECRET FINDER|SECRET FINDER]]
		- [[#SECRETFINDER + KATANA|SECRETFINDER + KATANA]]
	- [[#KATANA|KATANA]]
	- [[#SUBJS|SUBJS]]
	- [[#GETALLURLS (GAU)|GETALLURLS (GAU)]]
	- [[#GXSS|GXSS]]
		- [[#GXSS + GAU|GXSS + GAU]]
	- [[#DALFOX|DALFOX]]
		- [[#GAU + GXSS + DALFOX|GAU + GXSS + DALFOX]]
	- [[#automatizacion con GAU|automatizacion con GAU]]
- [[#GREPEAR LISTADO DE URLS (PARA NUESTROS OUTPUTS)|GREPEAR LISTADO DE URLS (PARA NUESTROS OUTPUTS)]]
- [[#RECURSOS|RECURSOS]]

## Table of contents

- [CONTENT DISCOVERY](#content-discovery)
  - [DICCIONARIO PARA ENCONTRAR VULNERABILIDADES WEB](#diccionario-para-encontrar-vulnerabilidades-web)
  - [TIPS](#tips)
  - [SCRAPPING JS](#scrapping-js)
    - [DIRSCRAPER](#dirscraper)
    - [RELATIVE URL EXTRACTOR (RUBY)](#relative-url-extractor-ruby)
    - [EXTRACT ENDPOINT FROM ANGULAR Y ANGULARJS](#extract-endpoint-from-angular-y-angularjs)
    - [JSEARCH](#jsearch)
    - [LINKFINDER](#linkfinder)
    - [SECRET FINDER](#secret-finder)
    - [SUBJS](#subjs)
    - [WAYBACKURLS](#waybackurls)
    - [CHECK BROKEN LINKS](#check-broken-links)
    - [DIRSEARCH](#dirsearch)
    - [GETALLURLS (GAU)](#getallurls-gau)
    - [HAKRAWLER](#hakrawler)
    - [GREPEAR LISTADO DE URLS (PARA NUESTROS OUTPUTS)](#grepear-listado-de-urls-para-nuestros-outputs)
  - [AUTOMATIZAR TODO CON RECONFTW](#automatizar-todo-con-reconftw)

## DICCIONARIO PARA ENCONTRAR VULNERABILIDADES WEB

- [https://github.com/fuzzdb-project/fuzzdb](https://github.com/fuzzdb-project/fuzzdb)

## TIPS

```bash
# Podemos reemplazar los caracteres especiales por los siguientes
# o agregar estos bytes al fuzzear
# puede resultar en un regex bypass, account takeover
0x00, 0x2F, 0x3A, 0x40, 0x5B, 0x60, 0x7B, 0xFF
%00, %2F, %3A, %40, %5B, %60, %7B, %FF
```

## SCRAPPING JS

### DIRSCRAPER

- [https://github.com/Cillian-Collins/dirscraper](https://github.com/Cillian-Collins/dirscraper)

>[!note]
>Funciona con python 3

```bash
#Puede analizar y raspar el contenido de JavaScript en un sitio web de destino para buscar subdominios ocultos o rutas interesantes
#A menudo, los puntos finales no son públicos, pero los usuarios aún pueden interactuar con ellos.

git clone https://github.com/Cillian-Collins/dirscraper

# Clasico
python discraper.py -u <url>

# Especificando un Output
python discraper.py -u <url> -o <output>

# Modo silencioso (no muestra resultado en consola)
python discraper.py -u <url> -s -o <output>
```

![[Pasted image 20230605155724.png]]

### EXTRACT ENDPOINT FROM ANGULAR Y ANGULARJS

```bash
# Extract all API endpoints from AngularJS & Angular javascript files
curl -s URL | grep -Po "(\/)((?:[a-zA-Z\-_\:\.0-9\{\}]+))(\/)*((?:[a-zA-Z\-_\:\.0-9\{\}]+))(\/)((?:[a-zA-Z\-_\/\:\.0-9\{\}]+))" | sort -u
```

### JSEARCH

- [https://github.com/incogbyte/jsearch](https://github.com/incogbyte/jsearch)

```bash
git clone https://github.com/incogbyte/jsearch

# simple script que extraa archivos JS
python3 -m pip install -r requirements.txt --user

python3 jsearch.py -u https://google.com -n google
```

![[Pasted image 20230605160348.png]]

### LINKFINDER

- [https://github.com/GerbenJavado/LinkFinder](https://github.com/GerbenJavado/LinkFinder)

```bash
git clone https://github.com/GerbenJavado/LinkFinder.git
cd LinkFinder
python setup.py install
```

```bash
# analiza un archivo y saca el output en HTML
python linkfinder.py -i https://example.com/1.js -o results.html

# CLI/STDOUT output
python linkfinder.py -i https://example.com/1.js -o cli

# analiza todo un dominio
python linkfinder.py -i https://example.com -d
```

![[Pasted image 20230605160735.png]]

#### collector.py

- [https://github.com/m4ll0k/BBTz/blob/master/collector.py](https://github.com/m4ll0k/BBTz/blob/master/collector.py)

Separa la salida de linkfinder  en jsfile,urls,params..etc

```bash
wget https://raw.githubusercontent.com/m4ll0k/BBTz/master/collector.py

python3 linkfinder.py -i https://www.test.com/a.js -o cli | python3 collector.py output
ls output
```

![[Pasted image 20230606004345.png]]

### jsAlert.py & getScriptTagContent.py

**jsAlert.py**: notifica si hay palabras clave interesantes, como postMessage,onmessage,innerHTML,etc.

```python
# by m4ll0k 
# github.com/m4ll0k
# jsalert.py - find interesting keywords in javascript files and extract the context..

import sys
import re

R = "\033[1;31m"
B = "\033[1;34m"
G = "\033[1;32m"
W = "\033[1;38m"
Y = "\033[1;33m"
E = "\033[0m"

regexs  = r"(-api|eyJ\S{3,10}|-api-key|-auth|-authorization|-back|-client|-config|-custom|-id|-integration|-oauth|-passwd|-private|-prod|-pwd|-redirect|-scope|-secret|-secure|-swagger|-token|-user|\.bash_history|\.bash_profile|\.cfg|\.client|\.cshrc|\.dockercfg|\.env|\.esmtprc|\.ftpconfig|\.git-credentials|\.history|\.htpasswd|\.ini|\.json|\.pem|\.pgpass|\.pwd|\.remote-sync\.json|\.s3cfg|\.secret|\.secure|\.sh_history|\.sql|\.stg|\.token|\.tugboat|_api|_api-key|_auth|_authorization|_authtoken|_back|_client|_config|_custom|_id|_integration|_oauth|_passwd|_password|_private|_prod|_pwd|_redirect|_scope|_secret|_secure|_token|_user|access_key|access_token|accesskey|accounts\.google\.com|admin_pass|admin_user|algolia_admin_key|algolia_api_key|alias_pass|alicloud_access_key|amazon_secret_access_key|amazonaws|amplitude\.com|ansible_vault_password|aos_key|api|api-|api\.applicationinsights\.io|api\.browserstack\.com|api\.datadoghq\.com|api\.github\.com|api\.googlemaps|api\.ipstack\.com|api\.iterable\.com|api\.keen\.io|api\.mailchimp\.com|api\.mailgun\.net|api\.mapbox\.com|api\.newrelic\.com|api\.pagerduty\.com|api\.sandbox\.paypal\.com|api\.sendgrid\.com|api\.twilio\.com|api\.twitter\.com|api\.wpengine\.com|api_|api_key_secret|api_key_sid|api_secret|apidocs|apikey|apisecret|app_debug|app_id|app_key|app_log_level|app_secret|appkey|appkeysecret|application_key|appsecret|appspot|auth-|auth2|auth_|authorization|authorization-|authorization_|authorizationtoken|authsecret|avastlic|aws_access|aws_access_key_id|aws_bucket|aws_key|aws_secret|aws_secret_key|aws_token|awssecretkey|b2_app_key|back-|back_|bash_history|bash_profile|bashrc|beanstalkd\.yml|bintray_apikey|bintray_gpg_password|bintray_key|bintraykey|bluemix_api_key|bluemix_pass|browserstack_access_key|bucket_password|bucketeer_aws_access_key_id|bucketeer_aws_secret_access_key|built_branch_deploy_key|bx_password|cache_driver|cache_s3_secret_key|cattle_access_key|cattle_secret_key|cccam\.cfg|certificate_password|ci_deploy_password|client-|client-token|client\.|client_|client_token|client_zpk_secret_key|clienttoken|clojars_password|cloud_api_key|cloud_watch_aws_access_key|cloudant_password|cloudflare_api_key|cloudflare_auth_key|cloudinary_api_secret|cloudinary_name|codecov_token|composer\.json|config|config-|config\.|config_|conn\.login|connectionstring|console\.jumpcloud\.com|credentials|cshrc|custom-|custom_|custom_token|cypress_record_key|database|database_password|database_schema_test|datadog_api_key|datadog_app_key|db_password|db_server|db_username|dbeaver-data-sources\.xml|dbpassword|dbuser|deploy\.rake|deploy_password|deployment-config\.json|dhcpd\.conf|digitalocean_ssh_key_body|digitalocean_ssh_key_ids|docker_hub_password|docker_key|docker_pass|docker_passwd|docker_password|dockercfg|dockerhub_password|dockerhubpassword|domain\.freshdesk\.com|dot-files|dotfiles|droplet_travis_password|dynamoaccesskeyid|dynamosecretaccesskey|elastica_host|elastica_port|elasticsearch_password|encryption_password|env\.heroku_api_key|env\.sonatype_password|environment|eureka\.awssecretkey|express\.conf|\.openshift|fabricapisecret|facebook_secret|fb_secret|fcm\.googleapis\.com|filezilla\.xml|firebase|flickr_api_key|fossa_api_key|ftp|ftp_password|gatsby_wordpress_base_url|gatsby_wordpress_client_id|gatsby_wordpress_user|gh_api_key|gh_token|ghost_api_key|git-credentials|gitconfig|github_api_key|github_deploy_hb_doc_pass|github_id|github_key|github_password|github_token|gitlab|gitlab\.example\.com|global|gmail_password|gmail_username|google_maps_api_key|google_private_key|google_secret|google_server_key|googleusercontent|gpg_key_name|gpg_keyname|gpg_passphrase|grant_type|graph\.facebook\.com|graph\.instagram\.com|hapikey|heroku_api_key|heroku_oauth|heroku_oauth_secret|heroku_oauth_token|heroku_secret|heroku_secret_token|herokuapp|history|homebrew_github_api_token|hooks\.slack\.com|htaccess_pass|htaccess_user|htpasswd|id-|id_|id_rsa|id_token|idea14\.key|identitytoolkit\.googleapis\.com|idtoken|incident_channel_name|integration-|integration-key|-internal|_internal|internal-|internal_|integration_|internal|jekyll_github_token|jupyter_notebook_config\.json|jwt_client_secret_key|jwt_lookup_secert_key|jwt_password|jwt_secret|jwt_secret_key|jwt_token|jwt_user|jwt_web_secert_key|jwt_xmpp_secert_key|keypassword|known_hosts|security-token|security_|_security|language:yaml|ldap_password|ldap_username|linux_signing_key|ll_shared_key|location_protocol|log_channel|login|login\.microsoftonline\.com|logins\.json|lottie_happo_api_key|lottie_happo_secret_key|lottie_s3_api_key|lottie_s3_secret_key|mail_password|mail_port|mailchimp|mailchimp_api_key|mailchimp_key|mailgun|mailgun_key|mailgun_password|mailgun_priv_key|mailgun_secret_api_key|makefile|manage_key|mandrill_api_key|mapbox|mapbox_apikey|master\.key|master_key|mg_api_key|mg_public_api_key|mh_apikey|mh_password|mile_zero_key|minio_access_key|minio_secret_key|mix_pusher_app_cluster|mix_pusher_app_key|mydotfiles|mysql|mysql_root_password|netlify_api_key|netrc|nexus_password|node_env|node_pre_gyp_accesskeyid|node_pre_gyp_secretaccesskey|npm_api_key|npm_password|npm_secret_key|npmrc|nuget_api_key|nuget_apikey|nuget_key|oauth-|oauth2|oauth2\.googleapis\.com|oauth_|object_storage_password|octest_app_password|octest_password|okta_key|omise_key|onesignal_api_key|onesignal_user_auth_key|openwhisk_key|org_gradle_project_sonatype_nexus_password|org_project_gradle_sonatype_nexus_password|os_password|ossrh_jira_password|ossrh_pass|ossrh_password|pagerduty_apikey|parse_js_key|passwd-|passwd_|password_|passwords|paypal_secret|paypal_token|personal_key|pgpass|playbooks_url|plotly_apikey|plugin_password|postgres_env_postgres_password|postgresql_pass|preprod|private|private-|private_|private_signing_password|prod|prod-|prod\.access\.key\.id|prod\.exs|prod\.secret\.exs|prod\.secret\.key|prod_|prod_password|proftpdpasswd|pt_token|publish_key|pusher_app_id|pwd-|pwd_|queue_driver|rabbitmq_password|rds\.amazonaws\.com|recentservers\.xml|redirect-|redirect_|redirect_uri|redis_password|registry\.npmjs\.org|response_auth_jwt_secret|rest_api_key|returnsecuretoken|rinkeby_private_key|robomongo\.json|root_password|ropsten_private_key|route53_access_key_id|rtd_key_pass|rtd_store_pass|s3_access_key|s3_access_key_id|s3_key|s3_key_app_logs|s3_key_assets|s3_secret_key|s3cfg|salesforce_password|sandbox_aws_access_key_id|sandbox_aws_secret_access_key|sauce_access_key|saucelabs\.com|scope=|scope_|secret-|secret\.password|secret_|secret_access_key|secret_bearer|secret_key_base|secretaccesskey|secrets|secrets\.yml|secure-|secure_|securetoken|security_credentials|send\.keys|send_keys|sendgrid_api_key|sendgrid_key|sendgrid_password|sendgrid_token|sendkeys|server\.cfg|ses_access_key|ses_secret_key|setdstaccesskey|setsecretkey|settings|settings\.py|sf_username|sftp-config\.json|sftp\.json|\.vscode|shodan_api_key|sid_token|signing_key_password|signing_key_secret|slack\.com|slack_api|slack_channel|slack_key|slack_outgoing_token|slack_signing_secret|slack_webhook|slash_developer_space_key|snoowrap_password|socrata_password|sonar_organization_key|sonar_project_key|sonatype_password|sonatype_token_password|soundcloud_password|spec|sql_password|sqsaccesskey|square_access_token|square_token|squaresecret|squareup\.com|ssh|ssh2_auth_password|sshd_config|sshpass|staging|stg|storepassword|stormpath_api_key_id|stormpath_api_key_secret|strip_key|strip_secret_key|stripe|stripe_key|stripe_secret|striptoken|svn_pass|swagger|swagger-|tenant_id|tesco_api_key|tester_keys_password|testuser|thera_oss_access_key|token-|token\.|token=|token_|trusted_hosts|tugboat|twilio_account_id|twilio_account_secret|twilio_account_sid|twilio_accountsid|twilio_acount_sid|twilio_api|twilio_api_auth|twilio_api_key|twilio_api_secret|twilio_api_sid|twilio_api_token|twilio_secret|twilio_secret_token|twilio_sid|twilio_token|twilioapiauth|twiliosecret|twine_password|twitter_secret|twitterkey|userid|userid-|userid_|ventrilo_srv\.ini|verifycustomtoken|wakatime\.com|webhook|webservers\.xml|wp-config|wp-config\.php|www\.googleapis\.com|x-api-key|zen_key|zen_tkn|zen_token|zendesk_api_token|zendesk_key|zendesk_token|zendesk_url|zendesk_username|s3\.amazonaws\.com|s3\.console\.aws\.amazon\.com|eyJ\w+|\.onmessage|\.postmessage|_session|session_|-session|HOMEBREW_GITHUB_API_TOKEN|ssh_key|sshkey|token|username|xoxa-2|xoxr|private-key|ssh:\/\/|ftp:\/\/|ws:\/\/|wss:\/\/|client_secret|clientsecret|consumer_key|consumer_secret|dbpasswd|encryption-key|encryption_key|encryptionkey|_encryption|-encryption|encryption_|encryption-|dencryption_|_dencryption|id_dsa|irc_pass|key_|_key|key-|-key|oauth_token|pass|password|private_key|privatekey|secret|secret-key|secret_key|secret_token|secretkey|session_key|session_secret|slack_api_token|slack_secret_token|slack_token|ssh-key|aws_secret_access_key|bearer|bot_access_token|bucket|client-secret|client_id|client_key|accesstoken|api-key|api_key|api_secret_key|api_token|auth_token|authkey|ConsumerSecret|consumer_|_consumer|consumer-|-consumer|merchant|-merchant|merchant-|_merchant|merchant_|sq0csp-|_live|live_|-live|live-|-sandbox|_sandbox|sandbox_|sandbox-|_stg|-stg|stg-|stg_|gtoken|gidtoken|acctoken|acc_token|access_|_access|-access|access-|dev_|_dev|developer_|_developer|-developer|developer-|authtoken|myaccesstoken)"


try:
    import jsbeautifier
    import requests
    import urllib3
except Exception as e:
    sys.exit(print("{0}.. please download this module/s".format(e)))

urllib3.disable_warnings(
	urllib3.exceptions.InsecureRequestWarning
)

def beauty(content:str)->str:
    return jsbeautifier.beautify(content.decode())

def getjs(url:str)->dict:
    try: return requests.get(url,verify=False) 
    except: return {'content':None}

def main()->None:
    try:
        url = sys.argv[1]
    except:
        sys.exit(print("\nUsage:\tpython3 {0} <url> <regex|string>\n".format(sys.argv[0])))
    if '.js' in url:
        r = getjs(url)
        if r.status_code == 200:
            js = beauty(r.content)
            for j in js.split('\n'):
                rr = re.findall(regexs,j,re.I)
                if rr:
                    rr = list(set(rr))
                    k = js.index(j)
                    a = js[js.index(j)-100 : js.index(j)+100]
                    for i in rr:
                        a = a.replace(i,f'{R}{i}{E}{Y}')
                    print(f'l:{G}{W}{str(k)}{E} -> {",".join(rr)}{E}\n{"-"*100}\n{Y}{a}{E}\n')
    else:
        sys.exit(print("\".js\" not found in URL ({}).. check your url".format(sys.argv[1])))

main()
```

con sinks y sources (MODIFICAR):

```python

import sys
import re

R = "\033[1;31m"
B = "\033[1;34m"
G = "\033[1;32m"
W = "\033[1;38m"
Y = "\033[1;33m"
E = "\033[0m"

regexs  = r"(-api|eyJ\S{3,10}|-api-key|-auth|-authorization|-back|-client|-config|-custom|-id|-integration|-oauth|-passwd|-private|-prod|-pwd|-redirect|-scope|-secret|-secure|-swagger|-token|-user|\.bash_history|\.bash_profile|\.cfg|\.client|\.cshrc|\.dockercfg|\.env|\.esmtprc|\.ftpconfig|\.git-credentials|\.history|\.htpasswd|\.ini|\.json|\.pem|\.pgpass|\.pwd|\.remote-sync\.json|\.s3cfg|\.secret|\.secure|\.sh_history|\.sql|\.stg|\.token|\.tugboat|_api|_api-key|_auth|_authorization|_authtoken|_back|_client|_config|_custom|_id|_integration|_oauth|_passwd|_password|_private|_prod|_pwd|_redirect|_scope|_secret|_secure|_token|_user|access_key|access_token|accesskey|accounts\.google\.com|admin_pass|admin_user|algolia_admin_key|algolia_api_key|alias_pass|alicloud_access_key|amazon_secret_access_key|amazonaws|amplitude\.com|ansible_vault_password|aos_key|api|api-|api\.applicationinsights\.io|api\.browserstack\.com|api\.datadoghq\.com|api\.github\.com|api\.googlemaps|api\.ipstack\.com|api\.iterable\.com|api\.keen\.io|api\.mailchimp\.com|api\.mailgun\.net|api\.mapbox\.com|api\.newrelic\.com|api\.pagerduty\.com|api\.sandbox\.paypal\.com|api\.sendgrid\.com|api\.twilio\.com|api\.twitter\.com|api\.wpengine\.com|api_|api_key_secret|api_key_sid|api_secret|apidocs|apikey|apisecret|app_debug|app_id|app_key|app_log_level|app_secret|appkey|appkeysecret|application_key|appsecret|appspot|auth-|auth2|auth_|authorization|authorization-|authorization_|authorizationtoken|authsecret|avastlic|aws_access|aws_access_key_id|aws_bucket|aws_key|aws_secret|aws_secret_key|aws_token|awssecretkey|b2_app_key|back-|back_|bash_history|bash_profile|bashrc|beanstalkd\.yml|bintray_apikey|bintray_gpg_password|bintray_key|bintraykey|bluemix_api_key|bluemix_pass|browserstack_access_key|bucket_password|bucketeer_aws_access_key_id|bucketeer_aws_secret_access_key|built_branch_deploy_key|bx_password|cache_driver|cache_s3_secret_key|cattle_access_key|cattle_secret_key|cccam\.cfg|certificate_password|ci_deploy_password|client-|client-token|client\.|client_|client_token|client_zpk_secret_key|clienttoken|clojars_password|cloud_api_key|cloud_watch_aws_access_key|cloudant_password|cloudflare_api_key|cloudflare_auth_key|cloudinary_api_secret|cloudinary_name|codecov_token|composer\.json|config|config-|config\.|config_|conn\.login|connectionstring|console\.jumpcloud\.com|credentials|cshrc|custom-|custom_|custom_token|cypress_record_key|database|database_password|database_schema_test|datadog_api_key|datadog_app_key|db_password|db_server|db_username|dbeaver-data-sources\.xml|dbpassword|dbuser|deploy\.rake|deploy_password|deployment-config\.json|dhcpd\.conf|digitalocean_ssh_key_body|digitalocean_ssh_key_ids|docker_hub_password|docker_key|docker_pass|docker_passwd|docker_password|dockercfg|dockerhub_password|dockerhubpassword|domain\.freshdesk\.com|dot-files|dotfiles|droplet_travis_password|dynamoaccesskeyid|dynamosecretaccesskey|elastica_host|elastica_port|elasticsearch_password|encryption_password|env\.heroku_api_key|env\.sonatype_password|environment|eureka\.awssecretkey|express\.conf|\.openshift|fabricapisecret|facebook_secret|fb_secret|fcm\.googleapis\.com|filezilla\.xml|firebase|flickr_api_key|fossa_api_key|ftp|ftp_password|gatsby_wordpress_base_url|gatsby_wordpress_client_id|gatsby_wordpress_user|gh_api_key|gh_token|ghost_api_key|git-credentials|gitconfig|github_api_key|github_deploy_hb_doc_pass|github_id|github_key|github_password|github_token|gitlab|gitlab\.example\.com|global|gmail_password|gmail_username|google_maps_api_key|google_private_key|google_secret|google_server_key|googleusercontent|gpg_key_name|gpg_keyname|gpg_passphrase|grant_type|graph\.facebook\.com|graph\.instagram\.com|hapikey|heroku_api_key|heroku_oauth|heroku_oauth_secret|heroku_oauth_token|heroku_secret|heroku_secret_token|herokuapp|history|homebrew_github_api_token|hooks\.slack\.com|htaccess_pass|htaccess_user|htpasswd|id-|id_|id_rsa|id_token|idea14\.key|identitytoolkit\.googleapis\.com|idtoken|incident_channel_name|integration-|integration-key|-internal|_internal|internal-|internal_|integration_|internal|jekyll_github_token|jupyter_notebook_config\.json|jwt_client_secret_key|jwt_lookup_secert_key|jwt_password|jwt_secret|jwt_secret_key|jwt_token|jwt_user|jwt_web_secert_key|jwt_xmpp_secert_key|keypassword|known_hosts|security-token|security_|_security|language:yaml|ldap_password|ldap_username|linux_signing_key|ll_shared_key|location_protocol|log_channel|login|login\.microsoftonline\.com|logins\.json|lottie_happo_api_key|lottie_happo_secret_key|lottie_s3_api_key|lottie_s3_secret_key|mail_password|mail_port|mailchimp|mailchimp_api_key|mailchimp_key|mailgun|mailgun_key|mailgun_password|mailgun_priv_key|mailgun_secret_api_key|makefile|manage_key|mandrill_api_key|mapbox|mapbox_apikey|master\.key|master_key|mg_api_key|mg_public_api_key|mh_apikey|mh_password|mile_zero_key|minio_access_key|minio_secret_key|mix_pusher_app_cluster|mix_pusher_app_key|mydotfiles|mysql|mysql_root_password|netlify_api_key|netrc|nexus_password|node_env|node_pre_gyp_accesskeyid|node_pre_gyp_secretaccesskey|npm_api_key|npm_password|npm_secret_key|npmrc|nuget_api_key|nuget_apikey|nuget_key|oauth-|oauth2|oauth2\.googleapis\.com|oauth_|object_storage_password|octest_app_password|octest_password|okta_key|omise_key|onesignal_api_key|onesignal_user_auth_key|openwhisk_key|org_gradle_project_sonatype_nexus_password|org_project_gradle_sonatype_nexus_password|os_password|ossrh_jira_password|ossrh_pass|ossrh_password|pagerduty_apikey|parse_js_key|passwd-|passwd_|password_|passwords|paypal_secret|paypal_token|personal_key|pgpass|playbooks_url|plotly_apikey|plugin_password|postgres_env_postgres_password|postgresql_pass|preprod|private|private-|private_|private_signing_password|prod|prod-|prod\.access\.key\.id|prod\.exs|prod\.secret\.exs|prod\.secret\.key|prod_|prod_password|proftpdpasswd|pt_token|publish_key|pusher_app_id|pwd-|pwd_|queue_driver|rabbitmq_password|rds\.amazonaws\.com|recentservers\.xml|redirect-|redirect_|redirect_uri|redis_password|registry\.npmjs\.org|response_auth_jwt_secret|rest_api_key|returnsecuretoken|rinkeby_private_key|robomongo\.json|root_password|ropsten_private_key|route53_access_key_id|rtd_key_pass|rtd_store_pass|s3_access_key|s3_access_key_id|s3_key|s3_key_app_logs|s3_key_assets|s3_secret_key|s3cfg|salesforce_password|sandbox_aws_access_key_id|sandbox_aws_secret_access_key|sauce_access_key|saucelabs\.com|scope=|scope_|secret-|secret\.password|secret_|secret_access_key|secret_bearer|secret_key_base|secretaccesskey|secrets|secrets\.yml|secure-|secure_|securetoken|security_credentials|send\.keys|send_keys|sendgrid_api_key|sendgrid_key|sendgrid_password|sendgrid_token|sendkeys|server\.cfg|ses_access_key|ses_secret_key|setdstaccesskey|setsecretkey|settings|settings\.py|sf_username|sftp-config\.json|sftp\.json|\.vscode|shodan_api_key|sid_token|signing_key_password|signing_key_secret|slack\.com|slack_api|slack_channel|slack_key|slack_outgoing_token|slack_signing_secret|slack_webhook|slash_developer_space_key|snoowrap_password|socrata_password|sonar_organization_key|sonar_project_key|sonatype_password|sonatype_token_password|soundcloud_password|spec|sql_password|sqsaccesskey|square_access_token|square_token|squaresecret|squareup\.com|ssh|ssh2_auth_password|sshd_config|sshpass|staging|stg|storepassword|stormpath_api_key_id|stormpath_api_key_secret|strip_key|strip_secret_key|stripe|stripe_key|stripe_secret|striptoken|svn_pass|swagger|swagger-|tenant_id|tesco_api_key|tester_keys_password|testuser|thera_oss_access_key|token-|token\.|token=|token_|trusted_hosts|tugboat|twilio_account_id|twilio_account_secret|twilio_account_sid|twilio_accountsid|twilio_acount_sid|twilio_api|twilio_api_auth|twilio_api_key|twilio_api_secret|twilio_api_sid|twilio_api_token|twilio_secret|twilio_secret_token|twilio_sid|twilio_token|twilioapiauth|twiliosecret|twine_password|twitter_secret|twitterkey|userid|userid-|userid_|ventrilo_srv\.ini|verifycustomtoken|wakatime\.com|webhook|webservers\.xml|wp-config|wp-config\.php|www\.googleapis\.com|x-api-key|zen_key|zen_tkn|zen_token|zendesk_api_token|zendesk_key|zendesk_token|zendesk_url|zendesk_username|s3\.amazonaws\.com|s3\.console\.aws\.amazon\.com|eyJ\w+|\.onmessage|\.postmessage|_session|session_|-session|HOMEBREW_GITHUB_API_TOKEN|ssh_key|sshkey|token|username|xoxa-2|xoxr|private-key|ssh:\/\/|ftp:\/\/|ws:\/\/|wss:\/\/|client_secret|clientsecret|consumer_key|consumer_secret|dbpasswd|encryption-key|encryption_key|encryptionkey|_encryption|-encryption|encryption_|encryption-|dencryption_|_dencryption|id_dsa|irc_pass|key_|_key|key-|-key|oauth_token|pass|password|private_key|privatekey|secret|secret-key|secret_key|secret_token|secretkey|session_key|session_secret|slack_api_token|slack_secret_token|slack_token|ssh-key|aws_secret_access_key|bearer|bot_access_token|bucket|client-secret|client_id|client_key|accesstoken|api-key|api_key|api_secret_key|api_token|auth_token|authkey|ConsumerSecret|consumer_|_consumer|consumer-|-consumer|merchant|-merchant|merchant-|_merchant|merchant_|sq0csp-|_live|live_|-live|live-|-sandbox|_sandbox|sandbox_|sandbox-|_stg|-stg|stg-|stg_|gtoken|gidtoken|acctoken|acc_token|access_|_access|-access|access-|dev_|_dev|developer_|_developer|-developer|developer-|authtoken|myaccesstoken)"


try:
    import jsbeautifier
    import requests
    import urllib3
except Exception as e:
    sys.exit(print("{0}.. please download this module/s".format(e)))

urllib3.disable_warnings(
	urllib3.exceptions.InsecureRequestWarning
)

def beauty(content:str)->str:
    return jsbeautifier.beautify(content.decode())

def getjs(url:str)->dict:
    try: return requests.get(url,verify=False) 
    except: return {'content':None}

def main()->None:
    try:
        url = sys.argv[1]
    except:
        sys.exit(print("\nUsage:\tpython3 {0} <url> <regex|string>\n".format(sys.argv[0])))
    if '.js' in url:
        r = getjs(url)
        if r.status_code == 200:
            js = beauty(r.content)
            for j in js.split('\n'):
                rr = re.findall(regexs,j,re.I)
                if rr:
                    rr = list(set(rr))
                    k = js.index(j)
                    a = js[js.index(j)-100 : js.index(j)+100]
                    for i in rr:
                        a = a.replace(i,f'{R}{i}{E}{Y}')
                    print(f'l:{G}{W}{str(k)}{E} -> {",".join(rr)}{E}\n{"-"*100}\n{Y}{a}{E}\n')
    else:
        sys.exit(print("\".js\" not found in URL ({}).. check your url".format(sys.argv[1])))

main()
```

**getScriptTagContent.py**: obtener contenido entre etiquetas de script

```python
desarrollar...
```

### getJSWords.py

- [https://raw.githubusercontent.com/m4ll0k/BBTz/master/getjswords.py](https://raw.githubusercontent.com/m4ll0k/BBTz/master/getjswords.py)

obtenga todas las palabras del archivo javascript, excepto las palabras clave de javascript

```bash
wget https://raw.githubusercontent.com/m4ll0k/BBTz/master/getjswords.py
python3 getjswords.py https://www.google.com/test.js
```

### getSrc

- [https://github.com/m4ll0k/BBTz/blob/master/getsrc.py](https://github.com/m4ll0k/BBTz/blob/master/getsrc.py)

```bash
wget https://raw.githubusercontent.com/m4ll0k/BBTz/master/getsrc.py

python3 getsrc.py https://www.quimiza.com/
```

![[Pasted image 20230606002820.png]]

### availableForPurchase.py

esta herramienta busca si un dominio está disponible para comprar, esta herramienta combinada con el buscador de enlaces y el recopilador (**linkfinder**) es realmente poderosa.

```bash
wget https://raw.githubusercontent.com/m4ll0k/Bug-Bounty-Toolz/master/availableForPurchase.py

cat sub-pur.txt | python availableForPurchase.py
```

![[Pasted image 20230606003722.png]]

### SECRET FINDER

- [https://github.com/m4ll0k/SecretFinder](https://github.com/m4ll0k/SecretFinder)

```bash
git clone https://github.com/m4ll0k/SecretFinder.git
cd SecretFinder
pip install -r requirements.txt
python3 SecretFinder.py
```

```bash
# Usa expresiones regular para buscar tokens, API keys, paths, ...

python3 SecretFinder.py -i https://example.com/1.js -o results.html
python3 SecretFinder.py -i https://example.com/1.js -o cli

#analiza todo el dominio
python3 SecretFinder.py -i https://example.com/ -e

#ignora ciertos archivos
python3 SecretFinder.py -i https://example.com/ -e -g 'jquery;bootstrap;api.google.com'

#usando regex
python3 SecretFinder.py -i https://example.com/1.js -o cli -r 'apikey=my.api.key[a-zA-Z]+'
```

![[Pasted image 20230606002009.png]]

#### SECRETFINDER + KATANA

```bash
katana -u https://www.quimiza.com -jc -d 2 | grep ".js$" | uniq | sort > /tmp/result.txt

cat /tmp/result.txt | while read url; do python3 SecretFinder.py -i $url -o cli; done
```

![[Pasted image 20230606002249.png]]

### KATANA

- [https://github.com/projectdiscovery/katana](https://github.com/projectdiscovery/katana)

```bash
git clone https://github.com/projectdiscovery/katana.git
cd katana/cmd/katana
go build .
mv katana /usr/bin
```

```bash
katana -u https://tesla.com
katana -u https://tesla.com -jc -d 2 | grep ".js$" | uniq | sort

katana -list url_list.txt

cat domains | httpx | katana
```

![[Pasted image 20230606001839.png]]

### SUBJS

- [https://github.com/lc/subjs](https://github.com/lc/subjs)

```bash
#subjs obtiene archivos javascript de una lista de URLS o subdominios.

cat urls.txt | subjs 
subjs -i urls.txt
cat hosts.txt | gau | subjs
```

### GETALLURLS (GAU)

- [https://github.com/lc/gau](https://github.com/lc/gau)
- [https://github.com/bp0lr/gauplus](https://github.com/bp0lr/gauplus)

descargando el release:

```bash
git clone https://github.com/lc/gau.git
cd gau/cmd/gau
go build
sudo mv gau /usr/local/bin/
gau --version
```

```bash
printf example.com | gau
cat domains.txt | gau --threads 5 #necesario poner los threads
gau example.com google.com
gau --o example-urls.txt example.com
gau --blacklist png,jpg,gif example.com
```

![[Pasted image 20230605224200.png]]

```bash
$ gau -h
```

| Flag | Description | Example |
|------|-------------|---------|
|`--blacklist`| list of extensions to skip | gau --blacklist ttf,woff,svg,png|
|`--fc`| list of status codes to filter | gau --fc 404,302 |
|`--from`| fetch urls from date (format: YYYYMM) | gau --from 202101 |
|`--ft`| list of mime-types to filter | gau --ft text/plain|
|`--fp`| remove different parameters of the same endpoint | gau --fp|
|`--json`| output as json | gau --json |
|`--mc`| list of status codes to match | gau --mc 200,500 |
|`--mt`| list of mime-types to match |gau --mt text/html,application/json|
|`--o`| filename to write results to | gau --o out.txt |
|`--providers`| list of providers to use (wayback,commoncrawl,otx,urlscan) | gau --providers wayback|
|`--proxy`| http proxy to use (socks5:// or http:// | gau --proxy http://proxy.example.com:8080 |
|`--retries`| retries for HTTP client | gau --retries 10 |
|`--timeout`| timeout (in seconds) for HTTP client | gau --timeout 60 |
|`--subs`| include subdomains of target domain | gau example.com --subs |
|`--threads`| number of workers to spawn | gau example.com --threads |
|`--to`| fetch urls to date (format: YYYYMM) | gau example.com --to 202101 |
|`--verbose`| show verbose output | gau --verbose example.com |
|`--version`| show gau version | gau --version|

### GXSS

Una herramienta ligera para verificar los parámetros reflejados en una URL.

si por ejemplo tenemos una URL asi:

```bash
http://www.quimiza.com/exchange/logon.asp?newwindow=1&viewer=1
```

verificara si los parametros `newwindow` y `viewer` se reflejan en la respuesta al consultar la URL.

```bash
git clone https://github.com/KathanP19/Gxss.git
cd Gxss
go build .
mv Gxss /usr/bin
```

ejemplos:

```bash
#definimos concurrencia en 100
echo "http://www.quimiza.com/exchange/logon.asp?newwindow=1&viewer=1" | Gxss -c 100

#leemos de un archivo
#con -p definimos que valor se probara en cada parametro, por defecto Gxss
cat urls.txt | Gxss -c 100 -p XssReflected

#guardar resultado
cat urls.txt | Gxss -c 100 -o Result.txt

#verbose
cat urls.txt | Gxss -c 100 -o Result.txt -v

#definir un header
cat urls.txt | Gxss -c 100 -p Xss -h "Cookie: Value"

#definir un user-agent
cat urls.txt | Gxss -c 100 -p Xss -h "Cookie: Value" -u "Google Bot"
```

![[Pasted image 20230605225211.png]]

#### GXSS + GAU

```bash
cat sub.txt | gau --threads 5 | grep "=" | Gxss -c 100
```

>[!note]
>Si sale en la respuesta la URL, es que se refleja el parametro

![[Pasted image 20230605225532.png]]

la cantidad de URL que reporta **GXSS** debe ser menor a los que **GAU** reporta, ya que no todos reflejan sus parametros:

### DALFOX

DalFox es una poderosa herramienta de código abierto que se enfoca en la automatización, lo que la hace ideal para escanear rápidamente fallas XSS y analizar parámetros. Su motor de prueba avanzado y sus características de nicho están diseñadas para agilizar el proceso de detección y verificación de vulnerabilidades.

```bash
git clone https://github.com/hahwul/dalfox.git
cd dalfox
go build .
mv dalfox /usr/bin
```

#### GAU + GXSS + DALFOX

```bash
echo "http://testphp.vulnweb.com/" | gau --threads 5 | grep "=" | Gxss -c 100 -p payload | dalfox pipe --mining-dict /root/Desktop/Arjun/arjun/db/params.txt --skip-bav
```

![[Pasted image 20230605233940.png]]

![[Pasted image 20230605234004.png]]

>[!tip]
>En lugar de usarlo con GAU (porque recopila mucha informacion y puede tardar demasiado) podemos usarlo con `Paramspider.py` sobre una unica URL y ver si uno de los parametros es reflejado y vulnerable a XSS.

fuente: [https://www.youtube.com/watch?v=6rkk3v2a7WQ](https://www.youtube.com/watch?v=6rkk3v2a7WQ)

### automatizacion con GAU

- [https://medium.com/@nynan/the-most-underrated-tool-in-bug-bounty-and-the-filthiest-one-liner-possible-cab14ef7faeb](https://medium.com/@nynan/the-most-underrated-tool-in-bug-bounty-and-the-filthiest-one-liner-possible-cab14ef7faeb)

## GREPEAR LISTADO DE URLS (PARA NUESTROS OUTPUTS)

- [https://github.com/tomnomnom/unfurl](https://github.com/tomnomnom/unfurl)

## RECURSOS

- [https://gist.github.com/m4ll0k/31ce0505270e0a022410a50c8b6311ff](https://gist.github.com/m4ll0k/31ce0505270e0a022410a50c8b6311ff)
- [https://github.com/kriko69/JS-Scrapping-Python-Scripts](https://github.com/kriko69/JS-Scrapping-Python-Scripts)
