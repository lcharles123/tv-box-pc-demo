NAME=deb-package-builder
IMAGE=lcharles060/$(NAME):latest
CONTAINER=$(NAME)-run

# you can pass args like "make drun arg='...'"
arg="help"

main:
	cat Makefile

dbuild:
	docker build . --tag $(IMAGE)

dpush:
	docker push $(IMAGE)

dpull:
	docker pull $(IMAGE)

#docker run -td -v $(PWD)/deb_packages:/root --name $(CONTAINER) $(IMAGE) 
drun:
	docker run -it --rm -v $(PWD)/do_build.sh:/do_build.sh -v $(PWD)/deb_packages:/root/deb_packages --name $(CONTAINER) $(IMAGE) $(arg)

denter:
	docker exec -it $(CONTAINER) /bin/bash

dclean:
	docker rm -f $(CONTAINER)

dpurge:
	docker image rm -f $(IMAGE) 

pclean:
	rm -rf deb_packages/*

examples:
	#make drun arg="build https://github.com/lcharles123/hello-debian-package 0.1"
	#make drun arg=""
	#make drun arg=""
	#/do_build.sh build https://github.com/lcharles123/hello-debian-package 0.1 

dfresh: dclean dbuild drun denter

drerun: dclean drun

