+++
title = "Nginx with ModSecurity"
tags = ["nginx", "modsecurity"]
date = "2013-11-06"
type = "post"
+++

At my current job we are using Gentoo on our servers in the form of Calculate Linux, so all I write in this post can be applied to this distribution. Earlier we have used Apache, but after we had changed web servers to Nginx it brought up about ModSecurity setup in combination with Nginx.

As ModSecurity support for Nginx is still in beta, only way to build the module and connect it to the nginx is to do it manually. And since nginx does not support dynamic module loading, ModSecurity should be enabled in `./configure` options before build.

The latest version of ModSecurity can be found on this page: [http://www.modsecurity.org/download/](http://www.modsecurity.org/download/).


## Setup

The first we need to download and unpack the modsecurity tarball:

	cd /tmp
	wget https://www.modsecurity.org/tarball/2.7.5/modsecurity-apache_2.7.5.tar.gz
	tar xvf modsecurity-apache_2.7.5.tar.gz

change directory:

	cd modsecurity-apache_2.7.5

To build the modsecurity module require some Apache libraries so we put:

	emerge www-servers/apache

and then configure and build:

	./configure --enable-standalone-module
	make
	make install

Moving the directory to a more appropriate location:

	mkdir /usr/src/modsecurity	
	mv /tmp/modsecurity-apache_2.7.5/* /usr/src/modsecurity

Next we need to make changes to the configuration files. Open the `/etc/make.conf` and add the following line:

	NGINX_ADD_MODULES="/usr/src/modsecurity/nginx/modsecurity/"

Now we can install nginx server:

	emerge www-servers/nginx

## Preparation

Please be aware that we use php-fpm as back-end using, so there is a section in nginx.conf:

	upstream backend {
		server 127.0.0.1:9000;
	}

We have an organized directory structure:

	# tree -d
	.
	├── conf.d
	├── modsec_sites
	└── modsecurity
	    └── base_rules

In conf.d directory there is common configuration files for virtual domains. In modsec_sites directory there is modsecurity configuration files.

	# pwd
	/etc/nginx/modsecurity
	# tree 
	.
	├── base_rules
	│   ├── modsecurity_35_bad_robots.data
	│   ├── modsecurity_35_scanners.data
	│   ├── modsecurity_40_generic_attacks.data
	│   ├── modsecurity_50_outbound.data
	│   ├── modsecurity_50_outbound_malware.data
	│   ├── modsecurity_crs_20_protocol_violations.conf
	│   ├── modsecurity_crs_21_protocol_anomalies.conf
	│   ├── modsecurity_crs_23_request_limits.conf
	│   ├── modsecurity_crs_30_http_policy.conf
	│   ├── modsecurity_crs_35_bad_robots.conf
	│   ├── modsecurity_crs_40_generic_attacks.conf
	│   ├── modsecurity_crs_41_sql_injection_attacks.conf
	│   ├── modsecurity_crs_41_xss_attacks.conf
	│   ├── modsecurity_crs_42_tight_security.conf
	│   ├── modsecurity_crs_45_trojans.conf
	│   ├── modsecurity_crs_47_common_exceptions.conf
	│   ├── modsecurity_crs_48_local_exceptions.conf.example
	│   ├── modsecurity_crs_49_inbound_blocking.conf
	│   ├── modsecurity_crs_50_outbound.conf
	│   ├── modsecurity_crs_59_outbound_blocking.conf
	│   └── modsecurity_crs_60_correlation.conf
	├── modsecurity.conf
	└── modsecurity-crs.conf

As you can see from listing, configuration files and configuration for Core Rule Set file are located within modsecurity. Base_rules directory keeps files from modsecurity-src.

You can download the archive with rule set [here](https://github.com/SpiderLabs/owasp-modsecurity-crs/tarball/master).

After unpacking the archive we should copy base_rules directory to `/etc/nginx/modsecurity`. After that we should copy modsecurity_crs_10_setup.conf.example file to `/etc/nginx/modsecurity/modsecurity-src.conf`.Because we used Apache earlier, we already have the modsecurity.conf file. So we will use it.

## Settings

I had two options:
1. I could arrange a full dump configuration files directly in the nginx directory for the reason that if you set up modsecurity in nginx.conf, modsecurity configuration file must be located close to the nginx configuration file.
2. To understand how best to arrange the files so not to make a mess :)

I came to the next option through an experience.

nginx.conf file does not include modsecurity configuration file. All settings will be applied to virtual domains, so we should include this section also in nginx.conf file:

	include /etc/nginx/conf.d/*.conf;

Virtual domain configuration files are located in conf.d directory:

	# tree conf.d/
	conf.d/
	└── test.com.conf

Connecting and enabling the modsecurity module:

	location / {
	    ModSecurityEnabled on;
	    ModSecurityConfig /etc/nginx/modsec_sites/modsec_test.com.conf;

	    index  index.php index.html index.htm;
	}

ModSecurityEnabled variable defines will modsecurity be applied for a particular domain or not (on or off). The second variable (ModSecurityConfig) defines the location of the modsecurity rules configuration file for this domain. If required for any given location for a site you can override whether modsecurity enabled or not.

Lets look at rules configuration file for the domain. It is located in the `/etc/nginx/modsec_sites`.
Here they are:

	include /etc/nginx/modsecurity/modsecurity.conf

	# Enable looking up geolocation data from MaxMind's GeoIP database
	SecGeoLookupDb /usr/share/GeoIP/GeoLiteCity.dat

	SecDataDir /var/cache/modsecurity
	SecTmpDir  /var/tmp/modsecurity

	# Restricted SQL Character Anomaly Detection Alert - Total # of special characters exceeded
	       SecRuleRemoveById 981172
	# Detects MySQL comment-/space-obfuscated injections and backtick termination
	       SecRuleRemoveById 981257
	# Detects classic SQL injection probings [data "8ora"]
	       SecRuleRemoveById 981242
	# Detects classic SQL injection probings 2/2"]
	       SecRuleRemoveById 981243
	# Detects basic SQL authentication bypass attempts
	       SecRuleRemoveById 981245
	# Detects basic SQL authentication bypass attempts 3/3"]
	       SecRuleRemoveById 981246
	# Detects chained SQL injection attempts.
	#       SecRuleRemoveById 981248
	# SQL Injection Attack: Common Injection Testing Detected
	       SecRuleRemoveById 981318
	# SQL Character Anomaly Detection Alert - Repetative Non-Word Characters
	        SecRuleRemoveById 960024

The first line here is the most important one. It defines location of modsecurity main file. Further there are other options and the rules that must be disabled for a given domain, as in the modsecurity.conf connected all the rules completely.

In modsecurity.conf we made ​​the necessary adjustments. Full description of variables is beyond the scope of this post. 
In our case we need just one line:

	include /etc/nginx/modsecurity/modsecurity-crs.conf

But in modsecurity-crs.conf we may edit the options and general rules that will be applied to all domains.

Now basic configuration is finished. Any other changes are dependend on the requirements of the rules which are applicable to specific domains.

## P.S.

Many thanks [@AndyClarkii](https://twitter.com/AndyClarkii) for help!
