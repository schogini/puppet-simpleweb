# A Recipe to Create a Simple Website in Puppet
Puppet-Docker Simpleweb
```
docker run -ti --rm --net puppet --name puppet  -p :8090:80 --hostname puppet schogini/docker-puppetserver-ubuntu
docker run -ti --rm --net puppet --name puppetnode1 -p :8091:80 --hostname puppetnode1 schogini/docker-puppetnode-ubuntu
```
##SERVER
service puppetserver start

##CLIENT
puppet agent -t

##SERVER 
puppet cert list
puppet cert sign puppetnode1
cd /etc/puppetlabs/code/environments/production/modules/
puppet module generate --skip-interview sree-simpleweb
nano simpleweb/manifests/init.pp
```
class simpleweb {
	
	$title1 = "Sree"
	$node1 = $hostname
	$desc = "Test Description via desc variable in manifest"
	$time = generate('/bin/date', '+%Y-%d-%m %H:%M:%S')

	exec { 'apt-update':
		command => '/usr/bin/apt-get update'
	}

	package { 'apache2':
		require => Exec['apt-update'],
		ensure => installed
	}

	service { 'apache2':
		ensure => running
	}

	package { 'php5':
		require => Exec['apt-update'],
		ensure => installed
	}

	package { 'libapache2-mod-php5':
		ensure => installed
	}

	file { '/etc/apache2/mods-available/php5.conf':
		content => '<IfModule mod_php5.c>
			<FilesMatch "\.php$">
				SetHandler application/x-httpd-php
			</FilesMatch>
		</IfModule>',
		require => Package['apache2'],
		notify => Service['apache2']
	}

	file { '/var/www/html/index.html':
	  ensure  => file,
	  content => template('simpleweb/index.html.erb')
	}

}
```


##SERVER
mkdir simpleweb/templates
nano simpleweb/templates/index.html.erb
```
<!DOCTYPE html>
<html>
<head>
	<title><%= @title1 %></title>
</head>
<body>
	<h1><%= @title1 %> from Node [<%= @node1 %>]</h1>
	<h3>The <%= @desc %></h3>
	<hr>
	<h3>I was created @ <%= @time %> by <%= @node1 %></h3>
</body>
</html>
```
##SERVER
puppet apply -e "include simpleweb"
curl localhost
BROWSER: http://0.0.0.0:8090/
```
nano ../manifests/site.pp
node 'puppetnode1' {
 include motd
 include simpleweb
}
node 'puppet' {
 include motd
 include simpleweb
}
```
##CLIENT
puppet agent -t
curl localhost
BROWSER: http://0.0.0.0:8091/
